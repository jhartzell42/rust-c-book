# RAII: GC without GC

C's memory management system is, true to its goals, very similar
to how you manage memory in an assembly language program:
you make a call to allocate memory, and another call to free it.
It's your responsibility as the programmer to remember to balance
these things and prevent leaks.

For a high-level language, this is unacceptable. The information of when
to free the memory is already implied by the program structure. Why force
the user to specify redundant information? It's tedious and error-prone.

Most programming languages use garbage collection or reference counting
to accomplish this, but both of those options have run-time costs, making
the performance worse than that of the comparable C or assembly program.
So another solution must be found.

Rust here borrows the solution from C++: RAII.

In general, Rust wouldn't be able to be the language it is without the
contributions and hard work of the C++ community all this time. So
many important Rust concepts have come from C++. But this particular
one, RAII, is part of the essential concept of how Rust works. Rust
would be inconceivable without this idea, which C++ invented and
matured.

## How RAII Works
