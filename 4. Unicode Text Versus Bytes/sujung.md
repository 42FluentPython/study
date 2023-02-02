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

`bytes`, `bytearray`는 `str`의 formatting method를 제외한 모든 method를 지원함.

Binary Sequences는 `str`클래스가 가지고 있지 않은 메소드인 `fromhex`를 가지고 있음.\
`fromhex`는 공백으로 구분된 16진수 쌍을 파싱해서 이진 시퀀스를 만들 수 있음.

**example.2 - fromhex**
```python
>>> bytes.fromhex('31 4B CE A9')
b'1K\xce\xa9'
```

## Basic Encoders/Decoders
파이썬 배포판은 번들은 100개 이상의 코덱을 지원함

## Understanding Encode/Decode Problems
### Coping with UnicodeEncodeError
대다수의 비 UTF 코덱은 유니코드의 작은 부분집합만을 다룸.\
정의되지 않은 유니코드를 바이트코드로 전환할때 특별한 처리를 하지 않으면 `UnicodeEncodeError`가 발생 할 수 있음.\

**example.1 - Encoding to bytes: success and error handling**
```python
>>> city = 'São Paulo'
>>> city.encode('utf-8')
b'S\xc3\xa3o Paulo'
>>> city.encode('cp437')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Developer/CommandLineTools/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8/encodings/cp437.py", line 12, in encode
    return codecs.charmap_encode(input,errors,encoding_map)
UnicodeEncodeError: 'charmap' codec can't encode character '\xe3' in position 1: character maps to <undefined>
>>> city.encode('cp437', errors='ignore')
b'So Paulo'
>>> city.encode('cp437', errors='replace')
b'S?o Paulo'
>>> city.encode('cp437', errors='xmlcharrefreplace')
b'S&#227;o Paulo'
```

### Coping with UnicodeDecodeError
모든 바이트가 유효한 ASCII를 보유하지 않고, 모든 `byte sequence`는 UTF-8, UTF-16로 유효하지 않음.\
그렇기에 `byte sequence`를 `text`로 변경할때 `UnicodeDecodeError`가 발생할 수 있음.\
8비트 인코딩을 해주는 레거사들은 무작의 소음을 포함한 어떤 바이트스트림도 에러 리포팅 없이 디코딩 할 수 있음.

**example.1 - Deconding from str to bytes: success and error handling**
```python
>>> octets = b'Montr\xe9al'
>>> octets.decode('cp1252')
'Montréal'
>>> octets.decode('utf-8')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 5: invalid continuation byte
>>> octets.decode('utf-8', errors='replace')
'Montr�al'
```

### SyntaxError When Loading Modules with Unexpected Encoding
UTF-8은 Python3가 사용하는 기본 인코딩 소스.\
만약 `old.py`를 `cp1252`로 만들게 되면 에러가 발생.\

**error**
```
SyntaxError: Non-UTF-8 code starting with '\xe1' in file ola.py on line
	1, but no encoding declared; see https://python.org/dev/peps/pep-0263/
	for details
```

### BOM: A Useful Gremlin
BOM은 `byte order mark`임.

## Handling Text Files
대다수 웹 프레임워크는 입력으로 들어온 바이트를 가능한 빠르게 문자열로 디코딩.\
비즈니스 로직을 처리하는 부분에서는 인코딩과 디코딩 과정이 있어서는 안됨.\
출력은 가능한한 문자열을 인코딩해서 보내야 함.

## Normalizing Unicode for Reliable Comparisons
유니코드의 스트링 표현은 같으나 실질 구성은 다른 경우가 존재.\
이 문제를 해결하기 위해 파이썬은 `unicodedata.normalize()` 지원.\
`unicodedata.normalize()`의 첫 번째 인자는 문자열로 `NFC`, `NFD`, `NFKC`, `NFKD`중 하나를 사용함.\
`NFC`는 가장 짧은 형태로 `NFD`는 결합된 문자를 기본 문자와 결합된 문자로 분해.

### Case Folding
`str.casefold()`메소드는 모든 text를 lowercase로 변경함.\
모든 text가 ascii로 구성되어있다면 `str.lower()`와 차이가 없으나 unicode가 존재한다면 지정된 규칙에 의해 변경.

### Utility Function for Normalized Text Matching
* nfc_equal
* fold_equal

## Sorting Unicode Text
파이썬은 모든 타입의 시퀀스를 각 시퀀스의 아이템을 비교함으로써 정렬함.\
그러나 `Unicode`를 정렬한 경우 원하는 정렬형태가 아닐 수 있음.\
그 이유는 `Non Ascii code`인 코드 포인트를 비교하는 과정에서 발생함. \
그렇기에 파이썬은 표준으로 `locale.strxfrm`를 사용하는 것.

**example.1 - using the locale.strxfrm function at the sort key**
```python
>>> import locale
>>> my_locale = locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8')
>>> print(my_locale)
pt_BR.UTF-8
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted_fruits = sorted(fruits, key=locale.strxfrm)
>>> print(sorted_fruits)
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```

## The Unicode Database
`Unicode` 표준은 전체 데이터베이스를 제공함.

## Dual-Model str and bytes APIs
파이썬 표준 라이브러리는 `str` 또는 `bytes`를 인자로 받는 함수를 가지고, 타압에 따라 다르게 동작함.

### str Versus bytes in Regular Expressions

### str Versus bytes in os Functions