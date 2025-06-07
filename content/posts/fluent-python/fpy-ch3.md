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

_dictcomps_ build `dict` instances using `key:value` pairs from any iterable.

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
For `api v1`, a food with `type == 'Fast Food'` should have a single ingredient.
A Japanese food should have a key `ingredients`, which is a list of ingredients.

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
`f1` has a key `name`, which does not appear in any `case` statement, yet it matches.

You can also capture extra key-value pairs using `**`.

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

Note that in Python, an object is hashable if it has a hash code that never changes during its lifetime (implements a `__hash__()` method), and can be compared to other objects (implements a `__eq__()` method).

In practice, custom implementations of `__hash__()` and `__eq__()` should only take into account instance attributes that never change duirng the life of the object.

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

Note that if `dd` is a `defaultdict` and `k` is a missing key, then `dd[k]` will create a default value, but `dd.get(k)` will not.

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

By providing a `__missing__` method, the standard `dict.__getitem__` (called via `d[k]`), will call it whenever a key is not found.

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

All mapping types provided by the standard library are mutable, but it may be important to prevent users from changing a mapping by accident.

The `MappingProxyType` from the `types` module can help to achieve this. Given a mapping, `MappingProxyType` returns a `mappingproxy` instance that is a read-only but dynamic view of the original mapping - this means that updates to the original mapping are reflected to the `mappingproxy`, but updates cannot be made directly to the `mappingproxy`.

See page 100 for an example.

## Dictionary Views

The `dict` instance methods `.keys()`, `.values()`, `.items()` return instances of read-only but dynamic views called `dict_keys`, `dict_values`, and `dict_items` respectively.

They have less memory overhead compared to their counterparts in Python 2, which returned lists that duplicated data in the target dict.

An example usage:

```python
>>> d = dict(a=1, b=2, c=3)
>>> values = d.values()
>>> values
dict_values([1, 2, 3])
>>> len(values)
3
>>> list(values)
[1, 2, 3]
>>> reversed(values)
<dict_reversevalueiterator object at 0x103362e30>
>>> values[0]
Traceback (most recent call last):
  File "<python-input-48>", line 1, in <module>
    values[0]
    ~~~~~~^^^
TypeError: 'dict_values' object is not subscriptable
>>> d['z'] = 99
>>> d
{'a': 1, 'b': 2, 'c': 3, 'z': 99}
>>> values
dict_values([1, 2, 3, 99])
```

Lastly, note that we cannot create a view object from scratch.

```python
>>> values_class = type({}.values())
>>> v = values_class()
Traceback (most recent call last):
  File "<python-input-53>", line 1, in <module>
    v = values_class()
TypeError: cannot create 'dict_values' instances
```

See page 102 for some practical implications of how dict works. Some pointers are that

- dicts have a significant memory overhead compared to an array of pointers.
- hash tables (the underlying data structure of dicts) have a third of their space empty for efficiency.

Also see [this](https://www.fluentpython.com/extra/internals-of-sets-and-dicts/) for a deep dive into the internals of sets and dicts

## Set Theory

A set is a collection of unique objects. Set elements need to be hashable.

While the `set` type is not hashable, its immutable counterpart, `frozenset`, is.

The set types implements several infix set operations, such as union (`|`), intersection(`&`), and symmetric difference (`^`).

### Set Literals

In Python 3, the syntax of set literals always uses the `{...}` notation, except for the empty set which uses `set()`.

```python
>>> s = {'a'}
>>> type(s)
<class 'set'>
>>> s
{'a'}
>>> s.pop()
'a'
>>> s
set()
```

Set literals like `{1,2,3}` is faster than calling the constructor like `set([1,2,3])` because in the latter case, Python will look up the `set` name to fetch the constructor, then build a list, and finally pass the list to the constructor.

On the other hand, in processing a literal like `{1,2,3}`, Python executes a specialized `BUILD_SET` bytecode.

Also try to import the `dis` function and compare the disassembled bytecodes:

- `dis('{1}])` versus
- `dis('set([1])')`

## Practical Consequences of How Sets Work

Like `dict`, the `set` and `frozenset` types are implemented with a hash table.

Thus, all set elements must be hashable.

Element ordering is not very useful, since for two elements with the same hash code, their position depends on which element was added first. Also adding elements to a set may change the order of existing elements due to table resizing.

See page 107 for more details, as well as [this](https://www.fluentpython.com/extra/internals-of-sets-and-dicts/)

### Set Operations

Page 108 ~ 109 lists the various operators supported by mutable and immutable sets derived from mathematical set theory, e.g. Intersection, Union, Difference, Symmetric Difference, Set Comparision, and Membership Testing.

Note that the infix operators require that both operans be sets, but all other methods only require the object its called on to be a set, and take in one or more iterable arguments.

For example,

```python
>>> s = {1,2,3}
>>> l = [2,3]
>>> s.intersection(l)
{2, 3}
```

In addition, Python offers other pratical set methods, like `.add()`, `.clear()`, and `.pop()`

See page 110 for more information.

## Set Operators on Dict Views

Dict view objects returned by the `.keys()` and `.items()` methods share common APIs with `frozenset`.

In particular `dict_keys` and `dict_items` support `&`, `|`, `-`, and `^`.

For example,

```python
>>> d1 = dict(a=1, b=2, c=3)
>>> d2 = dict(b=20, c=30, d=40)
>>> d1.keys() & d2.keys()
{'c', 'b'}
```

We get a `set` as a return value. Also, the set operators in dict views work with `set` instances.

```python
>>> s = {'a', 'd', 'e'}
>>> d1.keys() & s
{'a'}
>>> d1.keys() | s
{'a', 'e', 'c', 'd', 'b'}
```

Finally, note that for a `dict_values` view to support set operators, all values in the `dict` need to be hashable.

```python
>>> d3 = dict(a=1, b=[])
>>> d3.values() & {1}
Traceback (most recent call last):
  File "<python-input-19>", line 1, in <module>
    d3.values() & {1}
    ~~~~~~~~~~~~^~~~~
TypeError: unsupported operand type(s) for &: 'dict_values' and 'set'
```

In contrast, a `dict_keys` view can always be used as a set since every key is hashable by definition.
