# RAII: GC without GC

I don't want you to think of me as a hater of C++. In spite of the fact
that this book itself is a comparison between Rust and C++ in Rust's
favor, I am very aware that Rust as it exists would never have been
possible without C++. Like all new technology and science, Rust stands
on the shoulders of giants, and many of those giants contributed to C++.

And this makes sense if you think about it. Rust and C++ have very similar
goals. The C++ community has done a lot over all these years to pioneer
new programming language features in line with those goals. C++ has then
given these features years to mature in its humongous ecosystem. And
because Rust also doesn't have to be compatible with C++, it can then
steal those features without some of the caveats they come with in C++.

One of the biggest such features -- perhaps the biggest one -- is
RAII, C++'s and now Rust's (somewhat oddly-named) scope-based feature
for resource management. And while RAII is for managing all kinds of
resources, its biggest use case is as part of a compile-time alternative
to run-time garbage collection and reference counting.

As an alternative to garbage collection, RAII has deficits.  While many
allocations are created and freed neatly in line with variables coming in
and out of scope, sometimes that's not possible. To fully compete with
garbage collection and capture the diverse ways programs use the heap,
RAII needs to be combined with other features.

And C++ has done a lot of this. C++ added move semantics in C++11,
which Rust also has -- though cleaner in Rust because Rust was
designed with them from the start and so it can pull off [destructive
moves](/tags/cpp-move/). C++ also has opt-in reference counting, which,
again, Rust also has.

But C++ still doesn't have lifetimes (Rust got that from
[Cyclone](https://homes.cs.washington.edu/~djg/papers/cyclone.pdf),
which called them "regions"), nor the infamous borrow checker that goes
along with them in Rust. And even though the borrow checker is perhaps
the most hated part of Rust, in this post, I will argue that it brings
Rust's RAII-centric compile-time memory management system much closer
to feature-parity with run-time reference counting and other run-time
garbage-collection technologies.

I will start by talking about the problem that RAII was originally
designed to solve. Then, I will re-hash the basics of how RAII works,
and work through memory usage patterns where RAII needs to be combined
with these other features, especially the borrow checker. Finally, I will
discuss the downsides of these memory management techniques, especially
performance implications and handling of cyclic data structures.

But before I get into the weeds, I have some important caveats:

> *Caveat*: No Turing-complete programming language can completely
> prevent memory leaks. Even in fully-GC'd languages, you can still
> leak memory by filling up a data structure with increasing
> amounts of unnecessary data. This can be done by accident,
> especially when sophisticated callback systems are combined
> with closures. This is out of the scope of this post, which
> only concerns memory management issues that automated GC
> can actually help with.
>
> *Caveat #2*: Rust allows you to [leak memory on
> purpose](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.leak),
> even when a garbage collector would have reclaimed it. In extreme
> circumstances, the reference counting system can be abused to leak
> memory as well. This fact has been used in anti-Rust rhetoric to
> imply its memory safety system is somehow worthless.
> 
> For the purposes of this post, we assume a programmer who is
> trying to get actual work done and needs help not leaking memory
> or causing memory corruption, not an adversarial programmer
> trying to make the system leak on purpose.
>
> *Caveat #3*: RAII is a terrible name. OBRM (Ownership-Based Resource
> Management) is used in Rust sometimes, and is a much better name.
> I call it RAII in this article though, because that's what most people
> call it, even in Rust.

# The Problem: Manual Memory Management is Hard, GC is "Slow"

So. C-style manual memory management -- "just call `free` when you're
done with the allocation" -- is error prone.

It is error prone when it is easy and tedious, because programmers can
make stupid mistakes and just forget to write `free` and it isn’t
immediately broken. It is error prone when multiple programmers work
together, because they might make different assumptions about who is
supposed to free something. It is error prone when multiple parts of the
code need to use the same data, especially when that usage changes with
new requirements and new features.

And the consequences of doing it wrong are not just memory
leaks. Use-after-free can lead to memory corruption, and bugs in one
part of the program can abruptly show up when allocation patterns change
somewhere else entirely.

This is a problem that can be solved with discipline, but like many
tedious clerical disciplines, it can also be solved by computer.

It can be solved at run-time, which is what garbage collection and
reference counting do. These systems do two things:

* They keep allocations from lasting too long. When memory becomes
unreachable, it can be reclaimed. This prevents memory leaks.
* They keep allocations from being freed early. If memory is still
reachable, it will still be valid. This prevents memory corruption.

And for most programmers and applications, this is good enough. And so
for almost all modern programming languages, this run-time cost is well
worth not troubling the programmer with the error-prone tedious tasks
of C-style manual memory management, enabling memory safety and resource
efficiency at the same time.

> Caveat: To be clear, "slow" here is an oversimplification, and I address
> that more later. I mean it as a tongue-in-cheek way of saying that
> it has performance costs, whereas Rust and C++ try to adhere
> to a *zero*-cost principle.

## GC (including RC) Has Costs

But there are costs to having the computer do memory management at
run-time.

I lump mark-sweep garbage collection and reference counting together here.
Both mark-sweep garbage collection and reference counting have costs above
C-style manual memory management that make them unacceptable according to
the zero-cost principle. GC comes with pauses, and additional threads,
in the best case. RC comes with myriad increments and decrements to a
reference count. These costs might be small enough to be okay for your
application -- and that's well and good -- but they are costs, and
therefore they can't be the main memory management model in C++ or Rust.

This is a complicated issue, and so before continuing, here comes
another caveat:

> Caveat: GC is not *necessarily* slower, but it does have performance
> implications that are often unacceptable for situations where C++
> (or Rust) is used. To achieve its full performance, it needs to be
> enabled for the entire heap, and that has costs associated with it.
> For these reasons, C++ and Rust do not use GC. The details of these
> performance trade-offs are beyond the scope of this blog post.

## A Dilemma

But C++ and Rust are not most programming languages. They face a dilemma:

* On the one hand, manual memory management
is unacceptably error prone for a high level language, a detail the
computer should be able to handle for you.
* On the other hand, run-time garbage collection violates a fundamental
goal that C++ and Rust share: the zero-cost principle. Code written
in these languages is supposed to be as performant as the equivalent
manually-written C. To conform to that principle, reference counting
(or GC) have to be opt-in (because, after all, sometimes manually written
C code does use these technologies).

So, for the vast majority of situations, where a C programmer wouldn't
use reference counting (or mark-sweep), Rust and C++ need something more
sophisticated.  They need tools to prevent memory management mistakes --
that is, to at least partially automate this tedious and error-prone
task -- without sacrificing any run-time performance.

And this is the reason C++ invented (and Rust appropriated) RAII. Instead
of addressing the problem at run-time, RAII automates memory management
at compile-time. Analogous to how templates and trait monomorphization
can bring some but not all of the power of polymorphism without many of
the run-time costs, RAII brings some but not all of the power of garbage
collection without constant reference count updates or GC pauses.

But as we will see, RAII as C++ implements it only solves one of the
two problems addressed by garbage collection: leaks. It cannot address
memory corruption; it cannot keep allocations alive long enough for all
the code that could possibly need to use it.

# Raw RAII: How RAII Works on its Own

The simplest use case for RAII is underwhelming: it automatically inserts
calls to free up heap allocations at the end of the block where
we made the allocation. It replaces a `malloc`/`free` sandwich
from C with simply the allocation side, by inserting an
implicit (and unwritten) call to a destructor, which in its
simplest version is an equivalent of `free`. And if that
was all RAII did, it wouldn't be that interesting.

For example, take this C-style (no RAII) code:

```c++
void print_int_little_endian_decimal(int foo) {
    // Little endian decimal print of `foo`
    // i.e. backwards from how we normally write decimal numbers
    // e.g. 831 prints out as "138"

    // Big endian would be too hard
    // Little endian is as always actually simpler platonically,
    // if somehow not for humans.

    // Yes, this only works for positive ints. It's an example.

    char *buffer = malloc(11);
    for(char *it = buffer; it < buffer + 10; ++it) {
        *it = '0' + foo % 10;
        foo /= 10;
        if (foo == 0) {
            it[1] = '\0';
            break;
        }
    }
    puts(buffer); // put-string, not the 3sg verb form "puts"
    free(buffer); // Don't forget to do this!
}
```

Just using RAII (and `unique_ptr`s, which are an essential part of
the RAII model), but using no other features of C++, we get this very
unidiomatic and unimpressive version:

```c++
void print_int_little_endian_decimal(int foo) {
    std::unique_ptr<char[]> buffer{new char[11]};
    for(char *it = &buffer[0]; it < &buffer[10]; ++it) {
        *it = '0' + foo % 10;
        foo /= 10;
        if (foo == 0) {
            it[1] = '\0';
            break;
        }
    }
    puts(&buffer[0]);
}
```

It doesn't help us with our random guess of an appropriate buffer
size, our awkward redundant attempts to avoid a buffer-overflow, or
with any abstraction over the fact that we're trying to implement
a collection.

In fact, it makes the code more awkward, for a benefit that seems hardly
worth it, to just automatically call `free` at the end of the block --
which might not even be where we want to call free! We could instead
have wanted to return the data to the caller, or inserted it into
a bigger, greater data structure, or similar.

It's a bit less ugly when you use C++'s abstractions. Destructors
don't have to just call `free` (or rather its C++ analogue `delete`) as
`unique_ptr`'s does. Any C programmer can tell you that idiomatic
C code is rife with custom free functions to free all of the allocations
of a data structure, and C++ (and Rust) will choose which destructor
to call for you based on the type of the data. Calling `free` when
a custom destructor must be called is a common careless mistake in C.
This is true especially among beginners, and (hot take!) making
programming languages less needlessly tricky for beginners is a good
thing for everybody.

We can combine RAII with other features of C++ to get this more
idiomatic code, with the first `do`-`while` loop I've written
in years:

```c++
void print_int_little_endian_decimal(int foo) {
    std::string res;
    do {
        res += '0' + foo % 10;
        foo /= 10;
    } while (foo != 0);
    std::cout << res << std::endl;
}
```

Does `std::string` allocate memory on the heap? Maybe it only does
if the string goes above a certain size. But the custom destructor,
`~std::string`, will call `delete[]` only when the allocation was actually
made, abstracting that question away, along with handling terminating
nuls and avoiding overruns in a cleaner way.

This ability of RAII -- to call custom destructors that abstract away
allocation decisions -- gets more impressive when we consider that
many data structures don't make just 0 or 1 heap allocations, but whole
complicated trees of complicated heap allocations. In many cases, C++
(and Rust) will write your destructors for you, even for complicated
types like this:

```c++
struct PersonRecord {
    std::string name;
    uint64_t salary;
};

std::unordered_map<std::string, std::vector<PersonRecord>> thing;
```

To destroy `thing` in C, you'd have to loop through the hash map, free
all the keys, and then free all the values, which then requires freeing
all the strings in each `PersonRecord` before freeing the backing for
each vector. Only then could you free the actual allocations backing the
hash map.

And perhaps a C-based hash map library could do this for you, but only
by assuming that the keys are strings, and then taking a function
pointer to know how to free the values, which would ironically
be a form of dynamic polymorphism and therefore a performance hit. And
the function to free the values would then still have to manually free the
string, knowing which field of the `PersonRecord` was a pointer and
duplicating that information between the structure and the manually-written
"free" function, and still likely not supporting the small-string
optimization that C++ enables.

In C++, this freeing code is all automatically generated. `PersonRecord`
gets an automatic destructor that calls the destructor of each
field (`int`'s destructor is trivial), and the destructors of
`std::unordered_map` and `std::vector` are templated so that, at compile
time, a fresh destructor is built from those templates that handles all
of this, all without any indirect function calls or run-time cost beyond
what manually would be written for exactly this data structure in C.

See, with RAII, a destructor isn't just automatically and implicitly
called at the end of a scope in a function, but also in the destructors
of values ("objects" in C++) that *own* other values. Even if you
do write a custom destructor for aggregate types, that just specifies
what the computer should do on destruction *beyond* the automatic
calls to the destructors of the fields, which are still implicit.

# Ownership and its limitations

This is all possible based on the concept of "ownership," one of the key
principles of RAII. The key assumption is that every allocation has one
owner at any given time. Allocations can own each other (forming a tree
of allocations), or a scope can own an allocation (forming the root of
such a tree). RAII then can make sure the allocation ends when its owner
does -- by the scope exiting, or when the owning object is destroyed.

But what if the allocation needs to outlive its parent, or its scope?
It's not always the case that a function has primitive types as its
arguments and return value, and then only constructs trees of allocations
privately. We need to take these sophisticated collections and pass
them as arguments to functions. We need to have them be returned from
functions.

This becomes apparent if we try to refactor our
big-endian integer decimalizer to allow us to do other things with
the resultant string besides print it:

```c++
std::string render_int_little_endian_decimal(int foo) {
    std::string res;
    do {
        res += '0' + foo % 10;
        foo /= 10;
    } while (foo != 0);
    return res;
}

int main() {
    std::cout << render_int_little_endian_decimal(3781) << std::endl;
    return 0;
}
```

Based on our previous discussion of RAII, you might assume that
the `~std::string` destructor is called on the end of its scope,
rendering the allocation unusable for later printing, but instead
this code "Just Works."

We've hit one of many mitigations against the limitations of raw
RAII that are necessary for it to work. This mitigation is the "Named
Return Value Optimization (NRVO)," which stipulates that if a named
variable is used in all of the `return` statements in a function, it is
actually constructed (and destructed) in the context of the caller. It is
misnamed an "optimization" because it's actually part of the semantics:
It eliminates entirely the call to the destructor at the end of the scope,
even if that destructor call would have side effects.

This is just one of many ways RAII is made competitive with run-time
garbage collection, and we can have values that live outside of
a certain scope of a function. This one is narrow and peculiar to C++,
but many of the others lead to interesting comparisons. In the next
section, we discuss the others.

# Filling the Gaps in RAII
## Copying/Cloning

We're going to start with one of the oldest of these: copying.
When C++ was designed, the intention was that the programmer would
not see a difference between types that don't involve allocation
(like `int` or `double`) and types that do (like `std::string` or
`std::unordered_map<std::string, std::vector<std::string>>`.

When a function takes an `int` argument, as in
`print_int_little_endian_decimal`, that integer is copied.
Similarly, if we take a `std::string` argument without additional
annotation, C++ will also make a copy:

```c++
int parse_int_le(std::string foo) {
    int res = 0;
    int pos = 1;
    for (char c: foo) {
        res += (c - '0') * pos; // No input validation -- example!
        pos *= 10;
    }
    return res;
}

int main(int argc, char **argv) {
    std::string s = argv[1];
    std::cout << parse_int_le(s) << std::endl;
    return 0;
}
```

This is indeed consistent. Treating `int`s and `std::string` objects
in parallel ways is also in line with how higher-level programming
languages sometimes work: a string is a value, an `int` is a value,
why not give them the same semantics? Aliasing is confusing, why not
avoid it with copying?

It's made to work by an implicit function call. Just like destructor
calls are implicit in C++, copying also calls a function in the types
implementation. Here, it calls `std::string`'s "copy constructor."

The problem here is that this is slow. Not only is an unnecessary
copy made, but an unnecessary allocation and deallocation creep in.
There is no reason not to use the same allocation the caller already
has, here in `s` from the `main` function. A C programmer would never
write this copying version.

The only reason this feature is allowed under C++'s zero-cost
principle is because it is optional. It may be the default -- and
making it the default is one of the most questionable decisions C++
ever made -- but we can still alias if we want to. It just takes
more work.

Rust, as you can guess by my tone, requires explicit annotation to
copy types that have an allocation. In fact, Rust doesn't even use the
term "copy," which is reserved for types that can be copied without
allocations. It calls this cloning, and requires use of the `clone()`
method to accomplish it.

Some types don't use an allocation, and "copying" them is just a simple
memory copy. Some types do use an allocation, and "cloning" them requires
allocating. This distinction is important and fundamental to how computers
work. It's relevant and visible in Java and even Python, and pretending
it doesn't exist is unbecoming for a systems programming language like
C++.

## Moves

Returning an allocation from a function can't always use NRVO. So if
you want your value to outlast your function, but it's created inside
the function (and therefore "owned" by the function scope), what you
really need is a way for the value to change owners. You need to
be able to move the value from the scope into the caller's scope.
Similarly, if you have a value in a vector, and need to remove the
last value, you can move it.

This is distinct from copying, because, well, no copy is made --
the allocation just stays the same. The allocation is "moved" because
the previous scope no longer has responsibility for destroying the allocation,
and the new scope gains the responsibility.

Move semantics fix the most serious issue with RAII: your allocation
might not live exactly as long as its owner. The root of an allocation
tree might outlive the stack-based scope it's in, such as when you want to
return a collection from a function. The other nodes of an allocation tree
might leave that tree and be owned by another stack frame, or by another
part of the same allocation tree, or by a different allocation tree. In
general, "each allocation has a unique owner" becomes "each allocation
has a unique owner at any given time," which is much more flexible.

In Rust, this is done via "destructive moves," which oddly enough
means *not* calling the destructor on the moved-from value. In fact,
the moved-from value ceases to be a value when it's moved from, and
accessing that variable is no longer permitted.  The destructor is
then called as normal in the place where the value is moved to. This
is tracked statically at compile-time in the vast majority of situations,
and when it cannot be, an extra boolean is inserted as a ["drop
flag"](https://doc.rust-lang.org/nomicon/drop-flags.html) ("drop" is
how Rust refers to its destructors).

C++ didn't add move semantics until C++11; it was not part of the
original RAII scheme. This is surprising given
how essential moves are to RAII. Returning collections
from functions is super important, and you can't copy every time.
But before C++, there were only poor man's special cases for move,
like NRVO and the related RVO for objects constructed in the return
statement itself. These have completely different semantics than C++
move semantics -- they're still more efficient than C++ moves in
many cases.

When C++ did eventually add moves, the other established semantics of C++
forced it to add moves in a weird and deeply confusing way: it added
"non-destructive" moves. In C++, rather than the drop flag being a flag
inserted by the compiler, it is internal to the value. Every type that
supports moves must have a special "empty state," because the destructor
is called on the moved-from value. If the allocation had moved to
another value, there would be no allocation to free, and this had
to be handled by the destructor at run-time, which can amount to
a violation of the zero-cost principle in some situations.

C++ justifies this by making moves a special case of copy. Moves
are said to be like copies, but make no promises of preserving the
initial value.  In exchange, you might get the optimization of being
able to use the original allocation, but then the initial value will
not have an allocation, and will be forced to be different. This
definition is very different than what moves are actually used for
(cf. the name of the operation), and therefore, even though it is
[technically simple](https://herbsutter.com/2020/02/17/move-simply/),
claiming that focusing on that definition (as Herb Sutter does) will
simplify things for the programmer is disingenuous, as I discuss in more
detail in the next chapter on moves specifically.

In practice, this means that all types support the operation of moving --
even `int`s -- but even some types that manage an allocation might fall
back on copying if moves haven't been implemented for them. This
inconsistency, like all inconsistencies, is bad for programmers.

In practice, this also means that moved-from objects are a problem.
A moved-from object might stay the same, if no moving was done. It
might also change in value, if the move caused an allocation
(or other resource) to move into the new object. This forces
C++ smart pointers to choose between movability and non-nullability --
no moveable, non-nullable pointer is possible in C++. Nulls -- and the
other "moved-from" empty collections that you get from C++ move
semantics -- can then be referenced later on in the function,
and though they must be "valid" values of the object, they are
probably not the values you expect, and in the case of null pointers,
they are famously difficult values to reason about.

This is a consequence of the fact that C++ was a pioneer of RAII
semantics, and didn't design RAII and moves together from the
start. Rust has the advantage of having included moves from
the beginning, and so Rust move semantics are much cleaner.

In Rust also, all types can be moved. But in Rust, no resources or
allocations are ever copied. Instead, moves always have the same
implementation: copy the memory that is stored in-line in the value
itself, and then do not call the destructor. For copyable types like
`int` that do not manage an allocation or other resource, this does
amount to a copy, but the original is still not usable. But no allocation
or resource is ever copied; for those types, the pointer or handle is
simply brought along bit-by-bit just like other data, and the old value
is never touched again, making this a safe operation.

All types must then be written in such a way to assume that
values might not stay in the same place in memory. If some operations on
a type can't be written that way, they can be defined on "pinned"
versions of that type. A pin is a type of reference or box that
promises that the pointed-to value will never move again. The underlying
type is still movable, but these particular values are not.

This is a gnarly exception to Rust's "all types can be moved" rule that
make it false in practice, though still true in pedantic, language-lawyery
theory. But that's not important. What is important is that Rust's
move semantics are consistent, and do not rely on move constructors
and manual implementations of Rust's drop flags within the object. The
dangerous possibility of interacting with a moved-from object, whose value
is unpredictable and quite possibly a special "empty" state like null,
is not present in Rust.

## Borrows in Rust

While moves cover returning a collection (or other resource-managing
value) from a function, they don't cover passing such a value into a
function, or at least not in the general case. Sometimes, when we pass
a value into a function, we want to move the value in, so that the
function can consume it or add it to an allocation tree (like inserting
into a collection). But most times, we want the function to be able
to see and perhaps mutate it, but then we want to give it back to
the owner.

Enter the borrow.

In Rust, borrows are commonly introduced as a sort of an improvement on
moves. Consider our example function that parses a string to an `int`,
here implemented in C++ with copies:

```c++
int parse_int_le(std::string foo) {
    int res = 0;
    int pos = 1;
    for (char c: foo) {
        res += (c - '0') * pos; // No input validation -- example!
        pos *= 10;
    }
    return res;
}
```

Here is a Rust version, with moves, so that the function consumes
the string:

```rust
use std::env::args;

fn parse_int_le(foo: String) -> u32 {
    let mut res = 0;
    let mut pos = 1;
    for c in foo.chars() {
        res += (c as u32 - '0' as u32) * pos;
        pos *= 10;
    }
    res
}

fn main() {
    let mut args: Vec<String> = args().collect();
    println!("{}", parse_int_le(args.remove(1)));
}
```

As we can see with the "move" version of this, we are in the awkward
position of removing the string from the vector, so that `parse_int_le`
can consume the string, so it doesn't have multiple owners.

But `parse_int_le` doesn't need to own the string. In fact, it could
be written so that it can give the string back when it's done:

```rust
fn parse_int_le(foo: String) -> (u32, String) {
    let mut res = 0;
    let mut pos = 1;
    for c in foo.chars() {
        res += (c as u32 - '0' as u32) * pos;
        pos *= 10;
    }
    (res, foo)
}
```

"Taking temporary ownership" in real life is also known as
borrowing, and Rust has such a feature built-in. It is more powerful
than the above code that literally takes temporary ownership, though.
That code would have to remove the string from the vector and then
put it back -- which is even more inefficient than just removing it.
Rust borrowing allows you to borrow it even while it's inside
the vector, and stays inside the vector. This is implemented by
a Rust reference, which has this borrowing semantics, and is,
like most "references," implemented as a pointer at the machine level.

In order to accomplish these semantics, Rust has its infamous borrow
checker. While we are borrowing something inside the vector, we can't
simultaneously be mutating the vector, which could cause the thing we're
borrowing to move.  Rust statically ensures that this is impossible,
rejecting code that use a reference after a mutation, destruction,
or move somewhere else would invalidate it.

This enables us to extend the RAII-based system and both prevent
leaks and maintain safety, just like a GC or RC-based system. The
borrow checker is essential to doing so.

For completeness, here is the idiomatic way to handle the
parameter in `parse_int_le`, with an actual borrow, using
`&str`, the special borrowed form of `String` that also allows
slices:

```rust
use std::env::args;

fn parse_int_le(foo: &str) -> u32 {
    let mut res = 0;
    let mut pos = 1;
    for c in foo.chars() {
        res += (c as u32 - '0' as u32) * pos;
        pos *= 10;
    }
    res
}

fn main() {
    let args: Vec<String> = args().collect();
    println!("{}", parse_int_le(&args[1]));
}
```

## Dodging memory safety in C++

In C++, of course, there is no borrow checker. In the `parse_int_le`
example, it's still possible to use a pointer, or a reference, but then
you're on your own. When RAII-based code frees your allocation, your
reference is invalidated, which means it's undefined behavior to use it.
No coordination is performed by the compiler between the RAII/move
system and your references, which point into the ownership tree
with no guarantee that said tree won't move underneath it.
This can lead to memory corruption bugs, with security implications.

It's not just pointers and references. Other types that contain
references, such as iterators, can also be invalidated. Sometimes
those are more insidious because intermediate C++ programmers might
know about pointer invalidation, but let their guard down with
iterators. If you add to a vector while looping through it,
you've just done undefined behavior, and that's surprising because
no pointers or references even have to show up. Rust's borrow
checker handles these as well.

Even though the Rust borrow checker gets a bad reputation, its safety
guarantees often make it worth it. It's hard to write correct C++ when
references and non-owning pointers are involved.  Maybe some of you
have that skill, and are unsympathetic to those who don't yet have it,
but it is a specialized skill, and the compiler can do a lot of the work
for you, by checking your work. Automation is a good thing, and so is
making systems programming more accessible to beginners.

And of course, many C++ programmers do make mistakes. Even if it's not
you, it might be one of your colleagues, and then you'll have to clean
up the mess. Rust addresses this, and limits this more difficult mode
of thinking to writing unsafe code, which can be contained in modules.

## Multiple Ownership

In RAII, an allocation has one owner at a time, and if your owner is destroyed
before the allocation is moved to another owner, the allocation must be
destroyed along with it.

Of course, sometimes this isn't how your allocations work. Sometimes they need
to live until both of two parent allocations are destroyed, and sometimes
there is no way to predict which parent is destroyed first. Sometimes,
the only way to solve that situation -- even in C -- is to use runtime
information -- and so you can model *multiple ownership* through reference
counting: `std::shared_ptr` in C++, or `Rc` and `Arc` in Rust (depending
on whether it is shared between multiple threads).

This is something that C programmers will sometimes do in the face
of complicated allocation DAGs, and end up implementing bespoke on a
framework-by-framework basis (cf. GTK+ and other C GUI frameworks).
C++ and Rust are just standardizing the implementation of this, but, in
line with the zero-cost rule, making it optional.

Interestingly enough, reference counting is implemented in terms of
RAII and moves. The destructor for a reference-counted pointer decreases
the reference, and cloning/copying such a pointer increases it. Moves,
of course, don't change it at all.

# RAII+: What this all adds up to

Between RAII, moves, reference counting, and the borrow checker, we now
have the memory management system of safe Rust. Safe Rust is a powerful
programming language, and in it, you can write programs almost as easily
as in a traditionally GC'd programming language like Java, but
get the performance of manually written, manually memory managed C.

The cost is annotation. In Java, there is no distinction between
"borrowing" and "owning", even though sometimes the code follows
similar structures as if there were. In Rust, the compiler must
be informed about the chain of owners, and about borrowers.
Every time an allocation crosses scope boundaries or is referred
to inside another allocation, you must write different syntax
to tell Rust whether it's a move or a borrow, and it must
comply with the rules of the borrow checker.

But it turns out most code has a natural progression of owners,
and most borrows are valid in the borrow checker. When they're not,
it's usually straight-forward to rethink the code so that it can
work that way, and the resultant code is usually cleaner anyway.
And in situations where neither of them work, reference counting
is still an option.

At the cost of this annotation, Rust gives you everything a GC
does: Allocations are freed when their handles go out of scope,
and memory safety is still guaranteed, because the annotations
are checked. Memory leaks are as difficult as in a reference
counting language, and the annotations are checked, which is
most of the benefit of automating them. It's an excellent
happy medium between manual memory management and full run-time
GC with no run-time cost over a certain discipline of C memory
management.

Of course, other disciplines of C memory management are
possible.  And using this Rust system takes away flexibility
that might be relevant to performance. Rust, like C++, allows
you to sidestep the "compile-time GC" and use raw pointers,
and that can often be better for performance. [A recent blog
post](https://matklad.github.io/2022/10/06/hard-mode-rust.html) I read
explores some of that in more detail; encouragingly, that blog
post also considers RAII to be in-between manual memory management
and run-time GC -- serendipitously, because I had already drafted
much of this post when it came out.

But the standard memory management tools of Rust cover the common
cases well, and unsafe is available for when it's inappropriate --
and can be wrapped in abstractions for interfacing with code that
uses the RAII-based system.

In C++, the annotations of "borrows" vs "moves" can easily result
in undefined behavior. Leaks are prevented, but memory corruption is
not. So the C++ system is a much worse replacement for garbage collection
-- RAII is only doing some of its job, as it is not paired with
a borrow checker.

# Cycles

I leave the most awkward topic for the end. We've talked about allocation
trees and DAGs, but not general graphs. These require `unsafe` in Rust,
even something as supposedly basic as doubly linked lists. It's against
the borrow checker's rules, and the compiler will statically prevent
you from making them using safe, borrowing references. They simply aren't
borrows in the Rust sense, but are rather something else, something
about which Rust doesn't know how to guarantee safety.

This is not as bad as you might think, because cycles also form a hole in
reference counting, which is a popular run-time GC system. This is why
you can't use `Rc` or `Arc` to implement a doubly-linked list correctly in
Rust either: You'll get past the borrow checker and guarantee a memory
leak.. These systems generally can't detect cycles at all, and leak them,
which is arguably worse than forbidding them to be created.

In any case, the `unsafe` keyword is not poison. For things that Rust
doesn't know how to keep safe, you need to exercise extra responsibility,
but at least the programming language is making you aware of it --
unlike C++, which is unsafe all the time.
