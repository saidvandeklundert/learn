---
layout: post
title: Rust Option and Result
subtitle: dealing with the Option and Result enum
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---

## why is Option and Result important


## Introduction


In order to understand the `Option` and the `Result`, it is important to understand the following:
- the enum in Rust
- matching enum variants
- the Rust prelude

### The Rust Enum

In Rust, the the `Option` as well as the `Result` are _enumerations_, also referred to as _enums_. The enum in Rust is quite flexible, as it can contain tuples, structs and more. This allows you to create quite an elaborate enum. The Option and the Result are pretty straightforward though. 

Let's first look at an example enum:

```rust
enum Example {
    This,
    That,
}

let this = Example::This;
let that = Example::That;
```

In the above, we define an enum called `Example`. This enum has 2 variants called `This` and `That`. Next, we create 2 instances of the enum, variables `this` and `that`. Both are created with their own variant. It is important to note that an instance of an enum is always 1 of the variants. When you use a field struct, you can define struct with all it's possible fields. An enum is different because you assign only one of the fields (called variants).

### Displaying the enum variants

By default, the enum variants is not something you can print to screen. Let's bring `strum_macros` into scope. This makes it easy to derive 'Display' on the enum we defined, which we do using `#[derive(Display)]` above the enum definition:

```rust
use strum_macros::Display;

#[derive(Display)]
enum Example {
    This,
    That,
}

let this = Example::This;
let that = Example::That;

println!("Example::This contains: {}", this);
println!("Example::That contains: {}", that);
```

Now, we can use print to display the enum variant values to screen:

<pre>
Example::This contains: This
Example::That contains: That
</pre>

### Matching enum variants

Using the `match` keyword, we can do pattern matching on enums. The following function takes the Example enum as an argument:

```rust
fn mather(x: Example) {
    match x {
        Example::This => println!("We got This."),
        Example::That => println!("We got That."),
    }
}
```

We can pass it a value of the Example enum, and it will print something based on the variant of the enum value:

```rust
matcher(Example::This);
matcher(Example::That);
```

Running the above will print the following:

<pre>
We got This.
We got That
</pre>

### The Rust prelude

The Rust prelude, described [here](https://doc.rust-lang.org/std/prelude/index.html#), is something that is a part of every program. It is a list of things that are automatically imported into every Rust program. Most of the items in the prelude are traits that are used very often. But in addition to this, we also find the following 2 items:
- std::option::Option::{self, Some, None}
- std::result::Result::{self, Ok, Err}

The first one is the `Option` enum, described as 'a type which expresses the presence or absence of a value'. The later is the `Result` enum, described as 'a type for functions that may succeed or fail.'

Because these types are so commonly used, their variants are also exported. Let's go over both types in more detail.


## The Option

Brought into scope by the prelude, the [Option](https://doc.rust-lang.org/std/option/index.html) is available to us without having to lift a finger. The option is defined as follows:

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

The above tells us `Option<T>` is an enum with 2 variants: `None` and `Some(T)`. In terms of how it is used, the `None` can be thought of as 'nothing' and the `Some(T)` can be thought of as 'something'. Key thing that is not immediately obvious to those starting out with Rust is the '<T>'_-thing_. The `<T>` tells us the Option Enum is a `generic` Enum. 

### The Option is generic over type T.

The enum is 'generic over type T'. The 'T' could have been any letter, just that 'T' is used as a convention where 1 generic is involved.

So what does it mean when we say 'the enum is generic over type T'? It means that we can use it for any type. As soon as we start working with the Enum, we can (and must actually) replace 'T' with a concrete type. This can be any type, as illustrated by the following:

```rust
let a_str: Option<&str> = Some("a str");
let a_string: Option<String> = Some(String::from("a String"));
let a_float: Option<f64> = Some(1.1);
let a_vec: Option<Vec<i32>> = Some(vec![0, 1, 2, 3]);

#[derive(Debug)]
struct Person {
    name: String,
    age: i32,
}

let marie = Person {
    name: String::from("Marie"),
    age: 2,
};

let a_person: Option<Person> = Some(marie);
let maybe_someone: Option<Person> = None;

println!(
    "{:?}\n{:?}\n{:?}\n{:?}\n{:?}\n{:?}",
    a_str, a_string, a_float, a_vec, a_person, maybe_someone
);
```

The above code outputs the following:

<pre>
Some("a str")
Some("a String")
Some(1.1)
Some([0, 1, 2, 3])
Some(Person { name: "Marie", age: 2 })
None
</pre>

The code shows us the enum can be generic over standard as well as over custom types. Additionally, when we define an enum as being of type x, it can still contain the variant 'None'. So the Option is a way of saying the following:

<pre>
This can be of a type T value, which can be anything really, or it can be nothing. 
</pre>

### Matching on the Option

Since Rust does not use exceptions or null values, you will see the Option (and as we will learn later on the Result) used all over the place.

Since the Option is an enum, we can use pattern matching to handle each variant in it's own way:

```rust
let something: Option<&str> = Some("a String"); // Some("a String")
let nothing: Option<&str> = None;   // None

match something {
    Some(text) => println!("We go something: {}", text),
    None => println!("We got nothing."),
}

match nothing {
    Some(something_else) => println!("We go something: {}", something_else),
    None => println!("We got nothing"),
}
```

The above will output the following:

<pre>
We go something: a String
We got nothing
</pre>


### Unwrapping the Option

Oftentimes, you'll see unwrap being used. This looked a bit mysterious at first. Hoovering over it in the IDE offers some clues:

{:refdef: style="text-align: center;"}
![ID unwrap](/assets/img/ide_unwrap.png "ID unwrap"){:height="80%" width="80%"}
{: refdef}

In case you are using VScode, another nice thing to know is that simulteounously pressing Ctrl + left mouse button can oftentimes take you to the source code. In this case, it takes us to the place in `option.rs` where unwrap is defined:
```rust
pub const fn unwrap(self) -> T {
    match self {
        Some(val) => val,
        None => panic!("called `Option::unwrap()` on a `None` value"),
    }
}
```

In `option.rs`, we can see `unwrap` is defined in the `impl<T> Option<T>` block. When we call it on a value, it will try to 'unwrap' the value that is tucked into T. It matches on 'self' and if the `Some` variant is present, 'val' is 'unwrapped' and returned. If the 'None' variant is present, the panic macro is called:


```rust
let something: Option<&str> = Some("Something");
let unwrapped = something.unwrap(); 
println!("{:?}\n{:?}", something, unwrapped);
let nothing: Option<&str> = None;
nothing.unwrap();
```
The code above will result in the following:

<pre>
Some("Something")
"Something"
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src\main.rs:86:17
</pre>

Calling unwrap on an Option is quick and easy. Bot obviously, baking in the possibility of letting your program `panic` and crash is not the most elegant or safe approach. 

## Option examples

Let's look at some examples where 'Option' is used.

### passing an optional value to a function

```rust
fn might_print(option: Option<&str>) {
    match option {
        Some(text) => println!("The argument contains the following value: '{}'", text),
        None => println!("The argument contains None."),
    }
}
let something: Option<&str> = Some("some str");
let nothing: Option<&str> = None;
might_print(something);
might_print(nothing);
```

This outputs the following:

<pre>
The argument contains the following value: 'some str'
The argument contains None.
</pre>

### having a function return an optional value

```rust
// Returns the text if it contains target character, None otherwise:
fn contains_char(text: &str, target_c: char) -> Option<&str> {
    if text.chars().any(|ch| ch == target_c) {
        return Some(text);
    } else {
        return None;
    }
}
let a = contains_char("Rust in action", 'a');
let q = contains_char("Rust in action", 'q');
println!("{:?}", a);
println!("{:?}", q);
```
We can safely assign the return of this function to a variable and, later on, use match to determine how to handle a 'None' return. The pervious code prints the following:

<pre>
Some("Rust in action")
None
</pre>

Let's examine three different ways to work with the Optional return. 

The first one, which is the least safe, would be simply calling unwrap:

```rust
let a = contains_char("Rust in action", 'a');
let a_unwrapped = a.unwrap();   
println!("{:?}", a_unwrapped);

```

The second, safer option, is to use a `match` statement:

```rust
let a = contains_char("Rust in action", 'a');
match a {
    Some(a) => println!("contains_char returned something: {:?}!", a),
    None => println!("contains_char did not return something, so branch off here"),
}
```

The third option is to capture the return of the function in a variable and use `if let`:
```rust
let a = contains_char("Rust in action", 'a');
if let Some(a) = contains_char("Rust in action", 'a') {
    println!("contains_char returned something: {:?}!", a);
} else {
    println!("contains_char did not return something, so branch off here")
}
```

### Optional values inside a struct

We can also use the `Option` inside a struct. This might be usefull in case a field may or may not have any value:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: Option<i32>,
}

let marie = Person {
    name: String::from("Marie"),
    age: Some(2),
};

let jan = Person {
    name: String::from("Jan"),
    age: None,
};
println!("{:?}\n{:?}", marie, jan);
```

The above code outputs the following:

<pre>
Person { name: "Marie", age: Some(2) }
Person { name: "Jan", age: None }
</pre>



### Real world example


An real world example where the Option is used is inside Rust. The pop method for the vector returns an `Option<T>`. The vector has a `pop`-method that returns the last element. But it can be that a vector is empty. In that case, it should return None. An additional problem is that a vector can contain any type. In that case, it is convenvient for it to return `Some(T)`. So for that reason, `pop()` returns `Option<T>`. 

The `pop` method for the vec from Rust 1.53:

```rust
impl<T, A: Allocator> Vec<T, A> {
    // .. lots of other code
    pub fn pop(&mut self) -> Option<T> {
        if self.len == 0 {
            None
        } else {
            unsafe {
                self.len -= 1;
                Some(ptr::read(self.as_ptr().add(self.len())))
            }
        }
    }
    // lots of other code
}    
```

A trivial example where we output the result of popping a vector beyond the point where it is still containing items:

```rust
let mut vec = vec![0, 1];
let a = vec.pop();
let b = vec.pop();
let c = vec.pop();
println!("{:?}\n{:?}\n{:?}\n", a, b, c);
```

The above outputs the following:

<pre>
Some(1)
Some(0)
None
</pre>

## The result

### short description and picture

### defining a Result

### Examples from the field reqwest

### Returning a Result from a function
