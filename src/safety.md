# Safety and Performance

There is a persistent and persnickety little argument that I want to
talk specifically about. This argument is really persuasive on its face,
and so I think it deserves some attention -- especially since I am guilty
of having used this argument myself, many years ago when I still worked
at an HFT firm, to claim that C++ had a niche that Rust wasn't ready for.
I've also seen it a few times in a row in the wild, and it's made me
so emotional that I simply had to write this, and as a result, it's
a little more emotional than some of the other posts.

In this argument, array indexing stands in for a number of little
features. But -- I've seen array indexing cited so often as a canonical
example that I feel compelled to address it directly!

The argument goes like this:

> In Rust, array accesses are checked. Every
> time you write `arr[i]`, there is an extra prepended
> `if i >= arr.len() { panic!(..) }`. As you can see, that is more code,
> and worse, a run-time check. And while the optimizer might eliminate
> it, or the branch predictor may well predict it right every time,
> the extra code bloat and possible run-time check, is just
> unacceptable in [insert field here (I used HFT)], where every
> nanosecond matters. And until some acceptable solution is found to this,
> I just don't see Rust making it in [insert field].

When I made this argument, to a group of programming-language academics,
the defenders of Rust countered with a number of points, all of which
accepted the basic premise:

* Do I really need those extra nanoseconds? Yes.
* Is it really too much of a price to pay for all that extra
  safety? Yes.
* Do I really distrust the optimizer that much? Yes. If only
  Rust had a way to do optimizer assertions, a way to
  statically verify that the [panic had been optimized
  out](https://github.com/dtolnay/no-panic).
* Would dependent typing on integer values help? Yes. That sounds
  very promising. I think Rust will get there someday, but for right
  now we must use C++.

Now that I know more about Rust I'm happy to tell you that I was
completely off base. I wasn't off base about the performance considerations,
or the unacceptability of even the slightest risk of a run-time check.
I was off base about an even more basic premise: that Rust uses checked
array indexing, whereas C++ uses unchecked array indexing.

But wait! Isn't that the whole point? Doesn't C++ avoid checking everything,
to make sure all abstractions are zero-cost, to be blazing fast? Doesn't
Rust, while trying for performance, in the end always concede to the
demands of safety?

Well, let's look at the APIs in question. C++ apologists are always
saying to use the modern C++ features from C++11 and later,
rather than the more C-like "old style" C++ features, so on the
C++ side let's take a look at the
[documentation](https://en.cppreference.com/w/cpp/container/array)
for `std::array`, introduced in C++11.

Here we see two indexing methods. The first one, `at`, is bounds
checked and will throw an exception if the index is out of bounds,
whereas the second one, `operator[]`, is not, and will instead exhibit
undefined behavior of a very difficult-to-debug nature. It looks like C++
actually believes in free choice here, leaving the choice of method up
to the user. Not quite what we supposed, but the important part is that
unchecked indexing is available, so so far the argument can still stand.

Now let's look at Rust. Rust arrays and vectors can also be used with
methods from [slice](https://doc.rust-lang.org/std/primitive.slice.html),
as can slices, so the slice documentation is the best place to look.
And looking there, we immediately see -- drum roll please -- 4 methods. We
see `get` and `get_mut`, which are checked, and right underneath them,
in alphabetical order, `get_unchecked` and `get_unchecked_mut`, which
are not.

To review, where do Rust and C++, these programming languages with
their vastly different philosophies, Rust for the cautious, C++
for the fast and bold, stand? In the exact same place. Both programming
languages have both checked and unchecked indexing.

Let me say that again. This is the talking point form, what to say if you
need something quick to say, if you're ever debating programming languages
on a political-style talk show (or at a party or even a job interview):

> In both Rust and C++, there is a method for checked array indexing,
> and a method for unchecked array indexing. The languages actually
> agree on this issue. They only disagree about which version gets to
> be spelled with brackets.

The difference is simply in the default, which one gets
that old fashioned `arr[index]` syntax. And even that [can be
changed](https://docs.rs/unchecked-index/latest/unchecked_index/).
Even if the C++ default were superior -- and, as I will argue later,
it is not -- this is surely a minor issue. After all, don't we normally
use our fancy `for x in arr` syntax in Rust? This issue is just so small
as to be unlikely to be a deciding factor in what programming language
is better, even if we're in a special application domain where every
nanosecond matters.

## The Unsafe Keyword

So that's a wrap folks. We can all go home, and none of
us will ever see this extremely silly argument on the
Internet or in person again. It's just a misunderstanding,
the person making it was simply misinformed, and all it will
take is a link to this blog post -- or the [relevant method in the
docs](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked)
to set them straight.

But wait! The C++ apologists are still talking! What are they saying?
How have they not been completely flummoxed? They're pointing
at that method, chanting a word like a slogan at a protest march.
I can't quite make it out -- what it is it?

Oh. They're chanting `unsafe`. And credit where credit is due:
it's very difficult to chant in a monospace font.

Well, that is easy to respond with! The nerve, that C++ programmers would
call our unchecked array indexing method unsafe. For one, all unchecked
array indexing methods are unsafe: that's what unchecked means. If it
were safe, it would be at least statically checked. For another, isn't
this the pot calling the kettle black? Isn't C++ all about unsafety,
so much that C++ programmers don't even mark their unsafe code regions
becasue it all is, or their unsafe functions because they all are?

"But isn't that the whole point of Rust?" they cry. "If you have to
use `unsafe` to write good Rust, then Rust isn't a safe language
after all! It's a cute effort, but it's failing at its purpose!
Might as well use C++ like a [Real Programmer](http://www.catb.org/jargon/html/R/Real-Programmer.html)!"

This, my friends, is a
[straw](https://www.smbc-comics.com/comic/logical-fallacies)
[man](https://www.smbc-comics.com/comic/straw-men). No, the point
of Rust and specifically Rust's memory safety features is not
to create an entirely safe programming language that can't be
circumvented in any circumstance; you must be thinking of Sing#,
the programming language for Microsoft's defunct [research
OS](https://www.microsoft.com/en-us/research/project/singularity/).

Let me be abundantly clear: The point of memory safety, the unsafe
keyword, and friends in Rust is not to completely enforce memory safety,
to make it impossible for the programmer to do anything they want to
with the computer, even if they can't prove to the compiler that it's OK.
In fact, the point of memory safety isn't to make it *impossible* to do
anything at all -- it's to make it *possible* to reason about the program.

The premise of Rust is that the vast majority of code in a systems program
doesn't need to be unsafe, and so it might as well be safe. People used
to believe that you needed garbage collection for safety, but Rust
proved that you could use lifetimes to still get safety without that
performance cost. Now that we're there, why worry about null pointers?
Why not tell the compiler which things can be null, and which things
can't, so the compiler can check for you whether you're handling nulls
correctly? I've programmed C++ professionally for years without such a
feature.  You'd better believe I would have totally annotated the crap
out of the code so the compiler could've caught them ahead of time.

Sometimes, C++ apologists cite valgrind. I've had codebases where
I tried to use `valgrind`. Unfortunately, there was so much undefined
behavior and memory leaks already caked into this project that new
ones were simply impossible to see among all the noise. An army
of junior engineers was at some point required to clean this up
when finally the hierarcy decided that "valgrind" was something we
might want to be able to use in the future.

And a lot of those undefined behaviors were ticking time bombs.
Certainly, this codebase had its issues.  I've had memory corruption
issues where I poured over every line of code that I wrote, over and
over again, finding nothing. Ultimately, I learned that the issue was in
framework code -- code written by my boss's boss.  The code was untested,
and written extremely poorly, and had rotted, so that it didn't work
at all. In Rust, I might have had some idea that my code -- which in
Rust would have all been able to be "safe" -- couldn't possibly be the
source of the problem. Maybe my humble assumption that my code was to
blame would be a little less tenable.

If I wanted a language that was always safe, at the time I knew Java
or Python existed. Some companies even do finance in Java, for exactly
that reason. But sometimes you still need that extra bit of performance.
`unsafe` is sometimes necessary.

But given what gains safe Rust has made in predictable performance,
it's not as necessary as it used to be. The majority of the code I
wrote then could've been written in safe Rust, and not lost a single
clock cycle. The parts that needed to be unsafe could have been
isolated, delegated to specific sections, wrapped in
abstract data types, perhaps entrusted to a specific team.

And even then, I'm sure we would have been debugging memory
corruption issues. But we'd know where to look. We'd know where to
throw the tests. And we'd have saved programmer-years of time,
days if not months of my life.

Now, I'm proud of my C++ skills. There is some part of me that wishes
that C++ was better than Rust, that all that time getting better at
debugging memory corruption wasn't dedicated to a skill that is
becoming obsolescent through better technology. And to be honest, that's
part of why I dismissed Rust as a candidate for HFT programming
languages.

But it's possible to be proud of a skill that is also becoming obsolete.
And I am trying to replace it with a new skill to be proud of -- writing
Rust as performant as idiomatic C++, or even more performant, while
reaching for the `unsafe` keyword rarely and modularly. I think it's truly
possible, for where it's relevant.

Now I must turn to a subset of C++ apologists, who write using "modern
C++" which is "very safe now" and experience therefore no memory corruption
issues. To them I say, you are not doing high performance programming.
If you were, you'd have to do some wonky things with pointers to spell
the bespoke high-performance constructs you'd need.

There is indeed a safe subset of C++ heavy with modern features. If
you are disciplined and keep your programming in that realm, you can
avoid memory corruption mostly. But first, this safe subset covers fewer
high-performance features than Rust. I've read some of this code and its
idioms: It's full of `shared_ptr`s not to share ownership but simply to
avoid types that might be invalidated. It ironically leans on reference
counting more than idiomatic Rust. This is among other, similar problems.

Let me be clear: First off, instead of keeping in your brain which
features are "modern" and which are "edgy," why not have a distinction where
it's well-marked? Second off, if you are writing entirely in this safe
subset of C++, you can get much better performance instead out of the
safe subset of Rust. You have no right to complain about Rust's safety
trade-offs, as you're using a worse set, where you get no safety
promises from the compiler and none of Rust's surprising safe performance.

Rust's safe and "slow" subset is faster than C++'s while still being,
obviously, safer. Rust's unsafe subset is better factored and better
distinguished. Comparing apples to apples, Rust is better programming
language for extracting performance out of LLVM, because you'll be able
to code more often without fear, and with very focussed fear when you
do feel it.

A tool is even more useful if you can adjust it. The defenders
of C++ talk about choosing trade-offs, but really, Rust offers both
trade-offs. Mark your code as `unsafe` and convince yourself of its
safety manually, or rely on programming language features. It's up to
you, on a function-by-function, even block-by-block, basis.  In C++,
if you have a problem, every line of code is suspect; you simply
can't opt in to safety, but in Rust, for where you don't need the
performance of unchecked indexing and other unsafe features, you can
relax about the possibility of going [bankrupt due to inadvertent memory
reinterpretation](https://en.wikipedia.org/wiki/Knight_Capital_Group) --
and how do I wish my NDA permitted me to talk about consequences at my own
previous jobs!

And for where you do need to use `unsafe`, you can make sure your
debugging and overthinking efforts are well-directed, for the few places
in a large project you need it.

## Unchecked Indices

This has gotten a little far from the original question. Should array
indices be checked? Well, let me be clear about two facts that are both true,
but in tension with each other:

* Unchecked array indexing is sometimes absolutely necessary
* Unchecked array indexing is an edge-case feature, which you
  normally don't want.

If unchecked array indexing was unavailable in Rust, that would be a bug.
What is not a bug is making it inconvenient. C++ programmers probably
should be using `at` instead of `operator[]` more often. But in C++,
what would it gain? There's so many unsafe features, what's the cost
of one more?

But in Rust, where so much code can be written that's completely safe,
defaulting to the safe version makes more sense. Lack of safety is a cost
too, and Rust makes that cost explicit. Isn't that the goal of C++, making
costs explicit?

Let's look at situations where you are indexing memory. First off, most
of them I saw were in old C-style `for`-loops, where you loop over an
index rather than using iterators directly with a collection. Both Rust
and C++ have safe versions of `for` that loop over collections with
iterators, and those use the same check for the loop as they do for
bounds, so those are easy enough to address. Nevertheless, I think that
a lot of the noise about checked vs. unchecked array accesses comes from
people who use indexing for their `for`-loops instead of iterators,
and therefore mistakenly think that array indexing in general is a
far more common operation than it is.

For the remaining situations, most are implementing either gnarly
business logic, or a subtle, fast algorithm.

If it's gnarly business logic, in my experience, it's usually at config
time -- along with a good third to half to even more of the code in a
complicated production system.

What do I mean by config time? A running high-performance system, whether
optimized for latency or throughput, has a bunch of data structures
organized just so, a lot of threads set up just right to move data
between them in the perfect rhythm, and a lot of the work is in arranging
them. That work is generally not performance-sensitive, but often has
to be in the same programming language as the performance-intensive stuff.

Config-time is, depending on how you look at it, less of a thing or the
entire thing in a programming language like Python.  Python basically
exists to do config-time programming for performance-intensive code put
in very comprehensive "libraries" written in C or C++. But in C++, where
you have a constructor that runs only once or a few times at first,
and other methods related to it, in the same programming language as the
money-making do-it part, you have to really adjust programming style
between them.

Config-time is obviously when you read the configuration files.
It's where you open the relevant files. It's where you call `socket`
and `bind` and `listen` on your listening port. It's where you spin up
your worker threads, and make computations on how many worker threads
there are. It's where you construct your objects and your object pools.
It's where you memory map your log file. It's where you set your process
priorities. It's where you recursively call the constructors and `init`
functions of every object in your overwrought OOP hierarchy.

There is no need to sacrifice safety for performance at
config time -- especially since undefined behavior might lie latent and
destabilize the system once it's actually up and running. If you do
an unchecked array access at config time, you might put garbage data in
an important field, maybe one that determines how much money you're willing
to risk that day or how many of a thing to buy. And for what? To save a few
nanoseconds before your process has even "gone live"?

So, when do you truly need unchecked array accesses?  If it's a subtle
fast algorithm, probably deep in an inner loop, you should probably be
wrapping it in an abstraction anyway. The code that actually executes the
algorithm should be separate from the business logic, so that programmers
trying to maintain the business logic don't accidentally break it. And
that's exactly where it makes the most sense to use `unsafe` -- when
implementing a special algorithm. Maybe the proof that the index is
within bounds relies upon some number theory the compiler was never going
to understand without its own proof engine: great! You should probably
be explaining that in a comment in C++ anyway, and so the conventional
comment that goes with the `unsafe` block in Rust is a perfect place to
explain it.

But maybe I'm wrong about all of this. Maybe your experience hasn't
matched mine.  Maybe your particular application needs to make unchecked
array accesses a lot, needs them to be unchecked, and needs them littered
all over the codebase. I raise my eyebrows at you, suspect you need more
iterators and perhaps other abstractions, and wonder what problem you're
trying to solve. But even if you're absolutely right, I think it's still
a better idea to write Rust littered with `unsafe` every time you index
an array, than to write C++.

Because, as I keep emphasizing, Rust is still a better unsafe programming
language than C++. It would be better than C++ even if safety weren't
a feature.

## Post-Script: Some Perspective for the New Rustacean

I understand where this straw man argument comes from. The word
`unsafe` is scary, and advice, especially aimed at people coming
from safe languages like Python and Javascript, is to avoid `unsafe`
features while learning. And while I think adding `unsafe` to production
code should only be done once you've exhausted safe possibilities -- which
requires full understanding of safe possibilities -- this advice can
feel overbearing for a transitioning C++ programmer, especially when
it is immediately obvious that the safe features are very constrained
and can't literally do everything.

For that good-faith recovering C++ programmer, new to Rust: You're
right. The safe subset isn't enough to do everything you want to
do. And when it doesn't, that doesn't mean it failed. Its goal is to
make unsafe code rare, not non-existent.  But it might surprise you
how rarely you truly *need* `unsafe`.  And a good resource for you
might be, as it was for me, the excellent [Learn Rust the Dangerous
Way](http://cliffle.com/p/dangerust/) by Cliff L. Biffle.

For what it's worth, however, this criticism of Rust in general is often
levelled either in bad faith, or from a misunderstanding of what the
`unsafe` keyword is for. For all the philosophical discussion of what
`unsafe` truly means -- and how it interacts with the surrounding
module and encapsulation/privacy boundaries -- as well as principled
conventions for using it, please see the
[Rustonomicon](https://doc.rust-lang.org/nomicon/), the canonical
book on unsafe Rust, the same way [the book](https://doc.rust-lang.org/book/)
is canonical for introducing Rust.

Other criticisms of Rust from an HFT or low-latency point of view
are more relevant. Most specifically, `gcc` and `icc` are much better
compilers for those use cases -- empirically -- than is LLVM. Also,
the large codebases existing in C++ are often tested and contain
thousands upon thousands of programmer-years of optimizations and
bugfixes, where even small compiler upgrades are scrutinized closely
for performance regressions. Migrating to another programming language
from that starting point would be prohibitively expensive.

None of which is to say that if Rust gradually replaced C++ altogether,
eventually such ultra-optimizing compilers and ultra-optimized codebases
wouldn't start appearing in Rust. I hope to see that day within my
lifetime.
