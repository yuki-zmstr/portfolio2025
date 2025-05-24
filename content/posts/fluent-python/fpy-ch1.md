---
date: "2025-05-22T08:10:31+09:00"
draft: true
title: "Fluent Python Chapter 1: The Python Data Model"
author: "Yukihide Takahashi"
summary: "This is a summary"
---

## Chapter 1: The Python Data Model

---

This chapter is an introduction to how data objects are represented in Python. For custom objects, by implementing special 'dunder' methods, they can behave just like the built-in types like `str` and `int`.

For example, consider the following `Shelf` class:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author

    def __repr__(self):
        return f'Book({self.title!r}, {self.author!r})'

    def __str__(self):
        return f'{self.title} by {self.author}'


class Shelf:
    def __init__(self):
        self._books = [Book(title, author) for title, author in [
            ("Factfullness", "Hans Rosling"),
            ("The Da Vinci Code", "Dan Brown"),
            ("Politika", "Tom Clancy")
        ]]  # Our shelf has three books

    def __len__(self):
        return len(self._books)

    def __getitem__(self, position):
        return self._books[position]
```

Now we instantiate our `shelf` object.

```python
>>> shelf = Shelf()
```

The `len()` function and `[]` index operator come familiar to whoever has taken Python 101.
Our `Shelf` class responds to them(!)

```python
>>> len(shelf)
3
>>> shelf[1]
Book('The Da Vinci Code', 'Dan Brown')
```

These two commands made use of the `__len__` and `__getitem__()` methods we implemented.

For `__len__`:

- This means that users of our `Shelf` object do not need to remember special function names to "get its size" - just the usual `len` works.
- For comparison, in Java, the size of an `array` is found using `array.length`, while the size of a `set` is found using `set.size()` <- notice the presence/absence of the brackets too...

For `__getitem__()`:

- This means that our `shelf` is also iterable!

For example:

```python
>>> for book in shelf:
...     print(book)
Factfullness by Hans Rosling
The Da Vinci Code by Dan Brown
Politika by Tom Clancy
```

We have skipped two dunder methods: `__repr__` and `__str__` in our `Book` class.
Both of these methods provide the user with a string representation of the object.

In the code block immediately above, we `print`-ed each book - there, the `__str__` method is called.

On the other hand, `__repr__` is called in the interactive console:

```python
>>> book = Book("Politika", "Tom Clancy")
>>> book
Book('Politika', 'Tom Clancy')
```

The question is then which should be used to provide a string representation of our objects.

The author makes the following points:

- `__str__` should return a string suitable for display to end users.
- `__repr__` should return an unambiguous representation of the object, and if possible, match the code that enables the user to re-build the object.
  - Notice how we use `!r` in the f-string of our `Book` `__repr__` to get the standard representation of the attributed to be displayed.
- if `__str__` is not implemented, `print` can use `__repr__` as a fallback.
- However, if `__repr__` is not implemented, the interactive console will not use `__str__` as a fallback. Instead, you will get something like `<__main__.Book object at 0x104eebcd0>`

This [Stack Overflow thread](https://stackoverflow.com/questions/1436703/what-is-the-difference-between-str-and-repr) features an extensive discussion on this topic.

Arithmetic operators can be overloaded as well. Just a couple of examples:

| Operator, | implemented by |
| :-------: | :------------: |
|     +     |   `__add__`    |
|     -     |   `__sub__`    |
|    \*     |   `__mul__`    |
|     /     | `__truediv__`  |
|    //     | `__floordiv__` |

This allows for a wide selection of numeric types, such as `decimal.Decimal`, `fractions.Fraction`, and the custom `Vector` type that the author implements in the book.

## Summary

Utilizing special methods allows for objects to behave like the primitive data types.

I think this is one of the beauties of Python - the methods you learn in the first few weeks of Python 101 really do stick with you throughout your career!

Also, to me, whether other Python developers have a nice experience using my classes and APIs is important. It seems implementing dunder methods is one way I can achieve this.
