# Immutability

This means data cannot be altered once created

In Elixir all values are **immutable** - once a variable references a value, it will always reference the same values until you rebind the variable.

Elixir knows that data is immutable, it can reuse it, in part or as a whole, when building new structures.

Elixir uses a lot of processes each having its own heap. The data in your application is divided up between the processes, so each individual heap is much smaller than would have been the case if all the data had been in a single heap. 

As a result, garbage collection runs faster. When a process terminates before its heap becomes full, all of its data is discarded - no garbage collection required.
