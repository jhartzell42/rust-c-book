+++
title = "Sayonara, C++, and hello to Rust!"
date = "2021-10-26"
slug = "hello-rust"
author = "Jimmy Hartzell"
cover = ""
tags = ["programming", "Rust", "Rust vs C++", "C++", "computers"]
keywords = ["", ""]
description = ""
showFullContent = false
draft = false
+++

This past May, I started a new job working in Rust. I was somewhat
skeptical of Rust for a while, but it turns out, it really is all it's
cracked up to be. As a long-time C++ programmer, and C++ instructor,
I am convinced that Rust is better than C++ in all of C++'s application
space, that for any new programming project where C++ would make sense
as the programming language, Rust would make more sense.

## What Rust is not for

Before going into more detail about why I think that, I'd like to throw
out a few caveats, so you know I'm a reasonable person and not just an
extremist fanboy.

Caveat the first: Note that I said this about *new* programming
project. There are some people on the Internet who demand the re-writing
of all existing C and C++ projects in Rust, and while I think Rust is a
better language for new projects, and that many existing projects should
seriously consider integrating it, I realize, like a reasonable mature
programmer, that for most existing projects, a Rust re-write would be
a prohibitively expensive rabbit hole. In short, Rust is not so amazing
that it will protect you from second system syndrome or from the perils
of a complete rewrite. It is but a mortal programming language.

That said, new Rust versions of aging C and C++ projects are often
very worthwhile and exciting, like new versions of many aging projects
can be. It's just not a magical exception to basic economics.

Caveat the second: Note also that I said "where C++ would make sense."
Rust has a lot of enthusiastic fans, and so there are a lot of people
learning Rust expecting the magic when they first learned their favorite
programming language. And what they find is a programming language that
requires lots of arcane rules, where everything seems rather tedious,
and where a lot of their favorite features don't exist.

Rust is a systems programming language. It is not garbage collected,
meaning you do have to manually manage memory. While Rust makes it
much harder to do that egregiously wrong, it's still a very hard
problem, and there are trade-offs that Rust -- unlike GC'd languages
-- refuses to make for you.  Meanwhile, like C++, the emphasis is on
performance (or at least control over performance), whether latency
or throughput or memory footprint. Rust is trying to make sure that all
its organizational abstractions have no run-time cost, or, if they do,
to make sure it's abundantly clear exactly what that cost is.  If you
are a systems programmer, if you are used to C and C++ and to trying to
solve systems programming types of problems, Rust is magical, just like
when you learned your previous favorite programming language.

If you are not, Rust is overkill for your task at hand and you shouldn't
be using it. I earnestly recommend Haskell.

For more clarity on what I mean by systems programming: If you write
Python or JavaScript or Ruby, then you're running the code in a Python
interpreter, in Node or a web browser, in the Ruby interpreter, all on
top of an operating system with an operating system kernel.  Rust doesn't
replace those tools. Rather, the Python interpreter, the web browser,
and Node, and even the kernel, are programs written in C or C++, and
Rust replaces that. It's a whole 'nother level of programming,
where you have manage the actual hardware.

## Purpose of this series

But enough of what Rust is and is not for. It is an excellent systems
programming language, and one that was a long time coming. I plan
on writing several posts about Rust features, why they're an improvement
upon C++ features, and why Rust is a better, more modern programming
language. Mostly, this will be a discussion of why Rust is better than
C++, which I think is the most comparable existing programming language,
but it will also touch on why Rust is an improvement on C.

Because of this C++ focus, this series will at times be as much or
more a criticism of C++ as it is a commendation of Rust. I think that
is unavoidable, as this type of criticism of C++ is most truly credible
when an alternative is available, and similarly, Rust is most practically
evaluated in terms of its most viable alternative. Unfortunately, that also
means that I'll assume some level of familiarity with C++, but hopefully
not too much.

I know that this is a much-discussed topic. Perhaps this is the Rust
equivalent of the dreaded Haskell monad tutorial, where every person
new to the programming language excitedly writes the same thing, and so
thank you for reading.  I'm going to try and avoid the obvious tropes:
I'm going to try to do more than simply beat the table about type-safety
and memory safety and avoiding undefined behavior -- though of course
these topics will come up. In fact, I had until rather recently simply
assumed that the safety of Rust would lead to unacceptable performance
degradation, that Rust might be well and good for some applications but
could unfortunately never be useful in a true low-latency environment. I
had to be persuaded that memory safety wasn't a downside in such contexts,
that Rust could truly be a competitor to C or C++ and not just to Go
or Swift.

## The syntax of C++

So for today, I'm going to ignore memory safety completely. Even assuming
that C++ was more or less right that performance and optimization requires
a broad range of undefined behaviors, there were still problems with
C++ that left me regularly begging for at least a syntactic rewrite.
As Bjarne Stroustrup, creator of C++, famously said: "Within C++, there
is a much smaller and cleaner language struggling to get out."

He later clarified that he wasn't talking about a streamlined GC'd
language like Java, and of course he is aware of Rust and still on the C++
train. As he clarified, he was talking about the syntax of C++, and the
legacy of C. But just that category, just syntax, is I think enough to
justify a do-over of C++. I fantasized continually about a new syntax --
with identical semantics in my mind -- that could be migrated to in a
file-by-file basis, with its own file extension.  This, I realize now,
would in practice make for a new language, and a good opportunity to
introduce modern typing in it, and in Rust, I see my hope realized,
if a little more inconveniently than I imagined.

So that is what I want to focus on for the rest of this post: Why C++
syntax is rotten. Analyses of other Rust features I will reserve for
future posts.

Many of C++'s syntactic foibles have to do with its C heritage. This is
not to smear C: the same features that make sense in a simple "portable
assembly" like C begin to break down when they are preserved with
almost-identical syntax and naively extended semantics in a language that
promises powerful generic programming features that assist in automatic
code generation, resource management, and memory safety.

## Header Files

This is really clear in my first example: header files. In C, they
serve two purposes. For the programmer, they allow a separation of
interface and implementation, especially when considering that modern
IDE technology did not exist when C was developed. The header file shows
the external interface of how to use the module, and the C file shows
how it is implemented.

For the compiler, this arrangement simplifies implementation. The information
necessary to compile each module is all included in the C file for the
module plus all the headers included by it (and included by them, etc.).
None of the other `.c` C files need be consulted, only the much smaller `.h`
headers, in a practice known as *separate compilation*.

The problem when this is extended to C++ is templates. These are essentially
macros, in that they allow the on-demand generation of new code, based on
what is going on in the client code. So if we imagine that the module `broadcast`
depends on the module `connection`, and that the compiler is currently compiling
`broadcast`, it would not only be necessary to investigate the interface to
`connection`, but also the complete implementation of the templates.

If we are to preserve the programmers' perspective, and keep only the
interface in the header files, this means that "separate compilation"
is broken and that the compiler would need to fish around in the main
C++ files. If we instead preserve the concept of separate compilation,
the headers are no longer about just interfaces, but also implementation
details.

The inventors of C++ decided to preserve the concept of separate compilation,
and in the laziest way possible, literally the exact same implementation of C rather
than trying to apply the same principles and goals, and re-engineering something
better.

So now, we have the situation for the programmer where the header file contains
a duplicate of the interface, in addition to the implementation of all functions
that happen to be templates. To a compiler, a function and a function
template are very different things, but to a user, functions go back and forth
between template and not all the time, requiring the programmer to move them
between files.

Why does this distinction exist at all in C++? Computers are much faster now.
The compiler, to preserve efficiency, could automatically extract its own
binary file of what information it needs from each module to compile other
modules. The compiler could do this work for us. It is doing far more work
in all of its optimization steps.

C is famously a portable syntax for writing assembly. In C, the
information needed by the compiler, the *application binary interface*
or ABI, is exactly the same as the interface needed by the programmer,
the *application programming interface* or API -- or at least very
close to it. And so in C, the concept of header files makes sense,
if unnecessary with modern compiler technology.

And to be clear, templates are just the most egregious example of
non-interface code needed from other modules: bodies of inline functions
and private member variables in classes are also not part of the interface
from a programming perspective, but part of the binary interface, part
of what the compiler needs to know about a module to compile the other
modules that depend on it.

Now, you may think, why is this such a big deal? Why such a complaint about
the inconvenience of moving things between files, or duplicating some
information? Why does it matter if the rules of which things to put
in which file are on the arcane side? You might imagine, so what?
You'll mess it sometimes, the compiler will issue an error, you'll
say "oh, right" and move the code to the appropriate location.

To such objections I say: You don't know C++. Unfortunately, I've
seen this attitude taken by professional C++ programmers, who were
careless with header files, moving code around in bulk in a way that
was liable to break this particular set of arcane rules, and accusing
me of overreacting and wasting time when I objected to this.

For those who haven't had the misfortunate of finding this out the
hard way, when you break an arcane rule in C++ -- even rules that have
nothing to do with run-time behavior or memory safety -- you are lucky
if you get away with a simple compiler error -- or even the somewhat
more common arcane, incomprehensible compiler error. Unfortunately,
the result is regularly no error at all. The compiler cannot tell,
from its separate compilation point of view, if the information
provided in the headers is consistent. One module might import
one version of a header, and another module might import another.

This may sound unlikely, but many codebases have the practice of
separating out the actual interface in one header, and the template
implementations in another. At this point, it becomes important which
one is included, especially because "template specializations" mean
that additional template code doesn't just make more templates
available, but changes the meaning of existing templates.

If the templates included in different compilation units are inconsistent,
the result is undefinned behavior, and the program might potentially do
anything. Unfortunately, this also means the behavior might switch arbitrarily
between different compiler versions, different compiler vendors, or based on
seemingly unrelated permutations. Unpredictable behavior changes lead to bugs
and security vulnerabilities.

Worse, header files are implemented by textual inclusion. The compiler
proceeds as if the contents of the header were literally included in the
module that imports them. Cycles of inclusion don't result in error, but
instead, a header is simply not included (if common precautions are made)
when the second recursion happens.

Thus: Imports via header files are sensitive to ordering. A
seemingly-innocuous change, like alphabetizing the included header files
in each module, can break builds or change behavior. Such a change rolled
out over an entire company's codebase can be disastrous, and take many
programmer-months to unravel the consequences of. Ask me how I know.

So it should make sense that the first concept I had for my "new syntax"
for C++ was that header files should be auto-generated from source files,
preferably in a pre-compiled binary format.

This would be an implementation detail of the build artifact, maintained
a build directory, and be a compiler-specific optimization in favor of
better compilation times. Semantically, rather than textual inclusion,
there would simply be a declaration in one module to say that another
module's public interface could be used, where order wouldn't
matter.

Rust is a modern programming language, and the Rust `use` directive does
in fact work that way.

This isn't particularly a special point about Rust. This would be the
obvious way to construct any new programming language. The compiler
doesn't need header files -- the C preprocessor that implements `#include`
directives, along with the rule that functions and structures must be
declared before use, is a hold-over from the time when compilers ran on
computers slower than a modern thermostat. And programmers don't need
them either: A better place for interfaces to be put in a separate file
would be automatically-generated documentation.

So Rust here gets points for doing what any sensible modern programming
language would do, and C++ loses points for carrying over an implementation
detail from C to a context where it no longer makes any sense.

## Syntax and Layout

Since we're talking about the syntax of C++, I wanted to touch on something
very basic but very serious: basic syntax for control structures. C and its
syntactic descendants, including C# and Java, use something like this for
`if`-statements and `for`-statements:

```
if (!foo.is_empty()) {
    spin_up_thread(foo);
    destroy(&bar);
}
do_something_else();
```

In this example, the calls to `spin_up_thread` and `destroy` are inside
the `if` statement, and only happen if `foo` is indeed non-empty. The
call to `do_something_else` is not part of the `if` statement.

How do we know that? Well, the compiler knows that because after the `if`
statement there is an opening brace, and so all statements are included
until the matching closing brace, including the two mentioned. But,
depending on how fast we're skimming the code, we probably know that
because the `spin_up_thread` and `destroy` calls are indented.

In this situation, in what will be a recurring theme in this comparison,
the compiler and the programmer are getting their information from
different places. Therefore, the compiler and the programmer can disagree,
especially as braces aren't mandatory, and if omitted indicate that only
the first subsequent statement is included:

```
if (!foo.is_empty())
    spin_up_thread(foo);
    destroy(&bar); // Warning: This is done unconditionally
do_something_else();
```

This looks like it only destroys `&bar` conditionally, and to a human
following the indentation in code review or casual reading, that's
exactly what you would expect. But there's no braces and the compiler,
for whatever reason, ignores the same whitespace that human readers
rely on.

This has come up in personal projects of mine, usually when collaborating
with someone else. Even if you make the personal discipline of always
including the braces `{` around the body of your if-statements `}`, someone
else might not have that discipline, and therefore, you might be exposed
to this intermediate-state code:

```
if (!foo.is_empty())
    spin_up_thread(foo);
do_something_else();
```

Needing to add a call to `destroy(&bar)` in the condition, after `spin_up_thread`,
you find the line, add a new line at the same indentation level, and simply fail
to notice that the new line is not actually wrapped in any `{`.

This was, of course, the direct cause of a [major security vulnerability in iOS
and macOS](https://arstechnica.com/information-technology/2014/02/extremely-critical-crypto-flaw-in-ios-may-also-affect-fully-patched-macs/):

```
if (some_err_condition)
    goto fail;
    goto fail;
```

Since humans use indentation to read code, and to determine what is in a block
and what isn't, I would've wanted my wish-list "new C++ syntax" programming
language to take a page from Python and use significant whitespace:

```
if !foo.is_empty():
    spin_up_thread(foo)
    destroy(&bar)
do_something_else()
```

Rust is a minor disappointment in this department. It stuck to braces,
and whitespace being "insignificant." But it made a huge improvement,
far outweighing my disappointment: Rust at least prevents the `goto
fail` scenario by making braces mandatory, helping ergonomics by
instead removing bracketing around the condition. Having the body of the
`if`-statement without brackets is simply not worth it as a short-cut,
but if the braces are mandatory, then the parentheses aren't necessary:

```
if !foo.is_empty() {
    spin_up_thread(foo);
    destroy(bar);
}
do_something_else();
```

This is better because then the `goto fail` example would still be glaringly obviously
failsome, because even if the indentation does not match the braces, the braces still
have to go somewhere, and will jump out at you:

```
if some_err_condition {
    goto fail; }
    goto fail;
```

It disappoints me that these issues tend to be dismissed as "matters
of taste," because as Apple learned, there are actual consequences to
this misalignment of what programmers pay attention to and what the
compiler pays attention to. I would have liked Rust to go the whole
way, and remove altogether this strange concept that whitespace should
be insignificant, a concept that my oldest C and C++ books exclaimed
as a great feature without explanation or justification.  But at least
Rust has fixed the most egregious consequences of C++ syntax. Again,
this problem with C++ comes from a feature inherited from C, but in this
case C was just wrong to begin with, and should've done it the Rust way
(or the Python way) from the very start.

Additionally, Rust has good auto-formatting, which unlike C++ auto-formatting
tools, do not break code (by, for example, re-ordering headers). This fundamentally
replaces the whitespace provided by the programmer -- which might be misleading to
other programmers -- with whitespace that aligns with the compiler's interpretation,
and therefore is correct to rely on when skimming. A good `cargo fmt` should therefore
be run before every code review, to make sure that the code can be easily and correctly
read.

## C-isms vs "Modern C++"

I then have one more topic before I wrap up syntax. C++ programmers
nowadays are telling everyone who was upset with their language in
the 90's and aughts that it's better now, that C++ has cleaned up its
act. C++11 really has changed a lot, and C++ is innovating again, and
that's very good. C++ is full of new features, and part of its claim to
be a modern programming language involves claiming that programming in
C++ is good, if you use these new features.

Smart pointers, written `std::unique_ptr<Foo>`, allow automatic
implementation of construction and destruction for owning pointers, and
allows much clearer communication about ownership semantics in function
signatures, and so is preferable to writing the C-style `Foo *`. C++
arrays, `std::array<Foo, 12> arr;`, act like any other STL collection,
allow them and their iterators to be passed to standard templates that
expect STL interfaces, and provide a number of useful features as methods,
and so using them is preferable to the C-style `Foo arr[12];`.
`static_cast<A>(b)` is much more specific, and therefore less prone to
accident, than the C-style `(A)b`.

These are among the features that are trotted out whenever someone says
they used C++ in the 90's and it had all these problems. These features
are among the ones used to claim that C++ is a better, cleaner, tidier,
more modern programming language than it used to be. Whether or not
they've done enough to replace their old counter-parts -- they're
generally preferred whenever possible.

The problem? Convenience. Who wants to type `std::unique_ptr<Foo>` when
instead you can write `Foo *`? Why are the somewhat-deprecated options
the easy ones to write? Why isn't it something like `std::raw_ptr<Foo>`
with some convenient notation for `std::unique_ptr`?

But of course, that would break compatibility with C, and with earlier
versions of C++.

I don't want to get into the myriad reasons why smart pointers are to be
preferred to raw pointers, or why raw pointers occupy such an awkward place
in C++ -- those are topics for a future post. But for however many seemingly-principled
reasons some of my colleagues might state for why they used raw pointers in
this or that situation, I couldn't shake the feeling that it was partially
because raw pointers were given the old-fashioned, easy-to-type notation.

And so, when I imagined my new C++ syntax, it would have `Foo *` mean
`std::unique_ptr<Foo>`, and `Foo arr[12]` mean `std::array<Foo, 12>`.
Why have the not-entirely-deprecated-but-not-preferred legacy C features
be the easier ones to type?

## Conclusions

All in all, this shows that a lot of purely syntactic but still substantial
and consequential problems with C++ can be fixed with a syntax reboot, which
Rust mostly provides. And I haven't once mentioned type safety or memory safety!
This will be developed on further in this blog series, where I will maintain that
Rust is not only a better programming language than C++, but a better *unsafe*
programming language than C++. Even if I had to use `unsafe` for every function
in my module, I'd still rather write my module in Rust than C++, for all
these reasons. I say this as a pre-emptive strike against the argument
that occasionally having to use `unsafe` to achieve performance parity
with C++ (and it is very occasional) "defeats the whole purpose" of Rust.

But of course, there are deeper problems with C++ that Rust also
addresses, beyond just the syntactic. But those will have to wait for
future posts.
