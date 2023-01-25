# Chapter04 - Unicode Text Versus Bytes

## 학습목표
`Unicode`와 `byte sequences`에 대해 학습.
* Characters, code points, and byte representations
* Unique features of binary sequences: bytes, bytearray, and memoryview
* Encodings for full Unicode and legacy character sets
* Avoiding and dealing with encoding errors
* Best practices when handling text files
* The default encoding trap and standard I/O issues
* Safe Unicode text comparisons with normalization
* Utility functions for normalization, case folding, and brute-force diacritic removal
* Proper sorting of Unicode text with locale and the pyuca library
* Character metadata in the Unicode database
* Dual-mode APIs that handle str and bytes

## Character Issues
2021년 기준 `character`에 대한 최고의 정의는 `Unicode character`임.\
따라서 `Python3`의 `str`의 각 요소에서 얻을 수 있는것은 `Unicode character`임.\
`Python3`의 `str`에서 얻어진 요소는 `Python2`의 `unicode`의 오브젝트에서 얻어진 요소와 같음.\
`Python2`의 `str`에서 얻어진 raw byte와 다름.

**example.1 - Encoding and decoding**
```python
>>> s = 'café'
>>> len(s)
4
>>> b = s.encode('utf8')
>>> b
b'caf\xc3\xa9'
>>> len(b)
5
>>> b.decode('utf8') 'café'
```

`Python3`의 `bytes`은 `bytearray`타입과 더 밀접한 연관이 있다.

## Byte Essentials
`binary sequence`는 `Python2`의 `str`과 많은 면에서 다름.\
`binary sequence`를 위한 두 가지 빌트인 타입이 존재, 불변인 `bytes`와 가변인 `bytearray`.\
`bytes`, `bytearray`의 각 아이템은 정수형이며 0~255 까지.\
`binary sequence`의 slice는 같은 타입의 `binary sequence`를 만듦.

**example.1 - A five-byte sequence as bytes and bytearray**
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

`bytes`, `bytearray`는 `str`의 formatting method를 제외한 모든 method를 지원함.\
Binary Sequences는 `str`클래스가 가지고 있지 않은 메소드인 `fromhex`를 가지고 있음.\
