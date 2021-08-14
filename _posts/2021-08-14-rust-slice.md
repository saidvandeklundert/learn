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

We start of defining and array and look at a few ways to take a slice:

```rust
let array: [i32; 7] = [0, 1, 2, 3, 4, 5, 6];

let slice = &array[..]; // [ 0, 1, 2, 3, 4, 5, 6 ]
let slice = &array[0..3]; // [ 0, 1, 2 ]
let slice = &array[..3]; // [ 0, 1, 2 ]
let slice = &array[2..4]; // [ 2, 3 ]
let slice = &array[2..]; // [ 2, 3, 4, 5, 6 ]
```

The above shows an immutable array and a few ways in which we could create a slice out of it. The comment after the slice definition displays the `dbg!(slice);` output.


Let's proceed and create a mutable slice so we can also change the value later on. We can do that as follows:

```rust
let mut array: [i32; 7] = [0, 1, 2, 3, 4, 5, 6];
let array_slice = &mut array[0..5]; // [ 0, 1, 2, 3, 4 ]
```

![Rust slice](/assets/img/slice_1.png "Rust slice")

Checking the length of the slice and iterating the index / value:

```rust
slice.len(); // 5

for (index, item) in slice.iter().enumerate() {
    println!("index: {:?} element {:?}", index, item);
}
/*
index: 0 element 0
index: 1 element 1
index: 2 element 2
index: 3 element 3
index: 4 element 4
*/
```

Retrieving a value from the slice:

```rust
slice[1]; // 1
```

The slice lenght is not (always) known at compile time. The compiler will not save you in case you access an index value that is out of bounds:

```rust
slice[100];
```

Running the above for our slice will result in the following:

```
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 100'
```

To safely access values from a slice, we can use `get`:

```rust
slice.get(2); // Some(2)
slice.get(100); // None
```

Finding values in a slice:

```rust
slice.iter().position(|v| v == &120); // None
slice.iter().position(|v| v == &4); // Some(4)
```

Now, let's change the value of the slice and immediately thereafter, check the elements in the slice as well as in the array that owns the values that the slice is referencing:

```rust
slice[0] = 100;
dbg!(slice); // [100, 1, 2, 3, 4]
dbg!(array); // [100, 1, 2, 3, 4, 5, 6]
```