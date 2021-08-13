---
layout: post
title: Rust slice
subtitle: data types
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---

In Rust, the `slice` is a `primitive type` as well as a `sequence type`. I found slices very confusing at first. In the Rust book, slices are defined as 'a dynamically-sized view into a contiguous sequence'. 

But somewhere in the rust-lang github repo, the slice is defined as follows:

```
Slices are a view into a block of memory represented as a pointer and a length.
```

I found this more helpful. It tells us the slice is a `fat pointer`. So basically, when you have a slice of an array, the slice contains thw following:
- a pointer to the address of the element in the array that the slice starts with
- a value that describes the length of the slice

This similar to a slice in Go. In Rust though, the slice can be a view into a backing array as well as a view into other sequences, like vectors or Strings. In case the slice is a view into a `String`, it is called a `string slice`/`string literals` and it is usually seen in its borrowed form `&str`.

The following shows an example array and 2 slices from that array:

![Rust slice](/assets/img/rust_slice.png "Rust slice")


