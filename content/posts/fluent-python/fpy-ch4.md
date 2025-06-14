---
date: "2025-06-14"
title: "Fluent Python Chapter 4: Unicode Text Versus Bytes"
author: "Yukihide Takahashi"
summary: "The binary sequence types and codecs"
readTime: true
toc: true
---

## Character Issues

In Python 3, the identity of a character is its Unicode _code point_.

That's different from the bytes that represent the character itself.

Let's understand the distinction.

The identity of a character (its code point) is a number from 0 to 1,114,111 in base 10.
In Unicode standard this is shown using 4 to 6 hexadecimal digits with a "U+" prefix, from U+0000 to U+10FFFF.
About 13% of the valid code points have characters assigned to them in Unicode 13.0.0.

The bytes that represent the character, on the other hand, depend on the _encoding_ used.
For example, the code point for "A" (U+0041) is encoded as the single byte `\x41` in UTF-8, or as `\x41\x00` in UTF-16LE encoding. The code point for the Euro sign (U+20AC) is represented using three bytes `\xe2\x82\xac` in UTF-8, while in UTF-16LE only two bytes are necessary - `\xac\x20`

Also see this wikipedia page about [variable-width encoding](https://en.wikipedia.org/wiki/Variable-width_character_encoding)

## Byte Essentials

A quick overview of the binary sequence types in Python before talking about encoding/decoding.

In Python 3, there are two basic built-in types for binary sequences: the immutable `bytes` and mutable `bytearray`.

Each item in a `bytes` or `bytearray` is an integer in the range 0 to 255. Meanwhile, a slice of a binary sequence produces a binary sequence of the same type, including slices of length 1.

```python
>>> cafe = bytes('café', encoding='utf_8')
>>> cafe
b'caf\xc3\xa9'
>>> cafe[0]
99
>>> cafe[:1]
b'c'
>>> cafe_arr = bytearray(cafe)
>>> cafe_arr
bytearray(b'caf\xc3\xa9')
>>> cafe_arr[-1:]
bytearray(b'\xa9')
```

Note that actual ASCII text is used in the literal notion above. See page 121 for the four different displays used, depending on the byte value.

`bytes` and `bytearray` sypport every `str` method except formatting methods and methods that depend of Unicode data, such as `isdecimal` and `encode`. Familar string methods like `replace` and `strip` work on the binary sequences too.

Binary sequences have a class method that `str` doesn't have - `fromhex`. This builds a binary sequence from a string of hexadecimal digits.

```python
>>> bytes.fromhex('31 4B CE A9')
b'1K\xce\xa9'
```

See page 122 for other ways of building binary sequence instances, including building from a buffer-like object.

## Basic Encoders and Decoders

There are more than 100 codecs (encoder/decoders) bundled into the Python distribution.

Different codecs produce can different byte sequences.

```python
>>> for codec in ['latin_1', 'utf_8', 'utf_16']:
...     print(codec, 'El Niño'.encode(codec), sep='\t')
...
latin_1 b'El Ni\xf1o'
utf_8   b'El Ni\xc3\xb1o'
utf_16  b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

Some encodings, like ASCII and the multibyte GB2312 cannot represent every Unicode character.

On the other hand, the UTF encodings are designed to handle every Unicode code point.

See page 124 for a brief summary of common encodings, including `latin1`, `cp1252`, and `utf-8`.

## Understanding Encode/Decode Problems

Unicode errors in Python come in one of two types: `UnicodeEncodeError` and `UnicodeDecodeError`.

A `SyntaxError` can also be raised when loading Python modules when the source encoding is unexpected.

### Coping with UnicodeEncodeError

When converting text to bytes, if a character is not defined in the target encoding, a ``UnicodeEncodeError` is raised unless the handling behaviour is specified.

```python
>>> city = 'São Paulo'
>>> city.encode('utf_8')
b'S\xc3\xa3o Paulo'
>>> city.encode('cp437')
Traceback (most recent call last):
  File "<python-input-13>", line 1, in <module>
    city.encode('cp437')
    ~~~~~~~~~~~^^^^^^^^^
  File "/.../lib/python3.13/encodings/cp437.py", line 12, in encode
    return codecs.charmap_encode(input,errors,encoding_map)
           ~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^
UnicodeEncodeError: 'charmap' codec can't encode character '\xe3' in position 1: character maps to <undefined>
encoding with 'cp437' codec failed
>>> city.encode('cp437', errors='ignore')
b'So Paulo'
>>> city.encode('cp437', errors='replace')
b'S?o Paulo'
>>> city.encode('cp437', errors='xmlcharrefreplace')
b'S&#227;o Paulo'
```

We can also add custom error handling function to the `errors` argument. See [`codes.register_error`](https://docs.python.org/3/library/codecs.html#codecs.register_error)

### Coping with UnicodeDecodeError

Not every byte sequence is valid UTF-8 or UTF-16. Therefore, if we assume either encoding while converting a binary sequence to text, we will get a `UnicodeDecodeError` if unexpected bytes exist.

In contrast, many legacy 8-bit encodings like `cp1252` are able to decode any stream of bytes without reporting errors. This means that the program will silenetly decode garbage if it assymes the wrong 8-bit encoding.

Garbled characters are known as gremlins or mojibake (文字化け)

See page 127 for an example of how using the wrong codec may produce gremlins or a `UnicodeDecodeError`.

### SyntaxError When Loading Modules with Unexpected Encoding.

The default encoding for Python 3 source files is UTF-8 across all platforms.

Therefore, when opening a .py file on Windows with `cp1252`, a `SyntaxError` may be raised.

As a fix, the author suggests adding a `coding` comment at the top of the file:

```python
# coding: cp1252

print('Olá, Mundo!)
```

### How to Discover the Encoding of a Byte Sequence

Sadly, we cannot determine the encoding given a byte sequence, we must be told, says the author.

However we can make use of heuristics - that's how the package Chardet guesses one of more than 30 supoprted encodings.

For example, if we assume a stream of bytes is human plain text, and we see `b'\x00'` bytes often, it is most likely a 16- or 32-but encoding and not an 8-bit scheme, becasue null characters in plain text are bugs.

Also, although binary sequences of encoded text usually don't have explicit hints of their encoding, the UTF formats may prepend a byte order mark to the text content.

### Byte Order Mark: A Useful Gremlin

Let's look at the UTF-16 encoding of 'El Niño' again.

```python
u16 = 'El Niño'.encode('utf_16')
>>> u16
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

The bytes `b'\xff\xfe'` are a BOM denoting the 'little-endian' byte order of the Intel CPU where the encoding was performed.

On a little-endian machine, the least significant byte comes first for each code point.

Looking at the decimal representation of the byte string above,

```python
>>> list(u16)
[255, 254, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
```

we see that 'E' (code point U+0045, decimal 69) is encoded in byte offsets 2 and 3 as `69` and `0`.

On a big-endian machine, the most significant byte comes first: 'E' would be encoded as `0` and `69`.

To make clear the byte encoding, UTF-16 prepends the text to be encoded with the invisible character
`ZERO WIDTH NO-BREAK SPACE (U+FEFF)`. On a little-endian CPU, this is encoded as `b'\xff\xfe'`.

Since there is no U+FFFE character in Unicode, the byte sequence `b'\xff\xfe'` can only mean the `ZERO WIDTH NO-BREAK SPACE` on a little-endian encoding. Therefore the codec knows which byte ordering to use.

One advantage of UTF-8 is that it generates the same byte sequence no matter the machine endianness, so no BOM is needed.

See pages 130 and 131 for more details about LE versus BE.

### Handling Text Files

The author introduces the idea of the "Unicode Sandwich", where when handling text I/O, it is best practice to decode bytes to `str` on input as early as possible, execute your business logic on strings, and encode text to bytes on output as late as possible.

In Python 3, `open()` will decode when reading and encode when writing files in text mode. Therefore, `my_file.read()` and `my_file.write(text)` produce `str` objects.

Pages 132 ~ 139 explain encoding defaults, and why we should not rely on them.

In summary, the author advices to specify an `encoding=` argument explicitly.

\*Other topics covered in this chapter:

- Normalizing Unicode across Locales for Comparison
- the `unicodedata` module
- how `str` and `bytes` behave differently for certain standard library functions, such as RegExp and os functions.
