- str vs byte

# What’s New in This Chapter

Python 3은 Unicode를 포괄적, 안정적으로 제공

# Character Issues

“string”은 sequence of characters입니다. 문제는 characters의 정의입니다.

2021년 character의 적절한 정의는 a Unicode character입니다. Python 3의 str은 Unicode characters입니다.

- Unicode range: 0 - 1,114,111 or U+0000 - U+10FFFF
- 실제 바이트로 변환되는 값은 Encoding 방식마다 다릅니다. ‘A’ (U+0041)의 UTF-8 Encoding은 \x41이고 UTF-16LE encoding은 \x41\x00입니다.

```python
>>> s = 'café'
>>> len(s)
4
>>> b = s.encode('utf8')
>>> b
b'caf\xc3\xa9'
>>> len(b)
5
>>> b.decode('utf8')
'café'
```

- str.encode - method
- str.decode - method

Python 3 str type은 Python 2 unicode type과 비슷합니다. Python 3 byte type은 Python 2의 str과 비슷하지만 bytearray type과 밀접한 관련이 있습니다.

# Byte Essentials

두 개의 built-in binary sequence type이 있습니다. immutable → bytes, mutable → bytearray. 이 둘을 동시에 “byte string”이라고 부르기도 합니다. 그런데 싫어싫어.

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

- bytes - types
- bytearray - types

bytes type의 items는 range(256)입니다. cafe[0]은 integer type이 return되는데, cafe[:1]은 bytes type이 return됩니다.

- bytes의 32 ~ 126은 ASCII를 출력합니다.
    - tab → \t
    - newline → \n
    - carriage return → \r
    - \ → \\

bytes, bytearray type은 str method에서 formatting, Unicode method 빼고 대부분을 지원합니다. casefold, isdecimal, isidentifier, isnumeric, isprintable, encode, endswith, replace, strip, translate, upper…, re

```python
>>> bytes.fromhex('31 4B CE A9')
b'1K\xce\xa9'
```

- bytes.fromhex - method

```python
>>> import array
>>> numbers = array.array('h', [-2, -1, 0, 1, 2])
>>> octets = bytes(numbers)
>>> octets b'\xfe\xff\xff\xff\x00\x00\x01\x00\x02\x00'
```

# Basic Encoders/Decoders

Python은 100개 이상의 encoder, decoder (codec)을 내장하고 있습니다. open(), str.encode(), bytes.decode()

# Understanding Encode/Decode Problems

UnicodeError, UnicodeEncodeError, UnicodeDecodeError

## Coping with UnicodeEncodeError

```python
>>> city = 'São Paulo'
>>> city.encode('utf_8')
b'S\xc3\xa3o Paulo'
>>> city.encode('utf_16')
b'\xff\xfeS\x00\xe3\x00o\x00 \x00P\x00a\x00u\x00l\x00o\x00'
>>> city.encode('iso8859_1')
b'S\xe3o Paulo'
>>> city.encode('cp437')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/owen.ki/.pyenv/versions/3.11.0/lib/python3.11/encodings/cp437.py", line 12, in encode
    return codecs.charmap_encode(input,errors,encoding_map)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
UnicodeEncodeError: 'charmap' codec can't encode character '\xe3' in position 1: character maps to <undefined>
>>> city.encode('cp437', errors='ignore')
b'So Paulo'
>>> city.encode('cp437', errors='replace')
b'S?o Paulo'
>>> city.encode('cp437', errors='xmlcharrefreplace')
b'S&#227;o Paulo'
```

## Coping with UnicodeDecodeError

UTF-8, UTF-16은 DecodeError가 발생할 수 있지만, legacy 8-bit encoding은 어떤 byte sequence도 decode할 수 있습니다.

```python
>>> octets = b'Montr\xe9al'
>>> octets.decode('cp1252')
'Montréal'
>>> octets.decode('iso8859_7')
'Montrιal'
>>> octets.decode('koi8_r')
'MontrИal'
>>> octets.decode('utf_8')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 5: invalid continuation byte
>>> octets.decode('utf_8', errors='replace')
'Montr�al'
```

## SyntaxError When Loading Modules with Unexpected Encoding

UTF-8은 Python3의 Default encoding. Python2는 ASCII

magic coding comment

```python
# coding: cp1252

print('Olá, Mundo!')
```

## How to Discover the Encoding of a Byte Sequence

?

## BOM: A Useful Gremlin

Byte Order Mark

```python
>>> u16 = 'El Niño'.encode('utf_16')
>>> u16
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

\xff\xfe가 BOM입니다. little-endian이라는 뜻입니다.

little-endian이라 E\x00, 69 0.

big-endian은 \x00E, 0 69

ZERO WIDTH NO-BREAK SPACE: \xff\xfe - little-endian

explicit할 수 있습니다.

```python
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
>>> u16le = 'El Niño'.encode('utf_16le')
>>> list(u16le)
[69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
>>> u16be = 'El Niño'.encode('utf_16be')
>>> list(u16be)
[0, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111]
```

# Handling Text Files

“Unicode sandwich”

Django의 view의 output은 Unicode str입니다.

Python3은 Unicode sandwich를 더 잘 따릅니다.

Window환경과 Unix환경이 다릅니다. Window는 파일을 open할 때 cp1252로 default encoding합니다.

## Beware of Encoding Defaults

```python
import locale
import sys
expressions = """
        locale.getpreferredencoding()
        type(my_file)
        my_file.encoding
        sys.stdout.isatty()
        sys.stdout.encoding
        sys.stdin.isatty()
        sys.stdin.encoding
        sys.stderr.isatty()
        sys.stderr.encoding
        sys.getdefaultencoding()
        sys.getfilesystemencoding()
  """
my_file = open('dummy', 'w')

for expression in expressions.split():
  value = eval(expression)
  print(f'{expression:>30} -> {value!r}')
```

- GNU/Linux, macOS

```python
 locale.getpreferredencoding() -> 'UTF-8'
                 type(my_file) -> <class '_io.TextIOWrapper'>
              my_file.encoding -> 'UTF-8'
           sys.stdout.isatty() -> True
           sys.stdout.encoding -> 'utf-8'
            sys.stdin.isatty() -> True
            sys.stdin.encoding -> 'utf-8'
           sys.stderr.isatty() -> True
           sys.stderr.encoding -> 'utf-8'
      sys.getdefaultencoding() -> 'utf-8'
   sys.getfilesystemencoding() -> 'utf-8'
```

- Windows 10

```python
locale.getpreferredencoding() -> 'cp1252'
                 type(my_file) -> <class '_io.TextIOWrapper'>
              my_file.encoding -> 'cp1252'
           sys.stdout.isatty() -> True
           sys.stdout.encoding -> 'utf-8'
            sys.stdin.isatty() -> True
            sys.stdin.encoding -> 'utf-8'
           sys.stderr.isatty() -> True
           sys.stderr.encoding -> 'utf-8'
      sys.getdefaultencoding() -> 'utf-8'
   sys.getfilesystemencoding() -> 'utf-8'
```

# Normalizing Unicode for Reliable Comparisons

```python
>>> s1 = 'café'
>>> s2 = 'cafe\N{COMBINING ACUTE ACCENT}' >>> s1, s2
('café', 'café')
>>> len(s1), len(s2)
(4, 5)
>>>s1==s2
False
```

\N{COMBINING ACUTE ACCENT}

NFC: Normalization form C

D, Decompose

```python
>>> from unicodedata import normalize >>> s1 = 'café'
>>> s2 = 'cafe\N{COMBINING ACUTE ACCENT}' >>> len(s1), len(s2)
(4, 5)
>>> len(normalize('NFC', s1)), len(normalize('NFC', s2)) (4, 4)
>>> len(normalize('NFD', s1)), len(normalize('NFD', s2)) (5, 5)
>>> normalize('NFC', s1) == normalize('NFC', s2)
True
>>> normalize('NFD', s1) == normalize('NFD', s2)
True
```

## Case Folding

lower()과 비슷한데,  str.casefold()는 모든 문자를 소문자로 만듬

대략 300개 정도의 반례가 있음

## Utility Functions for Normalized Text Matching

## Extreme “Normalization”: Taking Out Diacritics

Google Search에서 Nomarlization을 사용하는데, 악센트가 강세가 검색결과에 영향을 미치지만, 사람들은 검색할 때 귀찮아서 악센트를 넣지 않고 검색하는 경우가 많다.

# Sorting Unicode Text

code 관점에서 비교해서 정렬하는데, 지역마다 정렬 방법이 다르다.

locale.strxfrm()

## Sorting with the Unicode Collation Algorithm

UCA, Unicode Collation Algorithm

pyuca

# The Unicode Database

Unicode Standard가 Unicode Database를 제공

## Finding Characters by Name

```python
from unicodedata import name
```

## Numeric Meaning of Characters

# Dual-Mode str and bytes APIs

## str Versus bytes in Regular Expressions

```python
import re

re_numbers_str = re.compile(r'\d+')
re_words_str = re.compile(r'\w+')
re_numbers_bytes = re.compile(rb'\d+')
re_words_bytes = re.compile(rb'\w+')
text_str = ("Ramanujan saw \u0be7\u0bed\u0be8\u0bef" " as 1729 = 13 + 123 = 93 + 103.")
text_bytes = text_str.encode('utf_8')

print(f'Text\n {text_str!r}')
print('Numbers')
print(' str :', re_numbers_str.findall(text_str))
print(' bytes:', re_numbers_bytes.findall(text_bytes))
print('Words')
print(' str :', re_words_str.findall(text_str))
print(' bytes:', re_words_bytes.findall(text_bytes))
```

## str Versus bytes in os Functions

- `sys.getfilesystemencoding()`

# Chapter Summary

bytes, bytearray, memoryview Error handling

# Further Reading