---
layout: post
comments: true
title: "Python - C++ bindings"
excerpt: "Python - C++ bindings are useful for several reasons. Performance is one of them. Exposing existing C++ classes to a python module is another important reason."
date:   2017-12-30 22:00:00
mathjax: true
image_url: "/assets/2017-12-30-python-cpp/dl-cpp-vs-python.jpg"
---

* TOC
{:toc}

I love python. It's just [so easy to develop anything in python](https://xkcd.com/353/). More often than not, all I need to do is add `import module` and few more lines of python code. I come from a systems programming background. Before python, I had only used strongly typed langauges. Coming from that background, python seems like magic.

Using python's easy interface is fun. It's often not as much fun to develop that interface though. It's not easy to write efficient and robust modules in python. It's easy to see why. While a lot of modules can be implemented in pure python, they often lag in [performance](https://benchmarksgame.alioth.debian.org/u64q/compare.php?lang=python3&lang2=gpp). If all you care about is performance from your code, one option is to simply write C++ code and use it in other C++ modules. But, it's really painful to go back to doing everything in C++ after using python for a while.

<img src="/assets/2017-12-30-python-cpp/dl-cpp-vs-python.jpg">

**Python-C++ binding** is a nice compromise that delivers the best of both worlds. We get to use Python's easy interface and we get to enjoy the performance benefits of C++. Let's review few of the options.

## CPython
The official python implementation is called **CPython**. I guess that's because the python core is implemented in C. There are other variants of python available out there: [PyPy](https://pypy.org/), [Jython](http://www.jython.org/), [IronPython](http://ironpython.net/). However, most people usually mean CPython when they say they wrote a python program. **CPython** exposes a C API to write modules in C. This is the most native way to add an optimized C/C++ module in python.

This is how most of the [python's core modules](https://github.com/python/cpython/tree/master/Modules) are written.

### Example module with functions

Let's consider the following example:
We want to create an `example` module containing a `pants()` function.
```
example module
|-> pants() function
```

First things first. Let's add the following line in our `example-pants.cpp` file.
```cpp
#include <Python.h>
```

It will import everything needed to define our `example` module. The most important part is `PyObject`. Everything in python is a `PyObject`. That includes everything from [`int`](https://docs.python.org/3/c-api/long.html#c.PyLongObject), `str`, `list`, `dict` and [`None`](https://docs.python.org/3/c-api/none.html#c.Py_None).

Next is `PyInit_example` function. It has to be in the format **PyInit_{module_name}**. `PyModule_Create` function creates a module object. Module is also a `PyObject`. We haven't yet defined `example_definition` - it's simply a struct listing module properties including module functions.

```cpp
PyMODINIT_FUNC PyInit_example(void) {
  Py_Initialize();
  PyObject *m = PyModule_Create(&example_definition);

  return m;
}
```

<img src="/assets/2017-12-30-python-cpp/pyinit_example.jpg">

Let's define our `example_definition` struct now!
```cpp
static PyMethodDef example_methods[] = {
    {"pants", pants, METH_VARARGS, "Returns a square of an integer."},
    {NULL, NULL, 0, NULL}
};

static struct PyModuleDef example_definition = {
    PyModuleDef_HEAD_INIT,
    "example",
    "A Python module containing Classy type and pants() function",
    -1,
    example_methods
};
```

<img src="/assets/2017-12-30-python-cpp/pymoduledef.jpg">

Next is creating our `pants` function in C/C++:
```cpp
#include <Python.h>

static PyObject *pants(PyObject *self, PyObject *args) {
  int input;
  if (!PyArg_ParseTuple(args, "i", &input)) {
    return NULL;
  }

  return PyLong_FromLong((long)input * (long)input);
}
```

<img src="/assets/2017-12-30-python-cpp/pants_function.jpg">

Putting everything together:
```cpp
#include <Python.h>

static PyObject *pants(PyObject *self, PyObject *args) {
  int input;
  if (!PyArg_ParseTuple(args, "i", &input)) {
    return NULL;
  }

  return PyLong_FromLong((long)input * (long)input);
}

static PyMethodDef example_methods[] = {
    {"pants", pants, METH_VARARGS, "Returns a square of an integer"},
    {NULL, NULL, 0, NULL},
};

static struct PyModuleDef example_definition = {
    PyModuleDef_HEAD_INIT,
    "example",
    "example module containing pants() function",
    -1,
    example_methods,
};

PyMODINIT_FUNC PyInit_example(void) {
  Py_Initialize();
  PyObject *m = PyModule_Create(&example_definition);

  return m;
}
```

Our **setup.py** would look something like this:
```python
from distutils.core import setup, Extension

example_module = Extension(
    'example',
    sources=['example-classy.cpp'],
    language='C++', )

setup(
    name='example',
    version='0.1.0',
    description='example module written in C++',
    ext_modules=[example_module], )
```

After creating **setup.py** and **example-pants.cpp**, this is how we would install the `example` package:
```bash
python3 setup.py install
```

However, I often prefer to use:
```bash
pip3 install -e .
```

Anyway, you can find this code and more information here - https://github.com/hardikp/pycpp/tree/master/cpython.

### Example module with classes
Let's try to add a class `Classy` in our example module. `Classy` class contains two methods: `miami()` and `new_york()`. `miami()` decrease an integer class variable and `new_york()` increases it.
```
example module
|-> Classy class

Classy class
|-> miami() method
|-> new_york() method
```

Adding a new type to a module requires more work. Since C doesn't have classes, CPython takes a longer route to adding classes. We need to define a **struct** - that's our class. In addition to that, we will define our methods `miami()` and `new_york()` as **static functions**. We also need to define the allocator, initializor and destructor as static functions. 

Let's first define the struct for our `Classy` class.
```cpp
typedef struct {
  PyObject_HEAD // no semicolon
  int number;
} Classy;
```

`PyObject_HEAD` is a macro pointing to the base class `PyObject`. `number` is the class variable we're adding for our use case.

We need to create a NULL-terminated array of PyMemberDef structs: `Classy_members`. We will list down the data members of our `Classy` type.
```cpp
static PyMemberDef Classy_members[] = {
    {"number", T_INT, offsetof(Classy, number), 0, "classy number"},
    {NULL} /* Sentinel */
};
```

Next step is to define the init/delete methods. `Classy_dealloc` frees the memory. If we had `PyObject *` members in our `Classy` struct, we would need to free that memory (i.e. decrease the ref count) before calling `tp_free` of the type object.
```cpp
static void Classy_dealloc(Classy *self) {
  Py_TYPE(self)->tp_free((PyObject *)self);
}

static PyObject *Classy_new(PyTypeObject *type, PyObject *args,
                            PyObject *kwds) {
  Classy *self;

  self = (Classy *)type->tp_alloc(type, 0);
  if (self != NULL) {
    self->number = 0;
  }

  return (PyObject *)self;
}

static int Classy_init(Classy *self, PyObject *args, PyObject *kwds) {
  self->number = 1;
  return 0;
}
```

<img src="/assets/2017-12-30-python-cpp/classy_new_init_del.jpg">

Next step is to define methods for the `Classy` type. We are defining two methods in the `Classy` class: `miami()` and `new_york()`. `PyMethodDef` struct at the end is listing those methods - this is how we will let python and compiler know about Classy methods.
```cpp
static PyObject *Classy_miami(Classy *self) {
  if (self->number > 1)
    self->number /= 2;

  return PyLong_FromLong((long)self->number);
}

static PyObject *Classy_new_york(Classy *self) {
  if (self->number < 1024 * 1024)
    self->number *= 2;

  return PyLong_FromLong((long)self->number);
}

static PyMethodDef Classy_methods[] = {
    {"miami", (PyCFunction)Classy_miami, METH_NOARGS, "Divides number by 2"},
    {"new_york", (PyCFunction)Classy_new_york, METH_NOARGS, "Doubles number"},
    {NULL} /* Sentinel */
};
```

<img src="/assets/2017-12-30-python-cpp/classy_methods.jpg">

After all of that is done, we define the following struct - this is similar to the `PyModuleDef` struct from the previous section. Several redundant members of that struct are set to NULL, becase we have not defined customized components.

```cpp
static PyTypeObject ClassyType = {
    PyVarObject_HEAD_INIT(NULL, 0) "example.Classy",  /* tp_name */
    sizeof(Classy),                           /* tp_basicsize */
    0,                                        /* tp_itemsize */
    (destructor)Classy_dealloc,               /* tp_dealloc */
    0,                                        /* tp_print */
    0,                                        /* tp_getattr */
    0,                                        /* tp_setattr */
    0,                                        /* tp_reserved */
    0,                                        /* tp_repr */
    0,                                        /* tp_as_number */
    0,                                        /* tp_as_sequence */
    0,                                        /* tp_as_mapping */
    0,                                        /* tp_hash  */
    0,                                        /* tp_call */
    0,                                        /* tp_str */
    0,                                        /* tp_getattro */
    0,                                        /* tp_setattro */
    0,                                        /* tp_as_buffer */
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE, /* tp_flags */
    "Classy objects",                         /* tp_doc */
    0,                                        /* tp_traverse */
    0,                                        /* tp_clear */
    0,                                        /* tp_richcompare */
    0,                                        /* tp_weaklistoffset */
    0,                                        /* tp_iter */
    0,                                        /* tp_iternext */
    Classy_methods,                           /* tp_methods */
    Classy_members,                           /* tp_members */
    0,                                        /* tp_getset */
    0,                                        /* tp_base */
    0,                                        /* tp_dict */
    0,                                        /* tp_descr_get */
    0,                                        /* tp_descr_set */
    0,                                        /* tp_dictoffset */
    (initproc)Classy_init,                    /* tp_init */
    0,                                        /* tp_alloc */
    Classy_new,                               /* tp_new */
};
```

<img src="/assets/2017-12-30-python-cpp/pytypeobject.jpg">

After all of the above is done, this is how we add the type we just defined to our `example` module:
```cpp
PyMODINIT_FUNC PyInit_example(void) {
  Py_Initialize();
  PyObject *m = PyModule_Create(&example_definition);

  if (PyType_Ready(&ClassyType) < 0)
    return NULL;

  Py_INCREF(&ClassyType);
  PyModule_AddObject(m, "Classy", (PyObject *)&ClassyType);

  return m;
}
```

<img src="/assets/2017-12-30-python-cpp/pymoduledef_classy.jpg">

You can find the complete example [here](https://github.com/hardikp/pycpp/blob/master/cpython/example-classy.cpp).

### Notes
* Almost everything shown above is **python3** specific.
* We have not explored memory management in this section. Python keeps track of the reference count for each object. Whenever the reference count comes back to 0, that object is freed from the memory. `Py_INCREF`, `Py_DECREF` and `Py_XDECREF` are the useful macros for this purpose.
* There are 2 main ways of extracting arguments in functions. https://docs.python.org/3/extending/extending.html#extracting-parameters-in-extension-functions
  1. `METH_VARARGS`: This is to process the arguments by position. `PyArg_ParseTuple()` is used to parse the arguments.
  1. `METH_KEYWORDS`: This is to process the arguments by keywords. `PyArg_ParseTupleAndKeywords()` is used to parse the arguments.
  1. `METH_NOARGS`: For functions without any arguments.
* We also skipped `list`, `tuple` and `dict` processing.

### Links
* https://github.com/hardikp/pycpp/tree/master/cpython
* https://docs.python.org/3/extending/extending.html - Getting started guide
* https://docs.python.org/3/c-api/index.html - Python C API
* https://docs.python.org/3/extending/newtypes.html
* https://gist.github.com/killeent/4675635b40b61a45cac2f95a285ce3c0 - Describes how [PyTorch](http://pytorch.org/) uses C API bindings.

## pybind11

`pybind11` is extremely simple to use. A major hurdle in adding python-C++ bindings is learning a whole new framework of APIs. `CPython`, `Cython` and almost all other options require you to learn the underlying APIs. The biggest advantage of `pybind11` is the ability to write normal C++ code and simply use it to export types and functions from a python module. It's the easiest framework to add C++ bindings.

Let's get started!

We don't need to worry about `PyObject` and other C APIs here. All we need to do is write normal C++ code and use `PYBIND11_MODULE` to define the module.

```cpp
#include <pybind11/pybind11.h>

int pants(int i) {
  // Return the square
  return i * i;
}

namespace py = pybind11;

PYBIND11_MODULE(example, m) {
  m.doc() = R"pbdoc(
        example module
        -----------------------
        .. currentmodule:: example
        .. autosummary::
           :toctree: _generate
           pants
    )pbdoc";

  m.def("pants", &pants, R"pbdoc(
        Returns the square of a number
    )pbdoc");

  m.attr("__version__") = "dev";
}
```

### Links
* https://github.com/hardikp/pycpp/tree/master/pybind11
* https://github.com/pybind/python_example
* http://pybind11.readthedocs.io/en/latest/

{% comment %}

## Boost Python

**Insert Basic image**

https://github.com/davisking/dlib/issues/293#issuecomment-285893491

## Other options

### Cython

### Numba

### Swig

### pypy

### CFFI

An additonal layer of indirection - https://www.reddit.com/r/Python/comments/6cific/do_you_use_cython/

https://www.reddit.com/r/Python/comments/6cific/do_you_use_cython/dhvnbcj/

https://www.reddit.com/r/MachineLearning/comments/767tb3/d_are_python_speedup_libraries_numba_cython_worth/

https://www.reddit.com/r/Python/comments/6df8gb/python_and_c_interoperability_using_boostpython/

**Insert Cython image**

**Insert Numba image**

**Insert Swig image**

**Insert pypy image**

## Comparisons

## Examples

### PyTorch
https://gist.github.com/killeent/4675635b40b61a45cac2f95a285ce3c0

http://pytorch.org/2017/06/27/Internals2.html

### Tensorflow

https://gist.github.com/Mistobaan/738e76c3a5bb1f9bcc52e2809a23a7a1
{% endcomment %}
