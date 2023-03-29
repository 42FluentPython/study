# Interfaces, Protocols, and ABCs

## 4 종류의 typing

- Duck typing
  - 파이썬의 기본 타이핑
- Goose typing
  - Python 2.6부터 지원. ABC 사용.
- Static typing
  - Python 3.5부터 지원. `typing` 모듈을 통해 지원
- Static duck typing
  - Python 3.8부터 지원. `typing.Protocol`을 통해 지원.
  - Golang에 의해 널리 알려짐.

## 두 종류의 Protocol

- `__getitem__()`만을 구현하는 것으로 인덱싱을 구현할 수 있다.
  - `PySequence_Check`는 dict가 아닌 클래스에 `__getitem__()`이 구현되었을 때 1을 리턴한다.
  - `len()` 함수를 사용할 수 없다.
- 여기서 프로토콜이란 "informal interface"이다.
- 정확히 구분하자면, 타이핑에서는 두 종류의 프로토콜이 있다.
  - 동적 프로토콜
    - Python이 항상 지원하는 프로토콜.
    - 인터프리터에 의해 지원된다.
  - 정적 프로토콜
    - Python 3.8부터 지원되는 구조적 subtyping.
    - `typing.Protocol`의 서브클래스를 통해 지원.
- 동적 프로토콜의 일부만을 구현했더라도 유용하게 사용할 수 있다.
- 정적 프로토콜은 필요가 없더라도 모든 프로토콜을 구현해야한다.
- 정적 프로토콜은 외부 타입체커에 의해 검사될 수 있다.
- 둘 다 이름을 직접 언급하지 않아도 (e.g. 상속) 프로토콜을 지원하는 클래스를 선언할 수 있다.
- 파이썬은 정적 프로토콜 외에, 명시적 인터페이스를 가지고있다. (ABCs, Goose typing)

## Duck Typing

### Python Digs Sequences

- `Sequence` ABC는 `__getitem__`과 `__len__`(`Sized`)를 구현해야한다.
- `FrenchDeck`에는 둘 다 구현되어 있다.
  - `__iter__`을 직접 구현하지 않아도 `__getitem__`을 이용하여 순회한다.
  - `__contains__`를 직접 구현하지 않아도 `in` 연산자를 사용할 수 있다.
- 두 함수만 구현하면 `Sequence`가 된다.

### Monkey Patching: 런타임에 프로토콜 구현하기

- Monkey patching은 기능 구현이나 버그 수정을 위해 모듈, 클래스, 함수를 동적으로 변화시키는 것을 뜻한다.
- `FrenchDeck`에는 `__setitem__`이 없어서 `random.shuffle`(in-place shuffle) 함수를 사용할 수 없다.

```python
def set_card(deck, position, card): ...
FrenchDeck.__setitem__ = set_card
```

- 위의 구현을 통해 `random.shuffle`을 사용할 수 있다.

  - 참고로, `deck`, `position`, `card` 자리에는 어떤 변수 이름이 와도 상관없다.
  - 그러나 컨벤션을 따라 `self`, `key`, `value`를 쓰는 것이 좋다.

- monkey patching은 강력한 도구이다.
  - 그러나 monkey patching 코드와 프로그램이 강하게 결합되게 한다.
  - 공개되지 않고 문서화되지 않은 속성들을 만든다.
- `shuffle`은 인자의 타입(클래스)에 상관하지 않는다.
  - mutable sequence 프로토콜만 만족하면 된다.
  - 동적으로 성립된 mutable sequence도 해당된다.

### 방어적 프로그래밍과 "Fail Fast"

- 명시적 런타임 타입 체크 없이 동적 프로토콜을 확인하는 방법에 대해 알아볼 것.
- 어떤 버그들은 런타임에서만 체크될 수 있다. (모든 언어에서)
- 동적 타입 언어에서는 "fail fast"가 좋은 전략이다.

```python
def __init__(self, iterable):
    self._balls = list(iterable)
```

- 이 메소드는 `list` 생성자에 의해 iterable을 인자로 받을 수 있다.
  - iterable이 아닌 인자는 TypeError를 발생시킬 것이다.
- `try`/`except` 문으로 감싸서 에러 메시지를 바꿀 수 있다.
  - 그러나 외부 API가 아닌 경우라면 권장하지 않는다.
- 알맞은 인자를 사전에 처리하지 않는다면 나중에 버그를 내뿜을 것이다.
  - 그러면 버그 원인을 찾기 어려워짐
- 데이터가 복사되면 안 된다거나, 데이터가 너무 크다거나, in-place로 인자를 변화시키는 함수라면 `list()`가 적절하지 않을 수 있다.
- 그 경우에는 `isinstance(x, abc.MutableSequence)`로 런타임 체크할 수 있다.
- 무한 generator를 피하고 싶다면 `len()`을 사용할 수 있다.
  - 튜플, 어레이, Sequence를 구현한 클래스를 제외한 것들을 골라낼 수 있다.
- 어떤 iterable 이라도 상관 없다면 함수 최상단에서 `iter()` 클래스를 사용하면 된다.
- 이런 방법으로 type hint가 오류를 사전에 찾아낼 수 있다.
  - type hint가 없는 타입(`Any`)의 경우는 찾아낼 수 없다.
- 방어적 코드는 duck typing이 `isinstance()`나 `hasattr()` 검사 없이 타입을 다룰 수 있게 해준다.

```python
try:
    field_names = field_names.replace(',', ' ').split()
except: AttributeError:
    pass
field_names = tuple(field_names)
if not all(s.isidentifier() for s in field_names):
    raise ValueError('field_names must all be valid identifiers')
```

- `str`의 메소드를 사용함으로써 `str`을 골라낼 수 있다.
- `str`이 아니면 iterable일 것을 가정한다.
- `tuple()`을 사용함으로써 iterable을 골라낼 수 있다.
- `isidentifier()`를 사용함으로써 identifier가 아닌 str을 골라낼 수 있다.
- 타입 힌트에서는 `isidentifier()` 체크를 사용할 수 없다.
  - duck typing이 타입 힌트보다 더 잘 체크한다.

## Goose Typing

- Python에는 `interface` 키워드가 없다.
  - 런타임에 사용하기 위하여 ABC가 있다.
- ABC를 이용하면 런타임 타입 체크가 가능하고, 정적 타입 체크도 가능하다.
- ABC의 virtual subclass를 사용한다면, 상속 없이 `isinstance()`, `issubclass()`를 사용할 수 있다.
- 행동이나 의미가 다르지만 우연히 이름이 같은 메소드를 가진 클래스들이 있을 수 있다.
  - ABC에 등록하면 같은 인터페이스로 분류되는 것을 피할 수 있다.
  - 어떤 인터페이스는 ABC에 등록하지 않아도 `isinstance()`, `issubclass()`를 만족한다.
    - e.g. `Sized`
- 해당하는 ABC가 있다면 모두 등록하는 것이 좋다.
  - 외부 라이브러리에 등록되지 않은 클래스가 있다면, 직접 해라.
- 새로운 ABC를 정의하지 마라.
  - 새로운 도구를 얻으면 써보고 싶기 마련이지만 그게 해답이 아닐 수도 있다.
- 타입 체크를 할 때는 구체 클래스가 아니라 ABC로 체크해라.
  - 구체 클래스로 체크하면 다형성을 제한한다.
  - ABC로 체크하면 더 유연하다.
- `isinstance`를 너무 많이 사용하는 것은 코드 냄새다.
  - 타입마다 다른 행동을 하기 위하여 `if`/`elif`/`elif`를 사용하는 것은 안 좋은 행위이다.
  - dispatch하라.
- API를 만드는 경우, 타입을 확인하기 위하여 `isinstance`를 사용하는 것은 괜찮다.

### Subclassing an ABC

- `collections.abc`의 ABC를 제대로 구체화하지 않아도 import time에 오류가 나지 않는다.
  - 인스턴스화 과정에서 오류가 난다.
- 추상 메소드가 아닌 구체 메소드도 더 효율적인 방법으로 override 할 수 있다.

### 표준 라이브러리의 ABC

- 17개의 `collections.abc`에 있다.
- `Callable`, `Hashable`은 collection이 아니지만 중요하기 때문에 포함되었다.
  - `isinstance(obj, Callble)`보다 `callble(obj)` 함수가 더 사용하기 쉽다.
  - `isintance(obj, Hashable)`이 `True`이더라도 hashable이 아닐 수 있다.
    - `__hash__` 함수가 있는지 아닌지 검사하기 때문이다.
    - 예를 들어, mutable한 내용물을 가진 튜플은 hashable이 아니다.
    - hashable을 체크할 때에는 `hash(obj)`를 쓰자.

### ABC 만들어보기

- 만들어보는 이유는 표준 라이브러리의 ABC를 이해하기 위해서지, 쓰라고 배우는게 아니다.
- `abc.ABC`를 상속하고, `@abc.abstractclass`를 사용함으로써 ABC를 정의할 수 있다.
  - `@abc.abstractclass`를 붙인 함수에 구현을 해놓을 수도 있다.
    - 하지만 데코레이터가 상속받은 클래스에서 구현할 것을 요구한다.
    - ABC의 구현은 `super()`로 호출할 수 있다.
  - `@abc.abstractclass`는 다른 데코레이터보다 먼저 붙여져야 한다.

### ABC의 virtual subclass

- `@{nameofABC}.register` 데코레이터로 등록할 수 있다.
- `{nameofABC}.register(cls)`함수로도 등록할 수 있다.
- `__mro__`(Method Resolution Order)에는 수퍼클래스들이 등록되어있는데, register로 등록한 클래스는 들어있지 않다.
- `collections.abc`에는 다음과 같은 코드가 있다.

```python
Sequence.register(tuple)
Sequence.register(str)
Sequence.register(range)
Sequence.register(memoryview)
```

### ABC를 이용한 구조적 타이핑

- nominal typing은 클래스 관계를 명시해주는 것.
- structural typing은 클래스 구조에 의해 타입이 결정되는 것.
- `Sized`는 구조적 타이핑을 지원한다.

```python
def __subclasshook__(cls, C):
    if cls is Size:
        if any("__len__" in B.__dict__ for B in C.__mro__)
            return True
    return NotImplemented
```

- `issubclass()`를 호출하면 위 함수가 실행된다.
- 추상 메소드 하나만 있는 ABC에서는 `__subclasshook__`을 구현하는 것이 좋을 수도 있다.
- ABC가 복잡하면 우연히 구조적으로 비슷할 수도 있다.
  - mapping은 `__len__`, `__getitem__`, `__iter__`을 구현하고 있지만 `Sequence`의 서브타입은 아니다.
    - 이런 연유로 `Sequence`는 `__subclasshook__`이 없다.

## Static Protocols

### 타입이 있는 `double` 함수

- python 3.8 이전에는 타입 힌트가 duck typing을 지원하지 않았다.
- `typing.Protocol`을 사용하면 `double`이 `x`에 대해 `x * 2`를 지원하도록 만들 수 있다. (static duck typing)

```python
from typing import TypeVar, Protocol

T = TypeVar('T')

class Repeatable(Protocol):
    def __mul__(self: T, repeat_count: int) -> T: ...

RT = TypeVar('RT', bound=Repeatable)

def double(x: RT) -> RT:
    return x * 2
```

- `self`에는 보통 타입 힌트가 붙지 않지만 리턴 값이 같은 타입이라는 것을 나타내기 위하여 `T`를 붙였다.

### 런타임에 체크 가능한 정적 프로토콜

- `typing.Protocol`의 서브 클래스에 `@runtime_checkable`을 붙이면 런타임에 타입 체크가 가능하다.
- python 3.9부터 미리 만들어진 프로토콜들이 있다
  - `typing.SupportsComplex`
    - `__complex__` 메소드를 구현한 경우
    - `complex(o)`를 사용하는 경우를 상정하고 만들어짐.
  - `typing.SupportsFloat`
    - `__float__` 메소드를 구현한 경우
    - `float(o)`를 사용하는 경우를 상정하고 만들어짐.
- `complex` 내장 타입은 `__complex__`를 구현하고 있지 않지만 `complex(o)`의 인자로 사용될 수 있다.
  - `isinstance(c, (complex, SupportsComplex))`를 사용하면 된다.
- `numbers`에 정의된 `Complex` ABC를 이용할 수도 있다.
  - `complex` 내장 타입도 `isinstance()`테스트를 통과한다.
  - 그러나 정적 타입 체커에 의해 검사되지 않는다.
- 다음의 duck typing을 이용하자.

```python
try:
    c = complex(o)
except: TypeError as exc:
    raise TypeError('o must be convertible to complex') form exc
```

- 그냥 타입 에러를 일으키려면 다음의 한 줄로 충분하다.

```python
c = complex(o)
```

### 런타입 프로토콜 체크의 한계점

- `__float__`가 `float`를 리턴하지 않더라도 프로토콜의 subclass로 간주된다.
- `complex`는 `__float__` 메소드를 구현했지만 이는 에러 메시지를 던지기 위해서이다.
  - `NoReturn` 리턴 타입에 해당하는 경우
- python 3.10에서 `complex`의 `__float__` 메소드가 삭제되었다.

### 사용자 정의 클래스에서의 정적 프로토콜

- 생략

### 정적 프로토콜 디자인하기

- 생략

### 프로토콜 디자인의 Best Practice

- '좁은' 프로토콜이 유용하다.
  - 메소드가 한두 개인 프로토콜을 말함.
- 프로토콜을 사용하는 함수와 프로토콜이 붙어있는 경우가 있다.
  - 프로토콜은 라이브러리가 아닌 `client code`에서 정의되는 경우를 말함.
  - 해당 프로토콜을 사용하는 함수를 만들기에 좋다.
- 프로토콜 이름 컨벤션
  - 명확한 개념을 표현하는 경우에, 분명한 이름을 써라
    - e.g. `Iterator`, `Container`
  - callable method를 지원하는 경우 `SupportsX`를 사용하라
    - e.g. `SupportsRead`, `SupportsReadSeek`
  - 읽을 수 있거나 쓸 수 있는 속성 혹은 getter나 setter를 가진 경우에 `HasX`를 사용하라.
    - e.g. `HasItems, HasFileNo`

### 숫자 ABC와 숫자 프로토콜들

- numeric tower는 정적 타입 체크를 위해 설계되지 않았다.
  - `numbers.Number`는 메소드가 없다.
- NumPy는 타입 힌트가 없다.
