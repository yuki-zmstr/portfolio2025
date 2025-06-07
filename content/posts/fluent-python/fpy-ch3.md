---
date: "2025-06-07"
title: "Fluent Python Chapter 3: Dictionaries and Sets"
author: "Yukihide Takahashi"
summary: "This chapter surveys the various mappings and sets available in the Standard library."
readTime: true
toc: true
---

## Syntax

### Dictionary comprehensions

`dictcomps` build `dict` instances using `key:value` pairs from any iterable.

For example:

```python
>>> student_ids = [
        (111, 'Yuki'),
        (222, 'David'),
        (333, 'Tom')
    ]

>>> nom_roll = (_id: name for _id, name in student_ids)
>>> nom_roll
{111: 'Yuki', 222: 'David', 333: 'Tom'}
```

### The | operator for map merging

Since Python 3.9, we are able to merge mapping using the `|` and `|=` operators.

```python
>>> d1 = {"Apples": 2,  "Oranges": 3}
>>> d2 = {"Apples": 1,  "Oranges": 5, "Bananas": 3}
>>> d1 | d2
{'Apples': 1, 'Oranges': 5, 'Bananas': 3}
```

Note how the second appearance of "Oranges" overwrites the first.

### Pattern matching with mappings

The `match/case` statament works with mapping patterns as well.

```python
def get_ingredients(food: dict) -> list:
    match food:
        case {'type': 'Fast Food', 'api': 2, 'ingredients': [*ingredients]}:
            return ingredients
        case {'type': 'Fast Food', 'api': 1, 'ingredient': ingredient}:
            return [ingredient]
        case {'type': 'Fast Food'}:
            raise ValueError(f"Invalid 'food' record: {food!r}")
        case {'type': 'Japanese', 'ingredients': [*ingredients]}:
            return ingredients
        case _:
            raise ValueError(f"Invalid record: {food!r}")
```

For `api v2`, a food with `type == 'Fast Food'` should have a list of ingredients.
For `api v1', a food with `type == 'Fast Food'`should have a single ingredient.
A Japanese food should have a key`ingredients`, which is a list of ingredients.

```python
>>> f1 = dict(api=1, type='Fast Food', name='Hamburger', ingredient='Beef')
>>> get_ingredients(f1)
['Beef']
>>> f2 = {'type':'Fast Food', 'ingredient':'Beef'}
>>> get_ingredients(f2)
Traceback (most recent call last):
    ...
    raise ValueError(f"Invalid 'food' record: {food!r}")
ValueError: Invalid 'food' record: {'type': 'Fast Food', 'ingredient': 'Beef'}
>>> f3 = "I am hungry"
>>> get_ingredients(f3)
Traceback (most recent call last):
    ...
    raise ValueError(f"Invalid record: {food!r}")
ValueError: Invalid record: 'I am hungry'
```

Unlike sequence patterns, mapping patterns allow for partial matches. In the example above,
f1 has a key `name`, which does not appear in any `case` statement, yet it matches.

You can also capture extra key-value pairs using `\*\*`.

```python
>>> food = {'type': 'Fast Food', 'name': 'Hamburger', 'price': '$10.00', 'calories': '700kca\
l'}
>>> match food:
...     case {'type': 'Fast Food', **details}:
...         print(f'Fast food details: {details}')
...
Fast food details: {'name': 'Hamburger', 'price': '$10.00', 'calories': '700kcal'}
```

## Standard API of Mapping Types

The `collections.abc` module offers the `Mapping` and `MutableMapping` ABCs.

These serve as the standard interfaces for mappings, and as criteria for `isinstance` tests in code.

```python
>>> import collections.abc as abc
>>> d = {}
>>> isinstance(d, abc.Mapping)
True
>>> isinstance(d, abc.MutableMapping)
True
```

Using the `isinstance` test with an ABC is often better than checking whether a function argument is of the concrete type `dict` so that alternative mapping types can be used. (More details in Chapter 13)

Whem implementing a custom mapping, the author recommends extending `collections.UserDict` or to wrap a `dict` by composition, instead of subclassing these ABCs. All concrete mapping classes including `collections.UserDict` encapsulate the basic `dict` in their implementation, which means all keys must be _hashable_.

Note that in Python, an object is hashable if it has a hash code that never changes during its lifetime (implements a `\_\_hash\_\_()` method), and can be compared to other objects (implements a `\_\_eq\_\_()` method).

In practice, custom implementations of `\_\_hash\_\_()` and `\_\_eq\_\_()` should only take into account instance attributes that never change duirng the life of the object.

## Common Mapping Methods

See page 86 for a list of methods defined on `dict`, `collections.defaultdict`, and `collections.OrderedDict`.

The way `d.update(m)` handles `m` is a good example of _duck typing_ (If it quacks, it's a duck).
If `m` has a `keys` method, it assumes `m` is a mapping. else, it will iterate over `m`, asssuming its items are `(key, value)` form.

`setdefault()` avoids redundant key look ups when we want to update the value of an item in place.

For example,

```python
d.setdefault(key, []).append(new_value)
```

is equivalent to

```python
if key not in d:
    d[key] = []
d[key].append(new_value)
```

but the latter does at least two searches for the `key`, three if it isn't found, while `setdefault` does it all with a single lookup.

## Automatic handling of missing keys

Sometimes we would like to have mappings that return some predetermined value when a missing key is searched. The author discusses two main approaches to achieve this.

### Approach 1: defaultdict

When using `defaultdict`, we pass a callable to produce a default value when a missing key is searched using
the `d[k]` syntax (which uses `__getitem__`).

The callable is held in an instance attribute called `default_factory`.

For example:

```python
index = collections.defaultdict(list)
```

Here, if some key `word` is not in the `index`, the `default_factory` is called to product an empty list, which is assigned to `index[word]` and returned.

Note that if `dd` is a `defaultdict` and `k` is a missing key, then `dd[k]` will create a default value, but dd.get(k) will not.

```python
>>> import collections
>>> dd = collections.defaultdict(list)
>>> dd["key1"]
[]
>>> dd.get("key2")
>>> dd.get("key2") is None
True
```

### Approach 2: \_\_missing\_\_

By providing a \_\_missing\_\_ method, the standard `dict.__getitem__` (called via `d[k]`), will call it whenever a key is not found.

See page 92 for an example of a custom class that implements `__missing__`.

Also see page 94 for a comparison of how diffferent standard library mappings treat missing keys.

## Variations of dict

Here we list mapping types included in the standard library, besides `defaultdict`

### collections.OrderedDict

Since Python 3.6, the built-in `dict` has been able to keep the keys ordered.

The author lists some differences between `dict` and `OrderedDict`, aside `OrderedDict` being backward compatibele with earlier Python versions.

- the equality operation for `OrderedDict` checks for matching order.
- `OrderedDict` was designed to be good at reordering operations - it can handle frequent reordering operations better than `dict`.

See [this](https://realpython.com/python-ordereddict/) for more info.

### collections.ChainMap

A `ChainMap` holds references to each input mapping in the constructor call. Lookups will be performed on the input mappings in order, while insertions and updates are only affect the first input mapping.

```python
>>> from collections import ChainMap
>>> d1 = dict(a=1, b=2)
>>> d2 = dict(a=3, b=4, c=5)
>>> chain = ChainMap(d1, d2)
>>> chain['a']
1
>>> chain['c']
5
>>> chain['c'] = -1
>>> d1
{'a': 1, 'b': 2, 'c': -1}
>>> d2
{'a': 3, 'b': 4, 'c': 5}
```

The author explains on page 96 how `ChainMap` is useful when implementing interpreters for languages with nested scopes.

Also see Chapter 18 for a `ChainMap` subclass used to implement an interpreter for a subset of Scheme.

### collections.Counter

`Counter` maintains an integer count for each key.

For example,

```python
>>> from collections import Counter
>>> ct = Counter('yukihide')
>>> ct
Counter({'i': 2, 'y': 1, 'u': 1, 'k': 1, 'h': 1, 'd': 1, 'e': 1})
>>> ct.update('yuki')
>>> ct
Counter({'i': 3, 'y': 2, 'u': 2, 'k': 2, 'h': 1, 'd': 1, 'e': 1})
>>> ct.most_common(3)
[('i', 3), ('y', 2), ('u', 2)]
```

`Counter` can be used as a multi-set too - consider each key as an element in the set, and the count as the number of times it appears.

### shelve.Shelf

The `shelve` module provides persistent storage for a mapping of string keys to Python objects serialised in the `pickle` binary format.

(Why `shelve`? Because pickle jars are stored on shelves, says the author.)

See more on the [Python documentation](https://docs.python.org/3/library/pickle.html).

Also see Ned Batchelder's ["Pickle's nine flaws"](https://nedbatchelder.com/blog/202006/pickles_nine_flaws.html) for a discussion about the caveats of using `pickle`

### Subclassing UserDict instead of Dict

The author suggests the it's 'better to subclass `UserDict` rather than `dict`'.

"Subclassing Built-in Types is Tricky' on page 492 describes the precise problem with subclassing `dict` (and other built-in types), but essentially `dict` has some implemention shortcuts that forces the user to override methods that can be inherited from `UserDict`.

See page 98 for a comparision of two versions of a custom `StrKeyDict` class - one that extends `dict` and another extending `colletions.UserDict`. The latter has a more succint implementation.

## Immutable Mappings
