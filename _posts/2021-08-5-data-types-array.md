---
layout: post
title: Rust arrays
subtitle: data types
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, rust basics]
---


In Rust, the `array` is the simplest of the sequence types (the others being slices and vectors). The array lives on the stack and has a fixed length. The length you declare an array with is part of it's type and it is going to remain the same throughout the program. This is very important, because to succesfully work with an array, you have to be able to specify the length up front at compile time.

The array can be declared as a mutable. This will let you change the values of the elements inside the array.

A constraint in addition to having a fixed lenght is that arrays can only store elements of the same type. An array created to hold `u8`'s can never be used to store an `i32`.


Arrays do not seem to be too pervasive in Rust. Some things they are used for could be:
- buffers
- fixed values ( days of the week, MAC address, IP address, UUID)


If you need a sequence type that can grow or shrink, you are in need of a vector. If you are still interested in using arrays, here are some commonly used operations.



## Working with arrays in Rust

Other then the above, when you are starting out with Rust, there is really not that much more you have to know about arrays. The best way to get more familiar with them is to play around with them. The following is a basic cheatsheet that should give you a good overal idea on how to start using arrays.

Creating an array:


```rust
// Specifying the type and length in square brackets when initializing:
let arr: [u8; 3] = [0, 1, 2];
// Have Rust infer the type:
let arr_type_inferred = [0, 1, 2];
// Create an array containing all 0's:
let arr_all_0s: [u8; 3] = [0; 3];
// show the entire array:
println!(
    "\
arr: {:?}
arr_type_inferred: {:?}
arr_all_0s: {:?}",
    arr, arr_type_inferred, arr_all_0s
);
```

This outputs the following:

<pre>
arr: [0, 1, 2]
arr_type_inferred: [0, 1, 2]
arr_all_0s: [0, 0, 0]
</pre>

Initializing an array as mutable so we can change the elements:

```rust
// initialize an array as mutable so we can change it later on:
let mut a: [u8; 4] = [0; 4];
println!("Before change: {:?}", a); // Before change: [0, 0, 0, 0]
a[1] = 30;
a[2] = 20;
a[3] = 10;
println!("After change: {:?}", a); // After change: [0, 30, 20, 10]
```

Sorting the array:

```rust
a.sort(); 
println!("After sorting: {:?}", a); // After sorting: [0, 10, 20, 30]
```

Accessing an element:

```rust
a[1]    // 10
```

Verifying a value is present in the array:

```rust
let x: u8 = 10;
if a.contains(&x) {
    println!("a contains 10")
}
```

We can also use a more 'rusty' approach. In the following example, we call `iter()` on the array. This returns an iterator and on this return, we call `any()`. Any takes a closure and will return a bool indicating whether or not there was a match:

```rust
let any_res = a.iter().any(|v| v == &10);
dbg!(any_res);  // any_res = true
```

We can also use this approch to test for other conditions. Some examples:

```rust
dbg!(a.iter().any(|v| v < &0)); // a.iter().any(|v| v < &0) = false
dbg!(a.iter().any(|v| v > &10)); // a.iter().any(|v| v > &10) = true
```

Iterating over an array:

```rust
for item in a.iter() {
    println!("{:?}", item);
}
/*
0
10
20
30
*/
```

Alternatively, we can also use the following:

```rust
for item in a {
    println!("{:?}", item);
}
/*
0
10
20
30
*/
```


Iterate the array and use enumerate to work with the index as well as the value at that index:
```rust
for (index, item) in a.iter().enumerate() {
    println!("index: {:?} element {:?}", index, item);
}
/*
index: 0 element 0
index: 1 element 10
index: 2 element 20
index: 3 element 30
*/
```

Iterate the array and change the value:

```rust
for item in a.iter_mut() {
    *item += 100;
}
dbg!(a)
/*
a = [
    100,
    110,
    120,
    130,
]
*/
```

Finding the position of an element inside an array:
```rust
// find the position of a value in an array:
dbg!(a.iter().position(|v| v == &120));
dbg!(a.iter().position(|v| v == &121));
// find the position of a value greater then 1 in the array:
dbg!(a.iter().position(|v| v > &1));
```

The above returns the following:

<pre>
[src\main.rs:83] a.iter().position(|v| v == &120) = Some(    
    2,
)
[src\main.rs:84] a.iter().position(|v| v == &121) = None     
[src\main.rs:86] a.iter().position(|v| v > &1) = Some(       
    0,
)
</pre>


Checking the size of an array in memory:

```rust
let i32_arr: [i32; 4] = [0, 1, 2, 3];
let u8_arr: [u8; 4] = [0, 1, 2, 3];
println!(
    "i32_arr is {} bytes\nu8_arr is {} bytes",
    std::mem::size_of_val(&i32_arr),
    std::mem::size_of_val(&u8_arr)
);
/*
i32_arr is 16 bytes
u8_arr is 4 bytes
*/
```

The array implements the `Copy` trait. So when we pass it into a function, the value is copied into a function:
```rust
fn array_copy(mut arr: [u8; 4]) {
    for elem in arr.iter_mut() {
        *elem = 0;
    }
    println!("after changing it in the func {:?}", arr);
}
println!("before array_copy(): {:?}", a);
array_copy(a);
println!("after array_copy(): {:?}", a);
```

This returns the following:

<pre>
before array_copy(): [100, 110, 120, 130]
after changing it in the func [0, 0, 0, 0]
after array_copy(): [100, 110, 120, 130]
</pre>

To make the changes persist, we have 2 basic options. We can have the `array_copy()` func return the new value. Or alternatively, we could pass a reference, like so:

```rust
fn array_ref(arr: &mut [u8; 4]) {
    for elem in arr.iter_mut() {
        *elem = 0;
    }
    println!("after changing it in the func: {:?}", arr);
}

println!("before array_ref(): {:?}", a);
array_ref(&mut a);
println!("after array_ref(): {:?}", a);
```

This last code will output the following:

<pre>
before array_ref(): [100, 110, 120, 130]
after changing it in the func: [0, 0, 0, 0]
after array_ref(): [0, 0, 0, 0]
</pre>


