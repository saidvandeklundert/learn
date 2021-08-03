---
layout: post
title: Rust scalar types
subtitle: Rust data types
cover-img: /assets/img/rust-logo.jpg
thumbnail-img: /assets/img/rust-logo.jpg
share-img: /assets/img/rust-logo.jpg
tags: [rust]
---

## Scalar types

Scalar types are types representing a single value. In Rust, we can identify the following scalar types:

![Rust scalar types](/learn/img/rust_scalar_types.png "Rust scalar types")  


Creating and printing these values in a program can be done as follows:

```rust
fn main() {
    let ch: char = 'z'; // four bytes, expresses single Unicode Scalar Value
    let b: bool = true; // true or false
    let i: i32 = -2323; // 32 bit signed integer, can be positive or negative
    let f: f32 = 3.4; // 32 bit float number
    println!("char: {}\nbool: {}\ni32: {}\nfloat32: {}\n", ch, b, i, f);
}
```

Running this will output the  following:

```
char: z
bool: true
i32: -2323
float32: 3.4
```

## Primitive types

Oftentimes, you will also read about primitive types. The language doc mentions the entire list of primitive types [here](https://doc.rust-lang.org/std/index.html#primitives). 

The primitive types include all of the scalar types as well as the following:

![Rust primitive types](/learn/img/rust_primitive_types.png "Rust primitive types")

I will cover most of the primitive types in other posts, but for now, one thing that I think is worth pointing out is that all primitive types have a few common traits. Traits in Rust are used to define shared behavior, and one particularly interesting trait for the primitive types is the `Copy` trait.

Types that have the copy traits are said to have `copy semantics`. What this means is that when you pass them into a function, you pass a copy of that value into that function.

Non-primitive types do not have the `Copy` trait by default. What this means is that they are subject to `move semantics`. This means that when you pass them into a function, you `move` the value into that function. Without going into details (that I am still learning about myself), this pretty much means that when the function scope ends, the value is destroyed.

Though it touches on some advanced concepts that I want to dedicate future posts to, I found it worth mentioning here because it was so increadibly confusing for me in the beginning. 

Using primitive types in a function:

```rust
fn main() {
    // Using primitive types in func is done using 'copy semantics':
    copy_semantics(i, ch, b, f);
    // We can still use the values after the function scope closes:
    println!("char: {}\nbool: {}\ni32: {}\nfloat32: {}\n", ch, b, i, f);
}

fn copy_semantics(number: i32, character: char, boolean: bool, float: f32) {
    println!(
        "char: {}\nbool: {}\ni32: {}\nfloat32: {}\n",
        number, character, boolean, float
    );
} 
```


Illustrating move semantics using a struct:

```rust
fn main() {
    // All non-primitive types move-semantics by default:
    let marie = Person {
        name: String::from("marie"),
        age: 2,
    };
    move_semantics(marie);
    // Next line is illegal because a move happend when we passed marie to a function:
    println!("{} is {} years old.", person.name, person.age);
}


fn move_semantics(person: Person) {
    println!("{} is {} years old.", person.name, person.age);
}

#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}
```


Running the above gives us the following:

<pre>
   Compiling b1 v0.1.0 (C:\dev-container\LearningRust\b\b1)
error[E0425]: cannot find value `person` in this scope
  --> src\main.rs:41:37
   |
41 |     println!("{} is {} years old.", person.name, person.age);
   |                                     ^^^^^^ not found in this scope

error[E0425]: cannot find value `person` in this scope
  --> src\main.rs:41:50
   |
41 |     println!("{} is {} years old.", person.name, person.age);
   |                                                  ^^^^^^ not found in this scope

error: aborting due to 2 previous errors
</pre>