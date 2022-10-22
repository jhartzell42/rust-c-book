Plan:
* Fill in outline with mostly existing material
    * Write rest, starting with OOP blog post
        * Either post now or wait until it's done
* Convert to Sphinx project
    * Get Google Analytics integration?
* Post, make announcement post
    * Along with OOP post?

Outline:
* Common Goals of Rust and C++ (Old book intro)
* What’s truly up with safety?
    * Address bad faith arguments.
        * Safety only covers what the Rust compiler says is safe
        * Memory leaks were considered a safety issue before, are not now
* Safety and Performance (/posts/unsafe/)
* Exceptions/error handling
    * /posts/multiparadigm/
    * Heavily revise
* RAII 
    * /posts/raii/
    * Lightly revise (a little reframing)
* Moves
    * /posts/cpp-move/
    * Lightly revise
    * Mention syntax!
    * Mention drop flag!
* Borrows
    * Entries
    * Slices
    * Iterators
* Nullability (New post)
* C++ Zombie Features:
    * Headers
    * Pointers vs References
    * Raw pointers vs Smart pointers
    * Arrays
    * Reframe from sources:
        * /posts/hello-rust/
        * A little from /posts/multiparadigm/
    * Function call craziness (New post)
        * Might call out/in-out parameters w pointers so you know can modify
        * Might call with references so you know it's not nullable
        * Both conventions make sense
* No OOP
* What I'm concerned about about Rust
