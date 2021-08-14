---
layout: post
title: Rust vectors
subtitle: data types
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---




https://doc.rust-lang.org/rust-by-example/std/vec.html

https://doc.rust-lang.org/src/alloc/vec/mod.rs.html#151


## What is a vector:

Vectors can be thought of as resizeable arrays. It is a data structure that can be used to store a sequence of elements. The stored elements have to be of the same type. In case you need to be flexible, you could choose to use an enum and define a variety of enum variants as a way to (kind of) store elements of a different type inside a vector (more on that later).

In Rust, a vector is represented using 3 parameters:

- pointer to the data (on the heap)
- length
- capacity

Though these parameters are held together in a struct, this is abstracted away. When working with sequences, the vector offers us quite some ergonomics as we will see later on. The following displays an example vector:

![Rust vector](/assets/img/rust_vector.png "Rust vector")

The top left code defines the vector:

```rust
let mut vec: Vec<i32> = Vec::with_capacity(6);
vec.push(1);
vec.push(2);
```

The above code starts of creating a vector. We use `with_capacity`, which is what Rust recommends whenever it is possible to specify how big the vector is expected to get. We can create vectors without specifying capacity. Directly after defining the vector, we have an empty vector with a lenght of 0 and a capacity of 6.

After we have created the vector, we push two elements onto it. Now, the vector contains the values `[ 1, 2]`. The length is 2 and the capacity remains the same. 

If we were to add 5 elements to this vector, we would exceed the capacity and Rust would resize the vector. This resizing basically involves creating a new vector that has double the capacity and copying over the old vector. To see this in action, we can do the following:

```rust
let mut vector = vec![3, 4, 5, 6, 7];
vec.append(vec![3, 4, 5, 6, 7]);
println!("--------------------------------------------");
println!("Length of the vec: {:?}", vec.len());
println!("Capacity of the vec: {:?}", vec.capacity());
println!("--------------------------------------------");
```
After adding this code, we would get to see the following:

<pre>
--------------------------------------------
Length of the vec: 7
Capacity of the vec: 12
--------------------------------------------
</pre>





## Common operations:

2 ways to define a vector:
let mut vec = Vec::new();
let mut vec = vec![1, 2, 3];