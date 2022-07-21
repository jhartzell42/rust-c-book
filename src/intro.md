# Introduction

## Goals of Rust and C++

Rust is a fresh attempt at accomplishing the most essential of C++'s goals.

In order to accomplish these essential goals well, Rust ends up ignoring
some of the other ones.  C++ has a lot of goals, and they distract from
each other.  The biggest goal that Rust drops from C++ is, of course:

* Be compatible with earlier versions of C++

This is a major burden on C++, as it incorporates a lot of older C++
goals by reference. Or, perhaps better said, it incorporates a lot
of features from times when those goals were more important.
These goals fit in with the older conceptualization of C++ as a
juxtaposition of C and OOP, when its original name of "C with Classes"
still rang true, and Rust also drops these:

* Be extremely C compatible, to the point of *almost* being a superset of C
* Be a traditional OOP programming language with all three pillars

With every one of these goals that Rust doesn't have to worry about,
there are ripple effects making the resultant programming language
tidier, so that Rust can focus on the *essential* goals it shares
with C++.

But what are those exactly?

### The Essential Goal: "Zero-Cost" Abstractions

Well, let's start with the goals of C, as Rust still tries to uphold the
goals of C (without the need to be tied to any literal source compatibility):

* Be a portable assembly language

This means that C is trying to be as *portable* as a high-level language, where
you can write a program once and run it on many platforms,
while offering an *assembly language* level of control over the computer,
as for as performance and exact control of memory.

And C does a great job of that! But missing from that goal is anything about
abstraction. Some abstraction over assembly language is necessary to ensure
portability, but C doesn't really go beyond that. It provides a standard
library, but again, that's for portability purposes. The C standard library
is not much more powerful than an assembly-language operating system API, just
more standardized. C is portable assembly, not abstracted assembly.

The essential goal of C++ over C -- and I would also argue, of Rust over C --
is therefore this:

* Create abstractions that cost no more than manual implementation in C
or assembly, whether in terms of speed or memory or any other resource.

For example, in C, like in many assembly environments, you have to call a
procedure to allocate memory, and another procedure to deallocate it.
In true portable assembly fashion, if you don't call the procedure,
the memory will just leak.

In a "normal" high-level programming language, in a high level
programming language where cost minimalization was not a goal, you
would layer an abstraction of garbage collection over this. You may
use mark-sweep with a sophisticated generational collector, like Java,
and accept that threads would sometimes have to be (hopefully briefly)
paused. You may use reference counting like Python or Swift and accept
that assigning references around would do little memory modifications.
In either case, you would add a high cost, and more importantly, the
computer would be doing operations under the hood that were different
and slower than what a skilled C programmer would have it do.

C++ has a genius solution to this: RAII. With RAII, the source code
looks simpler than in C, and is harder to get wrong. It's not quite
as straight-forward as in a garbage-collected language, but it has
many of the benefits of abstraction: You can't just forget to call
the destructor.

But! From the compiled binary, modulo such nit-picky accidents as symbol
names, you wouldn't be able to tell it was written in C++ instead of C!
The abstraction was resolved and disappeared at compile-time.

Similarly with templates, and therefore with the collections in the STL.
Similarly with many other C++ features.

And Rust has greatly benefitted from all of this innovation in C++.
C++ is one of the giants on whose shoulders Rust stands. Rust inherits
RAII from C++. Rust inherits templates as well, though it puts some
constraints on them and calls them "monomorphization."

And Rust needs these, because Rust is also striving to be the kind of
programming language where the compiled binary looks like something
someone could have written in C, but where the programmer actually had
a much easier task.

### OOP and Safety

There's a few wrinkles in this though, a few featuers in either programming
language that seem to detract from this goal. On the C++ side, we have
OOP and virtual methods, which are often less performant than the
equivalent hand-written C code would have been. On the Rust side, we
have safety: array indexes, by default, panic when the index is out
of bounds. What is "safety," and can Rust really be said to be interested
in minimizing the cost of abstractions if it's also trying to achieve
safety?

I know these seem like wildly related issues, but I think they're
actually connected.  Both C++ and Rust are trying to have their cake
and eat it too. They're trying to provide all of the conveniences of
a modern high-level programming language, while outputting binaries
equivalent to those a C programmer would make.

For most of C++'s history, OOP was an essential convenience of a
high-level programming language. Everyone's buzzwordy design patterns were
conceptualized in OOP. It was a low-level decision to make it optional,
to allow C-style code organization.  It was a low-level decision to make
`virtual` opt-in.

And throughout much of this time as well, C programmers *were* creating
similar code to this! It was very popular to write structs of function
pointers, or other complex mechanism to allow "object oriented design"
into C programs.

For Rust, memory safety is an essential convenience of a high-level
programming language. Unlike a Java or a Haskell, the goal isn't so much
being a memory-safe programming language as having a memory safe subset
and encouraging memory-safe abstractions around unsafe code. It was a
compromise to allow `unsafe` and raw pointers at all.

And even in C, the trend has gotten more towards linting and using
machines to verify safety, and leaving more checks in. I worked on a C
project myself recently where I manually checked every input argument
for `NULL`. Besides, contrary to the FUD, not only are most Rust
safety mechanisms are free, but the ones that aren't generally
have `unsafe` unchecked versions that can be used when you truly want
the performance rather than the safety.

And I can't emphasize this enough: `unsafe` Rust *is* part of Rust.
It's right there for you to use when you need it. Don't compare the
safe subset of Rust to C++; compare the real Rust where doing something
`unsafe` is just inconvenient enough to encourage thoughtfulness and
good abstraction.

## Comparison

So C++ and Rust both share the same essential goals:

* The C goal: Be portable while exposing the full power of assembly
language
* The C++ goal: Have modern, high-level programming language features
while still outputting code as good as what an assembly language
programmer would write

So if these goals appeal to you as a software developer, which
programming language should you use?

In my mind, that depends then either on the non-essential goals, or else
just the accidents of history and ecosystem.

If you have a large codebase already in C++ for example, then that
might mitigate against switching -- which we can file under C++'s goal
of being compatible with previous versions of C++. Similarly if there's
a C++ library that turns out to be the perfect match, that may make your
decision for you.

Strangely enough, it's similar in the other direction if you already have
a large Rust codebase -- there's just fewer people in that position. This
will probably change over time, though. I think it's already begun to
be true of the ecosystem, though.

I think discussing the goals is more interesting, however, especially in
the long term.

If you need object-oriented programming, which is another goal of C++,
then C++ might be your thing. I generally think object-oriented programming
is overrated and Rust's way of handling abstraction to be both more powerful
and less prone to problems, but many people disagree with me. Especially in
GUI programming, OOP patterns are far more established in C++, and Rust's
ecosystem is particularly behind in the GUI space.

However, if you're worried about memory corruption and the related security
vulnerabilities, it might be nice to have a guarantee that only certain
lines of code can cause such problems. It might be nice to have all those lines
marked with a special keyword and conventionally scrutinized and abstracted
in such a way as to help prevent these conditions.

And Rust's safety advantages go beyond simply delineating which features are
safe and which are unsafe. Rust is able to accomplish much more in its
safe subset without performance degradation than the average C++ programmer
might guess, because it has a more sophisticated type system.

For a system's programming language, I think memory safety is more
important than object-oriented programming and better GUI frameworks
(for now). GUI apps, a long time ago, used to be written locally in C
or C++ to run directly on the user's computer. Nowadays, they are more
likely to be deployed over the web and written in Javascript, and
even those apps that do run directly on the user's computer tend
to be written in other, less systems-oriented programming languages.

And over time, I suspect we'll find out that OOP isn't as
necessary to GUI frameworks as we had thought. My [favorite GUI
framework](https://reflex-frp.org/) personally is in Haskell
and doesn't use OOP at all. And once that happens, I think OOP
will simply be another legacy feature, as Rust's `trait` mechanism
is superior to OOP for non-GUI contexts.

Memory safety, on the other hand, is key for systems programs. Servers,
OS kernels, real-time systems, automobile controllers, cryptocurrency
wallets -- the domains where systems programming tends to be used are also
domains in which security vulnerabilities are absolutely unacceptable.
The fact that C++ doesn't have a safe subset, and makes it so
difficult to reason about undefined behavior compared to idiomatic Rust,
is a serious problem.

But even if Rust didn't have a specific memory safety advantage over C++,
it would still have quite a few things going for it. It avoids header
files and all the concomittant confusion. It gets rid of null pointers
in most contexts, called "the billion dollar mistake" by the inventor
of null pointers himself. It tidies up compile-time polymorphism and
brings it in line with run-time polymorphism. It has actual destructive
moves. In general, Rust takes advantage of the fact that it doesn't have
to also be C and also be old versions of C++, and uses it to create a
much cleaner experiences.

And that is what this book is about.

## This Book

This book is about how Rust has the goals of C++, many great ideas from
C++, without many of the problems of C++. And since memory safety is an
issue that has been discussed to death, and yelling about the borrow
checker has become a stereotype, this book is almost entirely about
other specific differences between the two programming languages.

The thesis of this book could thus be stated as follows:

> Rust accomplishes the essential goals of C++ and keeps the good ideas,
> while eliminating the cruft, which has far-reaching
> benefits over remaining compatible with C++.

This is more a persuasive document than an instructional document.
