

>

https://doc.rust-lang.org/book/ch04-03-slices.html

Slice does not have ownership. 



Slices let you reference a contiguous sequence of elements in a collection or the whole collection. 



A slice can be thought of as a pointer to the first element of a collection (like an array or a vector), together with a count that indicates how many elements are accessible in the slice.

Reference to an array is a thin pointer and a slice is a fat pointer.

https://stackoverflow.com/questions/57754901/what-is-a-fat-pointer


Slice and reference slice: https://stackoverflow.com/questions/61151041/i-dont-understand-the-difference-between-a-slice-and-reference

----

In Rust, memory is managed through a system of ownership. This is a set of rules that the compiler checks at compile time. The ownership features do not have any runtime costs and allow programs to run safely without the use of a garbage collector or programmers freeing memory manually.

Important ownership rules (from the book):
- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

In Rust, memory is automatically returned once the variable that owns it goes out of scope. When values go out of scope, Rust calls a function for us. This function is called `drop`. This executes the destructor for a type and this happens at the end of a scope (at the curly braces).

Deallocating at the end of a scope is called `RAII`: Resource Acquisition Is Initialization.

In Rust, primitive types have the `Copy` trait. Types that have the copy traits are said to have `copy semantics`. This means there value is copied when they are passed into a function.,
