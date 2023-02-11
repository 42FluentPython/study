# Functions as First-Class objects
- 파이썬은 함수형 프로그래밍 언어는 아니다. 하지만 함수형 프로그래밍의 특징을 일부 가지고 있다
- 일급 객체란 다음과 같은 특징을 가진 개체를 가리킨다.
  - 런타임에 생성될 수 있음.
  - 변수에 할당되거나 데이터 구조의 요소에 할당될 수 있음.
  - 함수의 인자로 넘겨질 수 있음.
  - 함수의 결과값으로 넘겨질 수 있음.

- 함수 외에도 정수, 문자열, 딕셔너리 등이 일급 객체이다.
- 함수가 일급 객체인 것은 함수형 프로그래밍 언어의 중요한 특징이다.
  - Clojure, Elixir, Haskell 등
- 인기 있는 몇 언어들에서도 함수가 일급 객체이지만 이들은 함수형 프로그래밍 언어는 아니다.
  - JavaScript, Java, Go 등

## 함수를 객체처럼 다루기.
```python
>>> def factorial(n):
...     """returns n!"""
...     return 1 if n < 2 else n * factorial(n - 1)
... 
>>> factorial(10)
3628800
>>> factorial.__doc__
'returns n!'
>>> type(factorial)
<class 'function'>
>>> help(factorial)
```
* help(x)는 별개의 화면을 띄운다.

- `factorial`에 별개의 이름을 지은 후, 함수에 인자로 넘겨주기
```python
>>> fact = factorial
>>> fact
<function factorial at 0x100b6c700>
>>> fact(5)
120
>>> map(fact, range(11))
<map object at 0x100b5ea70>
>>> list(map(fact, range(11)))
[1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]
```

## 고차 함수
- 함수를 인자로 받는 함수나 함수를 리턴하는 함수를 고차 함수라고 한다.
  - 예를 들자면 `key` 인자를 함수로 받는 `sorted`가 있다.
  ```python
  >>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
  >>> sorted(fruits, key=len)
  ['fig', 'apple', 'cherry', 'banana', 'raspberry', 'strawberry']
  ```

- 함수형 프로그래밍의 패러다임에서 잘 알려진 함수는 `map`, `filter`, `reduce`, `apply`(deprecated) 등이 있다.
  - `apply(fn, args, kwargs)`는 `fn(*args, **kwargs)`로 대체됐다.

### `map`, `filter`, `reduce`의 modern replacements
- listcomp, genexp는 `map`, `filter`을 대체할 수 있다.
```python
>>> list(map(fact, range(6)))
[1, 1, 2, 6, 24, 120]
>>> [fact(n) for n in range(6)]
[1, 1, 2, 6, 24, 120]
>>> list(map(fact, filter(lambda n: n % 2, range(6))))
[1, 6, 120]
>>> [factorial(n) for n in range(6) if n % 2]
[1, 6, 120]
```
- python3에서 `map`과 `filter`는 제너레이터를 리턴했다.
- `reduce` 함수는 python2에서 내장 함수였으나 python3에서는 `functools` 모듈로 이동되었다.
- 합을 구하는 `reduce`는 `sum`으로 대체될 수 있다.
- `reduce`를 대체하는 또 다른 내장 함수는 `all(iterable)`과 `any(iterable)` 등이 있다.
  - `all`은 iterable에 falsy값이 없으면 True를 리턴한다.
    - `all([])`은 `True`를 리턴한다.
  - `any`는 iterable에 truthy값이 있다면 True를 리턴한다.
    - `any([])`는 `False`를 리턴한다.
- `reduce`에 대한 자세한 사항과 대체하는 추가적인 여러 함수들은 나중에 다룰 것이다.

## 익명 함수
- `lambda` 키워드는 익명 함수를 만들어낸다.
  - `lambda`에는 expression만 들어갈 수 있다.
    - `while`, `for`, `try` 등의 다른 statement가 들어갈 수 없다.
    - `=` 대입 역시 statement이다.
    - `:=`는 사용할 수 있다. 그러나 이 정도 되면 람다가 너무 복잡해질 것이다.
- 익명 함수의 최선 사용 방식은 고차 함수의 인자로 넣는 것이다.
```python
sorted(fruits, key=lambda word: word[::-1])
```
- 이외의 상황에서 익명 함수는 그렇게 유용하지 않다. `def`를 쓰자.

## 9 종류의 호출 가능한 객체
- callable(호출 가능한 객체)인지 판별하려면 `callable` 내장 함수를 사용하자.
- 다음 9가지의 callable이 존재한다.
1. 사용자 정의 함수
  - `def`나 `lambda`를 통해 정의된 함수.
2. 내장 함수
  - (CPython의 경우) C로 구현된 함수들. `len`, `time.strftime` 등이 있다.
3. 내장 메소드
  - C로 구현된 메소드. `dict.get` 등이 있다.
4. 메소드
  - 클래스 본문에 정의된 함수들
5. 클래스
  - 호출되면 `__new__` 메소드를 통해 인스턴스를 생성한 후 `__init__`을 통해 초기화가 되고, 인스턴스가 리턴된다.
  - `new` 연산자가 없기 때문에 함수처럼 호출된다.
6. 클래스 인스턴스
  - 클래스에 `__call__` 메소드가 정의되어 있다면 인스턴스는 함수처럼 호출될 수 있다.
7. 제너레이터 함수
  - `yield` 키워드를 본문에 포함하고 있는 함수나 메소드. 
8. 네이티브 코루틴 함수
  - `async def`로 정의된 함수. 호출되면 코루틴 객체를 리턴한다.
9. 비동기 제너레이터 함수
  - `async def`로 정의되고 `yield` 키워드를 본문에 포함하고 있는 함수.
  - 호출되면 `async for`과 함께 사용할 수 있다.
- 7, 8, 9는 나중에 다룰 것이다.

## 사용자 정의 callable
- `__call__` 메소드를 정의함으로써 임의의 객체를 callable로 만들 수 있다.
```python
import random


class BingoCage:
    def __init__(self, items):
        self._items = list(items)
        random.shuffle(self._items)

    def pick(self):
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError("pick from empty BingoCage")

    def __call__(self):
        return self.pick()
```
```python
>>> from bingocall import BingoCage
>>> bingo = BingoCage((range(2)))
>>> bingo.pick()
1
>>> bingo()
0
>>> bingo()
Traceback (most recent call last):
...
LookupError: pick from empty BingoCage
```
- 이런 방법은 내부 상태를 가지고 있는 함수 비슷한 객체를 만드는데 유용하다.
- `__call__`은 데코레이터를 만드는데에도 이용된다.
  - 데코레이터는 callable이여야 한다.
  - 데코레이터의 호출 간에 값을 기억해야하는 경우가 있다.
  - 복잡한 구현을 여러 메소드로 분해하는 경우가 있다.
  - 데코레이터에 대한 자세한 것은 나중에 다룰 것.

## positional과 keyword-only argument

### keyword-only argument

```python
def func(arg, *args, key, **kwargs)
    print('arg: ', arg)
    print('args: ', args)
    print('key: ', key)
    print('kwargs: ', kwargs)
```
- `*`이 붙은 인자 뒤에는 keyword-only argument만이 올 수 있다.
```python
>>> func()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 1 required positional argument: 'arg'
>>> func('arg')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 1 required keyword-only argument: 'key'
>>> func('arg', key='key0')
arg:  arg
args:  ()
key:  key0
kwargs:  {}
>>> func('arg', 'arg1', 'arg2', key='key0', key1='key1')
arg:  arg
args:  ('arg1', 'arg2')
key:  key0
kwargs:  {'key1': 'key1'}
```
- positional argument의 수를 제한하고 싶거나, 아예 사용하고 싶지 않다면 `*`를 사용할 수 있다.
- `**kwargs` 를 사용하지 않는다면 keyword argument도 제한된다.
- `*`
```python
def func(*, key):
    print(key)
```
```python
>>> func()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 1 required keyword-only argument: 'key'
>>> func(key=1, key1=2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() got an unexpected keyword argument 'key1'
>>> 
>>> func(1, key=1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() takes 0 positional arguments but 1 positional argument (and 1 keyword-only argument) were given
```

### positional-only parameter
- python 3.8부터는 positional-only parameter를 사용할 수 있다.
- positional-only parameter는 position으로만 받고 keyword로 받을 수 없다.
```python
def divmod(a, b, /):
    return (a // b, a % b)
```
```python
>>> from divmod import divmod
>>> divmod(1, 2, 3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: divmod() takes 2 positional arguments but 3 were given
>>> divmod(a=1, b=2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: divmod() got some positional-only arguments passed as keyword arguments: 'a, b'
```

## 함수형 프로그래밍을 위한 패키지

### `operator` 모듈
- 팩토리얼 함수는 `reduce`를 이용하여 다음과 같이 작성할 수 있다.
```python
from functools import reduce

def factorial(n):
    return reduce(lambda a, b: a*b, range(1, n+1))
```
- 덧셈에는 `sum` 내장 함수가 있지만 곱셈에는 해당하는 내장 함수가 없기 때문에 람다를 써야한다.
- `operator` 모듈에는 내장 함수에서 제공하지 않는 이런 연산들을 제공한다.
  - 다음과 같이 `factorial`을 다시 쓸 수 있다.
```python
from functools import reduce
from operator import mul

def factorial(n):
    return reduce(mul, range(1, n+1))
```

- `itemgetter`은 팩토리 함수로, `[]` 연산자를 이용하여 객체의 요소를 가져오는 함수를 만든다.
  - `[]` 연산자를 이용하기 때문에 `__getitem__`이 구현되어있는 객체에서 사용할 수 있다.
  - 시퀀스 타입과 매핑 타입에서 모두 이용 가능하다.
```python
>>> itemgetter(5)(range(10))
5
>>> itemgetter(0, 1)((1, 2))
(1, 2)
>>> itemgetter('key0', 'key1')({'key1': 'value1', 'key0': 'value0'})
('value0', 'value1')
```
- `attrgetter`도 팩토리 함수로, 객체 속성(.)을 이름으로 가져오는 함수를 만든다.
  - 인자는 무조건 문자열이어야 한다.
  - 중첩도 된다.
```python
>>> from collections import namedtuple
>>> LatLon = namedtuple('LatLon', 'lat lon')
>>> ll = LatLon(lat=11.1, lon=22.2)
>>> ll
LatLon(lat=11.1, lon=22.2)
>>> attrgetter('lat')(ll)
11.1
>>> City = namedtuple('City', 'name coord')
>>> somecity = City(name='somewhere', coord=ll)
>>> attrgetter('coord.lat')(somecity)
11.1
```

- `methodcaller`는 어떤 객체의 메소드를 호출하는 함수를 만든다.
```python
>>> from operator import methodcaller
>>> s = 'The time has come'
>>> upcase = methodcaller('upper')
>>> upcase(s)
'THE TIME HAS COME'
>>> hyphenate = methodcaller('replace', ' ', '-')
>>> hyphenate(s)
'The-time-has-come'
```
- 그런데 `upper`를 함수로 쓰고싶다면 `str.upper`이 있다.

- `operator` 모듈에는 다음과 같은 함수들이 있다.
```python
>>> [name for name in dir(operator) if not name.startswith('_') and not name.endswith('_')]
['abs', 'add', 'attrgetter', 'concat', 'contains', 'countOf', 'delitem', 'eq', 'floordiv', 'ge', 'getitem', 'gt', 'iadd', 'iand', 'iconcat', 'ifloordiv', 'ilshift', 'imatmul', 'imod', 'imul', 'index', 'indexOf', 'inv', 'invert', 'ior', 'ipow', 'irshift', 'is_not', 'isub', 'itemgetter', 'itruediv', 'ixor', 'le', 'length_hint', 'lshift', 'lt', 'matmul', 'methodcaller', 'mod', 'mul', 'ne', 'neg', 'pos', 'pow', 'rshift', 'setitem', 'sub', 'truediv', 'truth', 'xor']
```

## `functools.partial`로 매개변수 고정시키기.
- partial은 positinal이나 keyword argument를 고정시킬 수 있다.
```python
>>> from operator import mul
>>> from functools import partial
>>> triple = partial(mul, 3)
>>> triple(7)
21
```
```python
>>> from unicodedata import normalize
>>> from functools import partial
>>> nfc = partial(normalize, 'NFC')
>>> s1 = 'café'
>>> s2 = 'cafe\u0301'
>>> s1, s2
('café', 'café')
>>> s1 == s2
False
>>> nfc(s1) == nfc(s2)
True
```
- `functools.partialmethod`는 메소드에 대한 `partial`을 제공한다.
