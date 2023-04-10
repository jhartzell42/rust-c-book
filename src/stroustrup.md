# Response to Dr. Stroustrup's Memory Safety Comments

The NSA recently published a [Cybersecurity Information
Sheet](https://media.defense.gov/2022/Nov/10/2003112742/-1/-1/0/CSI_SOFTWARE_MEMORY_SAFETY.PDF)
about the importance of memory safety, where they recommended
moving from memory-unsafe programming languages (like C and
C++) to memory-safe ones (like Rust). Dr. Bjarne Stroustrup, the
original creator of C++, has made some
[waves](https://developers.slashdot.org/story/23/01/21/0526236/rust-safety-is-not-superior-to-c-bjarne-stroustrup-says)
with his
[response](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2739r0.pdf).

To be honest, I was disappointed. As a current die-hard Rustacean
and former die-hard C++ programmer, I have thought (and
[blogged](/tags/rust-vs-c++/)) quite a bit about the topic of Rust vs
C++. Unfortunately, I feel that in spite of the exhortation in his title
to "think seriously about safety," Dr. Stroustrup was not in fact thinking
seriously himself. Instead of engaging conceptually with the article,
he seems to have reflexively thrown together some talking points --
some of them very stale -- not realizing that they mostly are not even
relevant to the NSA's Cybersecurity Information Sheet, let alone a
thoughtful rebuttal of it.

Fortunately, he does eventually discuss his own ideas of how to make C++
memory safe -- in the future. If these ideas are implemented well, it
will make C++ a safe programming language as the NSA's Cybersecurity
Information Sheet has defined it. But given that they are currently
just proposals in an early stage, it's unfair of him to expect the NSA
to mention them when advising people on what programming language to
use. C++ has been an unsafe language for a long time. Maybe someday that
will change, but we'll believe it when we actually see it.

But before I discuss that, I'd like to rebut and discuss my disappointment
at the talking points he uses earlier in his response, because I think
they unfairly frame the debate, shield C++ from legitimate and important
criticism, and slander memory-safe programming languages and downplay
memory safety as a concept, even though it's very important.

# Multiple Types of Safety?

One of the most interesting and conceptually relevant points that
Dr. Stroustrup harps on is that memory safety is not the only
type of safety:

> Also, as described, “safe” is limited to memory safety, leaving
> out on the order of a dozen other ways that a language could (and will)
> be used to violate some form of safety and security.

This might technically be true -- it's not entirly clear what other
forms of "safety" he's talking about -- but it's misleading. Memory
unsafety is not just one of a dozen equally important
forms of "unsafety."  Rather, memory unsafety is by far the
biggest source of security vulnerabilities and instability
in memory unsafe programming languages -- estimates as high as [70
percent](https://www.zdnet.com/article/microsoft-70-percent-of-all-security-bugs-are-memory-safety-issues/) in some contexts.

A 70% decrease in security vulnerabilities is worth committing
significant resources towards. Memory safety on its own is worth writing
a Cybersecurity Information Sheet about, and it is the area where C++ has
the most serious deficits. Given that, this feels like a car manufacturer
whose cars do not provide air bags responding to a government advisory
not to buy the C++ cars by saying "What about other types of safety?
By talking just about air bags, the government is clearly not thinking
seriously about safety." Sure, there's other types of safety features
besides air bags (or memory safety), but air bags are still important!

So, Dr. Stroustrup, what about memory safety in C++? Shouldn't C++
have memory safety? Are you saying it's not important, especially when
all of these other programming languages have it?

Of course, he doesn't go into detail about other types of safety,
which is telling. Of course, it's because C++ doesn't really have the
advantage in any of them. For example, Rust also has a lot of mechanisms
for thread safety and type safety, intimately connected with its memory
safety mechanisms, and baked into the design of Rust in a way that would
be next to impossible to retrofit into another programming language.

And, when you read later on about the "safety profiles" in the C++ Core
Guidelines that he makes such a big deal about, most of the focus there
is also about memory safety.

# Petty Irrelevancies

Let's look at some of the other points he makes.

> That specifically and explicitly excludes C and C++ as unsafe.

C++ does not enforce memory safety as a feature of the programming
language. This may change in the future (as Dr. Stroustrup discusses),
but is the current state of things. Dr. Stroustrup tries to downplay this,
but is not convincing.

> As is far too common, it lumps C and C++ into the single category C/C++,
> ignoring 30+ years of progress.

Writing "C/C++" to mean "C and C++" is considered a *faux pas* among C++
programmers, and among C programmers as well, because it is seen
as asserting that these two programming languages are near-identical
when there are in fact major differences between them. By pointing out
that the NSA does this, Dr. Stroustrup is trying to make them look
like they don't know what they're talking about, just because they used
a "/" character instead of the word "and."

He's reading too much into the orthography and the NSA's failure to use
insider *shibboleths* of the programming languages they're trying to
criticize. Outside of the "C" and "C++" communities, "C/C++" is a fairly
common way to refer to the two related programming languages.

And that's the most relevant thing here: C and C++ are indeed related
programming languages, and they have a lot in common: They are both
compiled programming languages with a focus on performance, and they are
(very relevantly) both not particularly focused on guaranteeing memory
safety. C and C++ have a substantial common subset, with many memory
unsafe features that are popular with programmers, perhaps even more
popular because they work similarly in both programming languages. For
the purposes of this document, it's often the features that C and C++
have in common that are the problematic ones, so it makes sense for the
NSA to lump them together.

While there might be 30+ years of divergence between C and C++, none
of C++'s so-called "progress" involved removing memory-unsafe C features
from C++, many of which are still in common use, and many of which still
make memory safety in C++ near intractible. Sure, new features in C++
have been added that (in some but by no means all cases) do not make it
as easy to corrupt memory, but the bad old features are not in any real
way being phased out: They are not guarded by any special opt-in syntax,
nor in many cases do they result in warnings. Given that, the combined
set of features is as strong as its weakest link.

> Unfortunately, much C++ use is also stuck in the distant past, ignoring
> improvements, including ways of dramatically improving safety.

This is a common C++ talking point, but it doesn't help Dr. Stroustrup's
position as much as he thinks it does.

He's trying to talk up how much C++ has improved, especially in the last
11 years -- and it has indeed improved. New ways of writing C++, emphasizing
relatively new features, can indeed result in more reliable C++ code with
less memory corruption.

But unfortunately, this talking point just serves to remind us that these
old memory-unsafe features are still in common use. When someone says
their project is written in Rust, we can guess that it likely uses only
the safe features (including using standard library functions that use
`unsafe` internally -- that truly doesn't count as unsafe), or maybe
uses the unsafe features when absolutely necessary. But when someone
says their project is written in C++, by Dr. Stroustrup's own admission,
there's a high likelihood that it uses old features "stuck in the distant
past, ignoring ... ways of dramatically improving safety." This is also
a reason to avoid C++.

However, I would also contest his claim about these new features.
Memory safety isn't just an absence of memory corruption, but a reliable
method for ensuring the absence of memory corruption. "Using new features"
isn't good enough. Even if using the new features in preference to the
old ones were a guarantee of memory safety -- which it isn't, they're
less memory corrupting but not truly memory safe -- the presence of the
old ones would still cause problems. You would need some mechanism to
ensure that the new features were only used safely, and that the old
features were not used, and no such mechanism exists, at least not in
the programming language itself. Someone who remembers the old features
can always still slip up and use one by accident.

# Static Analysis: Not Good Enough

Dr. Stroustrup points out that he's been working very hard on improving
memory safety in C++, for a very long time:

> After all, I have worked for decades to make it possible to write better,
> safer, and more efficient C++. In particular, the work on the C++ Core
> Guidelines specifically aims at delivering statically guaranteed type-safe
> and resource-safe C++ for people who need that without disrupting code
> bases that can manage without such strong guarantees or introducing
> additional tool chains.

Unfortunately, it's not done. The key word here is, of course, "aims." The
next sentences admit that this feature is not in fact available:

> For example, the Microsoft Visual Studio analyzer and its memory-safety
> profile deliver much of the CG support today and any good static analyzer
> (e.g., Clang tidy, that has some CG support) could be made to completely
> deliver those guarantees....

For memory safety, "much of" is not really good enough, and "could
be made" is practically worthless. Fundamentally, the point is that
memory safety in C++ is a project being actively worked on, and close
to existing. Meanwhile, Rust (and Swift, C#, Java, and others) already
implements memory safety.

It's worse than that, though. What Dr. Stroustrup is trying to downplay
is that this involves using static analyzers, considered separate from
the programming language, something the NSA's original article also
discusses. Theoretically, if a static analyzer could be used to guarantee
memory safety, that could be just as reliable as a programming language
that does it. An engineering team could have a policy that all code must
pass this static analysis before being put into production.

But unfortunately, human nature is more fickle than that. If it's not
built into the programming language, it's going to get skipped. If a
vendor says their software is written in C++, or if an engineer takes a
job in C++, how will they know that these static analyzers will in fact
be used? A programming language that takes memory safety seriously doesn't
provide it as an optional add-on that most people will simply ignore.

# But All The C++ Code!

The end of the last quote provides a common talking point in Rust vs
C++ arguments:

> [Static analyzers] could be made to completely deliver those guarantees
> at a fraction of the cost of a change to a variety of novel “safe”
> languages.

Besides the laughably condescending matter of calling Java (which first
appeared in 1995), C# (first appeared in 2000), and Ruby (first appeared
in 1995) "novel," this is a jab at a common trope that (some immature)
Rust programmers go around demanding that people rewrite their projects
in Rust (please don't do this!), and an attack on the idea that all code
can be written in safe programming languages, given the large body of
existing work in unsafe programming languages.

This is a bit of a straw man in this context. The NSA article that
Stroustrup is responding to addresses that switching existing codebases
might be expensive, even prohibitively so, saying:

> It is not trivial to shift a mature software development infrastructure
> from one computer language to another. Skilled programmers need to be
> trained in a new language and there is an efficiency hit when using a
> new language. Programmers must endure a learning curve and work their
> way through any “newbie” mistakes. While another approach is to
> hire programmers skilled in a memory safe language, they too will have
> their own learning curve for understanding the existing code base and
> the domain in which the software will function.

It then follows this up immediately with an explanation of how tools
like static analyzers can be used as a back-up plan for improving
memory safety in memory unsafe programming languages -- exactly what
Dr. Stroustrup discusses. He's criticizing this NSA document, implying
it is not thinking "seriously," while fundamentally making a point
that they already made for him.

Of course, this is a terrible endorsement of C++. It's far from ideal
to have to use add-on tools to work around a language's flaws. Coming
from Dr. Stroustrup, it reads more like a brag that his programming
language has locked everyone in than a defense of why C++ is good.
Or else, it's an admission that other programming languages should
be used for new projects, and that C++'s fate is now to gradually
fade like the elves from Middle Earth.

But he's also overstating his case. As I mention before, safe programming
languages have existed for a long time. Many programming projects that
in the early 90's would have been done in C or C++ *have* in fact been
done in safe programming languages instead, and according to the NSA's
recommendation, that was a good idea. As computers have gotten faster
and programming language technology has improved, there has been
fewer and fewer reasons to settle for languages like C or C++ that
don't have memory safety as a feature.

When I was a professional C++ programmer as early as 2013, some people
-- even some programmers -- already thought that C++ was a legacy
programming language like COBOL or Fortran. And outside of narrow
niches like systems programming (e.g. web browsers, operating systems,
and lower-level libraries), video games, or high performance programming,
it kind of has become one.  The former application niches of C++ have
been taken over by Java and C#, or more recently by Go.  If you have an
application program written in C++, chances are that it's a relatively
old codebase, or written at a shop that has reasons to write a lot of C++
(such as a high-frequency trading firm).

Now, even C++'s systems niche is under threat, with Rust, a powerful
memory-safe programming language that avoids many of C++'s problems. Now,
even the niches where C++ isn't at all "legacy" have a viable, memory-safe
alternative without a lot of the technical debt that C++ has.  Rust is
even allowed in the Linux kernel, a project that has only previously
accepted C, and whose chief maintainer has always [explicitly hated
C++](http://harmful.cat-v.org/software/c++/linus).

# A Memory-Safe C++

Fortunately, after all of these ill-thought out, tired talking points,
Dr. Stroustrup subtly changes his perspective. After his distractions,
after bashing memory safe programming languages as "novel," bragging about
how C++ is too entrenched to be removable, pretending memory safety is
just one of many equally important safety issues, and promising optional
add-on tools that will eventually be standardized, he finally begins to
tackle the question of how C++ could be made memory safe, in an opt-in
fashion:

> There is not just one definition of “safety”, and we can achieve a
> variety of kinds of safety through a combination of programming styles,
> support libraries, and enforcement through static analysis. P2410r0
> gives a brief summary of the approach. I envision compiler options
> and code annotations for requesting rules to be enforced. The most
> obvious would be to request guaranteed full type-and-resource safety.
> P2687R0 is a start on how the standard can support this, R1 will be more
> specific. Naturally, comments and suggestions are most welcome.
>
> ...
>
> For example, in application domains where performance is the main
> concern, the P2687R0 approach lets you apply the safety guarantees
> only where required and use your favorite tuning techniques where
> needed. Partial adoption of some of the rules (e.g., rules for
> range checking and initialization) is likely to be important. Gradual
> adoption of safety rules and adoption of differing safety rules will be
> important. If for no other reason than the billions of lines of C++ code
> will not magically disappear, and even “safe” code (in any language)
> will have to call traditional C or C++ code or be called by traditional
> code that does not offer specific safety guarantees.

This is a lot closer to what the NSA document actually specifies for
memory safe programming languages than he gives the document credit
for. For example, the document already provides for opting out of memory
safety via annotation, paired with an observation that that will
focus scrutiny on the code that opts out.

Dr. Stroustrup did not need to criticize the document for not thinking
"seriously" to reach this conclusion, but simply acknowledge that it's
true that C++ is not a memory safe programming language yet, but that
based on his work, it might soon become one. Maybe the next version
of the NSA document will endorse using C++, but only if it's C++*ZZ* --
where *ZZ* is some future version of the C++ standard.

I'm glad comments and suggestions are welcome, however, because
I have a huge one.

Opt-in for memory safety is unacceptable, and is almost as bad as having
a separate static analysis tool to enforce safety. Opt-out is fine --
Rust has a way to opt out of memory safety with the `unsafe` keyword, and
this concept is discussed and defended in the NSA's original document. But
the default should be to enforce memory safety unless otherwise specified.

For C++, this means that if these safety features are added in C++*ZZ*,
`--std=c++ZZ` should cause unsafe constructs to be rejected -- and the
C++ standard should require that these constructs be rejected for an
implementation to be a conforming implementation of C++*ZZ*.  Perhaps (but
only perhaps) other command line arguments could be added to override
this constraint on a file-by-file basis. Ideally, a new compiler command
(e.g. `g++ZZ`) should be created for each implementation that defaults
to this stricter behavior.

Parts of the codebase that use legacy features should have to
have at least a file-level annotation that that file is a legacy file --
and then this annotation could gradually be moved to the function level.
As a side benefit, this could also be used to phase out and deprecate
weird points of C++ syntax, similar to the Rust edition system: Anyone
using, for example, 0 literals to mean `nullptr` would have to declare
some sort of a legacy annotation on their file or in their build system.

Only with this sort of opt-out memory-safety system would I consider
C++ a memory safe programming language. I'd be very happy to see a
memory-safe C++. I earnestly hope Dr. Stroustrup is successful in his
endeavors. I'm not holding my breath, though, and in the meantime,
I will continue to use other programming languages, that are already
memory-safe, for my new projects, as will the majority of programmers.

In the meantime, it is unfair for Dr. Stroustrup to call safe programming
languages novelties or to pretend that C++ isn't already far behind the
times on this. This was already an important criticism of C++ decades ago,
when Java first came out in the 90's and was referred to as a "managed
programming language."  This was discussed in detail in my classes when
I was a college student in the late aughts. To read Dr. Stroustrup's writing,
C++ is being criticized by "novel" upstarts when it is well on its way to
getting the feature, but in actuality, the time to act was 1996.
