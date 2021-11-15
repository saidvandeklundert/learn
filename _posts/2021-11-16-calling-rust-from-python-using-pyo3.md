---
layout: post
title: Calling Rust from Python using PyO3 
subtitle: speed up your Python using Rust
image: /assets/img/rusty_logo_small.jpeg
cover-img: /assets/img/rusty_logo.jpeg
thumbnail-img: /assets/img/rusty_logo_small.jpeg
share-img: /assets/img/rusty_logo.jpeg
tags: [rust, python, PyO3 ]
---

Calling Rust code from Python is made very easy by something called [pyo3](https://github.com/PyO3/pyo3). PyO3 can be used to build a Rust library as a Python mode. After having done so, you can then import the Rust code you wrote in your Python and run it. Imagine having build a Rust library that contains a function called `multiply`. If you build the library as a Python module called `rust`, the following would be all that is required to run that function in your Python scripts:

```python
import rust

result = rust.multiply(2, 3)
print(result)
```
In this blog post, I will give a short introduction to PyO3. After that, I will discuss several example functions, written in Rust and called from Python. Some of these examples include
- multiply numbers and sum vectors
- print different types (dicts, lists, arrrays)
- use a Rust struct in Python code
- serialize a Pydantic basemodel as a struct in Rust
- calculate the n-th Fibonacci in Python as well as in Rust
- allow Rust to use the logger from the Python runtime
- generating an Error in Rust and catching it as an exception in Python

## Introduction to PyO3

PyO3 offers some ergonomics for people wanting to glue Rust and Python code together. It helps you calling Python code from Rust as well as calling Rust code from Python. Since I have been using it only to call Rust code from Python, that is the only thing I will write about here.

PyO3 offers some ergonomics in a vartiety of ways.

First of all, there is [maturin](https://github.com/PyO3/maturin), that will compile the Rust code for you and install the compiled code as a Python module that you can import using an import statement. After having things setup, you only have to run 1 command to use the Rust code in Python.

Apart from a tool that makes it easy to get started, PyO3 provides you with a lot of bindings to Python that make it so that you do not really have to bother that much with the interaction between Python and Rust. And to make things really convenient, PyO3 comes with a lot of macro's that prevent you from having to write too much boilerplate code.

## Calling a Rust function from Python

In this first example, we'll call a multiplication function from Rust. Normally, we could write something like this:

```rust
fn multiply(a: isize, b: isize) -> isize {
    a * b
}
```

Without adding too many things, we can make this function callable from Python. For starters, we need to:
- bring in the `pyo3` prelude
- annotate the function with `#[pyfunction]` to turn it into a PyCFunction
- wrap the result in a `PyResult`
- add the function to the `#[pymodule]`

The following code follows the above steps in that same order:
```rust
use pyo3::prelude::*;

#[pyfunction]
fn multiply(a: isize, b: isize) -> PyResult<isize> {
    Ok(a * b)
}

#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(multiply, m)?)?;
    Ok(())
}
```

The above will expose the multiply function in a Python module called `rust` (after the name of the last function). After we put in place the proper [Cargo.toml](), we write the Python that calls this function:
```python
import rust

result = rust.multiply(2, 3)
print(result)
```

To be able to actually run this code, the `rust` library needs to be compiled and installed as a Python library. This is where `maturin` comes in.

<pre>
root@rust:/# <b>git clone https://github.com/saidvandeklundert/pyo3.git</b>
Cloning into 'pyo3'...
remote: Enumerating objects: 36, done.
    ...
Resolving deltas: 100% (9/9), done.
root@rust:/# cd pyo3/multiply/
root@rust:/pyo3/multiply# python3 -m venv .env
root@rust:/pyo3/multiply# source .env/bin/activate
(.env) root@rust:/pyo3/multiply# <b>pip install maturin</b>
Collecting maturin
    ...
Installing collected packages: toml, maturin
Successfully installed maturin-0.11.5 toml-0.10.2
(.env) root@rust:/pyo3/multiply# <b>maturin develop</b>
🔗 Found pyo3 bindings
🐍 Found CPython 3.9 at python
   Compiling proc-macro2 v1.0.32
    ...
   Compiling multiply v0.1.0 (/pyo3/multiply)
    Finished dev [unoptimized + debuginfo] target(s) in 23.48s
(.env) root@rust:/pyo3/multiply# <b>python3 multiply.py</b>
6
</pre>

That was it!

## Calculating the n-th Fibonacci number in Python and in Rust

To calculate the n-th Fibonacci number, I will use a Python and Rust function that are very similar to each other. The following is the Python function:

```python
def get_fibonacci(n: int) -> int:
    """Get the nth fibonacci number."""
    fib_seq = [0]
    while n > 0:
        n -= 1
        if len(fib_seq) == 1:
            fib_seq.append(1)
        else:
            last = fib_seq[-1]
            second_to_last = fib_seq[-2]
            fib_seq.append(last + second_to_last)

    return fib_seq[-1]
```

To add the Rust equivalent, we need to do the following:
- add the function to `lib.rs`
- annotate the function with `#[pyfunction]`
- add the function to the module

```rust
#[pyfunction]
fn get_fibonacci(number: isize) -> PyResult<u128> {
    if number == 1 {
        return Ok(1);
    } else if number == 2 {
        return Ok(2);
    }

    let mut sum = 0;
    let mut last = 0;
    let mut curr = 1;
    for _ in 1..number {
        sum = last + curr;
        last = curr;
        curr = sum;
    }
    Ok(sum)
}

#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(multiply, m)?)?;    
    m.add_function(wrap_pyfunction!(get_fibonacci, m)?)?;
    Ok(())
}
```

<pre>
(.env) root@rust:/pyo3/pyo3# maturin develop
(.env) root@rust:/pyo3/pyo3# python -i
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.    
>>> import rust
>>> import timeit
>>> timeit.timeit("get_fibonacci(150)", setup="from fib import get_fibonacci")
33.936191699998744
>>> timeit.timeit("get_fibonacci(150)", setup="from rust import get_fibonacci")
3.6803346000015154
>>>
</pre>

Rust is almost 10x faster. But it gets better. We can also do a release build by adding `--release` as an argument to maturin:

<pre>
(.env) root@rust:/pyo3/pyo3# python -i
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.    
>>> import rust
>>> timeit.timeit("get_fibonacci(150)", setup="from fib import get_fibonacci")
37.771799099995405
>>> timeit.timeit("get_fibonacci(150)", setup="from rust import get_fibonacci")
0.13583959999959916
</pre>

Not sure what mechanism is at play here, but that is quite a difference.

When I do a release build and compare a function that scans a piece of text for a word, the difference is a little less absurd, but still a 3x improvement:

<pre>
timeit.timeit("count_occurences(text, 'words')", number=100, setup="""
from __main__ import count_occurences
from __main__ import text
""")
0.19804459999431856
>>>
timeit.timeit("rust.count_occurences(text, 'words')", number=100, setup="""
import rust
from __main__ import text
""")
0.07730440000159433
>>>
</pre>

## Working with different types

The PyO3 user guide describes the [mapping](https://pyo3.rs/main/conversions/tables.html) the Python type and the corresponding function argument types in Rust. This makes it relatively easy to work with function arguments of different types.

A few examples to illustrate this.

### returning the sum of the numbers in a list:

```rust
#[pyfunction]
fn list_sum(a: Vec<isize>) -> PyResult<isize> {
    let mut sum: isize = 0;
    for i in a {
        sum += i;
    }
    Ok(sum)
}

#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(list_sum, m)?)?;
    Ok(())
}    
```

Calling this function in Python:

```
>>> rust.list_sum([10, 10, 10, 10, 10])
50
```

### printing the values of a dict/HashMap:

```rust
#[pyfunction]
fn dict_printer(hm: HashMap<String, String>) {
    for (key, value) in hm {
        println!("{} {}", key, value)
    }
}
```
Calling this function from Python:
```python
>>> a_dict = {
...     "key 1": "value 1",
...     "key 2": "value 2",
...     "key 3": "value 3",
...     "key 4": "value 4",
... }
>>>
>>> rust.dict_printer(a_dict)
key 2 value 2
key 1 value 1
key 3 value 3
key 4 value 4
```

Note that the HasMap in Rust is very different from a Python dictionary. Given the fact that we defined the HasMap to use Strings for both key as well as value, it will fail in case we use something else in our Python dictionary. There is nothing that will try and coerce a type into another:

```python
>>> try:
...     rust.dict_printer({"a": 1, "b": 2})
... except TypeError as e:
...     print(f"Caught a type error: {e}")
...
Caught a type error: argument 'hm': 'int' object cannot be converted to 'PyString'
>>>
```

### printing a word n times

The following function will print a word a number of times. Additionally, it can also print the word in reverse and/or in uppercase:

```rust
#[pyfunction]
fn word_printer(mut word: String, n: isize, reverse: bool, uppercase: bool) {
    if reverse {
        let mut reversed_word = String::new();
        for c in word.chars().rev() {
            reversed_word.push(c);
        }
        word = reversed_word;
    }
    if uppercase {
        word = word.to_uppercase();
    }
    for _ in 0..n {
        println!("{}", word);
    }
}
```

Calling this function in Python:

```
>>> rust.word_printer("hello", 3, False, True)
HELLO
HELLO
HELLO
>>> rust.word_printer("eyb", 2, True, False)        
bye
bye
```

### printing the items in a list and printing the items in an array:

```rust
#[pyfunction]
fn array_printer(a: [String; 8]) {
    for string in a {
        println!("{}", string)
    }
}

#[pyfunction]
fn vector_printer(a: Vec<String>) {
    for string in a {
        println!("{}", string)
    }
}

```


Calling these from Python:
```python
>>> a_list = ["1", "2", "3", "4", "5", "6", "7", "8"]
>>> rust.vector_printer(a_list)
1
2
3
4
5
6
7
8
>>> rust.array_printer(another_list)
1
2
3
4
5
6
7
8
```
## Using a Rust struct in Python

We can also define a struct in Rust and use it in our Python script. I have not exhausted all possibilities, but you can output the struct with a constructor and some methods. 

I made the following example:

```rust
#[pyclass]
pub struct RustStruct {
    pub data: String,
    pub vector: Vec<u8>,
}
#[pymethods]
impl RustStruct {
    #[new]
    pub fn new(data: String, vector: Vec<u8>) -> RustStruct {
        RustStruct { data, vector }
    }
    pub fn printer(&self) {
        println!("{}", self.data);
        for i in &self.vector {
            println!("{}", i);
        }
    }
    pub fn extend_vector(&mut self, extension: Vec<u8>) {
        println!("{}", self.data);
        for i in extension {
            self.vector.push(i);
        }
    }
}
#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_class::<RustStruct>()?;

    Ok(())
}
```

Notice the use of 3 new annotations:
- `#[pyclass]`: above the struct definition
- `#[pymethods]`: above the impl block
- `#[new]`: above the constructor

Additionally, we add the struct to the module in a slightly different way.

Now, to use this in Python, we do the following:
```python
>>> rust_struct = rust.RustStruct(data="some data", vector=[255, 255, 255])
>>> rust_struct.extend_vector([1, 1, 1, 1])
Extending the vector.
>>> rust_struct.printer()
some data
255
255
255
1
1
1
1
```

## Sending over Json to Rust

One way to send over complex data structures could be by using JSON. The following example marshalls a string into a struct:

```rust
extern crate serde;
extern crate serde_json;
use serde::{Deserialize, Serialize};

#[pyfunction]
fn human_says_hi(human_data: String) {
    println!("{}", human_data);
    let human: Human = serde_json::from_str(&human_data).unwrap();

    println!(
        "Now we can work with the struct:\n {:#?}.\n {} is {} years old.",
        human, human.name, human.age,
    )
}

#[derive(Debug, Serialize, Deserialize)]
struct Human {
    name: String,
    age: u8,
}
```

On the Python side, I will use Pydantic to send over some Json:
```python
>>> from pydantic import BaseModel
>>> 
>>> class Human(BaseModel):
...     name: str
...     age: int
...
>>> 
>>> jan = Human(name="Jan", age=6)
>>> print(jan.json())
{"name": "Jan", "age": 6}
>>> rust.human_says_hi(jan.json())
{"name": "Jan", "age": 6}
Now we can work with the struct:
 Human {
    name: "Jan",
    age: 6,
}.
 Jan is 6 years old.
```

Note, `pydantic` is one of my favorite Python packages. The `.json()` method I used here convert the class to a JSON string.

## Have Rust use a logger from the Python runtime

Rust can be made to log using the logger we define in Python. This is possible using `pyo3-log`. 

We start off by adding the following to the Cargo.toml`:

```
[dependencies]
pyo3-log = "0.5.0"
log = "0.4.14"
```

Next, we create 2 example functions that will log a message:

```rust
use log::{debug, error, info, warn};
use pyo3_log;

#[pyfunction]
fn log_different_levels() {
    error!("logging an error");
    warn!("logging a warning");
    info!("logging an info message");
    debug!("logging a debug message");
}

#[pyfunction]
fn log_example() {
    info!("A log message from {}!", "Rust");
}
#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    pyo3_log::init();

    m.add_wrapped(wrap_pyfunction!(log_example))?;
    m.add_wrapped(wrap_pyfunction!(log_different_levels))?;

    Ok(())
}
```

THen, after we define a logger in Python, we can use it in Rust:

```python
>>> import logging
>>> FORMAT = "%(levelname)s %(name)s %(asctime)-15s %(filename)s:%(lineno)d %(message)s"
>>> logging.basicConfig(format=FORMAT)
>>> logging.getLogger().setLevel(logging.DEBUG)
>>> logging.info("Logging from the Python code")
INFO root 2021-11-15 20:24:46,979 <stdin>:1 Logging from the Python code>>> rust.log_example()
INFO rust 2021-11-15 20:24:51,336 lib.rs:118 A log message from Rust!
>>> rust.log_different_levels()
ERROR rust 2021-11-15 20:24:55,212 lib.rs:110 logging an error
WARNING rust 2021-11-15 20:24:55,212 lib.rs:111 logging a warning       
INFO rust 2021-11-15 20:24:55,213 lib.rs:112 logging an info message    
DEBUG rust 2021-11-15 20:24:55,213 lib.rs:113 logging a debug message   
>>>
```


## Catch an exception


```rust
use std::fmt;


// Define 'MyError' as a custom exception:
#[derive(Debug)]
struct MyError {
    /*
    the 'message' field that is used later on
    to be able print any message.
    */
    pub msg: &'static str,
}

// Implement the 'Error' trait for 'MyError':
impl std::error::Error for MyError {}

// Implement the 'Display' trait for 'MyError':
impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Error from Rust: {}", self.msg)
    }
}

// Implement the 'From' trait for 'MyError'.
// Used to do value-to-value conversions while consuming the input value.
impl std::convert::From<MyError> for PyErr {
    fn from(err: MyError) -> PyErr {
        PyOSError::new_err(err.to_string())
    }
}

#[pyfunction]
// The function 'greater_than_2' raises an exception if the input value is 2 or less.
fn greater_than_2(number: isize) -> Result<isize, MyError> {
    if number <= 2 {
        return Err(MyError {
            msg: "number is less than or equal to 2",
        });
    } else {
        return Ok(number);
    }
}

#[pymodule]
fn rust(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(greater_than_2, m)?)?;    

    Ok(())
}
```

Now, on the Python side, we can see the following:

```python
>>> rust.greater_than_2(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OSError: Error from Rust: number is less than or equal to 2
>>> rust.greater_than_2(3)
3
>>> rust.greater_than_2(11)
11
>>> rust.greater_than_2(-11)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OSError: Error from Rust: number is less than or equal to 2
```


The code from this blog can be found [here](https://github.com/saidvandeklundert/pyo3).


