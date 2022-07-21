* C++ vs Rust Book (don't actually PUBLISH publish, just maintain online)
    * "C++ to Rust"
    * Plan:
        * Draw from blog posts
            * Reorganize into this outline
        * Write new content
        * Develop in the open
        * https://twitter.com/pcwalton/status/1539112080590217217
    * Rust vs C++
        * C++ tackles a lot of important problems
            * List goals of C++ in some detail
                * C is portable assembly language
                     * But adds few abstractions
                * C++ goal:
                    * Zero-cost abstractions
                    * Essential abstractions at minimal cost
                * That is:
                    * C levels of control
                    * Automatic memory management
                        * Without RC or GC
                        * Fancy types without paying for it
                    * Templates
                * Follow overall outline of book as well
        * C++ nevertheless has problems
            * C++ is very old
                * Lots of C legacy
                * Lots of C++ legacy too
                    * OOP is legacy
                    * Refs vs pointers is legacy
                * A new PL may just be needed every once in a while
                * Rust can learn the lessons without being tied to the legacy
            * UB is very hard to deal with
                * Rust doesn't eliminate it
                    * Still has C levels of control
                * But does give tools for encapsulating/managing it
                    * Not every line of code should have to grapple with it
                    * Trusted safe abstractions
        * Some common objections to the Rust fandom
            * Utopianism in Programming Languages
                * Clearly exists
                * Haskell
                    * To Kata Haskellen Evangelion
                * Rust
                * However, the opposite is also false
                    * It's not the case that all tools are equal
                    * Shouldn't blame things on laziness learning the tool
                        * Or learning the many rules around it
                        * This might make sense for individuals
                            * But very bad corporate policy
            * Young enthusiastic people
                * But Torvalds, Cantrill...
                * Many have disliked C++ the whole time, like Rust
            * Tu quoque: Rust will have these problems in 40 years as well
                * C++ will have even more then! And so?
            * Not 100% memory safe
                * Not the goal
        * Meta-commentary on PL debates
            * "You're just being lazy"
                * Used in a variety of contexts
                    * As a rebuttal to claims that PLs are too complicated
                    * Or that you have to use best practices
                    * Or that there's undefined behavior
                * Unnecessary extra work is bad
                    * It would be a better argument if there was a pay-off
                    * But often, you can make it easier without making it worse
                * Making programmers work harder is actually bad
                    * Making programmers work harder makes code worse
                    * Why should the programming language get in the way of actual work??
                        * It distracts them from actually finding the problem
                        * Can make bugs harder to see because of cognitive load
                    * This isn't a manliness competition
                * Programming is engineering, not art
            * "A tool for every job"
                * Different tools for different jobs, sure
                * Some tools are just unilaterally worse than others
                * These tools are all free! What are you complaining about?
            * "It's a matter of personal preference"
                * Most programming is done for a job
                * Programmer comfort and familiarity is important
                * Other things are too
            * "Why care this much about programming language choice?"
                * The stakes actually are high
                * Security vulnerabilities
            * You just have to use the best practices of that PL
                * The more good practices can be enforced, the better
                    * Many programming languages create constructs that are hard to use
                        * To no upshot
                * Best practices without warnings or errors are bad
                    * Sometimes necessary
                    * But often, there are too many
            * All programming languages have dangerous constructs
                * That doesn't mean they all are affected the same amount
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
    * Part II: Compile-Time Polymorphism
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
