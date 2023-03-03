# A Pythonic Object
- 장 전체가 하나의 예시기 때문에 많은 부분 생략한다.
- [예시 코드](https://github.com/fluentpython/example-code-2e/blob/master/11-pythonic-obj/vector2d_v3_slots.py)

## `__iter__`
- `__iter__`을 사용하면 객체를 언패킹할 수 있다.
```python
>>> class Demo:
...     def __init__(self, a, b, c):
...             self.a = a
...             self.b = b
...             self.c = c
...     def __iter__(self):
...             return (i for i in (self.a, self.b, self.c))
... 
>>> d = Demo(1, 2, 3)
>>> a, b, c = d
>>> a, b, c
(1, 2, 3)
```

## byte representation, alternative constructor
- 다음의 코드를 통해 객체를 바이트로 변환할 수 있다.
```python
class Vector2d:
    typecode = 'd'

    ...

    def __bytes__(self):
        return (bytes([ordself.typecode]) + bytes(array(self.typecode, self)))
    
    @classmethod
    def frombytes(cls, octets):
        typecode = chr(octets[0])
        memv = memoryview(octets[1:]).cast(typecode)
        return cls(*memv)
```
```python
>>> v1 = Vector2d(1., 2.)
>>> octets = bytes(v1)
>>> v2 = Vector2d.frombytes(octets)
>>> v1 == v2
True
```

## @classmethod vs @staticmethod
- classmethod는 첫 번째 인자로 항상 클래스 자신을 가진다. 두 번째 인자부터 매개변수를 받는다.
- staticmethod는 첫 번째 인자가 첫 번째 매개변수다.
```python
class Demo:
    @classmethod
    def classmeth(*args):
        return args

    @staticmethod
    def staticmeth(*args):
        return args
```
```python
>>> Demo.classmeth()
(<class '__main__.Demo'>,)
>>> Demo.staticmeth()
()
```

## Formatted Displays
- f-문자열, `format()` 내장 함수, `str.format()` 메소드는 `.__format__(format_spec)`에 구현을 위임한다.
  - `format_spec`은
    - `format(my_obj, format_spec)`의 두 번째 인자이거나
    - f-문자열 {} 내부의 `:` 뒤에 들어오는 문자열이거나
    - `fmt.str.format()`의 `fmt`이다.
```python
>>> brl = 1 / 4.82
>>> brl
0.20746887966804978
>>> format(brl, '0.4f')
'0.2075'
>>> '1 BRL = {rate:0.2f} USD'.format(rate=brl)
'1 BRL = 0.21 USD'
>>> f'1 USD = {1 / brl:0.2f} BRL'
'1 USD = 4.82 BRL'
```
- `'{0.mass:.2f}, {1[0]:+}'`와 같이 어트리뷰트나 인덱스로 접근할 수도 있다.
- 몇 내장 타입들은 각각의 Format Specification Mini-Language를 가지고 있다.
```python
>>> format(42, 'b') # int의 경우
'101010'
>>> format(2 / 3, '.1%') # float의 경우
'66.7%'
```
- vector2d의 경우에는 다음의 `__format__`을 구현할 수 있다.
  - p의 경우에는 극좌표계이다.
```python
    def angle(self):
        return math.atan2(self.y, self.x)

    def __format__(self, fmt_spec=''):
        if fmt_spec.endswith('p'):
            fmt_spec = fmt_spec[:-1]
            coords = (abs(self), self.angle())
            outer_fmt = '<{}, {}>'
        else:
            coords = self
            outer_fmt = '({}, {})'
        components = (format(c, fmt_spec) for c in coords)
        return outer_fmt.format(*components)
```

- `__hash__`와 `__eq__`를 구현하면 hashable 객체가 된다.
  - set의 요소 dict의 키로 쓸 수 있다.
  - attribute의 튜플을 키로 사용하여 `hash` 내장 함수를 사용하면 쉽게 hashable을 만들 수 있다.
```python
def __hash__(self):
    return hash((self.x, self.y))
```
- `__int__`를 구현하면 `int` 내장 함수를, `__float__`를 구현하면 `float` 내장 함수를, `__complex__`를 구현하면 `complex` 내장 함수를 사용할 수 있다.

## Positional Pattern Matching
- 기본적으로 어트리뷰트를 이용한 클래스 패턴 매칭이 지원된다.
```python
def keyword_pattern_demo(v: Vector2d) -> None:
    match v:
        case Vector2d(x=0, y=0):
            print(f'{v!r} is null')
        case Vector2d(x=0):
            print(f'{v!r} is vertical')
        case Vector2d(y=0):
            print(f'{v!r} is horizontal')
        case Vector2d(x=x, y=y) if x==y:
            print(f'{v!r} is diagonal')
        case _:
            print(f'{v!r} is awesome')
```

- `__match_args__`를 사용하면 positional matching이 지원된다.
```python
class Vector2d:
    __match_args__ = ('x', 'y')
    ...
```
```python
def positional_pattern_demo(v: Vector2d) -> None:
    match v:
        case Vector2d(0, 0):
            print(f'{v!r} is null')
        case Vector2d(0):
            print(f'{v!r} is vertical')
        case Vector2d(_, 0):
            print(f'{v!r} is horizontal')
        case Vector2d(x, y) if x==y:
            print(f'{v!r} is diagonal')
        case _:
            print(f'{v!r} is awesome')
```
- `__match_args__`는 모든 퍼블릭 어트리뷰트를 포함해야하는 것은 아니다.
  - `__init__`에 optional argument가 있다면 빼는 것도 좋은 방법이다.

## protected attribute 만들기
```python
class Vector2d:
    ...

    def __init__(self, x, y):
        self.__x = float(x)
        self.__y = float(y)
    
    @property
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y
```
- 어트리뷰트 앞에 `__`를 붙이면 클래스 내부 `__dict__`에 클래스 이름과 같이 저장된다.
  - Dog 클래스 내의 `__mood`는 `_Dog__mood`로 저장된다.
  - Dog를 상속한 Beagle 클래스 내의 `__mood`는 `_Beagle__mood`로 저장된다.
  - 이 방법을 통해 어트리뷰트의 가림(shading)을 막을 수 있다.
- 이 방식은 '안전'을 위한 것이지, '보안'을 위한 것이 아니다.
  - 안전하다는 것은 실수를 방지한다는 것
  - 보안적인 측면은 악의적인 이용을 막는 것
```python
>>> v1 = Vector2d(1, 2)
>>> v1.__x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Vector2d' object has no attribute '__x'
>>> v1._Vector2d__x
1.0
```
- `__`를 이용하지 않고 그냥 `_x`와 같이 사용하는 경우도 많다.

## `__slots__`로 메모리 절약하기
- `__slots__`에 등록한 어트리뷰트는 `__dict__`이 생성되지 않고 추가적인 어트리뷰트도 등록할 수 없게 된다.
- 메모리도 절약할 수 있고, 속도도 약간 더 빠르다.

```python
>>> class Pixel:
...     __slots__ = ('x', 'y')
... 
>>> p = Pixel()
>>> p.__dict__
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Pixel' object has no attribute '__dict__'. Did you mean: '__dir__'?
>>> p.x = 10
>>> p.y = 20
>>> p.color = 'red'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Pixel' object has no attribute 'color'
```
- `__slots__`을 나중에 추가하거나 수정하는 것은 아무 영향이 없다.
- 상속하면 서브클래스의 `__dict__`가 생기고, 어트리뷰트 추가가 가능하다.
```python
>>> class OpenPixel(Pixel):
...     pass
... 
>>> op = OpenPixel()
>>> op.__dict__
{}
>>> op.x = 8
>>> op.__dict__
{}
>>> op.x
8
>>> op.color = 'green'
>>> op.__dict__
{'color': 'green'}
```
- 이를 방지하기 위해서는 서브클래스에도 `__slots__`을 등록해야한다.
```python
class ColorPixel(Pixel):
    __slots__ = ('color',)
```
- 혹은,
```python
class SamePixel(Pixel):
    __slots__ = (,)
```
- 객체가 약한 참조의 대상이 되기 위해서는 `__slots__`에 `'__weakref__'`를 포함시켜야 한다.
- `'__dict__'`를 포함시키면 새로운 어트리뷰트를 등록할 수 있다. (메모리 절약 효과를 반감시킨다.)
- `__slots__`를 사용한 클래스는 `@cached_property`를 사용할 수 없다.


## Further Readings
- `__fspath__`에 대해 다룬 글. [PEP519-Adding a file system path protocol](https://fpy.li/pep519)

## Security vs Safety
- Java의 경우 SecurityManager 상에서 배포하지 않으면 악의적인 이용을 막을 수 없다.
  - 그 외의 경우 악의적인 이용을 막는다고 해도 reflection 등으로 뚫을 수 있다.
  - 대부분 SecurityManager를 사용하지 않는다.
  