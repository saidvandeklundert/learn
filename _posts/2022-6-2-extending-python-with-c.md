---
layout: post
title: Extending Python with C 
subtitle: speed up your Python using C
image: /assets/img/python-logo.jpg
cover-img: /assets/img/python-logo.jpg
thumbnail-img: /assets/img/python-logo.jpg
share-img: /assets/img/python-logo.jpg
tags: [ python, C ]
---

A good deal of Python, or at least [CPython](https://github.com/python/cpython), is written in C. The language and the interpreter are implemented in C and in addition to this, you can also find a lot of modules in the Python standard library powered by C. 

My desire to read some of the innards of the Python language and to create C extensions lead me to pick up a copy of 'The C programming language'.

{:refdef: style="text-align: center;"}
![The C programming language](/learn/img/c_programming_language_book.jpg "The C programming language"){:height="40%" width="40%"}
{: refdef}

In this article, I will create a basic C extension using `ctypes` and the `Python C API` and share some of the things I learned.


# Why extend Python with C?


C is a compiled language that is very fast and efficient. In some cases, the speed of a Python program can be increased more then 50x. Additionally, using a C extension module can unlock some of the other advantages that C brings. To name a few in addition to speeding things up:
- C gives you low-level control over the hardware and memory allocations (which is also a bit of a downside)
- there is no GIL
- there are a lot of other C libraries that you will be able to leverage 
 
For me personally though, the only reason I have is getting more familiar with C and CPython. If I _really_ had to extend Python to speed things up, I would probably turn to Rust.


# How to extend Python with C?

There are multiple options available to call C code from Python. The options that come with CPython by default are the following:
- [ctypes](https://docs.python.org/3/library/ctypes.html#module-ctypes)
- [Python API](https://docs.python.org/3/c-api/intro.html)

In addition to these 2 options there is also the 'cffi', or 'C Foreign Function Interface' for Python. This package needs to be installed before you can use it and you can find more information on cffi [here](https://cffi.readthedocs.io/en/latest/index.html).


In this article, I will give a short example on how to call a C function using ctypes. After that, I will move on to using the Python API and give a detailed walkthrough. In closing, I will list some interested docs and things to checkout as next steps.


#  Extending Python with C using ctypes


When we use `ctypes`, we can write some C, compile it and then import the compiled C into our Python.

Let's look at an example function in a file called `square.c`:

```c
int square(int i)
{
    return i * i;
}
```

Before we can call this function from our Python code, we need to compile it. When we compile the code, we need to pass in a flag so that the produced file can be imported. To compile the C code, we run the following command:

<pre>
gcc -shared -o clib.so square.c
</pre>

This will output a file called `clib.so` which we can import into our Python script. Let's put a `square.py` script in the same directory:

```python
import ctypes

c_lib = ctypes.CDLL("./clib.so")

print(c_lib.square(2))
```

What happens in the above is we import `ctypes` so that we can use `ctypes.CDLL` to load the shared object library as a module in Python. The loaded 'clib.so' 'module' is stored in the `c_lib` variable. The C functions, or in our case function, can then be called like so: `c_lib.square()`.



# Extending Python with C using the Python API

The Python API is usable from C and C++. It is a maintained feature of Python and it is documented [here](https://docs.python.org/3/c-api/index.html). Interestingly enough, CPython also leverages this very same API. If you want to, you can check out the source code for the built-ins for instance right [here](https://github.com/python/cpython/blob/main/Python/bltinmodule.c). Reading it is not for the faint of heart though. This is a whopping 3.000 lines of code at the time of writing. And even though the next example I will walk through next is a little lighter, it does follow a similar flow.


# Python API example

What we will end up having is a C extension module called `c_extension` with a function called `multiplier`. We will end up being able to run the following code:

```python
>>> from c_extension import multiplier
>>> multiplier(2,2)
4
>>> multiplier(200,200)
40000
```


There are a number of steps to follow before you can call a C function from Python using the Python API. These are the steps I followed:
- include the proper header file (`Python.h`)
- write the C function
- put the function in a PyObject
- add the PyObject to the 'PyMethodDef' array
- define the 'PyModuleDef' struct
- define 'PyMODINIT_FUNC' so that the module can be initialized
- configure `setup.py` to compile and install the C-extension as a module



Some things about the C-extension up front:
- the C extension will be 1 file called `c_extension.c`
- the Python setup file will be used to build the extension and install it as a module called `c_extension`
- the function that we will expose from C to Python will be called `multiplier`

First, I will present the entirety of the C-code and then, I will walk through each of the steps.


## The C code


Our C-extension will provide a function that will be called `multiplier`. The C code for the extension is the following:

```c
#define PY_SSIZE_T_CLEAN
#include <Python.h>

int multiplier(int a, int b)
{
    return a * b;
}

static PyObject *c_multiplier(PyObject *self, PyObject *args)
{
    int a;
    int b;
    int ret;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
    {
        return NULL;
    }
    ret = multiplier(a, b);
    return Py_BuildValue("i", ret);
}

static PyMethodDef module_methods[] = {
    {"multiplier", c_multiplier, METH_VARARGS, "Multiply two numbers."},
    {NULL, NULL, 0, NULL}};

static struct PyModuleDef c_extension =
    {
        PyModuleDef_HEAD_INIT,
        "c_extension", // the name of the module in Python
        "",            // The docstring in Python
        -1,            /* size of per-interpreter state of the module, or -1 if the module keeps state in global variables. */
        module_methods};

PyMODINIT_FUNC PyInit_c_extension(void)
{
    return PyModule_Create(&c_extension);
}
```

That is a lot to take in, let's step through this code in the order that I wrote it.

### Including the proper header file

At the top of the file, we include the proper header file:

```c
#define PY_SSIZE_T_CLEAN
#include <Python.h>
```

This header file contains C declarations and macros. Basically, this pulls in the C-API. In this case, the `python.h` header file contains a lot of declarations that are relevant for us to be able to use the Python API.

Directing the C preprocessor to include this header file is like pasting in the code itself.

This header file is located in the `include` directory where you have installed Python. A good idea would be to point your IDE to also use header files from this location. This way, you get some information when you hover over the structs and functions etc. On the same note, another good idea for when you like to explore the CPython source code, is to clone the CPython repo and browse the code base using your IDE. This will make it easier to see how the whole code base is glued together.


### Writing the C function

The following is the actual C function that we will be able to call from Python:

```c
int multiplier(int a, int b)
{
    return a * b;
}
```

Lame, but the point here is sharing how to extend Python.

### Creating the PyObject

Next up is the PyObject. The PyObject is the base struct of which all other Python objects are an extension. Here we define what the function receives and returns from Python:

```c
static PyObject *c_multiplier(PyObject *self, PyObject *args)
{
    int a;
    int b;
    int ret;
    if (!PyArg_ParseTuple(args, "ii", &a, &b))
    {
        return NULL;
    }
    ret = multiplier(a, b);
    return Py_BuildValue("i", ret);
}
```

The first three lines define the integers 'a', 'b' and 'ret'. The values of 'a' and 'b' will be passed to the `multiplier` function, and the `ret` value will hold the return value of the `multiplier` function.

After our declarations, we see a call to `PyArg_ParseTuple()`. This will parse the arguments received from Python (`args`). The `ii` specifies that we are expecting 2 integer values. The `&a` and `&b` are the addresses that will be populated with the parsed values. These are pointing to the variables we declared earlier.

In case you want to use values other than integer values in your C extension, you can check the [PyArg_ParseTuple arguments](https://docs.python.org/3/c-api/arg.html) to understand what arguments you need to pass.

After this, we get to the last part:

```c
    ret = multiplier(a, b);
    return Py_BuildValue("i", ret);
```
On the first line, we run the 'multiplier' function and store the return value in `ret`. After this, we see the return statement that returns the value in `ret` to the Python runtime. Since the return type of the C function is not something that Python understands, we use `Py_BuildValue()`, which is the counterpart of `PyArg_ParseTuple`. Where `PyArg_ParseTuple` translates something from Python to C, `Py_BuildValue` does the opposite. In our case, we indicate that we will return the value in `ret` as a Python integer object.


### Adding the PyObject to the 'PyMethodDef' array

Next up is putting everything we want to expose to Python in the 'method table', or `PyMethodDef`:

```c
static PyMethodDef module_methods[] = {
    {"multiplier", c_multiplier, METH_VARARGS, "Multiply two numbers."},
    {NULL, NULL, 0, NULL}};
```

In the above example, `module_methods` is a static array containing `PyMethodDef` structures. Since we only want to expose 1 function, we only put in the following entry:

```c
{"multiplier", c_multiplier, METH_VARARGS, "Multiply two numbers."}
```

The above values are explained as follows:
- multiplier: the name the function will have in Python
- c_multiplier: the PyObject we created that calls the C function
- METH_VARARGS: bitfield, indicating the calling convention
- "Multiply two numbers.": the docstring for the Python function

More information can be found [here](https://docs.python.org/3/c-api/structures.html#c.PyMethodDef).

The other line, `{NULL, NULL, 0, NULL}`, is a sentinel value, indicating that the preceding value was the last relevant item in the array.


### Defining the 'PyModuleDef' struct

Now, we define the `PyModuleDef` struct, which will describe our Python C extension module:

```c
static struct PyModuleDef c_extension =
    {
        PyModuleDef_HEAD_INIT,
        "c_extension", // the name of the module in Python
        "",            // The docstring in Python
        -1,            /* size of per-interpreter state of the module, or -1 if the module keeps state in global variables. */
        module_methods};
```

The [doc](https://docs.python.org/3/c-api/module.html) says to always initialize the first member of this struct to 'PyModuleDef_HEAD_INIT', so I did that (without being able to figure out why).

The other lines are annotated and the final field is a pointer to the method table we defined earlier. This was the array of `PyMethodDef`, which contains all the functions (or 1 function in our case) that we want to expose to Python.


### Defining 'PyMODINIT_FUNC' so that the module can be initialized

Lastly, we define the `PyMODINIT_FUNC` function, which is called when Python imports the extension module. This function returns the module struct that we defined previously:

```c
PyMODINIT_FUNC PyInit_c_extension(void)
{
    return PyModule_Create(&c_extension);
}
```

Ensure that the `PyMODINIT_FUNC` is correctly named. The last part should be the same as the module name (in this case, `c_extension`). 


### Configuring setup.py to compile and install the C-extension module

Creating a `setup.py` will make things easier as you will be able to 'pip install' it and make it available to your local instance of Python. I put in place the following:

```python
from distutils.core import setup, Extension

module = Extension("c_extension", sources=["c_extension.c"])

setup(
    name="c_extension",
    version="0.1",
    description="An example of C extension made callable to the Python API.",
    ext_modules=[module],
)
```

After cloning the repo, all I had to do is the following:

<pre>
pip install src/
</pre>

This will compile the source file and install the C extension module. You will see the packages installed when you run `pip freeze`.

We can now run the test script and observe the following:

<pre>
python .\test.py
100
</pre>

Awesome!

You can go [here](https://github.com/saidvandeklundert/c_extension) to play with the example yourself.


# Next steps

In case you are interested in creating more serious C extension for Python, I think the official Python documentation is quite good, you can find the documentation on the API [here](https://docs.python.org/3/c-api/index.html). Additionally, there is a good talk you could look at [Paul Ross - Here be Dragons PyCon 2016](https://www.youtube.com/watch?v=Yq__HtUIH5Y). This person has put up quite a few examples and patterns for others to learn from right [here](https://pythonextensionpatterns.readthedocs.io/en/latest/index.html).

# Closing thoughts.

This was a nice exercise. I read K&R and played with C. I (tried) reading parts of CPython and I investigated how to extend Python with some C code.

Few personal things I learned:
- C may be a small language, reading and working with C is quite challenging
- the C Python source code is not for the faint of heart
- extending Python with Rust was easier due to the ergonomics put in place by PyO3



# Links relevant to this article:


[c modules](https://github.com/python/cpython/tree/main/Modules)<br>
[header files](https://github.com/python/cpython/tree/main/Include)<br>
[api docs](https://docs.python.org/3/c-api/index.html)<br>
[structures](https://docs.python.org/3/c-api/structures.html)<br>
[Extending Python with new types tutorial](https://docs.python.org/3/extending/newtypes_tutorial.html)<br>
[PyArg_ParseTuple arguments](https://docs.python.org/3/c-api/arg.html)<br>
[PEP 7](https://www.python.org/dev/peps/pep-0007/)<br>
[Paul Ross - Here be Dragons PyCon 2016](https://www.youtube.com/watch?v=Yq__HtUIH5Y)<br>
[Paul Ross - A faster Python? You Have These Choices](https://www.youtube.com/watch?v=5js_-pLGqwA)<br>
[Python Extension Patterns](https://pythonextensionpatterns.readthedocs.io/en/latest/index.html)<br>
[CPython internals](https://www.amazon.com/CPython-Internals-Guide-Python-Interpreter/dp/1775093344)<br>
[PyCon 2009 - nedbatchelder](https://nedbatchelder.com/text/whirlext.html)<br>