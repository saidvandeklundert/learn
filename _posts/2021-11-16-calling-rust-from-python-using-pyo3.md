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

Calling Rust code from Python is made easy by [PyO3](https://github.com/PyO3/pyo3). You can write a Rust library and rely on the combination of PyO3 and `maturin`, a supporting tool from the `PyO3` ecosystem, to compile the Rust library and have it installed directly as a Python module. Some of the things that PyO3 does is translate types from Python to Rust and vice versa. It also offers a set of macros that make exporting Rust functions to Python very straightforward.


In this blog post, I will give a short introduction to PyO3. After that, I will discuss several example functions, written in Rust and called from Python. These examples include:
- calculate the n-th Fibonacci in Python as well as in Rust
- having Python use a variety of types in Rust functions
- using a Rust struct in Python code
- using Python to send JSON to Rust and serialize that JSON as a struct
- allow Rust to use the logger from the Python runtime
- generating an Error in Rust and catching it as an exception in Python

## Introduction to PyO3

PyO3 offers some ergonomics for people wanting to glue Rust and Python code together. It helps you calling Python code from Rust as well as calling Rust code from Python. Since I have been using it only to call Rust code from Python, that is the only thing I will write about here.

So what does PyO3 give you?

For starters, there is [maturin](https://github.com/PyO3/maturin). This tool will compile the Rust code for you and install the compiled code as a Python module in your virtual environment. After this, you can import this module in your Python code and use it. After you <b>pip install maturin</b>, you only have to run 1 command (<b>maturin develop</b>) to use the Rust code in Python.

Apart from `maturin`, there is of course PyO3 itself. PyO3 offers Rust bindings to the Python interpreter. This makes it so that you do not really have to bother that much with the interaction between Python and Rust. For instance, you will not have to worry about how to translate a Python string to something in C, and then something else again in Rust. The same goes for integers, floats, lists, dictionaries, etc.  And to make things convenient, PyO3 comes with a lot of macros that prevent you from having to write too much boilerplate code. To expose Rust functions to Python, you annotate them with a macro. After this, PyO3 will take care of the rest. The same applies in case you want to export a struct or methods.

## Calling a Rust function from Python

In this first example, we'll call a Rust multiplication function from Python. Normally, we could write something like this:

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

The following code follows illustrates the above steps in the same order:

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

The above will expose the multiply function in a Python module called `rust` (after the name of the last function). We also need to put in place the proper [Cargo.toml](https://github.com/saidvandeklundert/pyo3/blob/main/multiply/Cargo.toml) file. 

To make things easy, make sure that the name of the library in Cargo.toml matches the name of the function that was annotated with `#[pymodule]`. In my example, I put the following in my `Cargo.toml`:

<pre>
[lib]
name = "rust"
</pre>

When these two names match, the `maturin` build tool will install the Rust library as a Python module using that name.

So, in this case, having chosen `rust` as the name of the package, we can write the following Python to call the multiply function:
```python
import rust

result = rust.multiply(2, 3)
print(result)
```

To be able to run this code, we need to compile the Rust code and install it as a Python library. This is where `maturin` comes in:

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

Now I can run [this](https://github.com/saidvandeklundert/pyo3/blob/main/multiply/multiply.py) script:

<pre>
(.env) root@rust: multiply# python3 multiply.py
6
</pre>

## Calculating the n-th Fibonacci number in Python and in Rust

To calculate the n-th Fibonacci number, I will use a Python and Rust function that are very similar to each other. The following is the Python function:

```python
def get_fibonacci(number: int) -> int:
    """Get the nth Fibonacci number."""
    if number == 1:
        return 1
    elif number == 2:
        return 2

    total = 0
    last = 0
    current = 1
    for _ in range(1, number):
        total = last + current
        last = current
        current = total
    return total
```

To add the Rust equivalent, we need to do the following:
- create the function in our `lib.rs`
- annotate the function with `#[pyfunction]`
- add the function to the module

This is illustrated in the following code snippet:

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
    m.add_function(wrap_pyfunction!(get_fibonacci, m)?)?;
    Ok(())
}
```

After pulling in the new code, we can use maturin to build and install the module and do a comparison:

<pre>
(.env) root@rust:/pyo3/pyo3# <b>maturin develop</b>
(.env) root@rust:/pyo3/pyo3# <b>python -i</b>
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.    
>>> <b>import timeit</b>
>>>
>>> <b>timeit.timeit("get_fibonacci(5)", setup="from fib import get_fibonacci")</b>
0.49461510000401177
>>>
>>> <b>timeit.timeit("get_fibonacci(5)", setup="from rust import get_fibonacci")</b>
1.1281064000068
>>>
>>>
>>> <b>timeit.timeit("get_fibonacci(150)", setup="from fib import get_fibonacci")</b>
9.057604000001447
>>> <b>timeit.timeit("get_fibonacci(150)", setup="from rust import get_fibonacci")</b>
3.5204217999998946
</pre>

The above tells us that when we call the function to calculate the 5-th Fibonacci number, Python is faster. But when we look for the 150th Fibonacci number, Rust is almost three times faster. 

But it gets better. 

We can also do a release build by adding `--release` as an argument to maturin:

<pre>
(.env) root@rust:/pyo3/pyo3# <b>python -i</b>
Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.    
>>> <b>import timeit</b>
>>> timeit.timeit("get_fibonacci(5)", setup="from fib import get_fibonacci")
0.4583319000012125
>>> timeit.timeit("get_fibonacci(5)", setup="from rust import get_fibonacci")
0.11867309999797726
>>> timeit.timeit("get_fibonacci(150)", setup="from fib import get_fibonacci")
8.990601400000742
>>> timeit.timeit("get_fibonacci(150)", setup="from rust import get_fibonacci")
0.15236040000309004
>>>
</pre>

With the release build, is a lot faster in both cases. 

I have also compared a lot of other functions. Scanning a text for a substring, manipulating texts, summing up numbers and a variety of other things. I have found that this type of performance increase is not that typical. Getting things to speed up 2x or 3x is usually not that hard. 

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

When we do a performance comparison, we can see that it only makes sense to try and speed things up if the function has to perform a lot of computations. The following example shows the difference in performance with a small input to the function:

```python
>>> timeit.timeit("rust.list_sum(a_list)", setup="""
... import rust
... a_list = [x for x in range(1,10)]
... """)
0.42956949999643257
>>>
>>> timeit.timeit("sum_list(a_list)", setup="""
... from __main__ import sum_list
... a_list = [x for x in range(1,10)]
... """)
0.4579178999993019
```

Hardly any difference. Now, when we increase the function input to 3.000 numbers, we can start seeing some real advantage over using the Rust function:

```python
>>> timeit.timeit("sum_list(a_list)", setup="""
... from __main__ import sum_list
... a_list = [x for x in range(1,3000)]
... """)
168.12326449999819
>>> timeit.timeit("rust.list_sum(a_list)", setup="""
... import rust
... a_list = [x for x in range(1,3000)]
... """)
95.2027356000035
>>>
```


Key takeaway is that how much you'll be able to speed things up really depends on what part of the code you outsource from Python to Rust. I'll skip any further comparisons between Rust and Python and focus on a few more scenario's that I think are worthwhile.
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

Note that the HashMap in Rust is very different from a Python dictionary. Given the fact that we defined the HashMap to use Strings for both key as well as value, it will fail in case we use something else in our Python dictionary. There is nothing that will try and coerce a type into another:

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

The following function will print a word several times. Additionally, it can also print the word in reverse and/or in uppercase:

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

One way to send over complex data structures could be by using JSON. The following example marshals a JSON-string into a struct:

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

Note, `pydantic` is one of my favourite Python packages. The `.json()` method I used here convert the class to a JSON string.

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

Then, after we define a logger in Python, we can use it in Rust:

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

In this last example, we will raise an error in Rust and catch that as an exception in Python. This part is quite involved, the steps I took are shown after the code snippet:

```rust
use std::fmt;

// 1
#[derive(Debug)]
struct MyError {
    /*
    the 'message' field that is used later on
    to be able print any message.
    */
    pub msg: &'static str,
}

// 2
impl std::error::Error for MyError {}

// 3
impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Error from Rust: {}", self.msg)
    }
}

// 4
impl std::convert::From<MyError> for PyErr {
    fn from(err: MyError) -> PyErr {
        PyOSError::new_err(err.to_string())
    }
}

#[pyfunction]
// 5
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
    // 6
    m.add_function(wrap_pyfunction!(greater_than_2, m)?)?;    

    Ok(())
}
```

1: We define 'MyError' as a custom Error. The struct has a field that is used to be able to send a custom message.
2: The 'Error' trait is implemented for 'MyError'.
3: We implement the 'Display' trait for 'MyError' and have it display the message from the 'msg' field in the 'MyError' struct.
4: The 'From' trait is implemented for 'MyError'. This trait is used to do value-to-value conversions while consuming the input value.
5: We create a function called 'greater_than_2'. This function will raise the error/exception in case the input value is 2 or less.
6: We add the function to the Python module in the 'regular' way.

Now we move to the Python side and run the function, triggering the excpetion:

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

It took some code, but in my opinion, not that hard to use after having it setup.

## Closing thoughts.

Working with PyO3 and using it to call Rust from Python is a very pleasant experience. I find that the ergonomics that have been put in place make everything a lot easier when compared to relying on `ffi` or `libc`. The macros offer a lot of convenience, the predefined types are very convenient and the build tool `maturin` is an absolute joy to work with.

The repo is actively maintained and the documentation is great.

{:refdef: style="text-align: center;"}
![PyO3 logo](https://github.com/saidvandeklundert/learn/blob/pyo3_post/img/pyo3_small.png "PyO3 logo"){:height="80%" width="80%"}
{: refdef}


- [code samples from this blog](https://github.com/saidvandeklundert/pyo3)
- [PyO3 repo](https://github.com/PyO3/pyo3)
- [PyO3 user guide](https://pyo3.rs/main/)
- [PyO3 architecture guide](https://github.com/PyO3/pyo3/blob/main/Architecture.md)
- [Maturin repo](https://github.com/PyO3/maturin)



