Plan:
* Revise blog posts that were literally copied in
    * Try to make it a little more coherent
    * Replace references to blog series with references to book :-)
* Write empty pages
* Convert to Sphinx project
    * Get Google Analytics integration?
* Post, make announcement post
    * Along with OOP post?

Specifically:
* `intro.md`
    * Post as blog post in blog post form?
* `complaints.md`
    * Address more bad faith arguments.
        * https://lwn.net/Articles/907685/
        * Safety only covers what the Rust compiler says is safe
        * Memory leaks were considered a safety issue before, are not now
* `behind.md`
    * Write me!
        * Compiler support
        * Library support
        * Platform support
            * Though "inside the Linux kernel is yes Rust, no C++"
        * Unsafe is hard
            * Undefined behavior is still not so rigorously defined
* `raii.md`
    * Remove some of the caveats
    * Talk with more authority about performance
    * Emphasize more that ownership tends to be the structure in GC langs too
* `moves.md`
    * Read whole thing for revisions
    * Mention weird reference syntax/function call weirdness!
    * Mention drop flag!
* `borrows.md`
* `entries.md`
    * Re-read
    * Emphasize a little more the ties to the book?
* `safety.md`
    * Re-read
    * Probably move some less relevant sections to other chapters
* `safe-limits.md`
    * Write
        * Directly address criticisms of safety
            * Safety isn't proof
* `null.md`
    * Write
        * What is null?
        * Billion-dollar mistake
        * Why Rust still has null for raw pointers?
            * Because it is a feature of the machine
            * But it is very easy to abstract away
* `result.md`
    * Write
    * This can pull in from `posts/multiparadigm.md`
* `oop.md`
* `enums.md`
    * Write
        * Discuss unions
        * Discuss run-time polymorphism with small class hierarchies
* `traits-rt.md`
* `traits-compile.md`
    * Write
    * This can pull in from `posts/multiparadigm.md`
* `syntax.md`
    * Take from `posts/hello-rust.md`
* `signatures.md`
    * Write
    * Just have fun!
        * Go on that unhinged rant
        * Demonstrate how crazed it is
    * Specifically about out params
        * Might call out/in-out parameters w pointers so you know can modify
        * Might call with references so you know it's not nullable
        * Both & and * conventions make sense
* `overloading.md`
    * Principled stance against overloading
        * What it's for is compile-time ifs, must use traits
        * What else it's for is implicit interfaces, use traits
        * Why do people like this feature besides??
* `headers.md`
    * Take from `posts/hello-rust.md`
* `ptrs.md`
    * Take from `posts/hello-rust.md`
