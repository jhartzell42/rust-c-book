* C++ vs Rust Book (don't actually PUBLISH publish, just maintain online)
    * Plan:
        * Move outline into book sections
        * Draw also from blog posts
        * Write new content
        * Develop in the open
        * https://twitter.com/pcwalton/status/1539112080590217217

    * Part I: Memory/Resource Management
        * Memory: What is C++ trying to solve?
            * C: Manual Memory Management
                * Corresponds closely to how computers work in raw way
                * Heap to pass large objects up or down stack
                    * Or to allocate variable sized objects within functions
                    * But heap is hard to manage manually
                    * Only sometimes used
                * Problem:
                    * Leaks
                    * Invalidations
                        * Dangling references
                        * Double-frees
                        * Or frees of stack/static objects
            * Normal solution:
                * Everything on heap
                    * Prevents requiring invalidation when functions return
                * GC
                    * Prevents misplacing frees to cause invalidations
                * This is slower in both ways
        * RAII: C++'s solution
            * How RAII works
            * Fixes in C++
                * Leaks
                * Double-frees
                * Frees of stack/static objects
            * Does not fix in C++
                * Dangling references
                * Stubborn, adversarial intentional abuse
                    * Also not fixed in Rust
            * Does fix more in Rust
                * Lifetimes from Cyclone
                    * Discuss borrows
        * Moves: An Essential Element
            * Pre-move C++
                * Manual constructor writing
                * Compiler would cascade constructor/destructor calls
                    * But not really generate them for you
                    * Because `unique_ptr` was not possible
                * std::vector<std::string> would be bad
                    * Lots of copies
                        * Code we dont think twice about today
                            * Asymptotic differences
                    * Requires pointers
                        * And then manual memory-management code
            * Post-move C++
                * This is fixed!
                * By-value semantics are possible
            * Two problems
                * Moves invalidate pointers into old values
                    * You can move even if someone else is referencing you
                        * Or into you
                        * This may or may not be OK
                            * But is chaotic and can result in UB
                * Non-destructive moves
                    * Forced by legacy of language
                    * Means `unique_ptr` *must be* nullable
                    * Very strange semantics, officially
                        * Move blog post
            * Destructive moves a la Rust
                * More sane semantics
                * Must have no outstanding borrows in order to move
        * Memory safety
            * Bring in from being fair about memory safety
    * Part II: OOP and Compile-Time Polymorphism
    * Part III: Legacy
        * C legacy
            * Header files
            * Null
        * C++ legacy
            * OOP
            * Exceptions

Old material:
* Rust vs C++: Function signatures in Rust and C++
    * Syntactic salt
        * Need to tell what's happening at the call site
        * Rust *could* auto-deref and auto-ref
        * Rust could automatically add ? and `async`
    * C++: Can't tell what's going to happen from call site
        * Taking by reference vs by copy is a huge difference
        * std::move does not guarantee move
    * What even are rvalue references?
        * They are the same as lvalue references
        * They constitute a heuristic to determine which function to call
    * ... what else?
* Move semantics part II:
    * Mention drop flag
    * rvalue vs lvalue overloading
        * You sometimes mean the other one
            * `const T &`
            * `std::stay`
* C as portable assembly?
    * The importance of a C ABI
        * Programmers' PC Sourcebook
        * Seeing the register assignments for EACH call
        * Quite tedious

Old:
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
