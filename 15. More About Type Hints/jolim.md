# More About Type Hints

- 오버로딩된 함수 시그니처
- 레코드용 dict를 위한 타입 힌트 `typing.TypedDict`
- 타입 캐스팅
- type hint에 런타임에 접근하기
- 제너릭 타입
  - 제너릭 타입 선언
  - Variane: invariant, covariant, contravariant
  - 제너릭 정적 프로토콜

## Overloaded Signatures

- 파이썬은 다양한 개수와 타입의 인자를 받을 수 있다.

```python
>>> help(sum)
sum(iterable, /, start=0)
...
```

- typeshed에는 sum 함수의 타입 힌트가 있다.

```python
@overload
def sum(__iterable: Iterable[_T]) -> Union[_T, int]: ...
@overload
def sum(__iterable: Iterable[_T], start: _S) -> Union[_T, _S]: ...
```

- `__iterable`앞에 `__`가 붙은 것은 이것이 positional only argument라는 것을 뜻한다. Mypy가 이것을 강제한다.
- 타입 체커는 모든 오버로드 중에서 맞는 오버로드가 있는지 검사한다.
- `@overload`를 사용하려면, 원본 함수 위에 선언하면 된다.
- pythonic한 코드에 타입 힌트를 적용하려면 힘들 때가 종종 있다.

## Max Overload

- `max`에 타입 힌트를 쓰기 위해서는 3개의 타입 변수와 6개의 오버로드가 필요하다.
- 경우의 수를 모두 따져야 한다.
- 타입 힌트를 표현하기는 어렵지만 `max` 함수를 이해하는 것은 어렵지 않다.
  ...
- 생략

## TypedDict

- JSON 데이터를 다루기 위해서 `TypedDict`를 사용하는 것은 좋지 않다.
  - 적절히 JSON을 다루는 일은 런타임에서 일어나는 일이고, 타입 힌트를 사용해서는 할 수 없다.
  - 런타입 체크는 [pydantic](https://fpy.li/15-5)을 사용하면 좋다.
- 레코드로써의 dict는 포맷이 정해져있다.
- 다음과 같은 레코드가 있다고 하자.

```python
{
    "isbn": "0134757599",
    "title": "Refactoring, 2e",
    "authors": ["Martin Fowler", "Kent Beck"]
    "pagecount": 478
}
```

- 위의 dict에 타입 힌트를 적용하려면 다음과 같을 수 있다.
  - `Dict[str, Any]`
  - `Dict[str, Union[str, int, List[str]]]`
    - 필드 이름과 값의 관계를 알 수 없다.
- `TypedDict`를 이용해서 다음과 같은 타입을 만들어낼 수 있다.

```python
from typing import TypedDict

class BookDict(TypedDict):
    isbn: str
    title: str
    author: list[str]
    pagecount: int
```

- 이것은 클래스 빌더가 아니다. 타입이다.
  - 런타임에 아무 것도 하지 않는다.
  - '필드'들은 인스턴스 어트리뷰트를 만들지 않는다.
  - '필드'에 기본 값을 넣을 수 없다.
  - 메소드 정의가 불가능하다.
    ...
- 생략
- `json.loads()`의 리턴값을 제대로 처리할 수 있는 타입 힌트는 없다.
  ...
- 생략

## 타입 캐스팅

- 타입 힌트는 완벽할 수 없다.
- `typing.cast()` 함수는 런타임에 아무것도 하지 않는다.
- 가끔은 올바른 타입으로 캐스팅하기 위하여 오랫동안 소스 코드를 봐야할 수도 있다.
  - 이럴 경우에는 타입 힌트를 포기하는 것도 좋은 방법이다.
  - `# type: ignore`를 사용하자.

## 런타입에 타입 힌트 읽기

- 너무 복잡해서 생략

## 제너릭 클래스 구현하기

- `typing.Generic`을 상속받아 구현한다.

```python
T = TypeVar('T')

class LottoBlower(Tombola, Generic[T]):
    ...
```

- 다음과 같이 쓴다.

```python
machine = LottoBlower[int](range(1, 11))
```

## 제너릭 타입에 대한 용어

- Generic type
  - 타입 변수와 함께 선언된 타입을 말한다.
  - `LottoBlower[T]`, `abc.Mapping[KT, VT]` 등
- Formal type parameter
  - 제너릭 타입 선언에 있는 타입 변수들
  - `LottoBlower[T]`에서 `T`
- Parameterized type
  - 실제 타입으로 선언된 타입을 말한다.
  - `LottoBlower[int]`, `abc.Mapping[str, float]` 등
- Actual type parameter
  - Parameterized type에 인자로 들어간 타입
  - `LottoBlower[int]`에서 `int`

## Variance

- input type들은 보통 contravariant이고 output type들은 보통 covariant이다.
  ...
- 생략

## 제너릭 정적 프로토콜 구현하기

...
