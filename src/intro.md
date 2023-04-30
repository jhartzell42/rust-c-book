Rust is a polarizing programming language, because of how radical it
is. It has [gone the furthest](/posts/paradigm-shift) in introducing
features from functional programming languages into the mainstream world,
and ignoring long-held programming language design principles from the
realm of [object-oriented programming](/tags/beyond-oop/). Its fans can be
very enthusiastic, sometimes off-puttingly so, stereotypically demanding
that all software be rewritten in Rust even when completely unfeasible --
a stereotype that is mostly untrue, but whose existence and occasional
true examples shows the intensity of the debate. But a lot of Rust's
criticism comes specifically from C++ programmers, and correspondingly
a lot of Rustaceans' criticisms of other programming languages is
directed specifically at C++ (including of course in this book). Even
the creator of C++, while not mentioning it by name, [entered the
fray](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2739r0.pdf)
(and along with other Rustaceans, I responded, and my response is
included in the appendices).

There's a good reason for this particular rivalry. While usable in other
domains, Rust is strongest where C++ has hitherto been unopposed: as a
high-level systems programming language. Many of Rust's greatest strengths
are directly based off of ideas originated in C++, such as RAII. And
Rust has, in many ways, the same goals that C++ has.

Specifically, in this book I shall argue that Rust has the exact same
overall goal that C++ does, albeit with a different interpretation of
how that goal is best accomplished. I will further argue that Rust
does a better job of accomplishing these goals. Thus, the thesis of
this book is a slightly longer version of the title:

> Rust is a better C++ than C++, as it is better at C++'s own goals.

# Zero-Overhead Abstractions

C++ has an explicit goal of providing zero-cost abstractions.

This is a bit of a confusing term of art and has the potential
to be misleading, but it comes attached with explanations that
clarify it some. It is also referred to as the "zero-overhead
principle," which Dr. Bjarne Stroustrup, father of C++,
[explains](https://www.stroustrup.com/ETAPS-corrected-draft.pdf) (see
pg. 4) describes as containing two components:

* What you don’t use, you don’t pay for (and Dr. Stroustrup means
"paying" in the sense of performance costs, e.g. in higher latency,
slower throughput, or higher memory usage)
* What you do use, you couldn’t hand code any better

There is also an [executive summary of the
concept](https://en.cppreference.com/w/cpp/language/Zero-overhead_principle)
at [CppReference.com](https://en.cppreference.com/w/).

A clearer term that is occasionally used in the trenches in the C++
community is "zero-overhead abstraction" -- there is zero overhead,
defined as cost in addition to what a reasonably-well hand-coded
implementation would do. Using this term, a third principle becomes
clearer, which was hidden all along, unstated among those other two
principles, and against which those other two principles are balanced. The
word "abstraction" is the key, and the third principle is:

* You can still get the abstractive and expressive power you expect from
a modern programming language.

This third principle is necessary to distinguish higher-level "zero cost"
languages like C++ and Rust from lower-cost languages like C.

To fully explain why I include this third principle, and to delve into
the history of the concept in general, I want to talk more about C.

# C: The Portable Assembly

C has often been described as a "portable assembly language." Unlike
other high level programming languages before it ("high level" at
the time meaning anything higher level than raw assembly language),
it exposed users directly to gnarly machine-language abstractions like
pointers, and to common assembly-language capabilities like shifting
and bitwise operators.

The goal was to give the programmer something minimally distinct from
assembly language, where the programmer had almost as much control over
the computer as an assembly language programmer without sacrificing
portability. Few higher-level features have been added, even now:
there was no built-in string type, and only a limited array type that
exposed the underlying concept of pointers the instant you poked at it.
Structures are little more than a way of calculating offsets, and memory
management is done by explicitly invoking memory management routines.

C's preference, in general, was to only add onto assembly those features
absolutely necessary for portability, and not to impose any other
structure on the programmer -- or, said another way, not to provide
any other structure to the programmer.

This was far from an iron-clad rule. And there are definitely exceptions:
C, built into the programming language, prefers null-terminated strings
(also known as "C strings") to arrangements that use specific lengths,
a substantial constraint on the programmer beyond assembly language and
probably a mistake overall.

More deeply, and probably less avoidably at the time, C assumes a
traditional call structure. Many techniques that can be used to
implement closures, co-routines, or other more radical alternatives
to a call stack are difficult to impossible to do with standard C --
while generally being possible in any assembly language.

But, with these exceptions, C generally does tend to only provide
one overarching abstraction, portability, and when it does, it
has the same zero-cost goals that C++ has, to only make the user
pay for the abstractions they actually use, and to provide abstractions
as efficiently as the equivalent hand-coded assembly.

Put another way, C++'s zero-cost overhead principle, as Dr. Stroustrup
defines it, is more or less inherited from C. Where C++ differs from
C is in the "abstraction" part of providing "zero-cost abstractions."
Everything you can do in C++ you can do in (potentially tedious and
repetitive and error-prone) C, but C++ provides more abstractions,
beyond just what is necessary for portability.

And C does a great job of portability!  But missing from that goal is
anything about a generally usable set of abstractions. Some abstraction
over assembly language is necessary to ensure portability, but C doesn't
really go beyond that. It provides a standard library, but again,
that's for portability purposes. The C standard library is not much
more powerful than an assembly-language operating system API, just more
standardized. C is portable assembly, not abstracted assembly.

# C++: A More Abstracted C

C++ goes beyond that. C++ tries to be competitive. Before, we dissected
"zero-overhead abstractions" into three goals:

* What you don’t use, you don’t pay for
* What you do use, you couldn’t hand code any better
* We give you the power of abstraction expected for a programming language
of the day

But really, they are one goal. The essential goal of C++ over C is
this:

* Create a competitive set of abstractions that cost no more than manual
implementation in C or assembly, whether in terms of speed or memory or
any other resource.

This includes the entire principle of zero-overhead abstraction. I include the
word "competitive," to describe how C++ in fact approaches what abstractions
it offers, and how C does not: In competition with other programming
languages, which directly implies our third point above, about abstractions
expected for a modern programming language.

This general single goal also implies the second point, that what
you don't use you shouldn't pay for. If a feature is not used, a C
programmer would simply not implement it and pay no cost for it. So if
the C++ programmer pays any cost for it at all when they don't use it,
that cost is all overhead.

Now we have C++ down to one coherent, easily-stated goal. And once
we understand this, everything else about C++ makes sense.

C++ was originally christened "C with Classes," and it tried to add
Object-Oriented Programming to C. All the mechanisms of OOP could be
portably added to C directly by an application or library developer
with judicious use of function pointers and structure nesting (and
[`glib`](https://en.wikipedia.org/wiki/GLib) is a famous example of a
library that does exactly that), but C++ built this abstraction
into the programming language itself.

Objective-C also did this (and according to Wikipedia it "first appeared"
one year sooner in 1984), but Objective-C has always felt like two
programming languages glued together. In Objective-C, the object-oriented
features do not inherit the zero-overhead principle from C -- nor do they
look like C at all. They look instead like a Smalltalk dialect, where
switching between C and this odd Smalltalk dialect was permitted on an
expression-by-expression basis using an odd mix of square brackets and
`@`-signs.

In C++, the added abstractions, including OOP, take on more of
a resemblance to C, and importantly, continue to try to retain
C's advantages in systems programming by making the new features
zero-overhead.

During much of the history of C++, OOP was considered to be the most
important abstraction that a programming language could offer. But
once it was added, it expanded the scope of C++ abstractions. Nowadays,
C++ is considered multi-paradigm, and provides not just OOP, but a
wide array of abstraction.

The only features C++ rejects out of hand are those that do not jive
with zero-cost abstraction.  This is why garbage-collection is not
offered in C++ (though it is still possible to implement manually) --
it cannot be offered in a zero-cost way. However, C++'s alternative to
garbage collection, namely RAII, continues to become
more effective as new features like move semantics and `std::unique_ptr`
were added, to the extent that in modern C++, it would be unimaginable
not to have those features, and they have become essential to C++'s
memory management model.

This also explains why C++ keeps accruing new features -- to have a
competitive set of abstractions -- whereas C maintains the features it has
-- because the only thing it needs abstractions for is portability. This
explains why C++ had to add templates -- as a zero-cost alternative to
OOP, or a zero-cost way of implementing collections. They explain why
C++ had to add move semantics -- because without it, RAII is a worse
abstraction than GC.

These are great features that C++ has had to develop to achieve
these goals.

RAII is a genius solution to the problem that garbage collection is
not zero-overhead. With RAII, the source code looks simpler than in
C, and is harder to get wrong. It's not quite as straight-forward as
in a garbage-collected language, but it has many of the benefits of
abstraction: You can't just forget to call the destructor.

But! From the compiled binary, modulo such nit-picky accidents as symbol
names, you wouldn't be able to tell it was written in C++ instead of C!
The abstraction was resolved and disappeared at compile-time.

Similarly with templates, and therefore with the collections in the STL.
Similarly with many other C++ features.

And Rust has greatly benefitted from all of this innovation in C++.
C++ is one of the giants on whose shoulders Rust stands. Rust inherits
RAII from C++. Rust inherits the core idea of templates as well, though
it puts some constraints on them and calls them "monomorphization."

And Rust needs these, because Rust is also striving to be the kind of
programming language where the compiled binary looks like something
someone could have written in C, but where the programmer actually had
a much easier task. And I will argue that Rust does a better job.

# Wrinkles: OOP and Safety

There's a few wrinkles in this though, a few features in either programming
language that seem to detract from this goal, to undermine the idea
that this is truly a focus for them at all. On the C++ side, we have
OOP and virtual methods, which are often less performant than the
equivalent hand-written C code would have been. On the Rust side, we
have safety: array indexes, by default, panic when the index is out
of bounds. What is "safety," and can Rust really be said to be interested
in minimizing the cost of abstractions if it's also trying to achieve
safety?

I know these seem like wildly unrelated issues, but they're actually
connected. Both C++ and Rust are trying to have their cake and eat
it too. They're trying to provide all of the conveniences of a modern
high-level programming language, while outputting binaries equivalent
to those a C programmer would make.

But safety and OOP actually do have non-zero overhead. OOP has virtual
functions, preventing inlining and other optimizations and requiring
indirect function calls. Safety, for its part, requires bounds checking,
an obvious non-zero overhead. And in Rust, it constrains heap usage
to certain layouts that are often less efficient than the ideal layout
would be.

So why do C++ and Rust make this decision? And how do they justify it?

For most of C++'s history, OOP was an essential convenience of a
high-level programming language. Everyone's buzzwordy design patterns were
conceptualized in OOP. Without OOP, a programming language could not
at all be taken seriously at the time, as it was widely believed that
scalable software architecture and intuitive reasoning about code required
OOP.

And throughout much of this time as well, C programmers *were* creating
similar code to this, so it was actually like code a C programmer would
write by hand! It was very popular to write structs of function
pointers, or other complex mechanism to allow "object oriented design"
into C programs. This is the entire premise of `GObject`, now part of
`glib` and used by `gtk`.

Nevertheless, OOP has run-time costs. Run-time polymorphism (known in
C++ as "virtual functions") is one of the pillars of OOP, the basis
of software decision-making, and a powerful abstraction. In most OOP
programming languages, this meant (and still means) there is a (non-zero
cost) run-time decision made at every method call.

C++ maintains its "zero-overhead abstraction" principle here on a
technicality: by making it optional. Rather than not including the
feature, C++ makes you only pay for it if you use it. C++ was considered
to be low-level by making `virtual` an opt-in keyword, rather than
the default. As we have said, not having it at all would've utterly
disqualified C++ as an application programming language. It was considered
necessary for usable abstractions.

After all, in programs where C++ programmers gets to just write `virtual`,
C programmers were implementing all of OOP, including runtime polymorphism,
by hand.

Rust, however, starts over trying to accomplish this goal from a much
more recent time. Now, OOP is no longer a *sine qua non* of programming
languages -- in fact, it's become old-fashioned. Rust uses features
from the functional programming paradigm instead to manage abstractions
and give users a chance at wrangling a large codebase. In doing so,
Rust has largely moved beyond the need for OOP.

However, just like OOP was considered essential to be taken seriously
when C++ was in its heyday, nowadays, new programming languages are
expected to be memory safe. Memory safety is no longer a feature for
"novel" programming languages to experiment with, as Dr. Stroustrup,
father of C++, unfortunately still thinks of it.

Memory safety comes with performance costs. Until Rust came along, it was
widely assumed that it required garbage collection, an unacceptable cost,
one that is impossible to opt out of and still use a heap, one that
generally is paid not as-used but on a per-program basis. Rust, however,
has managed to figure out a way to extend C++'s RAII and move semantics
with the borrow checker and lifetimes (from Cyclone's "regions"), and
make manual memory management with RAII safe.

There are still performance costs, but they are paid on an as-needed
basis, and opt-out is still available.Unlike a Java or a Haskell, the
goal isn't so much being a memory-safe programming language as having
a memory safe subset and encouraging memory-safe abstractions around
unsafe code. Similarly to how C++ is distinct from other OOP languages
by making virtual functions optional, Rust is distinct by having `unsafe`
and raw pointers at all. If there was no `unsafe` keyword to guard these
features and protect programmers from using them by accident, it would've
disqualified Rust, in 2015, from being an application programming language
at all.

Protections for safety are now considered necessary for usable
abstractions. I would say that C++ gets away with it because of its
venerable, established position, but in fact it does not get away with
it. No one uses C++ for new application-level programming, but rather only
for systems programming where the performance is absolutely necessary.

This is because of safety. Many C++ programmers, including Dr. Stroustrup
himself, don't realize that the rest of the world has already moved on to
safe programming languages and it's no longer considered a novelty. Often,
they think systems programming is the world rather than just a niche. But
make no mistake, now that Rust has brought safe programming to this niche,
it is only a matter of time.

But it still satisfies the zero-abstraction principle, even though there
is overhead. The overhead only applies when the feature is being used.
If there is a line of code where the overhead is unacceptable, the
solution is simple: use the `unsafe` keyword, and use a non-bounds
checked method (or an alternative data structure with raw pointers).
It is advised that you wrap this unsafe code in a safe abstraction, but
that is just advice. In the end, Rust only makes you pay for abstractions
you're actually using, just like C++.

## Should I use C++ or Rust?

So C++ and Rust both share the same essential goals:

* The C goal: Be portable while exposing the full power of assembly
language.
* The C++ goal, which implies the C goal: Have modern, high-level
programming language features while still outputting code as good as
what an assembly language programmer would write.

So if these goals appeal to you as a software developer, which
programming language should you use, Rust or C++?

In my mind, that depends then either on the non-essential goals, or else
just the accidents of history and ecosystem.

If you have a large codebase already in C++ for example, then that
might mitigate against switching. We can attribute this to C++'s goal
of being compatible with previous versions of C++, a goal C++ has paid
much for. Similarly if there's a C++ library that turns out to be the
perfect match, that may make your decision for you.

But to be clear, it's similar in the other direction if you already have
a large Rust codebase -- there's just fewer people in that position. This
will probably change over time, though. I think Rust's ecosystem is
already competitive with C++.

I think discussing the goals is more interesting, however, especially in
the long term.

If you need object-oriented programming, which is another goal of
C++, then C++ might be your thing. I generally think object-oriented
programming is overrated and Rust's way of handling abstraction to be
both more powerful and less prone to problems, but many people disagree
with me. And I must admit: the big use case everyone always mentions for
OOP is GUI programming, and Rust's ecosystem is particularly behind in
the GUI space.

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
(for now). GUI apps, a long time ago, used to be written locally in
C or C++ to run directly on the user's computer. Nowadays, they are
more likely to be deployed over the web and written in Javascript,
and even those apps that do run directly on the user's computer tend
to be written in other, less systems-oriented programming languages.
If you're writing a GUI app, the choice for the part of the app that
the user interacts with isn't between Rust and C++; it's between Rust,
C++, C#, Java, JavaScript, and many others. Neither Rust or C++ stand
much of a chance in the GUI space long-term.

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
moves.

In general, Rust takes advantage of the fact that it doesn't have
to also be C and also be old versions of C++, and uses it to create a
much cleaner experience.

## Rust Deficits

Rust has a few other specific downsides compared to C++.

Interfacing with C is an important goal for reasons besides
backwards-compatibility. On many platforms, C serves as a
lowest-common-denominator programming language, and its ABI serves as an
[inter-language protocol](https://faultlore.com/blah/c-isnt-a-language/).
C++ does provide smoother interfacing with this protocol than Rust does.

Relatedly, C++ generally has a relatively stable ABI on a given platform
for a given compiler vendor. This allows dynamic libraries to be used
as plugins with minimal glue code, something that in Rust normally
requires awkwardly working through a C ABI interface. Personally, I think
machine-language plugins as dynamically loaded libraries are mostly
a relic of past software distribution models, and haven't seen many
situations where they make sense, but I could think of a few edge cases.

In both of these cases, Rust is clumsier, but not completely incapable.
Rust still can speak the protocol that is the C ABI, just not as natively
and smoothly-integrated as C++.

Other downsides of Rust have to do with network effects and Rust
adoption. There is only one Rust compiler, while there are multiple
C++ compilers, that work together through a standards process. GCC
is currently in the process of getting Rust support, and we'll see
how well that works out for Rust.

Similarly, there are a lot of libraries that exist in C++ that don't
yet exist in Rust or have Rust bindings. Though that's true of any
pair of programming languages, it is a specific reason some developers
might still want to write new projects in C++ in favor of Rust.

Finally, while I still think Rust would be a better programming language
than C++ even if unsafe code were allowed everywhere, I think Rust
could do more to make its rules clearer in the unsafe realm. The fact
that the latest research on Rust's memory models seems so deeply
difficult to square with how `async` code often works [as in this
bug report](https://github.com/rust-lang/unsafe-code-guidelines/issues/148)
makes me nervous.

I'm sure there are other ways in which Rust is behind C++, and the
devil is as always in the details.

# This Book

But enough about the exceptions! Every thesis worth writing a book
about has caveats! Let's get back to the thesis of this book.

The groundwork for the thesis of this book is laid out above in this
introduction. It is as follows:

> Rust accomplishes the essential goals of C++ and keeps the good ideas,
> while eliminating the cruft, which has far-reaching
> benefits over remaining compatible with C++.

This book is all about the details and specific examples of this thesis.
It covers a number of topics, from the detals of how it expands RAII to
make it safe, how it makes move semantics less confusing, all the way
to clarifications on what safety means in practice.

It originated as a collection of blog posts on my blog, [The
Coded Message](https://www.thecodedmessage.com), and it's an
on-going project. New sections will continue to also be posted as
blog posts, though revisions to the existing sections will take
place sporadically with little fanfare. Please make issues and MRs on the [git
repo](https://github.com/jhartzell42/rust-c-book/) if you see any mistakes,
or would otherwise like to contribute thoughts, criticisms, or additional
content.

This book is more a persuasive document than an instructional document.
It's a work of apologetics, explaining in detail, topic by topic, why
Rust is good at these goals it shares with C++, and why a new programming
language was necessary to achieve them more effectively.

Like most books of apologetics, it's nominally aimed at the skeptics,
in this case the C++ developers who don't like Rust. But only nominally.
It will be far more interesting for the seekers and the proselytes:
Those who are interested in looking into Rust, or who have started
using Rust, but aren't fully sure of its benefits over C++, or whether
it can be truly used in as many ways as C++ can, or whether it can
truly be as high-performance.

We've known for some time that C++ was hampered by its C legacy.
Today, modern C++ is also hampered by the legacy of pre-modern C++.

Bjarne Stroustrup once famously said:

> Within C++, there is a much smaller and cleaner language struggling
> to get out.

I'm sure Bjarne Stroustrup has already said that this quote was not
about Rust, just as a long time ago he said that it wasn't about C#
or Java. But I think it resonated with so many people because we all
know how much cruft and complexity has accrued in C++. And the fact that
the quote resonated so much I think says more about C++ than whatever
Bjarne's original intentions were with these specific words.

And so, even though Bjarne explicitly said otherwise, I think
Java and C# were efforts to extract this smaller and cleaner language
-- one that had C++-style object-oriented programming without the
other bits that they considered the "cruft." And for a substantial
slice of C++ programmers, who didn't need control of memory and
could afford garbage collection, this is exactly what the doctor
ordered: Many use cases of C# an Java today would've previously
been filled by C++.

Remember, C++ used to be a general-purpose programming language. Now, it's
a niche systems programming languages. It has been edged out of
other niches in large part by "C-like programming languages" that
take what they like from it, and leave the rest, like Java and C#.

And I think Rust is finishing the job. It is a similar effort, but with
a different opinion about which bits constitute the "cruft." Zero-cost
abstraction remains a goal, but C compatibility does not. One of the
goals of this book is to convince you that this is the right decision
for a systems programming language.

Which brings me to this book's secondary thesis:

> With a few notable exceptions, between Rust and C++, Rust is the better
> language to start a new project in. If you were going to write something
> new in C++, I think you should almost always use Rust instead, or else
> another programming language if it's outside of Rust/C++'s core niche.

This is a corollary of this book's primary thesis: if Rust has the same
goals as C++, and accomplishes them just as well with fewer downsides
and fewer costs, why would you use C++? In my opinion, the exceptions
are already very limited, and will only decrease with time, until C++
is only appropriate for -- and ultimately only used for -- legacy projects.

Again, C++ is one of the giants on whose shoulders Rust stands. Without
C++, Rust would've been impossible, just like Java and C# would've been
impossible. But sometimes a clean, breaking re-design is required,
and Rust provides exactly that.
