+++
title = "Can you have too many programming language features?"
tag = "multiparadigm"
date = "2022-05-11"
author = "Jimmy Hartzell"
cover = ""
tags = ["rust", "programming", "computers", "C++", "Rust vs C++"]
description = ""
showFullContent = false
draft = false
+++

> There's more than one way to do it.
> - [Perl motto](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it)

> There should be one-- and preferably only one --obvious way to do it.
> - [The Zen of Python](https://peps.python.org/pep-0020/) (inconsistent
> formatting is part of the quote)

When it comes to statically-typed systems programming languages, C++ is
the Perl, and Rust is the Python. In this post, the next installment of
my [Rust vs C++](/tags/rust-vs-c++/) series, I will attempt to explain
why C++'s feature-set is problematic, and explain how Rust does better.

C++ fans brag that it is "multi-paradigm," and it is. You can do
everything the C way, as C++ has a subset almost exactly identical to
C. You can use pointers and virtual functions and inheritance to create
all the classic OOP design patterns, as C++ is object-oriented. Or you
can use templates, and "static" or "compile-time" polymorphism, and
program that way.

At first glance, this all seems like an unmitigated good thing, because
it gives you, as a programmer, flexibility. You can express your code
in OOP style if that matches the problem at hand, or even if you
just like it better. If you need the performance of
templates, you can use them, and if you don't (or you just find them
confusing), you can use run-time polymorphism instead. Or you
can just ignore all of it, and program in almost-plain C.
Flexibility is good: you can use the features you want, and not
use the features you don't want. Even if a feature is downright
harmful, in your opinion, that's easy enough to handle: Just don't
use it.

And this is all very well and good if you're programming a quick project
completely by yourself. But most code comes in long-lived projects, with
developers jumping in and out of the project all the time.  In such an
environment, as Robert C. Martin puts it:

> “Indeed, the ratio of time spent reading versus writing is well over
> 10 to 1. We are constantly reading old code as part of the effort to
> write new code. ...[Therefore,] making it easy to read makes it easier
> to write.”
> - Robert C. Martin, *Clean Code: A Handbook of Agile Software Craftsmanship*

(Sidenote: I will admit to knowing almost nothing about Robert C. Martin
besides this famous quote. I have no idea if the rest of his work is
as insightful as this quote, or not, and will probably try to find out
someday, but not today.)

Since programmers in general spend much more time reading code than
writing it, we very rarely actually get to reap the benefits of this
flexibility as writers. Much more often, as maintainers and readers,
we have to be flexible ourselves. We have to be ready to read code
in any style, in any paradigm, using any feature-set.

This is why Perl was commonly panned as a write-only programming language:
It had so many features that you could not be up to speed on all of them.
Each programmer at each point in time had a set that they used, but
no one could ever get proficient at working in the entire available
feature-set.

In Perl, the features were syntactic, so the programs would be unreadable
at a line-by-line level. In C++, the different features have more to do
with code organization, which is harder to make fun of, but I think
more insidious, because a lot of the features are structural.

Let me explain what I mean. Let's say you're a C++ maintenance programmer,
and you don't like exceptions. You're trying to maintain a program
that uses exceptions heavily, and add new features to it. Not only do
you have to be able to understand exceptions to read the code, you have
to write your own code so that it handles the exceptions where appropriate,
and so that it's exception-safe. Even if you're just using a third-party
library that throws exceptions, you have to understand exceptions to use
that library.

The entire programming language, with all the features, is part of
the necessary skill-set to program proficiently. Even if it is just
you writing your own project, you still will have to use libraries,
and the features involved with it.  And even if it is just you, if the
project lives long enough, you will have to deal with your previous
decisions. Migrating from dynamic to static polymorphism in C++ is no
joke. Ask me how I know.

And of course, every feature has to be considered when writing advice.
Every best practices manual for C++ is written for C++, not a subset
of C++ features. The more things it's possible for a future programmer
or future library writer to do, the more things you have to worry
about coding defensively, and the more things that have to be included
in best practices manuals, and finally the more things that a proficient
programmer has to stuff into their brain.

# Specific C++ Examples, Rust Responses

But I'm also not trying to advocate for absolute minimalism. There may
be a cost to every feature, and it may be that no feature is optional,
but that doesn't mean that we should have the bare minimum number of
features. Sometimes the cognitive and maintenance cost of a seemingly
extraneous feature is still worth it. Especially in a systems programming
context, different problems often do actually call for different
implementation strategies with different programming language features
to express them.

C++, however, does this poorly. I'm not even sure I'd claim that C++
has too many features; it's more that the features are not consistent.
They clash with each other. Different feature-sets make assumptions
that are violated by other feature-sets. C++ is not designed with
the costs of extra features in mind, and as such, the features
cost more than they have to.

Let's discuss a few specific ways in which C++'s features cause
problems and clash with each other. For each of these categories,
I then discuss how Rust handles the same topic, with a more
coherently-designed feature set.

## Value and Reference Semantics: Slicing

Slicing is a famous beginner error in C++, where the semantics of
combining certain features are surprising with a tendency to break
invariants, but no diagnostics are issued as the code is completely valid.
Perhaps unsurprisingly, this code comes from a mismatch between
two C++ features designed for two C++ programming styles.

Specifically, C++ has a distinction between value and reference semantics.

With value semantics, you can use operator overloading to make
your custom class look and act like a built-in type, supporting
operators like `+` and `+=`:

```
class complex {
    double re;
    double im;
public:
    complex &operator=(const complex &other) {
        re = other.re;
        im = other.im;
        return *this;
    }

    complex &operator+=(const complex &other) {
        re += other.re;
        im += other.im;
        return *this;
    }

    complex operator+(const complex &other) {
        complex res = *this;
        res += other;
        return res;
    }
};

// Sample usage
Complex a, b;
a = b;
Complex c = a + b;
```

With reference semantics, you can use polymorphism to create
many different types of object that support the same interface.
You can then access these objects through pointers or references
to the base class.

```
class Complex {
protected:
    double re;
    double im;
public:
    virtual double getMagnitude() {
        return sqrt(re * re + im * im);
    }
}

class Quaternion : public Complex {
protected:
    double j;
    double k;
public:
    double getMagnitude() override {
        return sqrt(re * re + im * im + j * j + k * k);
    }
}

// Sample usage
void print_magnitude(Complex &c) {
    std::cout << c.getMagnitude() << std::endl;
}

Quaternion a;
Complex b;
print_magnitude(a);
print_magnitude(b);
```

However, these two programming techniques cannot be combined.
You cannot assign a `Complex` object a `Quaternion` value:

```
Quaternion a;
Complex b;
b = a; // Non-sensical
```

Why? Well, unlike in Java, `Complex b` actually allocates the space for a
`Complex` number as a local variable on the stack. This means that it
only has room for the two fields, `re` and `im`.

But, unfortunately, if you include all the methods from both examples,
that code will compile, and run, and `b` will have only `re` and `im`
from `a`. This is almost certainly not what you want, and may
in fact break invariants (e.g. for this you might only be dealing
with values of magnitude 1, and this truncation would lower the
magnitude).

This comes from two alternative paradigms for objects: by value as
"primitive replacement," where `Complex` can be used like an `int`, and
by reference with traditional OOP inheritance and polymorphism. These
paradigms don't use different keywords, however. They can just all
be used in the same objects, causing this trouble.

Advice on how to prevent this includes rules like "give
all parent classes at least one pure virtual function,"
which would make `Complex b` as a by-value declaration
illegal. But if this rule is recommended in leading [books on
C++](https://www.amazon.com/Effective-Specific-Improve-Programs-Designs/dp/0321334876),
why isn't it enforced in the programming language itself?

### How Rust Handles This

C++'s slicing is caused by a conflict between two features, inheritance
and assignment. Rust handles both of those features
differently, so that they do not conflict.

So the most important difference here between Rust and C++ is that
Rust does not have implementation inheritance like C++ does. For
two given C++ concrete types, one of which surrounds the other,
there are two possible relationships between them: is-a, and has-a.
Rust only does has-a for concrete types.

C++ inheritance is a feature with many use cases, such as sharing
implementation, implementing policy, and implementing interfaces (what Rust
calls traits). Rust, rather than having one big broad feature,
instead implements individual features as appropriate. The closest
feature Rust has to inheritance is traits (including subtraits
and supertraits), but because traits are not concrete types, they
cannot be assigned, and so this issue is avoided.

But also in assignments, Rust implements a simpler feature that is
easier to reason about: Rust does not allow custom assignment operators.
Rust instead builds assignment out of two operations: move, and drop
(cf. C++ destructors). If drop is implemented correctly, so will
assignment. If you want to copy instead of move, you have to explicitly
call a `clone()` method. And moves are [not
customizable either](/posts/cpp-move/).

So, although Rust has some of the best parts of inheritance in traits,
and still allows assignment of custom types (but through customizing drop,
not assignment *per se*), it avoids this particular clash through
restricting the scope of those features.

## Exceptions and "Exception Safety"

It would be impossible to write a post criticizing C++ for
its problematic feature-clashes and not talk some
about exceptions.

Exceptions are another famous example of a C++ feature you simply can't
"not use."

Exceptions are viral by nature. If you call a function that might
throw and don't catch all the exceptions that it throws -- which might
be impossible to determine -- then your function can throw as well.

And lots of functions can cause exceptions. Allocating memory indicates
failure via exception. Exceptions are the only way for constructors to
signal failure, and C++ idiom encourages constructors to be written in
such a way that success guarantees that the object is usable. The
programming language was clearly not designed to be used without
exceptions.

But exceptions are gnarly and confusing. I already know people will
comment to this post and say that if you write and structure C++ code
correctly, it will be exception-safe. And that's almost trivially true,
since exception safety is part of correct C++ practice, but it's not
easy and it doesn't follow naturally from easy-to-learn principles,
which is why Herb Sutter, a huge name in C++, felt the need to write
[two](https://www.amazon.com/Exceptional-Engineering-Programming-Problems-Solutions/dp/0201615622)
[books](https://www.amazon.com/More-Exceptional-Engineering-Programming-Solutions/dp/020170434X)
about it. Of course, in practice, people just write exception-unsafe code,
all the time.

Every time you call a function -- which can happen in C++ simply by
declaring a variable, or even by ending a scope (though destructors are
supposed to avoid throwing exceptions) -- you have to worry about whether
that function throws an exception, and if you're leaving things within
that function in an inconsistent state. In C++, it is very common to
implement your own unsafe data structures, and exceptions are designed
to be sometimes recoverable from. Lack of exception safety can mean
memory corruption or even exploitable security vulnerabilities.

No wonder a lot of codebases ban exceptions. Unfortunately, many shops
simply avoid using exceptions instead of banning them, leaving exceptions
possible. Also, code from "exception-free" codebases can then later
be mixed back in with regular C++, re-opening it to exception-safety
concerns.

The fact that exceptions are so controversial can lead to
confusion as well. Consider this function signature:

```
std::unique_ptr<DatabaseConnection> connect(const ConnectionParameters&);
```

How does this function indicate failure? From the signature, there are
two possibilities: It could either return `nullptr`, or it could throw
an exception. Hopefully the documentation would clarify -- but again,
oftentimes, people don't write documentation, especially for internal
APIs.

### How Rust Handles This

#### Normal Error Handling in Rust

For recoverable errors, Rust encodes them in the type. Rust's
equivalent to `std::unique_ptr` -- `Box` -- is not nullable. If we
want to return one, but possibly also signal an error, we use
a sum type or what Rust calls an `enum`, and what C++ would call a
"tagged union" and make you implement by hand:

```
fn connect(param: &ConnectionParameters) ->
    Result<Box<DatabaseConnection>, OurError>;
```

This means that it can return either a database connection or an error.
This is the convention to return any error condition that is recoverable,
which is half of what exceptions are used for in C++. Since `Box` is not
nullable, you have to say more than just `Box` to signal that it's
possible to return an error, proving that you really mean it.

For unrecoverable exceptions -- for situations like logic and programming
errors that the program has caught -- Rust has panics, which work
much more like C++ exceptions in practice.

#### Panic Safety

Rust afficionados will know that Rust has not escaped exception
safety, having instead an analogous notion of "panic safety." How,
then, can I criticize C++ so boldly?

There are two notable differences between C++ exceptions and Rust panics.
The first is that Rust panics are used primarily for unrecoverable
errors, such as errors that indicate that a programmer's assumptions
were violated due to a bug or a circumstance that the program cannot
recover from or a misunderstanding from the programmer. These generally
are unrecoverable, and Rust by convention uses a different mechanism,
`Result`s, for recoverable errors. So most Rust code doesn't have to
care about maintaining invariants in the face of panics, because most
Rust code can presume that if it panics, that's the end. This is better
scoping for the panic feature, as opposed to exceptions.

But the fact remains that panics can be recovered from, and do still
do stack unwinding and destructor/drop calls, and safety issues can
still exist. Panics in Rust can cause memory corruption -- in unsafe
code. And that's where panic safety really still matters: in unsafe
code only. By cordoning off the implementations of sophisticated
data structures that require `unsafe`, Rust also cordons off who
has to worry about panic safety.

In C++, every function that calls another function has to be
written in an exception-safe way. In Rust, it's really only unsafe
code that has to worry about it. This, in my mind, is a huge win,
and it comes from both better scoping of panics, and better management
of the situations where panics can break things.

## C-style vs C++-style Pointers and Arrays

There is a subset of C++ that is almost identical to C, and C++
must maintain compatibility with this subset for tradition's sake.
It also must maintain compatibility with previous versions of itself.
Between the C and the C++, the concepts contained in C++20 stretch from
1972 to 2020, almost 50 years of active change in programming language
technology. This leads to features being duplicated, but differently,
and in ways that unfortunately clash with each other.

For example: How do you express indirection? How do you alias a
value? There are three different ways to do it, and rather than
breaking down by use case, the biggest difference between them
is era:

* Pointers, from the original C
* References, a newer innovation that attempts to solve some of
the issues with pointers
* Smart pointers, an even newer innovation that attempts to cover
some of the remaining use cases. For pointers into arrays, iterators
also cover a lot of the same territory as smart pointers, and can
be lumped together for this conversation.

These overlap a lot, and there is no single principle that will
tell you when to use which. You can invent some rules, and come
up with some principled reasonings for them, but your colleagues
won't necessary listen, and external libraries and other codebases
you have to interact with certainly won't, not even the standard
library, not even the programming language itself. Fundamentally,
the difference is era.

Nullability? Part of original pointers. Later, we learned it was harmful
and got rid of it in references, but due to issues with how C++ does
[move semantics](/posts/cpp-move/) it comes back with a vengeance for
smart pointers. (Of course, you still *can* make a null reference,
it's just undefined behavior. Ah well.)

Pointers and references have special syntax, whereas smart pointers,
because they came from a later era, use the more standard `ptr_type<T>`
syntax. Pointers and smart pointers can be used to manage ownership,
and references should not be.

How should out parameters be expressed? It's easy to say they should
be expressed with references, because otherwise they're nullable,
and you have to worry about whether to check for nulls or not. On
the other hand, expressing out parameters with references mean you
can't tell at the caller whether it's an out parameter, only
at the callee:

```
int foo_ptr(int in, int *out);
int foo_ref(int in, int &out);
int out;
foo_ptr(3, &out);
foo_ref(4, out); // Surprise, this changes `out`! Can't tell, though!

foo_ptr(5, nullptr); // Does this crash? Does this work? Who knows!
// Read the docs, I guess *shrug* hope there are docs
```

References should be used, in my practice and in the practice of many
people I respect, in every case where the reference is not owning,
will not be used for arithmetic, and is not optional. Of course, `this`
meets all of those requirements, but is a pointer, not a reference
(but a special pointer, where being null is undefined behavior, like a
reference), simply because references were invented after `this` was,
and for no stronger reason.

Similarly, my practice dictated that `std::unique_ptr` should be
used for owning pointers.  It's nullable, but at least it auto-frees,
and so you should use it everywhere you're conveying ownership. And
then, `Foo *` can be used when you want an optional non-owning
reference.  But old APIs and APIs from C exist all over the place
that will use `Foo *` invariably, and some will use `Foo *` for
out parameters because of the callee readability issue, or because
of [concerns](https://www.youtube.com/watch?v=rHIkrotSwcc) about
`std::unique_ptr`, or simply out of old habit, meaning you can't count
on this convention actually being upheld, not at all.

And of course, converting between these different representations is
sometimes as easy as `&` or `*`, and sometimes as difficult as having
`&` and `*` compile and seem to work but result in memory corruption,
and everywhere in between.

Similarly, `T foo[N]` and `std::array<T, N> foo` are different ways of
writing the same basic thing. It gets weird when `N = 0`, of course;
this is only supported by `std::array`. And (on compilers that support
it at all), having `N` be dynamic on the stack is only supported by `T
foo[N]`. And of course, `new T[N]` returns a raw pointer to `T` whereas
`new std::array<T, N>` returns a pointer to a `std::array`, which makes
much more sense.

So, basically, `T foo[N]` should be completely deprecated, but keeps on
being used even by new C++ programmers because it looks like it
should be the normal way to write an array, and because it looks like
the arrays from C. But they're completely different types -- one isn't
syntactic sugar for the other.

This gets unwieldy, because the ways with the syntactic sugar (like
`new` and `T*` instead of `std::make_unique` and `std::unique_ptr`)
are the old, more C-style ways, the ways that yield more memory leaks
(you have to explicitly free or delete a `T*`) and memory corruption (`T
foo[]` doesn't even have a safe indexing operation, or proper iterators).

And of course, even if you use the more modern formulations to save
on cognitive load because they're more consistent with the rest of
the programming language (where `std::unique_ptr` does RAII unlike
traditional pointers (spelled `*`) and `std::array` implements the
expected collections methods unlike traditional arrays (`[]`)), you still
have to understand the traditional pointers and arrays completely to call
yourself a C++ programmer. Due to C interop, people not changing their
ways, and old resources, lots of new code is still written with them,
and there are still situations where they're unavoidable, like pointer
arithmetic or `this`.

Besides, even if you do correctly discern that a `T*` must be freed,
how do you free it -- `free`, `delete`, or `delete[]`? Choose wisely,
because the consequences of mixing `malloc` and `delete` can go beyond
whether destructors are called, and lead to undefined behavior and
general memory corruption. The documentation (or lack thereof), however,
might just assume you know which one to call.

### How Rust Handles This

Rust also has references and various types of smart pointers and
iterators. It also has raw pointers, from which smart pointers
can be implemented. So in terms of the range of features, it's
actually the same as C++. What's the difference then?

Well, in Rust, the difference is that they don't overlap in the
same way. Each feature has its own purpose, unlike in C++ where it's
anyone's bet whether references or pointers are used for aliasing or
pass-by-reference, or whether raw pointers or smart pointers are used
to express ownership. Nullability is mostly a separate concern from
day-to-day use of Rust's types, and so it is implemented orthogonally
through `Option` and `Result`, rather than being available in some types
but not in others haphazardly.

References are for everyday aliasing and
pass-by-reference. They are not nullable. They represent
the primitive concept of aliasing and pass-by-reference, and
they are the only feature that does so. Unlike C++ references,
you must use the `&` operator to create a Rust reference, making
them explicit on the caller.

Smart pointers are, for the most part, also not nullable, possibly
partially because Rust has [destructive moves](/posts/cpp-move/).
They represent ownership semantics -- whether "unique" ownership (`Box`),
shared ownership (`Rc` or `Arc`), or locking (`Mutex` or `RefCell`).

Raw pointers in Rust are very special -- they are for implementing
smart pointers or other low-level data structures. They are for
situations where the structure of memory and the concept of a pointer
is actually key to the situation. They are kept within these narrow
bounds, and outside of everyday application programming, by having
most of their features considered `unsafe`.

If only that could be done for raw pointers in C++! But there is too
much momentum behind the C++ raw pointer.

## Dynamic vs Static Polymorphism

This is the most intense one, and could be a blog post all
on its own -- and probably I'll write it one day.

In response to comments, I'm going to add a caveat here even though
I address it later: In this section, I'm discussing the status of C++
pre-concepts, from C++17 and earlier, because that is the form of C++
that most people are still using, and that the vast majority of code is
still written in. It is too early to tell how much concepts will help,
but because they are an optional feature, I'm not at all optimistic.

We have two forms of polymorphism in C++, two very different
systems. One is a Turing-complete macro system that comprises
overloads, templates, and template metaprogramming. The other
is an object-oriented style system of polymorphism through
inheritance.

They were designed with different purposes in mind, and considering
their original purpose, it's clear to see why they must have
different implementations.

Templates were designed for collections and algorithms, for being
able to write a vector or linked list that could contain any arbitrary
type, without resorting to a C-style `void*` that would require both
indirection and type erasure. The lack of indirection is the point --
at least it was for C++ -- and so as a consequence templates had to be
carried out statically.

Dynamic polymorphism, on the other hand, was designed for OOP design
patterns. As such, in line with OOP principles, it supports heterogeneous
containers, especially necessary to support OOP's core use case of GUI
programming.

But in spite of this deep contrast between static and dynamic, they
overlap in use case. For example, Smalltalk, Objective-C, and
Java (pre Java 5) all show us that you can use dynamic polymorphism
to implement generic containers. If C++ had been less performance-centric,
and could tolerate the indirection, it could have used a similar strategy,
the (old school) Java approach to generic containers without generics
or templates:

1. Make all classes inherit from a universal base class, `Object`.
This way, `Object *` (just `Object` in Java) can refer to any object.
Make sure, for C++, that this has a virtual destructor, so you can
delete any object through its `Object*` handle.

2. Write "boxed versions" of all primitive types, classes that
extend `Object` to correspond to `int` and `double`, etc.

3. Write all collection classes (`std::vector`, `std::list`) in terms
of `Object *`, writing `Object *` instead of `T`.

4. Use RTTI and `dynamic_cast` (or in Java terms, casts) to allow
the user to get whatever object type they want out of them.

Voilà! You can now store anything in your collections without need
for generics or templates, using `dynamic_cast`, an obscure feature of
the OOP-style dynamic polymorphism that C++ has. And this system is in
fact still the basis of Java generics, and so we can project that C++
would have used something similar if performance weren't a concern and
indirections and RTTI were acceptable.

So that shows the overlap between templates and runtime polymorphism
in a theoretical sense, but do these very differently implemented
features in fact overlap in practice?

I've seen skepticism. I once interviewed
people for a job, and I asked candidates to explain to me the
similarities and differences between dynamic polymorphism and
templates. The candidate said there was no overlap; templates
were for generic programming (e.g. collections and algorithms and STL),
and dynamic polymorphism was for object-oriented programming.

But they do overlap in practice. I know, because I spent a lot of
time transitioning object-oriented dynamic code into static form,
and teaching the static equivalents to dynamic polymorphism patterns.
It wasn't easy, because even though the overlap is huge, the
semantics are vastly different.

Let me give an example. Let's start with one of my favorite patterns:
the policy pattern.  Let's imagine we have a function that sends messages
in a way that can fail, and let's also imagine that we have a policy
that indicates how we should delay and retry sending this message. I'll
start out writing it the object-oriented way, something like this:

```
struct RetryPolicy {
    virtual bool should_retry(mesg_send_err_t error_code) = 0;
    virtual uint32_t delay_microseconds() = 0;
};

mesg_send_err_t retry_send_message(Message &mesg, RetryPolicy &policy) {
    while (true) {
        auto err = send_message_once(mesg);
        if (err == mesg_send_err_t::SUCCESS) {
            return mesg_send_err_t::SUCCESS;
        } else if (!policy.should_retry(err)) {
            return err;
        } else {
            usleep(policy.delay_microseconds());
        }
    }
}
```

The policy can then do things like "retry 5 times, waiting 0.01 seconds
between each retry" or "exponential back-off, so that each retry waits
twice as long as the previous." It can also deem certain errors as
fatal, but others as worth sleeping and retrying for. Here's
an example of using this interface:

```
struct WaitOneSecondAndTryFiveTimes : RetryPolicy {
    int retry_count = 0;

    bool should_retry(mesg_send_err_t error_code) override {
        if (error_code == mesg_send_err_t::MALFORMED_MESG) {
            return false;
        }

        retry_count++;
        if (retry_count == 5) {
            return false; // do not retry
        }

        return true; // do retry
    }

    uint32_t delay_microseconds() override {
        return 1000000;
    }
};

WaitOneSecondAndTryFiveTimes policy;
auto err = retry_send_message(mesg, policy);
```

Now, it turns out we can do this exact same pattern with static polymorphsim.
The callee code now looks like this:

```
template <typename T>
mesg_send_err_t retry_send_message(Message &mesg, T policy) {
    while (true) {
        auto err = send_message_once(mesg);
        if (err == mesg_send_err_t::SUCCESS) {
            return mesg_send_err_t::SUCCESS;
        } else if (!policy.should_retry(err)) {
            return err;
        } else {
            usleep(policy.delay_microseconds());
        }
    }
}
```

This is no longer a function. It is a function template, which is a type
of macro. Its implementation must now move from the `.cpp` file to the
`.h` or `.hpp` file, for reasons that only make sense if you think about
how the programming language is implemented.

No longer is the policy interface spelled out separately. The only
thing the function signature says about the type of `policy` is
that it is `T` -- which can be any type. Only in the implementation,
in the body, do we see that `should_retry()` and `delay_microseconds()`
must be implemented on it. This is an implicit interface, defined by
usage, very similar to Python and Ruby's [duck
typing](https://en.wikipedia.org/wiki/Duck_typing). More importantly,
it is completely unrelated to the OOP-style explicit interface
using inheritance and virtual functions.

The errors are completely different, because the rules are completely
different.

```
test.cpp:57:34: error: variable type 'WaitOneSecondAndTryFiveTimes' is an abstract class
    WaitOneSecondAndTryFiveTimes policy;
                                 ^
test.cpp:22:22: note: unimplemented pure virtual method 'delay_microseconds' in 'WaitOneSecondAndTryFiveTimes'
    virtual uint32_t delay_microseconds() = 0;
                     ^
1 error generated.
```

With the template version, you get:

```
test.cpp:34:27: error: no member named 'delay_microseconds' in 'WaitOneSecondAndTryFiveTimes'
            usleep(policy.delay_microseconds());
                   ~~~~~~ ^
test.cpp:59:16: note: in instantiation of function template specialization 'retry_send_message<WaitOneSecondAndTryFiveTimes>' requested here
    auto err = retry_send_message(mesg, policy);
               ^
1 error generated.
```

The ad-hoc nature of template requirements should not be underestimated.
It means that objects that are designed to work with a whole library might
only work with the exact combinations of functions they've been used
with so far. It means that documentation, if it wants to be rigorous,
must do the work of defining the protocols itself of every argument
taken by every function. It means that it's not clear when you're
putting new requirements on arguments to a function, as there is no
warning and no clear red-line step to tell you that you're breaking
backwards-compatibility.

Concepts have been introduced recently to clean it up, and I think
it's still early to tell how good a job they will do. But even if
they do a great job, the polymorphism will still look very
different from the OOP style, and the old template-based code
will still exist, and so in the meantime the C++ programming language
has simply continued to grow.

And the concrete consequences: It's a perfectly reasonable decision to
use OOP-style polymorphism, for the benefits of cleaner structure and
explicit specification of the interface, even when the dynamic nature of
the polymorphism -- and its concomittant performance costs -- is never
actually called for. Meanwhile, using static polymorphism to accomplish
the same goals is simply harder, requiring much more skill and training.

### How Rust Handles This

Like C++, Rust has both static (compile-time) polymorphism, and dynamic
(run-time) polymorphism. Unlike C++, Rust integrates them closely into
a single feature, inspired by Haskell's typeclasses: traits.

Let's use the same example again, but in Rust, using static polymorphism,
which is the more Rusty way to write such a function:

```
trait RetryPolicy {
    // Return None to not retry at all
    // Takes `self` as `&mut` to implement counting and back-off
    fn retry_microseconds(&mut self, error: MesgSendError) -> Option<Duration>;
}

fn retry_send_message(mesg: &Message, mut policy: impl RetryPolicy) -> Result<(), MesgSendError> {
    loop {
        match send_message_once(mesg) {
            Ok(()) => {
                return Ok(());
            }
            Err(err) => match policy.retry_microseconds(err) {
                None => {
                    return Err(err);
                }
                Some(delay) => sleep(delay),
            },
        }
    }
}
```

I changed the example a little to showcase some other differences with
Rust. Instead of querying two functions, for example, to know whether
to try again and how long to delay, I feel in Rust it is more natural
to use sum types (and in particular `Option`) to fold them into a single
function. Similarly, rather than a `u32` count of microseconds,
`std::thread::sleep` takes a `Duration`, and so I felt the policy trait
should reflect that as well. Also, last but not least, in Rust it is
not necessary to consider `SUCCESS` to be one of the error options,
and so the types are more well-honed to the situation.

Notice, however, that this is the more performant static version,
and it has an explicit in-code specification of what the interface
is for the policy. However, the policy code and the generic code
are fully integrated just like in the C++ templated version,
through a process known as monomorphization. Fundamentally,
monomorphization exhibits the same behavior to C++ template
instantiation, but in a more principled, constrained fashion.

Here is the example of the usage of such a polymorphic function:

```
struct WaitOneSecondAndTryFiveTimes {
    retry_count: u32,
}

impl WaitOneSecondAndTryFiveTimes {
    fn new() -> Self {
        Self {
            retry_count: 0,
        }
    }
}

impl RetryPolicy for WaitOneSecondAndTryFiveTimes {
    fn retry_microseconds(&mut self, err: MesgSendError) -> Option<Duration> {
        if err == MesgSendError::MalformedMessage {
            return None;
        }

        self.retry_count += 1;
        if self.retry_count == 5 {
            return None;
        }

        Some(Duration::from_secs(1))
    }
}

let policy = WaitOneSecondAndTryFiveTimes::new();
let res = retry_send_message(&mesg, policy);
```

If we wanted to use dynamic polymorphism for some reason -- for example,
if we wanted to look the policy up in some sort of map based on a
user-supplied keyword, or load the policy from a dynamic library -- we
could, easily.

Unlike in C++, barely anything has to change. In fact,
only three lines have to change.

The function signature has to change, to indicate that it's using
dynamic polymorphism now. Dynamic polymorphism, due to its nature,
can only be done through indirection, so we have to add that (though
it does not affect the function body):

```
fn retry_send_message(mesg: &Message, policy: &mut dyn RetryPolicy) -> Result<(), MesgSendError> {
```

Similarly, the call site has to change, to implement the indirection:

```
let mut policy = WaitOneSecondAndTryFiveTimes::new();
let res = retry_send_message(&mesg, &mut policy);
```

And that's it! Now it's dynamic polymorphism!

When I first saw this is when I was truly convinced that Rust would
eclipse C++.

# Discussion and Conclusion

So, assuming I've convinced you that Rust has a better organized
feature-set than C++, we have to discuss what, in the big picture,
C++ has done wrong and Rust has done right.

The first and most obvious thing Rust did right was learn from the
mistakes of the past. Each new version of C++ has to be compatible with
previous versions to a great extent, including (in a lot of ways) C,
giving it a legacy back into the early 70's. Rust started maintaining
compatibility in 2015, and so it's only had 7 years or so of cruft,
but knew about all of C++'s later add-ons from the beginning.

And one of the things Rust learned from the experience of others is
how to mitigate this effect, so we can hope Rust retains its youthful
freshness for longer going forward. Rust has an edition system, so
that features actually can be deprecated and phased out, while still
maintaining compatibility.

But also, Rust's goal of separating safe and unsafe features -- and
keeping unsafe code encapsulated using the `unsafe` keyword -- forces
Rust's feature set to be more coherent. If two features clash in C++,
the standards committee can put the work of reconciling them on the
programmer, but in Rust, they often have to do the work to make
them make sense together, so they can continue to guarantee that
safe code can't cause undefined behavior.

Additionally, Rust believes in, and has, invariants. In C++, some
structs can be trivially copied. In Rust, all data types can
be trivially moved (`Pin` is almost but not quite an exception,
and the work that went into making `Pin` not break everything
shows how important the invariant is.) In Rust, a mutable reference
always means that a block of code has exclusive access to a value.
These invariants also structure other features, and force them
to work in concert.

Enough about Rust, though. I think there's deeper lessons to be learned
from the flaws in C++. Bjarne Stroustrup famously said, "Within C++,
there is a much smaller and cleaner language struggling to get out." I
think he regrets the quote, which he clarified is about the modern
semantics of C++, held down by the outdated syntax of C.  It's such a
compelling quote, though, because C++ is so messy and dirty, so we want
to believe in a small clean underlying core.

The truth, however, is that there isn't *one* smaller and cleaner
programming language struggling to get out. There's multiple. And
from the beginning, C++ was a glomming-together of multiple ideas: a
Simula-like OOP system glued awkwardly to C, without a unified motivating
vision. Operator overloading was intended to help integrate the two
parts, which it did but at the expense of creating its own entire
sub-paradigm. And then came templates, which tried to add generic
containers and algorithms but unexpectedly exploded into their own
programming paradigm.

So inside C++, struggling to get out, is of course C, the original
"portable assembly," which does its very simple job well. There's also
Java/C# in there, if we take the OOP features on their own.  For the
operator overloading and RAII and templates, the closest I can really
imagine is Rust, which I think if Bjarne was being fair, he would have
to admit is close to what he specified when he clarified his quote:
Rust does emphasize "programming styles, libraries and programming
environments that emphasized the cleaner and more effective practices
over archaic uses focused on the low-level aspects of C."

It's understandable that Bjarne glommed OOP, a foreign paradigm, onto
the otherwise-stable base of C.  OOP was extremely popular for a long
time, and has been awkwardly glommed on to many programming languages,
and I think Rust benefits from not even trying to be an OOP language in
the traditional 3-pillar sense (Rust doesn't have inheritance at all,
and has non-OOP concepts of encapsulation and polymorphism).

C++ wasn't even the only programming language to result from glomming
on object-oriented programming to C, and of the two big ones, it is
the more coherently integrated. Objective-C comes from a more dynamic
tradition of object-oriented programming, and it really feels like
two programming languages glued together, in this case C and Smalltalk.

I programmed Objective-C professionally for a while, and most of the
time, the C only came out when you had to do a little bit of pure logic
outside of the object-oriented framework. In the meantime, all of the OOP
code had to be written using the little whisps of syntax C left behind,
especially `@`, which basically served as a sigil to indicate that what
followed was to be interpreted in an Objective-C way ... which in an
Objective-C codebase basically should have been the default.

At the time, I dreamed of the leaner programming language inside
Objective-C (the non-C one), and even started designing a Smalltalk
dialect designed to interact with Apple's Cocoa APIs: CocoaTalk, I
think it was called. Ultimately, Apple unveiled their concept of it,
sharing many ideas with Rust, known as Swift. I felt very vindicated
the day Swift was announced.

Rust is C++'s chance to get a leaner, cleaner programming language.
The syntax is heavily influenced by C++, even as the semantics come
from a variety of sources. The design was done *de novo* with guiding
principles that allowed all of C++'s vast repertoire of features to
be reimagined but working in concert with each other.  As someone who
used to love programming in C++, which enabled programming techniques
no other programming language could, I continue to be deeply impressed
by the feature design of Rust.
