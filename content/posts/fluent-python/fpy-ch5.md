---
date: "2025-06-26"
title: "Fluent Python Chapter 5: Data Class Builders"
author: "Yukihide Takahashi"
summary: ""
readTime: true
toc: true
---

A "Data class" is a coding pattern featuring a collection of fields with little or no extra functionality.

This chapter covers three data class builders: `collections.namedtuple`, `typing.NamedTuple` and `@dataclasses.dataclass`

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

Next we consider the same Coordinate class but built with `collections.namedtuple`:

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

See page 165 for an example using `typing.NamedTuple`.
