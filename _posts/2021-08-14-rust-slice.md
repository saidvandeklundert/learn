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

I found this more helpful. It tells us the slice is a `fat pointer`. So basically, when you have a slice of an array, the slice contains the following:
- a pointer to the address of the element in the array that the slice starts with
- a value that describes the length of the slice

In Rust, the slice can be a view into a backing array as well as a view into other sequences, like vectors or Strings. In case the slice is a view into a `String`, it is called a `string slice`/`string literals` and it is usually seen in its borrowed form `&str`.

The following shows an example array and 2 slices from that array:

![Rust slice](/assets/img/rust_slice.png "Rust slice")


In the middle, you see an array that is defined as `array : [i32;10]`. The index runs from 0 to 9 and I gave the similar values.

On the left and on the right, there are 2 examples slices. At the top, we see the slice header and and the bottom, we see the index and the values associated with the slices.

The lefthand slice, `slice1`, is defined `&array[5..10]`. Because of this, the pointer of the slice is pointing to array index 5. We can also see `slice1` has a length of 5. This means the slice will contain 5 elements of the array. A little below the slice, we can see the index and the values associated with `slice1`. The index runs from 0 to 4 and the values are those we can see in the backing array. On the righthand side, we see `slice2`. That slice's pointer is pointing to element 3 and the slice has a length of 4.


### Some common slice operations

We start of defining and array and a slice. The slice will be a view into the first 5 elements of the array. And since we will be changing a value in the slice later on, both are defined as mutable:

```rust
let mut array: [i32; 7] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
let array_slice = &mut array[0..5]; // [ 0, 1, 2, 3, 4 ]
```

We now have the following:

![Rust slice](/assets/img/slice_1.png "Rust slice")