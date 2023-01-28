# 4. Unicode Text Versus Bytes
- Python 3에서는 문자열과 바이트를 나누었다.
  - 문자열은 human-readable이고 컴퓨터는 바이트를 다룬다.
  - byte sequence와 unicode간의 묵시적 변환은 더 이상 지원하지 않는다.
- 이 장에서는 유니코드 문자열, binary sequence, 두 sequence 간의 변환에 이용되는 인코딩을 다룬다.
- 유니코드를 다루는 것은 중요하다.
  - `str`과 `byte`의 차이에서 생기는 문제는 피할 수 업삳.
- binary sequence 타입은 python2의 다용도 `str`가 지원하지 않는 기능들을 지원한다.

### 이 챕터에서 다루는 토픽들
- 문자, 유니코드 코드포인트, 바이트 표현
- `bytes`, `bytearray`, `memoryview`: binary sequence의 특이점들
- 전체 유니코드와 legacy character set의 인코딩
- 인코딩 오류를 피하는 법과 다루는 법
- 텍스트 파일을 다루는 좋은 방법
- 기본 인코딩의 함정과 표준 입출력 문제
- 정규화를 통한 안전한 유니코드 비교
- 정규화를 위한 유틸리티 함수, 대소문자 변경, brute force를 통한 발음 구별 기호 제거
- `locale`과 _pyuca_ 라이브러리를 통한 적절한 유니코드 정렬
- 유니코드 데이터베이스에서의 문자 메타데이터
- `str`과 `bytes`를 다루는 이중 모드 API

### 다루지 않고 링크로 대신합니다.
- [Parsing binary records with struct](https://fpy.li/4-3)
- [Building Multi-character Emojis](https://fpy.li/4-4)

## 문자 문제
- 문자열이라는 것은 문자의 sequence이다.
- 문자란 무엇인가?
  - 현재로써 최선의 정의는 유니코드 문자
- python3의 `str`은 유니코드 문자의 sequence이다.
  - python2에서 이는 `unicode` 객체였다.
  - python2에서 `str`은 날 byte였다.
- 유니코드 표준은 문자의 identity와 바이트 표현을 구분한다.
  - 문자의 identity(== code point)는 0에서 1,114,111(10진수)에 이르는 숫자이다.
    - 유니코드 표준에서는 "U+" 접두어가 붙은 4-6자의 16진수로 표기된다.
      - U+0000에서 U+10FFFF까지
    - A: U+0041, €: U+20AC
    - Python 3.10.0b4에서 쓰이는 유니코드 표준 13.0.0 에서는 13% 정도의 code point가 문자를 배정받았다.
  - 실제 바이트 표현은 사용하는 인코딩에 따라 달라진다.
    - 인코딩이란 code point와 바이트 열 간을 변환하는 알고리즘이다.
    - code point A (U+0041)는,
      - UTF-8 인코딩에서 단일 바이트 `\x41`로 표현된다.
      - UTF-16LE에서는 2바이트 `\x41\x00`으로 표현된다. 
    - code point € (U+20AC)은,
      - UTF-8 인코딩에서 3바이트 `\xe2\x82\xac`로 표현된다.
      - UTF-16LE에서는 2바이트 `\xac\x20`으로 표현된다.

- code point에서 바이트로의 변환은 인코딩이고, 바이트에서 code point로의 변환은 디코딩이다.
  - human-readable로 만드는 과정을 'decoding'이라고 생각하면 외우기 쉽다.
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
- `byte`의 문자열 표현은 `b` 접두어를 가진다.

## 바이트의 핵심
- 바이너리 sequence는 두 종류 있다.
  - immutable인 `byte`
  - mutable인 `bytearray`
- `byte`와 `bytearray`의 각 요소는 0부터 255까지의 정수이다.
  - 인덱스로 접근했을 때, 정수를 얻을 수 있다.
- 바이너리 sequence의 slice는 같은 타입의 바이너리 sequence가 된다.
  - 길이 1의 slice도 포함
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
- `cafe[0]`과 `cafe[:1]`이 다른 이유는 원래 sequnce 타입이 그렇기 때문이다.
  - `list` `l`이 있을 때, `l[0]`은 요소의 타입이고 `l[:1]`은 `list` 타입이다.
  - `str`만이 예외이다.

- 바이너리 sequence는 정수의 나열이지만 리터럴 표기법은 ASCII를 포함하고 있음을 반영한다.
- 4 가지 다른 표시법이 쓰인다.
  - 10진수로 32에서 126까지(printable ASCII)는 ASCII 자체를 출력한다.
  - 탭, 개행, 캐리지 리턴, \\(이스케이프 시퀀스)는 각각 `\t`, `\n`, `\r`, `\\`으로 표시된다.
  - 문자열 구획 문자인 `'`과 `"`이 모두 바이트 시퀀스에 사용되면 전체 시퀀스는 `'`으로 묶이고 포함된 `'` 문자는 `\'`으로 이스케이프 된다.
  - 다른 바이트 값들은 16진수 이스케이프 시퀀스로 표현된다. (e.g. `\x00`은 널문자)

- `bytes`와 `bytearray`는 `str`의 메소드를 거의 모두 지원한다.
  - 제외되는 함수
    - 포매팅 함수를 제외 (`.format`, `.format_map`)
    - 유니코드 데이터를 이용하는 함수들 제외 (`casefold`, `isdecimal`, `isidentifier`, `isnumeric`, `isprintable`, `encode`)
  - 지원하는 함수의 일부
    - `endswith`, `replace`, `strip`, `translate` 등을 모두 사용할 수 있다.
- `bytes`와 `bytearray`에서만 지원하며, 이들만을 인자로 받는 함수도 있다.
- 정규표현식 모듈 `re`도 바이너리 시퀀스를 지원한다.
  - 단, 정규표현식이 바이너리 시퀀스를 이용하여 컴파일 되었을 때만 지원한다.
- Python 3.5부터는 % 연산자가 바이너리 시퀀스를 지원한다.
- 바이너리 시퀀스에는 `fromhex`라는 클래스 메소드가 있다.
  - 다음을 참조
```python
>>> bytes.fromhex('31 48 CE A9')
b'1H\xce\xa9'
```
- `bytes`와 `bytearray`는 생성자에서 다음의 인자를 받아 인스턴스를 생산한다.
  - 하나의 `str` 인자와 `encoding` 키워드 인자
  - 0에서 255 사이의 값만을 제공하는 iterable
  - 원본 객체에서 새롭게 생성될 바이너리 시퀀스로의 buffer protocol를 가진 객체들
    - `bytes`, `bytearray`, `memoryview`, `array.array`
    - buffer 유사 객체에서 바이너리 시퀀스를 만드는 것은 타입 캐스팅이 필요할 수도 있는 저수준 연산이다.
```python
>>> from array import array
>>> numbers = array('h', [-2, -1, 0, 1, 2])
>>> octets = bytes(numbers)
>>> octets
b'\xfe\xff\xff\xff\x00\x00\x01\x00\x02\x00'
```
    - buffer 유사 객체에서 바이너리 시퀀스를 만드는 것은 항상 '바이트' 복사를 한다.

## 기본 인코더/디코더
- 파이썬 배포에는 텍스트와 바이트 변환을 위한 100개 넘는 코덱이 들어있다.
  - 각 코덱은 이름이 있다. 하나의 코덱은 이름을 여러 개 가질 수 있다. (alias)
    - e.g. `'utf_8'`, `'utf8'`, `'utf-8'`, `'U8'`
    - `open()`, `str.encode()`, `bytes.decode()`같은 함수에 인자로 넣을 수 있다.
  - 코덱마다 문자열에 대한 바이트 표현이 매우 달라진다.
```python
>>> for codec in ['latin_1', 'utf_8', 'utf_16']:
...     print(codec, 'El Niño'.encode(codec), sep='\t')
... 
latin_1 b'El Ni\xf1o'
utf_8   b'El Ni\xc3\xb1o'
utf_16  b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```
- 모든 인코딩이 모든 유니코드 코드 포인트를 표현할 수 있는 것은 아니다. 
  - UTF 인코딩은 모든 유니코드 코드 포인트를 표시할 수 있게 설계되었다.
- 몇 가지 중요한 인코딩
  - `latin1` _a.k.a._ `iso8859_1`
  - `cp1252`
  - `cp437`
  - `gb2312`
  - `utf-8`
  - `utf-16le`

## 인코딩/디코딩 문제 이해하기
- 파이썬에는 유니코드와 관련된 예외가 있다.
  - 좀 더 포괄적인 `UnicodeError` 예외가 있지만, 사용되지 않는다.
  - 좀 더 구체적인 인코딩 예외와 디코딩 예외가 있다.
    - `UnicodeEncodeError`(`str`에서 바이너리 시퀀스)
    - `UnicodeDecodeError`(바이너리 시퀀스에서 `str`)
- 원본 객체의 인코딩이 예상되지 않는 것이라면 `SyntaxError`을 발생시킨다..

### `UnicodeEncodeError` 다루기
- 대부분의 UTF 아닌 인코딩은 유니코드 문자 집합의 일부만을 다룬다.
- 텍스트에서 바이트로 인코딩 할 때 텍스트에 존재하는 문자가 목표 인코딩에 존재하지 않으면 `UnicodeEncodeError`가 발생한다.
  - 인코딩 메소드/함수에 오류를 처리하는 `error` 인자를 넘겨주면 이를 피할 수 있다.
```python
>>> city = 'Sāo Paulo'
>>> city.encode('utf_8')
b'S\xc4\x81o Paulo'
>>> city.encode('utf_16')
b'\xff\xfeS\x00\x01\x01o\x00 \x00P\x00a\x00u\x00l\x00o\x00'
>>> city.encode('iso8859_1')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'latin-1' codec can't encode character '\u0101' in position 1: ordinal not in range(256)
>>> city.encode('iso8859_1', errors='ignore')
b'So Paulo'
>>> city.encode('iso8859_1', errors='replace')
b'S?o Paulo'
>>> city.encode('iso8859_1', errors='xmlcharrefreplace')
b'S&#257;o Paulo'
```
- `error='ignore`은 인코딩 될 수 없는 문자를 건너뛴다.
  - 데이터 손실을 알 수 없다.
- `error='replace`은 인코딩 될 수 없는 문자를 ?로 대체한다.
  - 데이터는 손실되지만 손실되었다는 사실을 알 수 있다.
- `error='xmlcharrefreplace'`는 인코딩 될 수 없는 문자를 XML entity로 대체한다.
  - `utf`를 사용할 수 없으며, 데이터 손실을 원하지 않을 때 유일한 선택지다.
- Python 3.7에서 `str.isascii()` 불리언 메소드가 추가되었다.
  - ASCII는 거의 대부분의 인코딩과 호환이 된다.
  - 따라서 텍스트가 ASCII로 이루어져 있다면 인코딩은 성공한다.
  - 해당 메소드로 `UnicodeEncodeError`가 발생할 지 아닐 지 알 수 있다.
- `codecs.register_error(name, error_handler)`

### `UnicodeDeocdeError` 다루기
- 모든 바이트가 유효한 ASCII 문자는 아니며, 모든 바이트 시퀀스가 유효한 UTF-8이나 UTF-16인 것은 아니다.
  - 따라서 이런 인코딩을 이용하여 바이너리 시퀀스를 텍스트로 디코딩 할 때, `UnicodeDecodeError`가 발생할 것이라고 예상해야한다.
- 반대로, `cp1252`, `iso8859_1`, `koi8_r`같은 인코딩들은 어떤 바이트의 흐름, 임의적인 노이즈라고 하더라도 에러 발생 없이 디코딩할 수 있게 되어있다.
  - 따라서 이런 인코딩을 사용할 때에 잘못된 8비트 인코딩을 만난다면, 에러를 일으키지 않고 조용히 쓰레기 값을 디코딩할 것이다.
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
- "�"는 유니코드 공식 `REPLACEMENT CHARACTER`로, U+FFFD이다.

### 예상치 못한 모듈을 로딩할 때 발생하는 `SyntaxError`
- UTF-8은 Python 3의 기본 인코딩이다.
  - Python 2에서는 ASCII였다.
- 올바른 인코딩 선언 없는, UTF-8이 아닌 데이터가 포함된 `.py` 모듈을 로딩하면 다음과 같은 메시지가 뜬다.
```
    SyntaxError: Non-UTF-8 code starting with '\xe`' in file ola.py on line 1, but no encoding declared; see https://pythonorg/dev/peps/pep-0263/ for details
```
- 일반적인 경우는 Windows에서 `cp1252`를 사용해서 `.py` 파일이 작성된 경우이다.
  - GNU/Linux, macOS에서는 UTF-8이 널리 사용되고 있다.
  - 이 에러는 Python for Windows에서도 일어난다.
    - Python 3의 기본 인코딩은 어느 플랫폼에서든 UTF-8이다.
- 이는 `coding` 매직 코멘트를 파일 최상단에 달면 해결할 수 있다.
```python
# coding: cp1252

pring('Olá, Mundo!')
```
- 문제를 해결하기 위해서는 `coding` 매직 코멘트를 사용하는 것보다 파일을 UTF-8으로 변환하는 것이 낫다.
  - 편집기가 UTF-8을 지원하지 않는다면 바꾸시오

### 바이트 시퀀스의 인코딩 알아내는 법
- 바이트 시퀀스의 인코딩을 알아내는 법: 그럴 수 없다.
  - 미리 주어져야 알 수 있다.
  - HTTP나 XML같은 일부 통신 프로토콜들은 내용이 어떻게 인코딩 되었는지를 명시적으로 알려주는 헤더를 가지고 있다.
  - 127을 넘는 바이트가 있다면 적어도 ASCII는 아니다.
  - UTF-8과 UTF-16도 가능한 바이트와 불가능한 바이트가 있다.
hack!
  - 127을 넘는 바이트를 포함한 시퀀스를 UTF-8로 디코딩할 수 있다면 아마도 UTF-8일 것이다.
  - UTF-8로 디코딩을 시도하고, `UnicodeDecodeError`가 발생한 경우 `cp1252`를 시도하는 것은 못생겼지만 효율적인 방법이다.
- 인간 언어는 규칙성을 가지고 있다.
  - 통계적으로 보았을 때,
  - `b'\x00'`이 자주 나온다면 8 비트가 아니라 16 비트나 32 비트 인코딩일 확률이 높다.
    - 단순 텍스트에서 널 문자가 자주 나온다면 버그일 것이다.
  - `b'\x20\x00'`이 자주 나온다면 UTF-16LE 인코딩에서의 공백 문자(U+0020)일 것이다.
    - `EN QUAD` 문자인 U+2000은 아닐 것이다.
- [Chardet - The Universal Character Encoding Detector](https://fpy.li/4-8)
  - Chardet 라이브러리 (CHARacter DETector)
  - 30 개가 넘는 인코딩 중에서 하나를 골라준다.
  - 명령줄 유틸리티도 제공한다.
```s
$ chardetect 04-text-byte.asciidoc
04-text-byte.asciidoc: utf-8 confidence 0.99
```

### BOM: 유용한 그렘린
```python
>>> u16 = 'El Niño'.encode('utf_16')
>>> u16
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```
- UTF-16 인코딩의 경우에 가장 앞에 2 바이트의 `b'\xff\xfe'` 시퀀스가 붙는다.
  - 이는 BOM - byte-order mark이다.
  - 이것이 뜻하는 것은 Intel CPU가 인코딩할 때 사용한 "little-endian" 바이트 배치법이다.
- 리틀 엔디언 기계에서는 최하위 바이트가 먼저 온다.
  - `'E'` (U+0045, 10진수 69)는 바이트 오프셋 2번째 자리와 3번째 자리에 `69`와 `0`으로 나타난다.
  ```python
  >>> list(u16)
  [255, 254, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
  ```
- 빅 엔디언 기계에서는 `0`과 `69` 순서로 나타날 것이다.
- 헷갈리지 않게 하기 위해서 UTF-16 인코딩은 첫번째 자리를 보이지 않는 특이 문자인 `ZERO WIDTH NO-BREAK SPACE`(U+FEFF) 문자로 채운다.
- UTF-16LE(litte-endian), UTF-16BE(big-endian)은 명시적으로 엔디언이 표시되기 때문에 BOM이 없다.
```python
>>> u16le = 'El Niño'.encode('utf_16le')
>>> list(u16le)
[69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111, 0]
>>> u16be = 'El Niño'.encode('utf_16be')
>>> list(u16be)
[0, 69, 0, 108, 0, 32, 0, 78, 0, 105, 0, 241, 0, 111]
```
- BOM이 존재한다면 UTF-16 코덱에 의해서 걸러지도록 되어있다.
  - 따라서 디코딩된 텍스트에는 BOM이 존재하지 않는다.
- 유니코드 표준은 BOM이 존재하지 않는다면 UTF-16BE으로 간주하는 것으로 되어있다.
  - 그러나, Intel x86 아키텍처는 리틀 엔디언이기 때문에, BOM 없는 리틀 엔디언이 많다.
- 엔디언의 문제는 한 문자를 이루는데 1 바이트 초과가 필요한 UTF-16과 UTF-32에서 생긴다.
- UTF-8은 CPU의 엔디언과 관계없이 똥일한 바이트를 만들어내고, BOM이 필요 없다.
  - 윈도우즈 애플리케이션(Notepad같은)은 UTF-8에도 BOM을 삽입한다.
    - BOM이 없으면 윈도우 코드 페이지로 간주된다.
    - BOM이 삽입된 UTF-8은 파이썬에서 UTF-8-SIG로 등록되어있다.
    - UTF-8에 삽입된 BOM(U+FEFF)는 `b'\xef\xbb\xbf'`이다.
      - 따라서 이 바이트로 시작된다면 UTF-8-SIG일 것이다.
  - 파이썬에서, UTF-8에 BOM을 삽입하는 것은 권장되지 않는다.

## 텍스트 파일 다루기
- 텍스트 입출력을 다루는 가장 좋은 방법은 "유니코드 샌드위치"이다.
  - `bytes` -> `str`: 입력 바이트를 디코딩한다.
  - 100% `str`: 텍스트를 처리한다.
  - `str` -> `bytes`: 출력 문자열을 인코딩한다.
- 이는 바이트가 입력받고 최대한 이른 때에 디코딩되어야 한다는 것을 의미한다.
  - e.g. 파일을 읽어들일 때
- 샌드위치의 '내용물'은 `str` 객체로만 이루어지는 프로그램의 비즈니스 로직이다.
  - 이때, 인코딩이나 디코딩을 하지 말아야한다.
- 출력 시에, `str`은 `bytes`로 최대한 늦게 인코딩 되어야한다.
- `bytes`를 사용 시 수정할 일은 거의 없다.
  - Django에서, view에서의 출력은 `str`으로 이뤄지고, 프레임워크가 이를 `bytes`로 해석하여 응답을 생성한다.
- Python 3에서 유니코드 샌드위치를 구현하기는 쉽다.
  - `open()` 내장 함수는 읽을 때 디코딩이, 쓸 때 인코딩을 필수적으로 하지는 않는다.
    - 따라서 `my_file.read()`에서는 `str`이 리턴되고, `my_file.write(text)`에서 `text`는 `str`일 수 있다.
- 텍스트 파일을 사용하는 것은 쉬워보이지만, 기본 인코딩에만 의존한다면 곤란해질 수 있다.
  - 윈도우즈의 기본 인코딩은 code page 1252이다.
  - 플랫폼이나 locale 설정에 따라 호환성 문제가 발생할 수 있다.
- 여러 컴퓨터나 여러 다른 경우에 실행될 코드를 짤 때에는 `open()` 함수를 사용할 때 반드시 `encoding=` 인자를 넣어주어야 한다.
- 텍스트 파일을 바이너리 모드(`'b'`)로 열지 마시오.
  - 인코딩을 추정해야하는 경우에는 chardet을 사용하시오.
  - 바이너리 모드는 바이너리 파일을 읽을 때만 사용하여야 한다.

### 기본 인코딩을 조심하자
- 윈도우즈와 GNU/Linux, macOS에서는 `locale.getpreferredencoding()` 값이 다르다.
- 이전에는 오류가 났지만, 이제는 윈도우즈에서 인코딩 오류 없이 유니코드를 터미널에 출력할 수 있다.
  - 리다이렉트 제외.
    - 이 경우, `sys.stdout.isatty()`는 `False`가 되고,
    - `sys.stdout.encoding`이 `locale.getprefererredencoding()`, 즉 'cp1252'가 되며, 
    - `sys.stdin.encoding`, `sys.stderr.encoding`은 `utf-8`로 유지된다.
    - 리다이렉트 시 `cp1252`에 없는 문자를 출력하려고 하면 `UnicodeEncodeError`가 발생한다.

- `open()` 함수 사용시 `encoding` 인자를 누락하면 기본값은 `locale.getpreferredencoding()`이 된다.
- `sys.stdout|stdin|stderr`(표준 입출력)은 interactive I/O의 경우 UTF-8이고, 리다이렉트 시에는 `locale.getpreferredencoding()`이 된다.
  - [`PYTHONLEGACYWINDOWSSTDIO`](https://fpy.li/4-13)이 빈 값이 아니라면 `PYTHONENCODING` 환경 변수에 의해 설정된다.
- `sys.getdefaultencoding()`은 파이썬 내부적으로 바이너리 데이터와 `str` 간의 변환에 이용된다.
  - 이 설정은 변경할 수 없다.
- `sys.getfilesystemdencoding()`은 파일 이름을 인코딩/디코딩 하는데 이용된다
  - `open()` 사용 시 파일 이름으로 들어오는 인자 `str`에 사용된다.
  - `open()`에 파일 이름이 `bytes`로 들어오면 변경되지 않고 OS API에 넘어간다.
- GNU/Linux, macOS의 기본 설정은 이들 모두 UTF-8이다.
  - 따라서 입출력은 모든 유니코드 문자를 다룰 수 있다.
- 윈도우즈에서는 `'cp850'`이나 `'cp1252'`같은 것이 섞여있다.
  - 윈도우즈에서는 인코딩에 매우 신경써야한다.
- 요약: 가장 중요한 인코딩 설정은 `locale.getpreferredencoding()`이다.
  - 이는 텍스트 파일을 열거나 리다이렉트시 표준 입출력으로 사용된다.
  - 그러나 이는 정확한 시스템 설정을 반영하지 않을 수 있다.
  - 따라서 기본 인코딩에 의존하지 마시오.

tip.
- 유니코드 문자를 출력할 때 `'\N{}'` 이스케이프를 활용할 수 있다.
  - e.g. `'\N{HORIZONTAL ELLIPSIS}'`
  - 이름이 존재하지 않을 경우 `SyntaxError`가 발생한다.
  - 16진수 코드를 입력할 때보다 실수를 알아채기 쉽다.

## 믿을 수 있는 비교를 위한 유니코드 정규화
- 유니코드 샌드위치과 인코딩을 항상 명시적으로 표시하는 방법은 많은 일을 줄여준다.
- 다만, 유니코드 다루기는 정확하게 디코딩 된다고 해도 수고로움을 피할 수는 없다.
  - 텍스트 정규화와 정렬에서.

- 유니코드에서 문자열 비교는 복잡하다.
  - 발음 구별 기호같은 여러 기호가 문자에 덧붙여질 수 있다.
```python
>>> s1 = 'café'
>>> s2 = 'cafe\N{COMBINING ACUTE ACCENT}'
>>> s1, s2
('café', 'café')
>>> len(s1), len(s2)
(4, 5)
>>> s1 == s2
False
```
- `COMBINING ACUTE ACCENT` (U+0301)은 e 뒤에 붙어 é를 표현한다.
- 유니코드에서 `é`와 `e\u0301`은 표준 동일체('canonical equivalents`)라고 불린다.
  - 그러나 파이썬에서는 다른 코드 포인트의 시퀀스로 표현되므로 다르다고 평가된다.
- 이 문제를 해결하기 위해서는 `unicodedata.normalize()`가 필요하다.
  - 첫 번째 인자로 `'NFC'`, `'NFD'`, `'NFKC'`, `'NFKD'`를 받는다.
    - NFC: Normalization Form C는 가장 짧은 동일체를 생성하기 위해 코드 포인트들이 합성되어있다.
    - NFD는 기초 문자와, 분리된 결합 문자로 동일체를 분리한다.
    - 이 둘 모두 비교를 예상되로 작동하게 만들어준다.
- NFC:
  - 키보드 드라이버는 일반적으로 합쳐진 문자를 생성하도록 되어있어서 유저 입력 문자는 NFC가 기본이다.
    - 그러나 안전한 코딩을 위해서는 문자열을 저장하기 전에 모두 `normalize('NFC', user_text)`로 저장하는 것이 안전하다.
  - NFC는 W3C에서 권장되는 정규화 형태이기도 하다.
    - [Character Model for the World Wide Web: String Matchiing and Searching](https://fpy.li/4-15)
  - 어떤 문자들은 한 개의 문자가 NFC에서 다른 하나의 문자로 표현되기도 한다.
    - 전기 저항을 표현하는 옴(Ω) 단위는 그리스어 오메가 대문자로 정규화된다.
    - 둘은 같은 형태로 표현되지만 다른 값으로 비교된다.
```python
>>> from unicodedata import normalize, name
>>> ohm = '\u2126'
>>> name(ohm)
'OHM SIGN'
>>> ohm_c = normalize('NFC', ohm)
>>> name(ohm_c)
'GREEK CAPITAL LETTER OMEGA'
>>> ohm, ohm_c
('Ω', 'Ω')
>>> ohm == ohm_c
False
>>> normalize('NFC', ohm) == normalize('NFC', ohm_c)
True
```
- 나머지 정규화 방법
  - NFKC와 NFKD: K는 compatibility(호환성)를 뜻한다.
  - 소위 '호환성 문자'라고 불리는 문자들에 영향을 미치는 더 강한 정규화를 제공한다.
  - 유니코드의 목적은 한 문자에 대해 '표준적인' 하나의 코드 포인트를 만드는 것이다. 그러나 이미 존재하는 표준들을 위한 호환성을 위해 한 문자가 여러 개의 코드 포인트를 가지고 있기도 한다.
    - `latin1`에는 그리스 문자가 없지만 `MICRO SIGN`, `µ`가 존재한다.
      - 해당하는 유니코드 문자는 `µ`(U+00B5)이며, 이는 `latin1`로의 변환을 위해 추가되어있다.
    - 동일한 그리스 문자가 코드포인트 U+03BC(`GREEK SMALL LETTER MU`, `μ`)로 추가되어있다.
    - 따라서 micro sign은 호환성 문자로 취급된다.
  - NFKC와 NFKD에서 각 호환성 문자는 더 '선호되는 형태'인 하나 이상의 문자로 이루어진 '호환성 분해 형태'에 의해 대체된다
    - 포맷 손실이 있더라도 변경이 된다.
      - 이상적으로는 포맷은 외부적인 표시에 책임이 있지 유니코드에 책임이 있지 않다.
      - e.g. 이분의 일 분수 형태인 `'½'`(U+00BD)의 호환성 분해 형태는 3 문자의 나열인 `'1/2'`이다.
      ```python
      >>> from unicodedata import normalize, name
      >>> half = '\N{VULGAR FRACTION ONE HALF}'
      >>> print(half)
      ½
      >>> normalize('NFKC', half)
      '1⁄2'
      ```
  - NFKC와 NFKD는 검색과 인덱싱에서 편리한 매개 표현이 될 수 있다.
  - 단, 유니코드는 생각보다 복잡하다.
  ```python
  >>> for char in normalize('NFKC', half):
  ...     print(char, name(char), sep='\t')
  ... 
  1       DIGIT ONE
  ⁄       FRACTION SLASH
  2       DIGIT TWO
  >>> print('/', name('/'), sep='\t')
  /       SOLIDUS
  ```
  - `FRACTION SLASH`와 일반적인 slash(`SOLIDUS`)는 다르다.
    - 일반 slash로 검색하면 검색되지 않는다.
  - NFKC와 NFKD는 데이터 손실이 있을 수 있기 때문에 특별한 상황에서만 사용해야 한다.
    - 영구적인 데이터 저장에는 절대 사용하지 마시오

### 대소문자 변경 (case folding)
- case folding이란 소문자화인데, 추가적인 변화가 붙어있다.
  - `str.casefold()` 메소드에 의해 지원된다.
- 문자열 `s`가 `latin1` 문자만을 담은 경우, `s.casefold()`는 `s.lower()`과 거의 동일하다.
  - micro sign 'µ'가 그리스어 소문자 mu 'μ'로 변한다.
  - German Eszett(sharp s) 'ß'가 'ss'로 변한다.
- 300개가 넘는 코드 포인트에서 `str.casefold()`와 `str.lower()`이 다른 결과를 낸다.

### 정규화된 텍스트 매칭을 위한 유틸리티 함수
- NFC와 NFD는 유니코드 문자열 사이에 합리적인 문자열 비교에 사용될 수 있음을 보았다.
- NFC는 대부분의 애플리케이션에서 가장 좋은 정규화이다.
- 대소문자 구별 없는 비교를 할 때에는 `str.casefold()`를 사용하면 된다.

- 다언어 텍스트를 사용한다면 다음 코드로 정의할 수 있는 `nfc_equal`과 `fold_equal`이 유용하다.

normeq.py
```python
from unicodedata import normalize

def nfc_equal(str1, str2):
    return normalize('NFC', str1) == normalize('NFC', str2)

def fold_equal(str1, str2):
    return (normalize('NFC', str1).casefold() ==
            normalize('NFC', str2).casefold())
```
```python
>>> from normeq import nfc_equal, fold_equal
>>> s1 = 'café'
>>> s2 = 'cafe\u0301'
>>> s1 == s2
False
>>> nfc_equal(s1, s2)
True
>>> nfc_equal('A', 'a')
False
>>> s3 = 'Straße'
>>> s4 = 'strasse'
>>> s3 == s4
False
>>> nfc_equal(s3, s4)
False
>>> fold_equal(s3, s4)
True
>>> fold_equal(s1, s2)
True
>>> fold_equal('A', 'a')
True
```

### 극한의 표준화: 발음 구별 기호 떼어내기
- 유니코드 표준의 일부인 유니코드 표준화와 case folding를 넘어서, 더 깊은 변형이 필요할 때가 있다.
  - 'café'를 'cafe'로 변환한다던가
- 발음 구별 기호 떼어내기
  - 구글 검색은 검색 시 특정 context에서는 발음 구별 기호를 무시하고 검색한다.
    - 단, 발음 구별 기호를 떼어내는 것은 단어의 뜻을 바꿔버리거나 검색 시 비정확한 결과를 초래할 수 있다.
    - 근데 검색 시 발음 구별 기호를 붙이지 않거나 정확한 표기를 모르는 사람들이 많다.
  - 발음 구별 기호를 제외하는 것은 가독성이 좋은 URL을 만드는데 도움이 될 수 있다.

simplify.py
```python
import unicodedata
import string

def shave_marks(txt):
    """Remove all diacritic marks"""
    norm_txt = unicodedata.normalize('NFD', txt)
    shaved = ''.join(c for c in norm_txt
                     if not unicodedata.combining(c))
    return unicodedata.normalize('NFC', shaved)
```
```python
>>> order = '“Herr Voß: • ½ cup of Œtker™ caffè latte • bowl of açaí.”'
>>> shave_marks(order)
'“Herr Voß: • ½ cup of Œtker™ caffe latte • bowl of acai.”'
>>> Greek = 'Ζέφυρος, Zéfiro'
>>> shave_marks(Greek)
'Ζεφυρος, Zefiro'
```
- 일반적으로 발음 구별 기호를 제외하는 이유는 라틴 문자를 순수 ASCII로 만들기 위해서이다.
  - `shave_marks`는 라틴 문자가 아닌 그리스 문자까지 변형시킨다.
    - 그리스 문자는 발음 구별 기호를 떼어낸다고 ASCII가 되지는 않는다.

- 라틴 문자에서만 발음 구별 기호를 떼어내는 예.
```python
def shave_marks_latin(txt):
    """Remove all diacritic marks from Latin base characters"""
    norm_txt = unicodedata.normalize('NFD', txt)
    latin_base = False
    preserve = []
    for c in norm_txt:
        if unicodedata.combining(c) and latin_base:
            continue # ignore diacritic on Latin base char
        preserve.append(c)
        # if it isn't combining char, it's a new base char
        if not unicodedata.combining(c):
            latin_base = c in string.ascii_letters
    shaved = ''.join(preserve)
    return unicodedata.normalize('NFC', shaved)
```

- 서양 문자을 ASCII로 변환하는 예시
```python
single_map = str.maketrans("""‚ƒ„ˆ‹‘’“”•–—˜›""",
                           """'f"^<''""---~>""")

multi_map = str.maketrans({
    '€': 'EUR',
    '…': '...',
    'Æ': 'AE',
    'æ': 'ae',
    'Œ': 'OE',
    'œ': 'oe',
    '™': '(TM)',
    '‰': '<per mille>',
    '†': '**',
    '‡': '***',
})

multi_map.update(single_map)


def dewinize(txt):
    """Replace Win1252 symbols with ASCII chars or sequences"""
    return txt.translate(multi_map)


def asciize(txt):
    no_marks = shave_marks_latin(dewinize(txt))
    no_marks = no_marks.replace('ß', 'ss')
    return unicodedata.normalize('NFKC', no_marks)
```
- 언어마다 발음 구별 기호를 제거하는 방법이 다르므로 유의해야한다.
  - 독일어에서, 'ü'는 'ue'와 같다.

## 유니코드 텍스트 정렬하기
```python
>>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
>>> sorted(fruits)
['acerola', 'atemoia', 'açai', 'caju', 'cajá']
```
- 발음 구별 기호가 있으면 원하지 않는 정렬 결과가 나올 수 있다.
- ASCII가 아닌 문자열을 정렬하는 방법은 `locale.strxfrm`을 사용하는 것이다.
  - [`locale` module docs](https://fpy.li/4-16)
  - 문자열을 locale이 이용된 비교에서 사용될 수 있게 변형한다.
- `locale.strxfrm`을 활성화시키기 위해서는 애플리케이션을 위한 적절한 locale이 필요하다.
```
>>> import locale
>>> my_locale = locale.setlocale(locale.LC_COLLATE, 'pt_BR.UTF-8')
>>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
>>> sorted_fruits = sorted(fruits, key=locale.strxfrm)
>>> print(sorted_fruits)
['açai', 'acerola', 'atemoia', 'cajá', 'caju']
```
- sorting key로 `locale.strxfrm`을 사용하기 전에 `setlocale(LC_COLLATE, <<my_locale>>)`로 설정해야한다.

caveats
- locale 설정은 전역이기 때문에 `setlocale`을 라이브러리에서 사용하는 것은 권장되지 않는다.
  - 애플리케이션, 혹은 프레임워크에서 locale은 프로세스가 시작될 때 설정되고 이후에 변경해서는 안된다.
- locale은 OS에 설치되어 있어야한다.
  - 그렇지 않으면 `setlocale`은 `locale.Error: unsupported locale setting` 예외를 발생시킨다.
- locale 이름을 정확히 알아야한다.
- OS 생산자에 의해 locale이 올바르게 구현되어야 한다.
  - Ubuntu 19.10에서는 성공적이었는데, macOS에서는 실패할 수 있다. macOS에서 제대로 구현되어 있지 않다.
- 단순한 해결책: _pyuca_ 라이브러리를 사용하는 것.

### 유니코드 정렬 알고리즘(Unicode Collation Algorithm, UCA)로 정렬하기
- [_pyuca_](https://fpy.li/4-17)
```python
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açai', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
['açai', 'acerola', 'atemoia', 'cajá', 'caju']
```
- `pyuca`의 정렬 방식은 GNU/Linux, macOS, Windows에서 모두 작동한다.
- `pyuca`는 locale을 건드리지 않는다.
- 정렬을 사용자화할 필요가 있을 때는 `Collator()` 생성자에 인자를 줄 수 있다.
  - 이 인자는 정렬표(collation table)이다.
  - 기본적으로는 [_allkeys.txt_](https://fpy.li/4-18)을 사용한다.
    - 이는 Default Unicode Collation Element Table의 복사본이다.
- _pyuca_의 대안으로 locale처럼 동작하는 (그러나 locale을 변경하지는 않는) [PyICU](https://fpy.li/4-20)이 있다.
  - PyICU에는 컴파일되어야 하는 확장 프로그램이 있다.

## 유니코드 데이터베이스
- 유니코드 표준은 문자 전체의 데이터베이스를 제공한다.
  - 코드포인트와 문자 이름의 매핑 뿐만 아니라 각 캐릭터의 메타데이터와 관계까지 적혀있다.
    - 유니코드 데이터베이스는 캐릭터가 출력 가능한지, 알파벳인지, 십진수 숫자인지, 아니면 다른 수학적 기호인지를 기록하고 있다.
    - 이를 통해서 `isalpha`, `isprintable`, `isdecimal`, `isnumeric`, `casefold`같은 `str` 메소드들이 작동한다.
- `unicodedata.category(char)`은 2문자로 된 `char`의 유니코드 카테고리를 리턴한다.
  - 고수준의 `str` 메소드([`label.isapha`](https://fpy.li/4-21))가 더 사용하기 쉽다.
    - 모든 문자가 다음의 카테고리라면 `True`를 반환한다.
    - `Lm`, `Lt`, `Lu`, `Ll`, `Lo`

### 문자 메타데이터
- `unicodedata`모듈은 문자 메타데이터를 가져오는 함수가 있다.
  - `unicodedata.name(char)`: 문자의 유니코드 표준 이름을 가져온다.
  - `unicodedata.numeric(char)`: 문자가 나타내는 숫자값을 리턴한다.
    - 불리언 값을 리턴하는 `str.isdecimal()`, `str.isnumeric()`와는 다르다.
      - 정규표현식 모듈 `re`에서 숫자를 뜻하는 `\d`는 더 협소한 범위를 가진다.
      - PyPI의 `regex` 모듈은 유니코드에 대한 더 좋은 지원을 한다.
    - `str.isnumeric()`이 참이면 `unicodedata.numeric(char)`은 값을 가진다.
    - `str.isnumeric()`이 거짓이면 `ValueError`가 발생한다.
  - [`unicodedata` 모듈의 문서](https://fpy.li/4-25)

## 이중 모드 문자열, `bytes` API
- `str`을 인자로 받느냐 `bytes`을 인자로 받느냐에 따라 행동이 달라지는 함수들이 있다.
  - `re`와 `os`모듈에 그런 함수들이 있다.

### 정규표현식에서 `str`과 `bytes`의 차이
- `bytes`로 만들어진 정규표현식은 `\d`와 `\w`가 ASCII에만 매칭된다.
- `str`으로 만들어진 정규표현식은 `\d`와 `\w`가 유니코드 범위에서 매칭된다.

### `os` 함수에서 `str`과 `bytes`의 차이
- GNU/Linux 커널은 유니코드를 알지 못한다.
  - 어떤 인코딩에도 해당하지 않는 바이트 시퀀스를 파일 이름에서 찾을 수 있다.
  - 특히 여러 종류의 OS가 섞여있는 파일 서버와 클라이언트에서 발생한다.
- 이러한 문제 때문에 `os` 모듈은 파일 이름이나 경로 이름을 `str`이나 `bytes` 둘 모두 지원한다.
  - 이 때, `str`은 코덱 `sys.getfilesystemencoding()`에 의해 `bytes`로 인코딩된다.
    - OS 응답도 해당 코덱으로 디코딩된다.
  - 이런 방법으로 다룰 수 없는 파일 이름을 다룰 때에는 `bytes`를 인자로 넘겨야한다.
- `str`이나 `bytes` 시퀀스를 직접적으로 다루기 위한 함수들이 있다.
  - `os.fsencode(name_or_path)`, `os.fsdecode(name_or_path)`
    - 둘 모두 인자로 `str`또는 `bytes`, 혹은 `os.PathLike` 인터페이스를 구현한 객체를 받는다.

## 읽을 거리
- [Unconfusing Unicode: What Is Unicode?](https://fpy.li/4-31)
- 공식 문서: [Unicode HOWTO](https://fpy.li/4-31)
- [_Dive_ Chapter 4, "Strings"](https://fpy.li/4-33)
- [Python 3 and ASCII Compatible Binary Protocols](https://fpy.li/4-38)
- [Processing Text Files in Python 3](https://fpy.li/4-39)
- 파이썬에서 지원하는 인코딩의 목록: `codecs` 모듈 문서에 있는 [Standard Encodings](https://fpy.li/4-40)
  - 코드에서 얻고 싶으면 [_/Tools/unicodelist/listcodecs.py](https://fpy.li/4-41)
- _Unicode Explained_
- _Unicode Demystified_
- [_Programming with Unicode_](https://fpy.li/4-44)
- W3C
  - [Case Folding: An Introduction](https://fpy.li/4-45)
    - 친절한 소개문
  - [Character Model for World Wide Web: String Matcher](https://fpy.li/4-15)
    - 건조한 표준 문체

- 
