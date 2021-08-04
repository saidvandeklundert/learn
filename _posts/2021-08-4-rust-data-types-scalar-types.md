---
layout: post
title: Rust scalar types
subtitle: data types
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---

## Scalar types

Scalar types are types representing a single value. In Rust, we can identify the following scalar types:

![Rust scalar types](/learn/img/rust_scalar_types.png "Rust scalar types")  


Creating and printing these values in a program can be done as follows:

```rust
fn main() {
    let ch: char = 'z'; // four bytes, expresses single Unicode Scalar Value
    let b: bool = true; // true or false
    let i: i32 = -111; // 32 bit signed integer, can be positive or negative
    let f: f32 = 3.4; // 32 bit float number
    println!("char: {}\nbool: {}\ni32: {}\nfloat32: {}\n", ch, b, i, f);
}
```

When we `cargo run` the above, we get the following output:

<pre>
char: z
bool: true
i32: -111
float32: 3.4
</pre>

<p style="font-size:10px;">To run the above without having Rust installed, go <a href="https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=22533c3a2fae48a860168a0d49b53f4c">here</a>).</p>

The `let` keyword is used the bind a value to a variable. In the previous example, we explicitly specified a type for every variable. The Rust compiler can also infer the type, so the following would have been valid code as well:

```rust
fn main() {
    let ch = 'z';
    let b = true;
    let i = -111;
    let f = 3.4;
    println!("char: {}\nbool: {}\ni32: {}\nfloat32: {}\n", ch, b, i, f);
}
```

The above is valid, though it is not exactly the same as the first code snippet. When the compiler has to infer the type, it settles on using `i32` for the `i` variable and `f64` for the `f` variable. 


Another thing worth pointing out is that variables are immutable by default. In the following code, we (try) to change the value of `b` from `true` to `false`:

```rust
fn main() {
    let b = true;
    println!("b contains: {}", b);
    b = false;
    println!("b now contains: {}", b);
}
```

When we run the above code, we get the following:

<pre>
error[E0384]: cannot assign twice to immutable variable `b`
 --> src\main.rs:3:5
  |
2 |     let b = true;
  |         -
  |         |
  |         first assignment to `b`
  |         help: consider making this binding mutable: `mut b`
3 |     b = false;
  |     ^^^^^^^^^ cannot assign twice to immutable variable
</pre>

The compiler does not let us run the code. Additionally, it is also nice enough to give us a hint:

<pre>
help: consider making this binding mutable: `mut b`
</pre>

If we want to be able to change the value of a variable, we need to specify that when we bind the value to the variable using the `mut` keyword:

```rust
fn main() {
    let mut b = true;
    println!("b contains: {}", b);
    b = false;
    println!("b now contains: {}", b);
}
```
Now, we can run the code and we get the following:

<pre>
b contains: true
b now contains: false
</pre>



## Copy semantics and move semantics

Oftentimes, you will also read about primitive types. The language doc mentions the entire list of primitive types [here](https://doc.rust-lang.org/std/index.html#primitives). The primitive types include all of the scalar types as well as the following:

![Rust primitive types](/learn/img/rust_primitive_types.png "Rust primitive types")

I will cover most of the primitive types in future posts but I think is worth pointing out is that all primitive types have a few common traits. In Rust, traits are used to define shared behavior. One particularly interesting trait for the primitive types is the `Copy` trait.

Types that have the copy traits are said to have `copy semantics`. What this means is that when you pass them into a function, you pass a copy of that value into that function.

Non-primitive types do not have the `Copy` trait by default. What this means is that they are subject to `move semantics`. This means that when you pass them into a function, you `move` the value into that function. Without going into details (that I am still learning about myself), this pretty much means that when the function scope ends, the value is destroyed.

Though it touches on some advanced concepts that I want to dedicate future posts to, I found it worth mentioning here. This is because it was increadibly confusing for me in the beginning when I started passing different types to functions I had created.

Here is an example function that takes in primitive types:

```rust
fn main() {
    let ch: char = 'z';
    let b: bool = true;
    let i: i32 = -2323;
    let f: f32 = 3.4; 
    // Using primitive types in func is done using 'copy semantics':
    copy_semantics(i, ch, b, f);
    // We can still use the values after the function scope closes:
    println!("char: {}\nbool: {}\ni32: {}\nfloat32: {}\n", ch, b, i, f);
}

fn copy_semantics(i: i32, c: char, b: bool, f: f32) {
    println!(
        "char: {}\nbool: {}\ni32: {}\nfloat32: {}\n",
        i, c, b, f
    );
} 
```

In the code above, we define a few variables. We then pass them to a function. Here, the `Copy` trait ensures that the value is copied into the function. After the function completes, we print the variables we defined to screen. This code will run without any errors. 

The following illustrates the move semantics using a struct:

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

In the above code, we define a struct and pass it into a function. In this case, the value is `moved` into the function. When the function completes, the variable is destroyed and `marie` is no more. Running the above code will give us the following error:

<pre>
cargo run
   Compiling testing v0.1.0 (example)
error[E0425]: cannot find value `person` in this scope
 --> src\main.rs:9:37
  |
9 |     println!("{} is {} years old.", person.name, person.age);
  |                                     ^^^^^^ not found in this scope

error[E0425]: cannot find value `person` in this scope
 --> src\main.rs:9:50
  |
9 |     println!("{} is {} years old.", person.name, person.age);
  |                                                  ^^^^^^ not found in this scope

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0425`.
error: could not compile `testing`
</pre>

Here we see the compiler complain about the fact that `person` is not found in the scope of `main`. This is because the value was moved into the `move_semantics` function. And when that function completed, it was destroyed.


### Wrapping up

As I learn more about Rust, I will dive into more complicated features of the language. My goal is to learn it proper and cover the foundations first. This first post on Rust covered coverd the scalar types and some other basics, like defining variables. For more on scalars and the other primitive types, you can check the docs. [Rust primitives](https://doc.rust-lang.org/std/index.html#primitives) offers pretty comprehensive examples and explanations.

