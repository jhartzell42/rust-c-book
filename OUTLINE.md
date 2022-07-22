* C++ vs Rust Book (don't actually PUBLISH publish, just maintain online)
    * "C++ to Rust"
    * Plan:
        * Draw from blog posts
            * Reorganize into this outline
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
