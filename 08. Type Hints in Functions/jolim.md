# Type Hints in Functions
- 파이썬은 동적 타입 언어이고 계속 그럴 것이다.
  - 타입 힌트가 컨벤션으로라도 강제될 일은 없다.
- IDE와 CI에 가장 큰 도움이 된다.
- 모든 사용자가 파이썬으로 깊은 프로그래밍을 하는 것은 아니다. 타입 힌트는 선택 사항이다.
- 이번 챕터에서 다룰 내용은
  1. Mypy와 함께하는 점진적 typing
  2. duck typing과 nomial typing의 보조적 시각
  3. 타입 어노테이션에 나올 수 있는 타입의 카테고리
  4. 가변인자 함수의 타입 힌트
  5. 타입 힌트의 한계와 단점. static typing

## 점진적 타이핑
- 점진적 타입 시스템은
  - 선택 사항이다.
    - 점진적 타입 시스템의 가장 좋은 점
    - 타입 힌트가 없으면 타입 체커는 오류를 띄우지 않는다.
    - 타입 힌트가 없으면 암시적으로 `Any` 타입을 가정한다.
  - 런타임에 오류를 내지 않는다.
    - 정적 타입 검사기와 린터, IDE는 경고를 띄울 것이다.
    - 런타임에는 아무런 영향을 미치지 않는다.
  - 성능을 향상시키지 않는다.
    - 이론적으로는 타입 힌트가 성능을 향상시키는 인터프리터를 만들 수 있다
    - 그러나 그런 인터프리터는 만들어지지 않았다.
- 패키지 전체에 타입 힌트가 없을 수도 있다.
  - 그런 모듈에 대해서는 타입 체크 기능을 사용하지 않을 수 있다.

## 점진적 타이핑 사용해보기
- 타입 힌트가 없는 곳에서 Mypy가 경고를 띄우게 할 수 있다. `--disallow-untyped-defs`
  - Mypy config 파일을 사용해도 가능하다.
- 리턴 타입을 명시하면 Mypy가 그 함수를 인지하고 타입을 추가할 것을 원한다.

### 코드 스타일
- flake8과 blue를 사용할 것을 권한다.
  - black formatter는 기본적으로 쌍따옴표를 사용한다.
  - blue는 기본적으로 홑따옴표를 사용한다.

## 기본 인자로 `None`을 사용하기.
- 예를 들어, `str`을 받을 수 있는 자리에 `None`을 기본 인자로 받고 싶다.
  - 기본 인자로 mutable을 사용하는 것은 좋지 않은 생각이고 대신에 None을 써야 한다.
  - `Optional[str]`은 그 값이 `None` 또는 `str`이라는 것을 뜻한다.

## 타입은 지원하는 연산에 의해 정의된다.
- 다음에 타입 힌트를 준다면?
```python
def double(x):
    return x * 2
```
- `x`는 숫자 타입이거나 시퀀스이거나, N-차원 어레이일 수 있고, `__mul__`이 구현된 어떤 객체일 수 있다.
- 다음은 틀린 타입 힌트이다.
```python
def double(x: abc.Sequence):
    return x * 2
```
- `abc.Sequence`는 `__mul__`이 없다.

## 덕 타이핑과 노미널 타이핑??
- 덕 타이핑
  - Smalltalk, Python, JavaScript, Ruby 등.
  - 객체는 타입이 있고, 변수는 타입이 없다. ??
  - 객체의 선언된 타입이 무엇이든, 지원하는 연산에 의해서 타입이 결정된다.
  - 오직 런타임에만 덕 타이핑이 작동할 수 있다.
  - 노미널 타이핑에 비해 유연하지만 에러가 더 날 수 있다.
- 노미널 타이핑
  - C++, Java, C# 등.
  - 객체와 변수는 타입을 가진다.
  - 객체는 런타임에만 존재한다.
  - 타입 검사기는 타입 힌트로 표시된 변수에만 작동한다.
  - 수퍼클래스가 들어갈 자리에 서브클래스가 대신 들어갈 수 있다.
  - 런타임이 아니라 코드만 가지고 타입 검사를 할 수 있다.
  - 덕 타이핑보다 더 엄격하고, 빌드 파이프라인에서 일찍 버그를 잡을 수 있다.

## 어노테이션에 사용 가능한 타입들
- `typing.Any`
- 기본 타입과 클래스
- `typing.Optional`, `typing.Union`
- 제너릭 콜렉션. 튜플과 매핑 포함.
- ABCs
- 제너릭 이터러블
- 매개변수화된 제너릭과 `TypeVar`
- `typing.Protocol` - 정적 덕 타이핑의 핵심
- `typing.Callble`
- `typing.NoReturn`

### `Any` 타입
- 동적 타입이라고도 불린다.
- `object` 타입과는 다르다.
  - 모든 타입은 `object` 타입이지만 `object` 타입은 어떤 타입이 아니다.
  - `object` 타입은 연산이 거의 정의되어있지 않다.
- 좀 더 일반적인 타입은 더 좁은 인터페이스를 가지고 있다.
  - `object` 타입은 `abc.Sequence`보다 적은 연산이 구현되어있고, 또, `abc.Sequence`는 `abc.MutableSequence`보다 적은 연산이 구현되어있다.
- `Any` 타입은 가장 일반적인 타입부터 가장 구체적인 타입까지 지원한다.
- `Any` 타입의 존재는 타입 검사기의 목적을 방해한다.

### subtype-of VS consistent-with
- `T2`가 `T1`을 상속받는다면 `T2`는 subtype-of `T1`이다.
  - `T1`을 호출할 수 있는 곳에서는 `T2`를 호출할 수 있다.
  - 이는 LSP(Liskov Subsitute Prinsiple)의 예이다.
    - 하위 클래스는 상위 클래스를 대체할 수 있다.
- 점진적 타입 시스템에는 다른 타입 관계가 있다: contsistent-with
  1. `T1`의 하위 타입 `T2`은 `T1`과 consistent-with이다. (LSP)
  2. 모든 타입은 `Any`와 consistent-with이다: type이 `Any`인 곳에 모든 타입을 넣을 수 있다.
  3. `Any`타입은 모든 타입과 consistent-with이다: 모든 타입에 `Any`를 넣을 수 있다.

## 기본적인 타입과 클래스
- 기본적인 타입에는 `int`, `float`, `str`, `bytes` 등이 있다.
- 사용자 지정 class들도 타입으로 사용할 수 있다.
- ABC도 타입 힌트로 유용하다. -> 나중에 다룰 것
- 클래스 간에는 __subtypeof__가 __consistent-of__와 비슷하게 작동한다.
- `int`와 `float`와 `complex`에 관해서...
  - 셋은 모두 `object`의 직접적 서브클래스이다.
  - `int`는 `float`와 consistent-of이고, `float`는 `complex`와 consistent-of이다.

## `Optional`과 `Union` 타입
- `Optional[str]`은 `Union[str, None]`과 같다.
  - Python 3.10부터는 `str | None`과 같다.
- `ord`는 다음과 같은 타입 시그니처를 가진다.
  `def ord(c: Union[str, bytes]) -> int: ...`
- 다음과 같이 `str` 혹은 `float`를 리턴하는 함수를 생각해볼 수 있다.
```python
from typing import Union

def parse_token(token: str) -> Union[str, float]:
    try:
        return str(token)
    except ValueError:
        return token
```
- 가능한 특정한 타입을 리턴하고 `Union` 타입을 리턴하는 것은 피해야 한다.
- 중첩된 `Union`은 단일 층 `Union`과 같다.
  - `Union[a, b, Union[c, d, e]]` == `Union[a, b, c, d, e]`
  - `Union[int, float]`는 중복이다. 왜냐하면 `int`는 `float`에 대해 consistent-with이기 때문에.

## 제너릭 콜렉션
- 대부분의 Collection은 비균질하다. 즉, 여러 타입을 담고 있다.
  - 실례에서는 이런 상황은 바람직하지 않다. 여러 타입을 한꺼번에 처리할 수는 없다.
- 매개변수화된 collection에서는 한 가지 타입으로 한정된 collection을 다룬다.
```python
def tokenize(text: str) -> list[str]:
    return text.upper().split()
```
- Python 3.9 이상에서는 `tokenize`가 모든 요소가 `str`인 `list`를 반환한다는 것을 나타낸다.
- `stuff: list`는 `stuff: list[Any]`와 같다.
- `collections`나 `abc`를 이용하면 여러 가지 매개변수화 된 타입을 이용할 수 있다.
  - `list`
  - `set`
  - `frozenset`
  - `collections.deque`
  - `abc.container`
  - `abc.Collection`
  - `abc.Sequence`
  - `abc.Set`
  - `abc.MutableSequence`
  - `abc.MutableSet`
- Python 3.10까지는 `array.array`의 적절한 타입 어노테이션이 없다.

### 레거시 서포트
- Python 3.7, 3.8에서는 `from __future__ import annotations`가 필요하다.
- Python >= 3.5에서는 `from typing import List` 등이 필요하다.

## 튜플 타입
- 튜플 타입은 다음과 같은 세 경우가 있다.
1. 레코드로써의 튜플
2. 레코드인 named tuple
3. immutalbe sequence

### 레코드로써의 튜플
- `tuple[str, float, str]`과 같이 사용하면 된다.

### named tuple의 튜플
- class 이름을 타입으로 사용할 수 있다.
```python
class Coordinate(NamedTuple):
    lat: float
    lon: float
```
- 이는 `tuple[float, float]`와 호환된다.
- 즉, `Coordinate`는 `tuple[float, float]`에 대해서 consistent-with이다.
  - 반대는 참이 아니다. e.g. `_asdict`와 같은 메소드가 없다.

### immutable sequence로써의 튜플
- immutable sequence로 사용하려면 단일 타입으로밖에 어노테이션이 안된다.
  - `tuple[str, ...]`과 같이...
- `tuple[Any, ...]`과 `tuple`은 같다.

### 제너릭 매핑
- 제너릭 매핑 타입은 `MappingType[KeyType, ValueType]`으로 타이핑할 수 있다.
- e.g. `dict[str, set[str]]`

### ABCs
- 보내는 것은 보수적으로, 받는 것은 자유롭게.
  - Postel's Law
- `collections.abc.Mapping`은 `dict`, `defaultdict`, `ChainMap`, `UserDict`와 `Mapping`의 서브클래스 등과 consistent-with이다.
- `dict`보다는 `Mapping`이나 `MutableMapping`을 타입 힌트로 사용하는 것이 좋다.

### Iterable
- 가능하다면 `list`보다는 `Iterable`이나 `Sequence`를 매개변수 타입으로 사용하는 것이 좋다.
- `Iterable`이 `Sequence`보다 더 범위가 넓다.
  - 제너레이터의 경우 반복이 끝나지 않으면서 `Iterable` 타입을 받는 경우 메모리를 꽉 채울 위험이 있다.
  - 그럼에도 불구하고 `Iterable`을 사용하는 것이 좋다.

### 매개변수화된 제너릭과 `TypeVar`
- `list[T]`와 같은 것.
- 사용하기 전에 `T = TypeVar('T')`와 같이 초기화해주어야 한다.
  - 이는 파이썬 인터프리터에 깊은 변화를 피해기위한 것.
```python
...
from typing import TypeVar
T = TypeVar('T')

def sample(population: Sequence[T], size: int): list[T]: ...
```
- `sample`에서 `population`의 `Sequence` 내의 타입과 리턴인 `list` 내부의 타입이 같음을 나타낸다.
- 참고로 `str`은 `Sequence[str]`과 consistent-with이다.

### 제한된 `TypeVar`
- `NumberT = TypeVar('NumberT', float, Decimal, Fraction)`
  - 이와 같이 사용하면 `NumberT`자리에 `float`, `Decimal`, `Fraction`이 올 때만 유효하다.

### 경계지어진 `TypeVar`
- `HashableT = TypeVar('HashableT', bound=Hashable)`
  - 이와 같이 사용하면 Hashable과 그 하위 타입만 `HashableT` 타입으로 올 수 있다.
- `TypeVar`은 `covariant`와 `covariant`인 keyward 인자가 있지만, 나중에 다룰 것.

#### 정의되어있는 `AnyStr`
- `typing.AnyStr`
  - `AnyStr = TypeVar('AnyStr', str, bytes)`와 같이 정의되어있다.

### 정적 프로토콜
- 프로토콜이란 Go의 인터페이스와 같다.
  - 프로토콜 타입은 메소드를 특정함에 따라 정의된다.
- 프로토콜은 `typing.Protocol`의 서브클래스에 의해 정의된다.
  - 단, 프로토콜에 해당하는 클래스는 그것을 상속할 필요가 없다.
```python
def top(series: Iterable[T], length: int) -> list[T]:
    ordered = sorted(series, reversed=True)
    return ordered[:length]
```
- `sorted`는 `<` 연산이 정의되어야 하기 때문에 위의 정의로는 부족하다.
  - 즉 `__lt__` 연산이 정의되어 있어야 한다.
- `__hash__`가 정의되어있어야 할 때는 `typing.Hashable`을 사용할 수 있지만 이 경우는 적절한 타입이 없다.
- 다음과 같은 프로토콜을 정의하자.
```python
from typing import Protocol, Any

class SupportsLessThan(Protocol):
    def __lt__(self, other: Any) -> bool: ...
```
- 프로토콜에서 정의되어야할 연산의 본문은 `...`으로 갈음한다.
- 이제, 다음과 같이 정의하자.
```python
LT = TypeVar('LT', bound=SupportsLessThan)
def top(series: Iterable[LT], length: int) -> list[LT]:
    ordered = sorted(series, reversed=True)
    return ordered[:length]
```
- 이는 정적 덕 타이핑으로 불린다.
  - 명목적인 타입과는 다르게, 연산에 의해 정의되기 때문에.
  - 또한, 런타임이 아니라 코드에 의해 정의되기 때문에.

#### Mypy의 `TYPE_CHECKING`과 `reveal_type`
- `typing.TYPE_CHECKING`은 런타임에는 항상 `False`이고, 타입 검사 시에는 `True`이다.
- `reveal_type()`은 런타임에는 호출될 수 없다. import 되지 않기 때문. 타입 검사 시에만 실행된다.


### `Callable`
- `Callble[[ParamType1, ParamType2], ReturnType]`과 같이 쓴다.
- 인자가 정해지지 않았을 경우 `Callable[..., ReturnType]`와 같이 사용할 수도 있다.

#### `Callable` 타입의 variance
- `int`는 `float`의 하위 타입이다.
- `Callable[[], int]`는 `Callable[[], float]`의 하위 타입(subtype-of)이다.
  - 이를 covariant라고 부른다.
- `Callable[[int], None]`은 `Callable[[float], None]`의 하위 타입이 아니다.
  - 반대이다. 이를 contravariant라고 부른다.

### `NoReturn`
- 예외를 발생시키는 등의 이유로 리턴 값이 아예 없는 함수가 있다.
  - 예를 들어서 `sys.exit()` 함수
- 참고로 `object`와 `Optional[object]`는 같다.
  - `None`이 `object`의 하위 타입이기 때문에.

## Positional-Only, Variadic Parameters
- recall: `def tag(name, /, *content, class_=None, **attrs) -> str:`
- 타이핑 후에...
```python
def tag(
  name: str,
  /,
  *content: str,
  class_: Optional[str] = None,
  **attrs = str
) -> str:
```
- `content`의 타입은 `tuple[str]`이 될 것이다.
- `attrs`의 타입은 `dict[str, str]`이 될 것이다.
  - `attrs`가 여러 타입을 받아야 한다면 `Union[]`을 쓰거나 `Any`를 써야 한다.
- positional-only 인자는 >= 3.8에서만 해당한다.
  - 그 이전에는 인자 이름에 `__`를 붙이면 타입 검사기에서 확인한다. 그것이 컨벤션임.
  ```python
      def tag(__name: str, *content: str, class_: Optional[str] = None, **attrs: str) -> str:
  ```
  - Mypy가 알아본다.

## 완벽하지 않은 타이핑과 엄밀한 테스트
- 엄밀한 테스트는 타임 힌트가 있기 전에도 가능했다.
- 타입 힌트에는 가양성과 가음성 문제가 있다.
- 어떤 기능들은 타입 체크를 하기 힘들다.
  - `config(**settings)`와 같은 언패킹에는 타입을 적용하기 힘들다.
  - 프로퍼티, 디스크립터, 메타클래스, 메타프로그래밍에는 타입을 적용하기 힘들거나 불가능하다.
  - 타입 검사기의 릴리즈는 파이썬 릴리즈보다 느리다.
- 어떤 조건은 타입 힌트로 표현할 수 없다.
  - 값이 0보다 큰 정수
  - 길이가 10인 문자열
  - ASCII가 6과 12 사이인 바이트
- 비즈니스 로직에서 오류를 잡는데에는 적합하지 않다.
