---
layout: post
title: Underscores in Python
description: Underscores in Python.
summary: Underscores in Python.
tags: [python, underscore, inheritance, name mangling]
permalink: "/python/UnderscoresInPython"
---

#### Keywords: Leading/Trailing Underscore, Name Mangling

### Double Underscore Name Mangling: __var

Python has no privacy model and there are no truly `protected` or `private` attributes. Names with a leading double-underscore and no trailing double underscore are mangled to protected them from clash when inherited

[Document on Identifiers (Names)](https://docs.python.org/3.6/reference/expressions.html#atom-identifiers)
> **Private name mangling**: When an identifier that textually occurs in a class definition begins with two or more underscore characters and does not end in two or more underscores, it is considered a private name of that class. Private names are transformed to a longer form before code is generated for them. The transformation inserts the class name, with leading underscores removed and a single underscore inserted, in front of the name. For example, the identifier `__spam` occurring in a class named `Ham` will be transformed to `_Ham__spam`. This transformation is independent of the syntactical context in which the identifier is used.

```python
class Parent(object):
    def _protected(self):
        pass

    def __private(self):
        pass

class Child(Parent):
    def foo(self):
        self._protected()

    def bar(self):
        self._Parent__private()
```

We can look at the attributes on the `Parent` object with the built-in `dir()` function:

```python
>>> p = Parent()
>>> dir(p)
['_Parent__private', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__',
'__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__',
'__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__',
'__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__',
'__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_protected']
```

### Single Leading Underscore: _var

```python
# my_module.py

def external_func():
    return "external_func"

def _internal_func():
    return "internal_func"
```

If we import with wildcard, Python will skip names with a leading underscore (unless the module defines an `__all__` list that overrides this behavior):

```python
>>> from my_module import *
>>> external_func()
external_func
>>> _internal_func()
NameError: "name '_internal_func' is not defined"
```

However, regular imports are not affected by the leading single underscore naming convention:

```python
>>> import my_module
>>> my_module.external_func()
external_func
>>> my_module._internal_func()
internal_func
```

### Single Trailing Underscore: var_

If the name is already taken by a Python keyword, then we can append a single underscore to break the naming conflict:

```python
>>> def func(class):
SyntaxError: "invalid syntax"
>>> def func(class_):
...     pass
```
