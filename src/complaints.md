# Common Complaints about Rust

Before I get into the specific topics, though, I'd like to clear up and
respond to a few anti-Rust talking points I've seen in the discourse.

> Rust fans all want to rewrite everything in Rust immediately
as a panacea.

Unfortunately, we have a vocal minority who do! But most Rust
developers have a much more moderate perspective. Most are aware
that a large project cannot and should not be rewritten lightly.

> Rust is a fad

Rust might be popular right now, but it also is already a part
of a lot of critical infrastructure, and is currently being put
in the Linux kernel.

> Rust fans are all young and naive people with little real-life
> programming experience

Sure, there's fans of all languages who are like that.

But there's also Bryan Cantrill embracing Rust after a
lifetime of disliking C++. There's Linus Torvalds allowing it
in the kernel, after his very vocal anti-C++ statements.

There's a long tradition of people who like C, but don't like C++.
See the [Frequently Questioned Answers](https://yosefk.com/c++fqa/) for
a taste. Their criticisms are, in many cases, legitimate. This is
unfortunate, because C has such limited capacity for abstraction,
and so they're missing out on all the abstractive power that
a higher-level language can provide.

For some of these people, Rust addresses the most important criticisms.

Personally, I agreed with many of those criticisms, personally, but was
a professional C++ programmer for 5 years, working in positions where
the zero-cost abstractions were absolutely necessary. I wasn't in a
position to choose programming languages at the company where I was,
but I agreed with the choice of C++ over C, in spite of what in my mind
were the clear costs of overcomplexity. I enjoy Rust now because it
addresses many of those problems.

> But you must at least admit that Rust fans are annoying.

Many of the ones that annoy you, annoy me too, if not more.

I also know that I've annoyed C++ fans by advocating for Rust on the C++
subreddit. My post was taken down by moderators as irrelevant to C++,
but how could you be more relevant than a thorough critique?

> Rust will be in the same place as C++ in 40 years

First, if there needs to be a new systems language in another 30
or 40 years that reboots Rust like Rust is rebooting C++, I don't
see that as a failure. I certainly don't see that as a reason
not to use Rust now.

And maybe Rust will be in 40 years as messy as C++ is now. Probably C++
will be even messier by then. Maybe C++ will clean up certain problems;
probably Rust will help spur it to do so by means of competition.

But also: Maybe Rust will be able to avoid some of C++'s mistakes;
it's certainly trying to.

> No programming language is better than another; there's simply
> different tools for different jobs.

I have a hard time taking this line of argument very seriously,
and yet it comes up a lot. There are some tools for which almost
no job is the right job. There are some tools that are just worse
than other tools. No one uses VHS tapes anymore; there's no job
for which they're the right tool.

Programming languages are technology. Some technologies simply
dominate others. There are currently still some things that the C++
ecosystem has that Rust doesn't yet: I'm thinking about the GUI library
space, and gcc support. Also, C++ has undeniably better interoperability
with C, which is relevant.

But both of those things might change. There is no natural reason why
C++ and Rust would be on equal fitting, or why Rust wouldn't at some
point in the future be better than C++ at literally every single thing
besides support for legacy C++ codebases.  Some tools are simply better
than others. No one's writing new production code in COBOL anymore;
it's a bad tool for a new project.

> C++ undefined behavior is avoidable if you're actually good at C++/if
> you just try harder and learn the job skills. You just have to use
> established best practices and a lot of problems go away.

First off, my experience working at a low-latency C++ shop shows that
that's not true. Avoiding undefined behavior in high-performance C++
is extremely hard. It's hard in Rust too, but at least Rust gives
you tools to manage this risk explicitly. If you're avoiding
memory corruption errors in C++, you've either found a safe subset,
or you're coding easy problems, or likely both.

But even if there are use cases where this is true,
to me that means that an experienced C++ programmer can be just
as good at avoiding undefined behavior as a novice Rust programmer.
So what does this mean for a business considering whether to use
C++ or Rust? In C++ everything a junior programmer writes
requires more scrutiny from senior programmers. Everyone requires
more training and more time to do things correctly. It's not
that good of a selling point.

Similarly, using best practices makes it sound easy, or at least
achievable. But the more complicated and arcane best practices are,
and the higher the stakes of following them, the higher the cognitive
load on the programmers, and again, the more you need senior programmers
to look over everyone else's work or even do parts of the work themselves.

When we've been doing something the hard way for a long time,
and it's successful, it's tempting to see other people struggling
and to tell them it's not that hard, that they can just up
their knowledge and their work ethic and do it the hard way
like us. But in the end, everyone benefits if the work is just easier
with better tools.

And what's a better tool than a "best practice"? An error message.
A lint. A programming language structured in such a way that it
doesn't even come up.

> Programming language is a matter of personal preference.

For your hobby project, sure, this is true. But there are real differences
between programming languages in terms of many things that matter for
business purposes.

> The existence of `unsafe` defeats the purpose of Rust. You have
> to use `unsafe`, and since the standard library uses it, you're almost
> certainly using it too. That makes Rust unsafe just like C++ in practice,
> and so there's no advantage to switching.

I would call this a straw man, but people do call Rust a "safe"
programming language, and some people say you should never have to use
`unsafe` (which I disagree with). So this takes some addressing.

First of all, memory safety, while important, is not the only
purpose of Rust. I would switch to Rust even if it didn't have the
`unsafe` keyword. There are many other problems about C++, and this
book focuses primarily on the other problems.

Second of all, in every "memory-safe" language, safe abstractions are
built from unsafe foundations.  You have to -- assembly language is
unsafe. `unsafe` allows those foundations to be written in Rust.
And that is what a memory safe language is, not one that is 100%
memory safe in all situations, but one in which it's possible to
explicitly manage and scope memory safety, and do most regular
tasks in the safe subset.

You can't both have the guard rails that Rust provides and write certain
types of high-performance code at the same time, but the `unsafe`
keyword allows you to make the decision on whether to have your cake or
eat it on a situation-by-situation basis, rather than giving up one or
the other for the entire programming language.

If you don't use `unsafe`, and you trust the libraries you import, then
you're in a safe language. If you do use `unsafe`, you are temporarily
in a language as flexible as C++, while still having many advantages of
Rust -- including safety features, which are still fully in place for
most programming constructs even in `unsafe` blocks.

Use the `unsafe` code to build more safe components, and expand the safe
language, and you get to only worry about safety a small percentage of
the time, as opposed to all the time in C++.

> Rust is taking the easy way out. Or: You can do C++ well, you just
> have to work harder at it, so there's no point to Rust.

I do think people regularly underestimate their ability to write safe C++.
Other people underestimate how much performance they're giving up on by
making sure they're confident their C++ is safe.

But even if you have put in the work to be good at writing C++ safely,
why does that mean that someone else shouldn't be happy to get the same
results with less training and less work, if the technology exists?

Who wouldn't want to take the easy way out? Do you exit your house by
climbing through the windows? This phrase only makes sense when there's
a downside, in which case the response depends on the alleged downside.
In which case, the actual downside is more important than this rhetorical
trick.

Because, after all, businesses *should* use programming languages that
make programming easier.

> Safety means that Rust is not as high-performance

It's true that some operations in Rust are checked by default which
are unchecked by default in C++. Array indexing is the typical example.

However, both checked and unchecked indexing are available in both
Rust and C++. The difference is in Rust, to use the unchecked one,
you have to use a named method and an `unsafe` block. This is easy
enough to do in situations where indexing matters.

Most code is not the tight loops in performance-sensitive parts of
performance-sensitive code. Most code by volume is configuration
and other situations where the check is well worth it to prevent
the possibility of memory corruption. Rust does make a less performant
default decision than C++, but it is not that hard to override, and
then you still get all the other benefits of Rust.

> All programming languages have foot-guns.

Some have more than others. In some you run across them more
frequently than others. And in some, they come with a safety.

> Modern C++ fixes the problems with C++.

Modern C++ has all the bad features of pre-modern C++. It
has to, to be compatible with it.

In my experience, it's not enough to have good features. To make promises
about memory safety, or even to have a sane programming ecosystem,
it's also important to not have bad features. And redundant features of
varying quality are often the worst of both worlds.
