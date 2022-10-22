Plan:
* Revise blog posts that were literally copied in
    * Try to make it a little more coherent
    * Replace references to blog series with references to book :-)
* Write damn thing
    * Mostly with existing material
* Convert to Sphinx project
    * Get Google Analytics integration?
* Post, make announcement post
    * Along with OOP post?

Specifically:
* Common Goals of Rust and C++ (Old book intro)
    * Split old book intro into two docs
    * Objections doc will be important, can be maintained separately
        * Might grow into its own thing
        * Address more bad faith arguments.
            * https://lwn.net/Articles/907685/
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
