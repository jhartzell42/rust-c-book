# Moves

* TODO: Integrate the following ideas from this introduction into the
  old blog post. Each sentence from this introduction probably deserves
  to be its own several paragraphs.

Both Rust and C++ support move operations for custom data types, as a more
RAII-friendly alternative to aliasing and a more performant alternative
to copy-then-destroy. Any language with the goals of modern C++ and Rust
must support moves to be both performant and ergonomic. Unfortunately,
C++ only got support for move semantics in C++11, after C++ had already
been in existence for decades.

As a consequence, C++ had to work moves around the existing syntax and
semantics. Copying remains the default operation in most contexts, even
though they're more expensive than moves and sometimes impossible. The
syntax surrounding moves is awkward, and involves two new types
of reference (rvalue and universal), which is confusing. But most
importantly, and I would argue, most confusingly, C++ did not
implement moves destructively.

* The snippet from the original blog post starts here

In 2011, C++ finally fixed a set of long-standing deficits in the
programming language with the shiny new C++11 standard, bringing it into
the modern era. Programmers enthusiastically pushed their companies to
allow them to migrate their codebases, champing at the bit to be able to
use these new features. Writers to this day talk about "modern C++,"
with the cut-off being 2011. Programmers who only used C++ pre-C++11 are
told that it is a new programming language, the best version of its old
self, worth a complete fresh try.

There were a lot of new features to be excited about. C++ standard
threads were added then -- and thread standardization was indeed good,
though anyone who wanted to use threads before likely had their choice
of good libraries for their platform.  Closures were also very exciting,
especially for people like me who came from functional programming, but
to be honest, closures were just syntactic sugar for existing patterns
of boilerplate that could be readily used to write function objects.

Indeed, the real excitement at the time, certainly the one my colleagues
and I were most excited about, was move semantics. To explain why this
feature was so important, I'll need to talk a little about the C++
object model, and the problem that move semantics exist to solve.

## Value Semantics

Let's start by talking about a primitive type in C++: `int`. Objects --
in C++ standard parlance, `int` values are indeed considered objects --
of type `int` only take up a few bytes of storage, and so copying them
has always been very cheap. When you assign an `int` from one variable
to another, it is copied. When you pass it to a function, it is copied:

```
int print_i(int arg) {
    arg += 3;
    std::cout << arg << std::endl;
}

int foo = 3;
int bar = foo; // copy
foo += 1; // foo gets 4
std::cout << bar << std::endl; // bar is still 3
print_i(foo); // prints 4+3 ==> 7
std::cout << foo << std::endl; // foo is still 4
```

As you can see, every variable of type `int` acts independently of each
other when mutated, which is how primitive types like `int` work in many
programming languages.

In the C++ version of object-oriented programming, it was decided that
values of custom, user-defined types would have the same semantics, that
they would work the same way as the primitive types. So for C++ strings:

```
std::string foo = "foo";
std::string bar = foo; // copy (!)
foo += "__";
bar += "!!";
std::cout << foo << std::endl; // foo is "foo__"
std::cout << bar << std::endl; // bar is "foo!!"
```

This means that whenever we assign a string to a new variable, or
pass it to a function, a copy is made. This is important, because
the `std::string` object proper is just a handle, a small structure
that manages a larger memory allocation on the heap, where the
actual string data is stored. Each new `std::string` that is made
via copy requires allocating a new heap allocation, a relatively
expensive operation in performance.

This would cause a problem when we want to pass a `std::string` to a
function, just like an `int`, but don't want to actually make a copy
of it. But C++ has a feature that helps with that: `const` references.
Details of the C++ reference system are a topic for another post, but
`const` references allow a function to operate on the `std::string`
without the need for a copy, but still promising not to change the
original value.

The feature is available for both `int` and `std::string`; the principle
that they're treated the same is preserved. But for the sake of performance,
`int`s are passed by value, and `std::string`s are passed by `const`
reference in the same situation. In practice, this dilutes the benefit
of treating them the same, as in practice the function signatures
are different if we don't want to trigger spurious expensive deep copies:

```
void foo(int bar);
void foo(const std::string &bar);
```

If you instead declare the function `foo` like you would with an `int`,
you get a poorly performing deep copy. The default is something you
probably don't want:

```
void foo(std::string bar);
void foo2(const std::string &bar);
`
std::string bar("Hi"); // Make one heap allocation
foo(bar); // Make another heap allocation
foo2(bar); // No copy is made
```

This is all part of "pre-modern" C++, but already we're seeing negative
consequences of the decision to treat `int` and `std::string` as identical
when they are not, a decision that will get more gnarly when applied to
moves. This is why Rust has the `Copy` trait to mark types like `i32`
(the Rust equivalent of `int`) as being copyable, so that they can be
passed around freely, while requiring an explicit call to `clone()`
for types like `String` so we know we're paying the cost of a deep copy,
or else an explicit indication that we're passing by reference:

```
fn foo(bar: String) {
    // Implementation
}

fn foo2(bar: &str) {
    // Implementation
}

let bar = "hi".to_string();
foo(bar.clone());
foo2(&bar);
```

The third option in Rust is to move, but we'll discuss that after
we discuss moves in C++.

## Copy-Deletes and Moves

C++ value semantics break down even more when we do need the
function to hold onto the value. References are only valid as long
as the original value is valid, and sometimes a function needs it
to stay alive longer. Taking by reference is not an option when
the object (whether `int` or `std::string`) is being added to a vector
that will outlive the original object:

```
std::vector<int> vi;
std::vector<std::string> vs;
{
    int foo = 3;
    foo += 4;
    vi.push_back(foo);
} // foo goes out of scope, vi lives on
{
    std::string bar = "Hi!";
    bar += " Joe!";
    vs.push_back(bar);
} // bar goes out of scope, vs lives on
```

So, to add this string to the vector, we must first make an
allocation corresponding to the object contained in the variable
`bar`, and then must make a new allocation for the object that lives in
`vs`, and then copy all the data.

Then, when `bar` goes out of scope, its destructor is called, as is
done automatically whenever an object with a destructor goes out
of scope. This allows `std::string` to free its heap allocation.

Which means we copied an allocation into a new heap allocation, just to free
the original allocation. Copying an allocation and freeing the old one
is equivalent to just re-using the old allocation, just slower. Wouldn't
it make more sense to make the string in the vector just refer to the
same heap allocation that `bar` formerly did?

Such an operation is referred to as a "move," and the original C++ --
pre C++11 -- didn't support them. This was possibly because they didn't
make sense for `int`s, and so they were not added for objects that were
trying to act like `int`s -- but on the other hand, destructors were
supported and `int`s don't need to be destructed.

In any case, moves were not supported. And so, objects that managed
resources -- in this case, a heap allocation, but other resources could
apply as well -- could not be put onto vectors or stored in collections
directly without a copy and delete of whatever resource was being managed.

Now, there were ways to handle this in pre-C++11 days. You could add
an indirection, and make a heap allocation to contain the `std::string`
object, which is only a small object with a pointer to another allocation,
but would at least let you pass around a `std::string *` which is a
raw pointer that would not trigger all these copies by automatically
managing the heap allocation with this façade of value semantics. Or
you could manually manage a C-style string with `char *`.

But the most ergonomic, clear `std::vector<std::string>` could not be
used without performance degradation. Worse, if the vector ever needed
to be resized, and had to itself switch to a different allocation, it
would have to copy all those `std::string` objects internally and
delete the originals, N useless reallocations.

As a demonstration of this, I wrote a [sample
program](/string_example/string.cpp) with a vastly simplified
version of `std::string`, that tracks how many allocations it makes.
It allows C++11-style moves to be enabled or disabled, and then it
takes all the command line arguments, creates `string` objects out
of them, and puts them in a vector. For 8 command line arguments,
the version with move made, as you might expect, 8 allocations,
whereas the version without the move, that just put these strings
into a vector, made 23. Each time a string was added to a vector,
a spurious allocation was made, and then N spurious allocations
had to be made each time the vector doubled.

This problem is purely an artifact of the limitations of the tools
provided by C++ to encapsulate and automatically manage memory, RAII and
"value semantics."

Consider this snippet of code:

```
std::vector<std::string> vec;
{ // This might take place inside another function
  // Using local block scope for simplicity
    std::string foo = "Hi!";
    vec.push_back(foo);
}
{
    std::string bar = "Hello!";
    vec.push_back(bar);
}
// Use the vector
```

If we didn't use this `string` class, we would then
have not done a copy, just to free the original allocation. We would
have simply put the pointer into the vector. We would then have been
responsible for freeing all the allocations -- once -- when we're done:

```
std::vector<char *> vec;
{
    // strdup, a POSIX call, makes a new allocation and copies a
    // string into it, here used to turn a static string into one
    // on the heap. We will assume we have a reason to store it
    // on the heap -- perhaps we did more manipulation in the
    // real application to generate the string.

    char *foo = strdup("Hi!");
    vec.push_back(foo);
}
{
    char *bar = strdup("Hello!");
    vec.push_back(bar);
}

// Use the vector

// Then, later, when we are done with the vector, free all the elements once
for (char *c: vec) {
    free(c);
}
```

The copy version of the C++ code instead does -- after de-sugaring the
RAII and value semantics and inlining -- something that no programmer
would ever write manually, something equivalent to this (the vector is
left in object-oriented notation for readability):

```
std::vector<char *> vec;
{
    char *foo = strdup("Hi");
    vec.push_back(strdup(foo)); // Why the additional allocate-and-copy?
    free(foo); // Because the destructor of foo will free the original
}
{
    char *bar = strdup("Hello!");
    vec.push_back(strdup(bar));
    free(bar);
}

// Use the vec
for (char *c: vec) {
    free(c);
}
```

C++ without move semantics fails to reach its goal of zero-cost abstraction.
The version with the abstraction, with the value semantics, compiles to code
less efficient than any code someone would write manually, because what
we really want is to allocate the allocation while it's a local variable
`foo`, use the same allocation on the vector, and then only free it on the
vector.

The abstractions of only supporting "copy" and "destruct" mean that the
destructor of the variable `foo` must be called when `foo` goes out of
scope. This means that the "copy" operation must make an independent
allocation, as it cannot control when the original goes out of scope,
or will be replaced with another value.  If we had instead re-used the
same allocation, it would be freed by `foo`s destructor.

But copying just to destroy the original is silly -- silly and
ill-performant.  What any programmer would naturally write in that
situation results in a "move". So this gap -- and it was a huge gap --
in C++ value semantics was filled in C++11 when they added a "move"
operation.

Because of this addition, using objects with value semantics that managed
resources became possible. It also became possible to use objects with
value semantics for resources that could not meaningfully be copied,
like unique ownership of an object or a thread handle, while still
being able to get the advantages of putting such objects in collections
and, well, moving them. Shops that previously had to work around value
semantics for performance reasons could now use them directly.

It is not, therefore, surprising that this was for many the most exciting
change in C++11.

## How Move Is Implemented in C++

But for now, let's put ourselves in the place of the language designers
who designed this new move operation. What should this move operation
look like? How could we integrate it into the rest of C++?

Ideally, we would want it to output -- after inlining -- exactly the
code that we would expect to write manually. When `foo` is moved into the
vector, the original allocation must not freed. Instead, it is only freed
when the vector itself is freed. This is an absolute necessity to solve
the problem as we must remove a free in order to remove the allocation,
but we also cannot leak memory. If there is to be exactly one allocation,
there must be exactly one deallocation.

Calls to `free` (or `delete[]` in my example program) are made in the
destructor, so the most straight-forward way to go forward is to say
that the destructor should only be called when the vector is destroyed,
but not when `foo` goes out of scope. If `foo` is moved onto the vector,
then the compiler should take note that it has been moved from, and simply
not call the destructor. The move should be treated as having already
destroyed the object, as an operation that accomplishes both initialization
of the new object (the string on the vector) from the original object and the
destruction of the original object.

This notion is called "destructive move," and it is how moves are done
in Rust, but it is not what C++ opted for.  In Rust, the compiler would
simply not output a destructor call (a "drop" in Rust) for `foo` because
it has been moved from. But, in fact, the C++ compiler still does. In
destructive move semantics, the compiler would not allow `foo` to be
read from after the move, but in fact, the C++ compiler still does,
not just for the destructor, but for any operation.

So how is the deallocation avoided, if the compiler doesn't remove it
in this situation? Well, there is a decision to make here.  If an object
has been moved from, no deallocation should be performed. If it has not,
a deallocation should be performed. Rust makes this decision at compile-time,
but C++ makes it at run-time.

When you write the code that defines what it means to move from
an object in C++, you must make sure the original object is in a
run-time state where the destructor will still be called on it, and
will still succeed. And, since we established already that we must save
a deallocation by moving, that means that the destructor must make a
run-time decision as to whether to deallocate or not.

The more C-style post-inlining code for our example would then look
something like this:

```
std::vector<char *> vec;
{
    char *foo = strdup("Hi!");
    vec.push_back(foo);
    foo = nullptr;
    if (foo != nullptr) {
        free(foo);
    }
}
{
    char *bar = strdup("Hi!");
    vec.push_back(bar);
    bar = nullptr;
    if (bar != nullptr) {
        free(bar);
    }
}
```

This null check is hidden by the fact that in C++, `free` and `delete`
and friends are defined to be no-ops on null, but it still exists.
And while the check might be very cheap compared to the cost of calling
`free`, it might not be cheap when things are moved in a tight loop,
where `free` is never actually called. That is to say, this
run-time check is not cheap compared to the cost of not calling free.

So, given the semantics of move in C++, it results in code that is not
the same as -- and not as performant as -- the equivalent hand-written
C-style code, and therefore it is not a zero-cost abstraction, and
doesn't live up to the goals of C++.

Now, it looks like the optimizer should be able to clean up an adjacent
set to null and check for null, but not all examples are as simple as this
one, and, like in many situations where the abstraction relies on the
optimizer, the optimizer doesn't always get it.

## Arguing Semantics

But that performance hit is small, and it is usually possible to optimize
out. If that were the only problem with C++ move semantics, I might find
it annoying, but ultimately I'd say, like about many things in about both
C++ and Rust, something like: Well, this decision was made, remember to
profile, and if you absolutely have to make sure the optimizer got it
in a particular instance, check the assembly by hand.

But there's a few further consequences of that decision.

First off, the resource might not be a memory allocation, and null pointers
might not be an appropriate way to indicate that that resource doesn't
exist. This responsibility of having some run-time indication of what
resources need to be freed -- rather than a one-to-one correspondence
between objects and resources -- is left up to the implementors of classes.
For heap allocations, it is made relatively easy, but the implementor
of the class is still responsible for re-setting the original object.
In my example, the move constructor reads:

```
string(string &&other) noexcept {
    m_len = other.m_len;
    m_str = other.m_str;
    other.m_str = nullptr; // Don't forget to do this
}
```

The move constructor has two responsibilities, where a destructive
version would only have one: It must set up state for the new object,
and it must set up a valid "moved from" state for the old object.
That second obligation is a direct consequence of non-destructive moves,
and provides the programmer with another chance to mess something up.

In fact, since destructive moves can almost always be implemented by
just copying the memory (and leaving the original memory as garbage
data as the destructor will not be called on it), a default move
constructor would correctly cover the vast majority of implementations,
creating even fewer opportunities to introduce bugs.

But furthermore, the moved-from state also has obligations. The destructor
has to know at run-time not to reclaim any resources if the object no
longer has any, but in general, there is no rule that moved-from objects
must immediately be destroyed. The programming language has explicitly
decided not to enforce such a rule, and so, to be properly safe, moved-from
objects must be considered -- and must be -- valid values for those objects.

This means that any object that manages a resource now must manage either
1 or 0 copies of that resource. Collections are easy -- moved from collections
can be made equivalent to the "empty" collection that has no element. For
things like thread handles or file handles, this means that you can have
a file handle with no corresponding file. Optionality is imported to all
"value types."

So, smart pointer types that manage single-ownership heap allocations, or
any sort of transferrable ownership of heap allocations, now of necessity
must be nullable. Nullable pointers are a serious cause of errors, as
often they are used with the implicit contract that they will not be null,
but that contract is not actually represented in the type. Every time a
nullable pointer is passed around, you have a potential miscommunication
of whether `nullptr` is a valid value, one that will cause some sort
of error condition, or one that may lead to undefined behavior.

C++ move semantics of necessity perpetuate this confusion. Non-nullable
smart pointers are unimplementable in C++.

## Move, Complicatedly

This leads me to Herb Sutter's
[explanation of C++ move semantics](https://herbsutter.com/2020/02/17/move-simply/) from his blog. I respect Herb Sutter greatly as someone explaining
C++, and his materials helped me learn C++ and teach it. An explanation
like this is really useful if programming in C++ is what you have to do.

However, I am instead investigating whether C++'s move semantics are
reasonable, especially in comparison to programming languages like Rust
which do have a destructive move. And from that point of view, I think
this blog post, and its necessity, serve as a good illustration of the
problems with C++'s move semantics.

I shall respond to specific excerpts from the post.

> C++ “move” semantics are simple, and unchanged since C++11. But they
> are still widely misunderstood, sometimes because of unclear teaching
> and sometimes because of a desire to view move as something else instead
> of what it is.

Given what he's going to say is the definition of C++ move semantics, I
think this is unfair. The goal of move is clear: to allow resources to be
transferred when copying would force them to be duplicated. It is obvious
from the name. However, the semantics as the language defines them,
while enabling that goal, are defined without reference to that goal.

This is doomed to lead to confusion, no matter how good the teaching is.
And it is desirable to try to understand the semantics as they connect
to the goal of the feature.

To explain what I mean, see the definition given for moving:

> In C++, copying or moving from an object `a` to an object `b` sets `b` to
> `a`’s original value. The only difference is that copying from `a` won’t
> change `a`, but moving from a might.

This is a fair statement of C++'s move semantics as defined. But it is
made without any reference to the goals. Note that, in this definition,
we are discussing the assignment written as `b = a` or as `b = std::move(a)`.

The reason why moving might change `a`, as we've discussed, is that
`a` might contain a resource. Moving indicates that we do not wish
to copy resources that are expensive or impossible to copy, and that
in exchange for this ability, we give up the right to expect that `a`
retain its value.

This definition is the correct one to use for reasoning about C++
programs, but it is not directly connected to why you might want
to use the feature at all. It is natural that programmers would
want to be able to reason about a feature in a way that aligns with
its goals.

The goal of this post is to obscure the goal, and to treat
move as if it were a pure optimization of copy, which will not
help a programmer understand why `a`'s value might change, or why
move-only types like `std::unique_ptr` exist.

The explanation of the goal of this operation is reserved in this post
for the section entitled "advanced notes for type implementors".

Of course, almost all C++ programmers in a sufficiently large project
have to become "type implementors" to understand and maintain custom
types, if not to write fresh implementations of them, so I think
most professional programmers should be reading these notes, and
so I think it's unfair to call them advanced. But beyond that,
this explanation is core to why the operation exists, and the only
explanation for why move-only types exist, which all C++ programmers
will have to use:

> For types that are move-only (not copyable), move is C++’s closest
> current approximation to expressing an object that can be cheaply moved
> around to different memory addresses, by making at least its value
> cheap to move around.

He follows up with an acknowledgement that destructive moves are
a theoretical possibility:

> (Other not-yet-standard proposals to go further
> in this direction include ones with names like “relocatable” and
> “destructive move,” but those aren’t standard yet so it’s
> premature to talk about them.)

For his purposes, this is extremely fair, but since my purposes are to
compare C++ to Rust and other programming languages which have destructive
moves, it is not premature for me to talk about them.

This gets more interesting in the Q&A.

> How can moving from an object not change its state?
>
> For example, moving an int doesn’t change the source’s value because
> an int is cheap to copy, so move just does the same thing as copy. Copy
> is always a valid implementation of move if the type didn’t provide
> anything more efficient.

Indeed, for reasons of consistency and generic programming, move is
defined on all types that can be moved or copied, even types that don't
implement move differently than copy.

What makes this confusing in C++, however, is that types that manage
resources might be written without an implementation of move. They might
pre-date the move feature, or their implementor might not have understood
move well enough to implement them, or there might be a technical reason
why moving couldn't be implemented in a way that elides the resource
duplication. For these types, a move falls back on a copy, even if the
copy does significant work. This can be surprising to the programmer,
and surprises in programming are never good. More direly, there
is no warning when this happens, because the notion of resource management
is not referenced in the semantics.

In Rust, a move is always implemented by copying the data in the object
itself and then not destructing the original object, and never by copying
resources managed by the object, or running any custom code.

> But what about the “moved-from” state, isn’t it special somehow?
> 
> No. The state of a after it has been moved from is the same as the
> state of a after any other non-const operation. Move is just another
> non-constfunction that might (or might not) change the value of the
> source object.

I disagree in practice. For objects that use move as intended, to avoid
copying resources, move will (at least usually) drain its resource. This
means that an object that often manages a resource will enter a state in
which it is not managing a resource. That state is special, because it is
the state when a resource-managing object is doing something other than
its normal job, and is not managing a resource. This is not a "special
state" by any rigorous definition, but is guaranteed to be intuitively
special by virtue of being resource-free. (It is also a special state
in that the value is unspecified in general, whereas most of the time,
the value is specified.)

Collections can, as I said before, get away with becoming the
empty collection in this scenario, but even for those, the empty
state is special: It is the only state that can be represented without
holding a resource. And many other types of objects cannot even
do this. `std::unique_ptr`'s moved-from state is the null pointer,
and without these move semantics, it would be possible to design a
`std::unique_ptr` that did not have a null state.

Once `std::unique_ptr` is forced to be allowed to have null values, it
makes sense that there be other ways to create a null `std::unique_ptr`,
e.g. by default-constructing it. But it is the design of move semantics
that force it to have a null value in the first place.

Put another way: `std::unique_ptr` and thread handles are therefore
collections of 0 or 1 heap allocation handles or thread handles, and
once defined that way, the "empty" state is not special, but it is move
semantics that force them to be defined that way.

> Does “but unspecified” mean the object’s invariants might not hold?
>
> No. In C++, an object is valid (meets its invariants) for its entire
> lifetime, which is from the end of its construction to the start of its
> destruction.... Moving from an object does not end its
> lifetime, only destruction does, so moving from an object does not make
> it invalid or not obey its invariants.

This is true, as discussed above. The moved-from object must be able to
be destructed, and there is nothing stopping a programmer for instead
doing something else with it. Given that, it must be in some state that
its operations can reckon with. But that state is not necessarily
one that would be valid if move semantics didn't force its conclusion,
and so again, we are close to the problem.

> Does “but unspecified” mean the only safe operation on a moved-from
> object is to call its destructor?
> 
> No.
> 
> Does “but unspecified” mean the only safe operation on a moved-from
> object is to call its destructor or to assign it a new value?
> 
> No.
> 
> Does “but unspecified” sound scary or confusing to average
> programmers?
> 
> It shouldn’t, it’s just a reminder that the value might have changed,
> that’s all. It isn’t intended to make “moved-from” seem mysterious
> (it’s not).

I disagree firmly with the answer to the last question. "Unspecified"
values are extremely scary, especially to programmers on team projects,
because it means that the behavior of the program is subject to arbitrary
change, but that change will not be considered breaking.

For example, `std::string` does not make any promises about the contents
of a moved-from string. However, a programmer -- even a senior programmer
-- may, instead of consulting the documentation, write a test program
to find out what the value is of a moved-from string. Seeing an empty
string, the programmer might write a
[program](/string_example/split.cpp) that relies on the string
being empty:

```
std::vector<std::string>
split_into_chunks(const std::string &in) {
    int count = 0;
    std::vector<std::string> res;
    std::string acc;
    for (char c: in) {
        if (count == 4) {
            res.push_back(std::move(acc));
            // Don't need to clear string.
            // I checked and it's empty.
            count = 0;
        }
        acc += c;
    }
}
```

Of course, you should not do that. A later version of `std::string`
might implement the small string optimization, where strings of
below a certain size are not stored in an expensive-to-copy heap
resource, but in the actual object itself. In that situation, it would
be reasonable to implement move as a copy, which is allowed, and then
this program would no longer do the same thing.

But this is a surprise. This is a result of the "unspecified value."
And so while it may, strictly speaking, be "safe" to do things
with a moved-from object other than destruct them or assign to them,
in practice, without documentation to the contrary making stronger
guarantees, the only way to get "not surprising" behavior is to
greatly limit what you do with moved-from objects.

> What about objects that aren’t safe to be used normally after being moved
> from?
>
> They are buggy....

By this definition, `std::unique_ptr` should likely be considered buggy,
as null pointers cannot be used "normally". Similarly, a `std::thread`
object that does not represent a thread handle. It is only by stretching
the definition of "used normally" to include these special "empty values"
that `std::unique_ptr` gets to claim to not be buggy under that definition,
although a null pointer simply cannot be used the way a normal pointer
can.

Again, this attitude, that a null pointer is a normal pointer, that an
empty thread handle is a normal type of thread handle, is adaptive to
programming C++. But it will inevitably exist in a programmer's blind
spot, as null pointers always have. The "not null" invariant is often
expressed implicitly. Many uses of `std::unique_ptr` are relying on
them never being null, and simply leave this up to the programmer
to ensure.

Herb Sutter himself discusses this:

> Since the problem is that we are not expressing the “not null”
> invariant, we should express that by construction — one way is
> to make the pointer member a `gsl::not_null<>` (see for example the
> Microsoft GSL implementation) which is copyable but not movable or
> default-constructible.

In a programming language with destructive moves, it would be possible
to have a smart pointer that was both "non-null" and movable. If we
need both movability and the ability to express this invariant in the
type system, well, C++ cannot help us.

> But what about a third option, that the class intends (and documents) that
> you just shouldn’t call operator< on a moved-from object… that’s a
> hard-to-use class, but that doesn’t necessarily make it a buggy class,
> does it?
> 
> Yes, in my view it does make it a buggy class that shouldn’t pass
> code review.

But in a sense, this is exactly what `std::unique_ptr` is. It has a
special state where you cannot call its most important operator, the
dereference operator. It only avoids being called buggy because it
expands this state so it can be arrived at by other means.

Again, everything Herb Sutter says is true in a strict sense. It is
memory-safe to use moved-from objects other than to destroy or
assign to them, even if the move operation makes no further guarantees.
It simply isn't safe in a broader sense, in that it will have surprising,
changeable behavior. It is true that the null pointer is a valid
value of `std::unique_ptr`, but smart pointers that implement move
are forced to have such a value.

And therefore, it should not be surprising that these questions come
up. The misconceptions that Herb Sutter is addressing are an unfortunate
consequence of the dissonance between the strict semantics of
the programming language, where his statements are true, and the
practical implications of how these features are used and are
intended to be used, where the situation is more complicated.

## Moves in Rust

So the natural follow-up question is, how does Rust handle move semantics?

First off, as mentioned before, Rust makes a special case for types
that do not need move semantics, where the value itself contains all
the information necessary to represent it, where no heap allocations
or resources are managed by the value, types like `i32`. These types
implement the special `Copy` trait, because for these types, copying
is cheap, and is the default way to pass to functions or to handle
assignments:

```
fn foo(bar: i32) {
    // Implementation
}

let var: i32 = 3;
foo(var); // copy
foo(var); // copy
foo(var); // copy
```

For types that are not `Copy`, such as `String`, the default function
call uses move semantics. In Rust, when a variable is moved from, that
variable's lifetime ends early. The move replaces the destructor call
at the end of the block, at compile time, which means it's a compile
time error to write the equivalent code for `String`:

```
fn foo(bar: String) {
    // Implementation
}

let var: String = "Hi".to_string();
foo(var); // Move
foo(var); // Compile-Time Error
foo(var); // Compile-Time Error
```

`Copy` is a trait, but more entwined with the compiler than most traits.
Unlike most traits, you can't implement it by hand, but only by deriving
from primitive types that implement copy. Types like `Box`, that manage
a heap allocation, do not implement copy, and therefore structs that
contain `Box` also cannot.

This is already an advantage to Rust.  C++ pretends that all types are the
same, even though they require different usage patterns in practice. You
can pass a `std::string` by copy just like an `int`. Even if you have a
vector of vectors of strings, you can pass by copy and that's usually the
default way to pass it -- moves in many cases require explicit opt-in. For
`int` it's a reasonable default, but for collections types it isn't,
and in Rust the programming language is designed accordingly.

If you want a deep copy, you can always explicitly ask for it with
`.clone()`:

```
fn foo(bar: String) {
    // Implementation
}

let var: String = "Hi".to_string();
foo(var.clone()); // Copy
foo(var.clone()); // Copy
foo(var);         // Move
```

What this actually does is create a clone, or a deep copy, and then
move the clone, as `foo` takes its parameter by move, the default for
non-`Copy` types.

What does a move in Rust actually entail? C++ implements moves
with custom-written move constructors, which collections and other
resource-managing types have to implement in addition to implementing
copying (though automatic implementation is available if building out of
other movable types). Rust requires implementations for clone, but for
all moves, the implementation is the same: copy the memory in the value
itself, and don't call the destructor on the original value. And in Rust,
all types are movable with this exact implementation -- non-movable types
don't exist (though non-movable values do). The bytes encode information
-- such as a pointer -- about the resource that the value is managing,
and they must accomplish that in the new location just as well as they
did in the old location.

C++ can't do that, because in C++, the implementation of move has to
mark the moved-from value as no longer containing the resource. How this
marking works depends on the details of the type.

But even if C++ implemented destructive moves, some sort of "move
constructor" or custom move implementation would still be required.
C++, unlike Rust, does not require that the bytes contained in an
object mean the same thing in any arbitrary location. The object could
contain a reference to itself, or to part of itself, that would be
invalidated by moving it. Or, there could be a data structure somewhere
with a reference to it, that would need to be updated. C++ would have
to give types an opportunity to address such things.

Safe Rust forbids these things. The lifetime of a value takes moves into
account; you can't move from a value unless there are no references
to it. And in safe Rust, there is no way for the user to create a
self-referential value (though the compiler can in its implementation
of `async` -- but only if the value is already "pinned," which we will
discuss in a moment).

But even in unsafe Rust, such things violate the principle of move.
Moving is always safe, and unsafe Rust is always responsible for keeping
safe code safe. As a result, Rust has a mechanism called "pinning" that
indicates, in the type system, that a particular value will never move
again, which can be used to implement self-referential values and which
is used in `async`. The details are beyond the scope of this blog post,
but it does mean that Rust can avoid the issue of move semantics for
non-movable values without ruining the simplicity of its move semantics.

For these rare circumstances, the features of moving can be accomplished
by indirection, and using a `Box` that points to a pinned value on
the heap. And there is nothing stopping such types from implementing a
custom function which effectively implements a custom move by consuming
the pinned value, and outputs a new value, which can then be pinned
in a different location. There is no need to muddy the built-in move
operation with such semantics.

## Practical Implications for C++ Programmers

So, obviously, in light of the themes of this book, I recommend using Rust
over C++. For Rust users, I hope this clarifies why the move semantics
are the way they are, and why the `Copy` trait exists and is so important.

But of course, not everyone has the choice of using Rust. There are a
lot of large, mature C++ codebases that are well-tested and not going
away anytime soon, and many programmers working on those codebases.
For these programmers, here is some advice for the footgun
that is C++ move semantics, both based on what we've discussed, and
a few gotchas that were out of the scope of this post:

 * Learn the difference between rvalue, lvalue, and forwarding references.
   Learn the rules for how passing by value works in modern C++. These
   topics are out of the scope of this blog post, but they are core parts
   of C++ move semantics and especially how overloading is handled in
   situations where moves are possible. Scott Meyers's *Effective Modern
   C++* is an excellent resource.
 * Move constructors and assignment operators should always be `noexcept`.
   Otherwise, `std::vector` and many other library utilities will simply
   ignore them. There is no warning for this.
 * The only sane things to do with most moved-from objects are to
   immediately destroy it or reset its value. Comment about this in your
   code! If the class specifically defines that moved-from values are
   empty or null, note that in a comment too, so that programmers don't
   get the impression that there are any guarantees about moved-from
   values in general.

## Conclusion

Move semantics are essential to the performance of modern C++. Without
them, much of its standard library would become much more difficult
to use. However, the specific design of moves in C++:

 * is misaligned with the purpose of moving
 * fails to eliminate all run-time cost
 * surprises programmers, and
 * forces designers of types to implement an "empty-yet-valid" state

Why, then, does C++ use such a definition? Well, C++ was not originally
designed with move semantics in mind. Proposals to add destructive move
do not interact well with the existing language semantics.
One [interesting blog post](https://www.foonathan.net/2017/09/destructive-move/)
that I found even says, when following through on the consequences
of adding destructive move semantics:

> ...  if you try to statically detect such situations, you end up with Rust.

C++ has so many unsafe features and so many existing mechanisms, that
this was deemed the most reasonable way to add move semantics to C++, harmful
as it is.

And perhaps this decision was unnecessary. Perhaps there was a way --
perhaps there still is a way -- to add destructive moves to C++. But
for right now, non-destructive moves are the ones the maintainers of
C++ have decided on. And even if destructive moves were added, it's unlikely
that they'd be as clean as the Rust version, and the existing non-destructive
moves would still have to be supported for backwards-compatibility sake.

In any case, Rust has taken this opportunity to learn from existing
programming languages, and to solve the same problems in a cleaner,
more principled way. And so, for the move semantics as well as for the
syntax, I recommend Rust over C++.

And to be clear, this still has very little to do with the safety features
of Rust. A more C++-style language with no `unsafe` keyword and no safety
guarantees could have still gone the Rust way, or something similar to
it. Rust is not just a safer alternative to C++, but, as I continue to
argue, unsafe Rust is a better unsafe language than C++.
