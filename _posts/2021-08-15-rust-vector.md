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

A vector is represented using 3 parameters:

- pointer to the data (on the heap)
- length
- capacity



## Common operations:

2 ways to define a vector:
let mut vec = Vec::new();
let mut vec = vec![1, 2, 3];