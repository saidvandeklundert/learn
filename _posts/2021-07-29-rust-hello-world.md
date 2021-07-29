---
layout: post
title: Rust hello workd
subtitle: For starters
tags: [rust, python, go]
comments: true
---

### Rust:

Create a new cargo package and move to that directory:

```
cargo new hello_world
cd hello_world
```
You will find a `main.rs` file created for you already. This file contains the Rust's "Hello world":

```rust
fn main() {
    println!("Hello, world!");
}
```

We can run it by issuing the following:

```
cargo run
   Compiling hello_world v0.1.0 (C:\dev\hello_world)
    Finished dev [unoptimized + debuginfo] target(s) in 3.75s
     Running `target\debug\hello_world.exe`
Hello, world!
```


### Python:

```python

if __name__ == "__main__":
    print("Hello world")
```



### Go:


```go
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```

Running the Go equivalent:

```
go run hello_world.go
hello world
```
