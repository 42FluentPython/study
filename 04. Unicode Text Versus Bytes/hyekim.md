# 4장

- Characters, code points, and byte representations
- Unique features of binary sequences: `bytes`, `bytearray`, and `memoryview`
- Encodings for full Unicode and legacy character sets
- Avoiding and dealing with encoding errors
- Best practices when handling text files
- The default encoding trap and standard I/O issues
- Safe Unicode text comparisons with normalization
- Utility functions for normalization, case folding, and brute-force diacritic removal
- Proper sorting of Unicode text with `locale` and the *pyuca* library
- Character metadata in the Unicode database
- Dual-mode APIs that handle `str` and `bytes`

## ****Character Issues****

String이란 Character의 연속이다. 그럼 Character란 무엇인가?

2021년, character의 정의는 unicode 문자이다.

이러한 정의는 chracter의 정체성과 byte를 명확하게 구분한다.

코드포인트를 바이트로 변환하는 것을 인코딩, 바이트를 코드포인트로 변환하는 것을 디코딩 이라고한다.

파이썬3의 str은 파이썬 2의 unicode type과 비슷하다. 하지만 python3 의 bytes는 python2 의 str을 단순히 이름만 변경한게 아니고 이와 관련된 bytearray 타입도 있다.

## **Byte Essentials**

새로 도입된 이진 시퀀스 형은 파이썬 2의 str과 다르다. bytes 타입은 파이썬 3에서 추가된 불변형, bytearray는 파이썬 2.6에 추가된 가변형. 

byte, bytearray는 0 ~ 255 사이의 정수로 구성되어있다.

## **Basic Encoders/Decoders**

텍스트를 바이트로 혹은 바이트를 텍스트로 변환하기 위해 파이썬 배포본에는 100여 개의 코덱(인코더/디코더 )이 포함되어 있다.

하나의 텍스트를 세개의 서로다른 바이트 시퀀스로 인코딩한  결과이다.

```python
>>> for codec in ['latin_1', 'utf_8', 'utf_16']:
...     print(codec, 'El Niño'.encode(codec), sep='\t')
...
latin_1 b'El Ni\xf1o'
utf_8   b'El Ni\xc3\xb1o'
utf_16  b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'
```

## **Understanding Encode/Decode Problems**

UnicodeError라는 범용 예외가 있지만, 거의 항상 UnicodeEncodeError나 UnicodeDecodeError같은 구체적인 예외가 발생한다.

### UnicodeEncodeError

대부분의 비UTF 코덱은 유니코드 문자의 일부만 처리한다. 따라서 텍스트를 바이트로 인코딩할 때 문자가 대상 인코딩에 정의되어있지 않으면 UnicodeEncodeError 가 발생한다.

### **UnicodeDecodeError**

디코드할 때 바이트가 정당한 문자로 변환될수 없으면 UnicodeDecodeError가 발생한다

UnicodeEncodeError, UnicodeDecodeError 둘다 에러처리기를 사용하면 에러를 묵인할 수 있다. 

## **Handling Text Files**

텍스트를 처리하는 최고의 방법은 유니코드 샌드위치다.

- 입력시 바이트를 디코딩한다.
- 텍스트만 처리한다.
- 출력시 텍스트를 인코딩한다.

이 과정을 따르지 않으면 여러 문제가 발생할 수 있다.

## **Normalizing Unicode for Reliable Comparisons**

유니코드에는 combining characters가 있기 때문에 문자열 비교가 간단하지 않다. 앞문자에 연결되는 발음 구별 기호는 인쇄할 때 앞 문자와 하나로 결합되어 출력된다.

예를 들어 'café'라는 단어는 네 개나 다섯 개의 코드 포인트를 이용해서 두 가지 방식으로 표현할 수 있지만 결과는 동일하게 나타난다.

이 문제를 해결하려면 unicodedata.normalize( )를 써야한다.

유니코드 정렬할 때는 locale.strxfrm( ) 로 비아스키 텍스트를 정렬해야한다
****

##
