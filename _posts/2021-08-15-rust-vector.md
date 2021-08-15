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
## What is a vector:

Vectors can be thought of as resizeable arrays. It is a data structure that can be used to store a sequence of elements. The stored elements have to be of the same type. In case you need to be flexible, you could choose to use an enum with several variants as a way to store elements of a different type inside a vector (more on that later).

In Rust, a vector is represented using 3 parameters:
- pointer to the vector's data on the heap
- length
- capacity

These parameters are held together in a struct. When working with sequences, the vector offers us quite some ergonomics as we will see later on. 

Let's start off creating a vector:

```rust
let mut vec: Vec<i32> = Vec::with_capacity(6);
vec.push(1);
vec.push(2);
```

We use `with_capacity`. Not required, though it is what Rust recommends whenever it is possible to specify how big the vector is expected to get. Directly after defining the vector, we have an empty vector with a lenght of 0 and a capacity of 6. Next, we push two elements onto it. Now, the vector contains the values `[ 1, 2]`. This increases the length to 2 while the capacity remains the same. Our vector now looks like this:

{:refdef: style="text-align: center;"}
![Rust vector](/learn/img/rust_vector_1.png "Rust vector"){:height="80%" width="80%"}
{: refdef}

To better understand vectors, we can print several of it's properties to screen:

```rust
println!("--------------------------------------------");
println!("Values inside vec: {:?}", vec);
println!("Length of the vec: {:?}", vec.len());
println!("Capacity of the vec: {:?}", vec.capacity());
println!("--------------------------------------------");
```

The code outputs the following:

<pre>
--------------------------------------------
Values inside vec: [1, 2]
Length of the vec: 2
Capacity of the vec: 6
--------------------------------------------
</pre>

Here we display the values, the lenght and the capacity of the vector.

Now, if we were to add 5 elements to this vector, we would exceed the capacity and Rust would resize the vector. This resizing basically involves creating a new vector that has double the capacity and copying over the old vector. To see this in action, we can do the following:

```rust
let mut vector = vec![3, 4, 5, 6, 7];
vec.append(&mut vector);
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

The following are some examples on how you could use vectors. Most of the comments detail the content of the vector after an operation. They (mostly) come from printing the variable using `println!("{:?}", x);`. To reduce clutter, I removed the print statements.

Now, for starters, creating an empty vector and pusing some values onto it:

```rust
let mut vector: Vec<i32> = Vec::new(); // []
vector.push(0); // [0]
vector.push(1); // [0, 1]
vector.push(2); // [0, 1, 2]
```

Removing the last value from a vector, and getting an Option returned that contains that value:

```rust
vector.pop()); // Some(2)
vector; // [0, 1]
```

Retrieving values from a vector:

```rust
vector[1] // 1
```

The above will panic when you refer to an index that does not exist. To safely access an index, use `.get()` (or `get_mut()` to get access to a mutable reference):

```rust
vector.get(1) // Some(1)
vector.get(100) // None
```

Here we create a second vector using the macro:

```rust
let mut vec = vec![2, 2, 3, 4, 5];
vec; // [2, 2, 3, 4, 5]
```

We can use `remove` to remove an index from the vec:

```rust
vec.remove(0);
vec; // [2, 3, 4, 5]
```

Use `append` to move all elements from a vector into the vector that is calling the method:

```rust
vector.append(&mut vec);
vec; // []
vector; // [0, 1, 2, 3, 4, 5]
```

Check if a vector is empty:

```rust
vec.is_empty(); // true
vector.is_empty(); // false
```

Return the length of a vector:

```rust
vector.len(); // 6
```

Iterating a vector. After defining the iterator, we loop over it:

```rust
let vecter_iterator = vector.iter();
for elem in vecter_iterator {
    println!("{}", elem);         
}
/*
0
1
2
3
4
5
*/
```

The previous method will not allow us to alter any of the values inside the vector. To iterate over the elements in the vector AND change them, we can do the following:

```rust
let vecter_iterator_m = vector.iter_mut();
for elem in vecter_iterator_m {
    *elem = *elem * 2;
}
println!("{:?}", vector); // [0, 2, 4, 6, 8, 10]
```

To verify whether or not a value exists inside a vector:

```rust
vector.contains(&200); // false
vector.contains(&2); // true
```

Insert an element into a vector:

```rust
vector.insert(2, 1);
vector; // [0, 2, 1, 4, 6, 8, 10]
```

Sorting a vector and then execute a binary search:

```rust
vector.sort()
vector; // [0, 1, 2, 4, 6, 8, 10]
vector.binary_search(&4); // Ok(3)
vector.binary_search(&400); // Err(7)
```

Resize a vector and filling empty elements with 0's:

```rust
vector.resize(10, 0);
vector; // [0, 1, 2, 4, 6, 8, 10, 0, 0, 0]
```

We can use the same method to shrink the vector:

```rust
vector.resize(2, 0);
vector; // [0, 1]
```

Working with different types inside a single vector using an Enum with Variants:

```rust
#[derive(Debug)]
enum ManyAsOne {
    String(String),
    I32(i32),
    F64(f64),
}
let vec = vec![
    ManyAsOne::I32(65444),
    ManyAsOne::String(String::from("Hello world.")),
    ManyAsOne::String(String::from("Foo bar.")),
    ManyAsOne::F64(3.14159265),
    ManyAsOne::I32(1984),
];
for elem in vec {
    println!("{:?}", elem);
    match elem {
        ManyAsOne::I32(value) => println!("value: {}", value),
        ManyAsOne::String(value) => println!("value: {}", value),
        ManyAsOne::F64(value) => println!("value: {}", value),
    }
}
```

Running the above code outputs the following:

<pre>
I32(65444)
value: 65444
String("Hello world.")
value: Hello world.
String("Foo bar.")
value: Foo bar.
F64(3.14159265)
value: 3.14159265
I32(1984)
value: 1984
</pre>

Putting a vector into a hashmap:

```rust
use std::collections::HashMap;
let mut job_results: HashMap<String, Vec<i32>> = HashMap::new();
job_results.insert(String::from("1"), vec![3, 2, 2, 2, 2]);
job_results.insert(String::from("2"), vec![2, 3, 2, 2, 2]);
job_results.insert(String::from("3"), vec![2, 2, 3, 2, 2]);
job_results;  // {"2": [2, 3, 2, 2, 2], "3": [2, 2, 3, 2, 2], "1": [3, 2, 2, 2, 2]}
```

And finaly, it is also possible to put a vector inside a struct:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
    interests: Vec<String>,
}

let mut marie = Person {
    name: String::from("Marie"),
    age: 31,
    interests: vec![
        String::from("Rust"),
        String::from("Python"),
        String::from("History"),
    ],
};
marie.interests.push(String::from("Astronomy"));
println!("{:?}", marie);
```
Running the above will output the following:

<pre>
Person { name: "Marie", age: 31, interests: ["Rust", "Python", "History", "Astronomy"] }
</pre>
