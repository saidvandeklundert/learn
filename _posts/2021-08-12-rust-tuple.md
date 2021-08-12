---
layout: post
title: Rust tuple
subtitle: data types
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---


The Rust `tuple` is placed in several categories of types. 

The tuple is a `primitive type`. Among others, this means tuples have the `Copy` trait implemented, making them pass by value.

Together with the array, the `tuple` also falls into the category of `primitive compound types`. These types group multiple values into a single type. Though, unlike the array, the values in the tuple can be of the same type or they can be of different types.

Additionally, together with arrays and slices, the tuple is a `sequence type`. The tuple can be used to hold a list of values. Though, unlike slices and arrays, tuples do not support iteration.



### Basic tuple operations

We can define a tuple with or without type annotations:

```rust
let tup = ("hello world", true, 220);
let tuple: (&str, bool, u8) = ("hello world", true, 220);
```

Individual values inside a tuple are accessed by their index:

```rust
println!("{:?}\n{:?}\n{:?}", tuple.0, tuple.1, tuple.2);
/*
"hello world"
true
220
*/
```

Tuples can also be 'destructured' or 'unpacked' like so:

```rust
let (a, b, c) = tuple;
dbg!(a, b, c);
/*
[src\main.rs:10] a = "hello world"
[src\main.rs:10] b = true
[src\main.rs:10] c = 220
/*
```

Oftentimes, you will also see the tuple being used to express 'nothing'. In such cases, it is an empty tuple that is referred to as 'unit':

```rust
();
```

Tuples can also be useful in case you want to return multiple values from a single function.

```rust
fn some_operation(input: i32) -> (&'static str, bool, i32) {
    if input < 10 {
        return ("not good", false, 1);
    } else {
        return ("good", true, 0);
    }
}
let result = some_operation(1);
println!("{:?}", result);
/*
("not good", false, 1)
*/
let (msg, value, exit_code) = some_operation(11);
println!("{:?}", (msg, value, exit_code));
/*
("good", true, 0)
*/
```

Tuples can be compared:

```rust
let tup_1 = ("hello world", true, 220);
let tup_2 = ("hello world", true, 220);
let tup_3 = ("hello", false, 220);
assert_eq!(tup_1, tup_2); // this is fine
assert_eq!(tup_1, tup_3); // thread 'main' panicked at 'assertion failed: `(left == right)

if tup_1 == tup_2 {
    println!("equal")
} else {
    println!("not equal")
}
/*
equal
*/
```

Tuples, like arrays, cannot grow or shrink in size. If you need a type that can grow or shrink, you are probably in need of a vector. 


