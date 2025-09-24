---
date: "2025-09-24"
title: "Fluent Python Chapter 5: Data Class Builders"
author: "Yukihide Takahashi"
summary: "Ways to represent data, and anti-patterns"
readTime: true
toc: true
---

A "Data class" is a coding pattern featuring a collection of fields with little or no extra functionality.

This chapter covers three data class builders: `collections.namedtuple`, `typing.NamedTuple` and `@dataclasses.dataclass`

`Data Class` is also the name of a code smell, which will be discussed later.

## Overview of Data Class Builders

Consider the following `Coordinate` class:

```python
class Coordinate:
    def __init__(self, lat, lon):
        self.lat = lat
        self.lon = lon
```

Notice that if we were to add another attribute, we will need to mention it three times.

Also string representation and comparison between objects do not work the way we expect:

```python
>>> moscow = Coordinate(55.76, 37.62)
>>> moscow
<__main__.Coordinate object at 0x104868590>
>>> location = Coordinate(55.76, 37.62)
>>> location == moscow
False
>>> (location.lat, location.lon) == (moscow.lat, moscow.lon)
True
```

The data class builders considered in this chapter provide the necessary `__init__`, `__repr__` and `__eq__` methods.

For example, here is the same Coordinate class but built with `collections.namedtuple`:

```python
from collections import namedtuple
Coordinate = namedtuple('Coordinate', 'lat lon')
>>> moscow = Coordinate(55.76, 37.62)
>>> moscow
Coordinate(lat=55.76, lon=37.62)
>>> moscow == Coordinate(lat=55.76, lon=37.62)
True
```

`__repr__` and `__eq__` are meaningful here.

Whereas `typing.NamedTuple` will add a type annotation to each field:

```python
>>> import typing
>>> Coordinate = typing.NamedTuple('Coordinate',
...     [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```

A more readable option is to give the type annotation as keyword arguments, like

`Coordinate = typing.NamedTuple('Coordinate', lat=float, lon=float)`

Also, `typing.NamedTuple` can be used in a class statement, with type annotations written in PEP526 syntax.

```python
from typing import NamedTuple
class Coordinate(NamedTuple):
    lat: float  # <- PEP 526 syntax
    lon: float
```

Here's the same class written with the `@dataclass` decorator.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float
```

Note that in the `typing.NamedTuple` example, what seems to be inheritance is actually usage of a metaclass, and
in the `@dataclass` example, we use a class decorator.

Both are ways of custimizing class behavior beyond what is offered by inheritance. See Chapter 24 'Class Metaprogramming' for more information.

### Main Features

See page 167 for a table comparing the characteristics of the three data class builders.

A discussion of each feature:

1. Mutability

`collections.namedtuple` and `typing.NamedTuple` build `tuple` subclasses, and therefore the instances are immutable.

When using `@dataclass`, the instances of the class are mutable by default, but the decorator accepts a keyword argument `frozen`. By setting `frozen=True`, the class will raise an exception if we try to assign a value to a field after the instance has been created.

2. Class statement syntax

`typing.NamedTuple` and `@dataclass` support the `class` syntax, making it easier to add methods and docstrings to the class definition.

3. Constructing a dict

For `collections.namedtuple` and `typing.NamedTuple`, we have an instance method `._asdict()` which will construct a
`dict` object from the fields in the data class instance.

For `dataclasses` module provides the same functionality via the function `dataclasses.asdict()`.

4. Getting field names and default values

All three data class builders provide methods to get the field anmes and defailts values that may be configured.

- `collections.namedtuple`: the `._fields` and `._fields_defaults` class methods
- `typing.NamedTuple`: the `._fields` and `._fields_defaults` class methods
- `@dataclass`: the `dataclasses.fields` function

5. Getting field types

For `typing.NamedTuple` and `@dataclass` classes, we can use the `typing.get_type_hints` function to get the mapping of field names to type.

6. New instance with changes

Calling `x._replace(**kwargs)` on a named tuple instance x or `dataclasses.replace(y, **kwargs)` on an instance of a `@dataclass` decorated class y will return a new instance with some attribute values replaced according to the `kwargs` given.

7. New class at runtime

The `class` statement syntax is readable but hardcoded.

To build data classes on the fly, at runtime, we can use the default function call syntax for named tuples: `collections.namedtuple()` and `typing.NamedTuple()`, and the `dataclasses.make_dataclass()` function for `@dataclass` classes.

Given this overview of data class builders, we now focus on each of them.

## Classic Named Tuples

The `collections.namedtuple()` function builds subclasses of `tuple` with field names, a class name, and a useful `__repr__` method.

An example of a data class about a city.
