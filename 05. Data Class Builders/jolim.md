# Data Class Builders

- `collections.namedtuple` since Python 2.6
- `typing.NamedTuple` since Python 3.5, class syntac since 3.6
- `@dataclasses.dataclass` since Python 3.7

## Overview of Data Calss Builders

```python
class Coordinate:
    def __init(self, lat, lon):
      self.lat = lat
      self.lon = lon
```
- `lat`, `lon`은 3번 씩 써야한다.
- 위의 class는 적절한 repr과 동등 연산을 지원하지 않는다.
  - 원한다면 `__repr__`, `__eq__`를 굳이 정의해아한다.
```python
>>> somewhere = Coordinate(11.1, 22,2)
>>> location = Coordinate(11.1, 22.2)
>>> somewhere
<...>
>>> (location.lat, location.lon) == (somewhere.lat, somewhere.lon)
True
>>> location == somewhere
False
```

- `collections.namedtuple`의 해결법
```python
>>> from collections import namedtuple
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True
>>> somewhere = Coordinate(11.1, 22.2)
>>> somewhere
Coordinate(lat=11.1, lon=22.2)
>>> somewhere == Coordinate(11.1, 22.2)
True
```

- `typing.NamedTuple`의 해결법
```python
>>> from typing import NamedTuple
>>> Coordinate = NamedTuple('Coordinate',
      [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```
- 다른 생성 방법
  - 이 방법은 ``**kwargs`로 필드와 타입을 생성할 수 있게 한다. 
```python
Coordinate = NamedTuple('Coordinate', lat=float, lon=float)
```
- 다른 생성 방법
  - 변수의 순서는 `__init__`에서 보존된다.
```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float
```
- 주의할 점. 위의 생성방법으로 만들었을 때 다음과 같다. (metaclass)
```python
>>> issubclass(Coordinate, NamedTuple)
False
>>> issubclass(Coordinate, tuple)
True
```

- `@dataclasses.dataclass`의 생성 방법은 다음과 같다.
```python
from dataclasses import dataclass
class Coordinate:
    lat: float
    lon: float
```
- `dataclass`는 상속이나 metaclass와는 상관이 없다.

## 주 기능
- mutability
  - `namedtuple`, `NamedTuple`은 언제나 immutable이다.
  - `dataclass`는 `Frozen=True`일 때만 immutable이다.
- 클래스 문법
  - `namedtuple`은 생성자 문법만을 지원한다.
  - `NamedTuple`은 생성자 문법과 클래스 문법을 지원한다.
  - `dataclass`는 클래스 문법만을 지원한다.
- `namedtuple`, `NamedTuple`
  - `dict`로 변환: `x._asdict()`
  - 필드 이름 구하기: `x._fields`
  - 기본 값 구하기: `x._field_defaults`
  - 값이 변경된 새 인스턴스 만들기: `x._replace(...)`
  - 새 클래스 만들기: `namedtuple(...)`, `NamedTuple(...)`
- `dataclass`
  - `dict`로 변환: `dataclasses.asdict(x)`
  - 필드 이름 구하기: `[f.name for f in dataclasses.fields(x)]`
  - 기본 값 구하기: `[f.default for f in dataclasses.fields(x)]`
  - 값이 변경된 새 인스턴스 만들기 `dataclasses.replace(x, ...)`
  - 새 클래스 만들기: `dataclasses.make_dataclass(...)`
- 필드의 타입 구하기
  - `namedtuple`은 타입 어노테이션이 없기 때문에 구할 수 없다.
  - `x.__annotations__`
    - 이는 권장되지 않는다. 후에 다룬다.
    - `inspect.get_annotations(MyClass)`같이 쓰자.
    - 또는 `typing.get_type_hints(MyClass)`
- `dataclasses.fields(x)`는 `Fields` 객체의 `tuple`을 반환한다.
- 새 클래스 만드는 기능은 동적으로 생성할 때 이용된다.

## Classic Named Tuple
- `collections.namedtuple`은 factory이다.
  - 필드 이름, 클래스 이름, `__repr__`을 제공하는 tuple임.
  - `tuple`이 사용되는 곳이면 어디든 사용될 수 있다.
  - 인스턴스의 메모리 사용량도 `tuple`과 같다. 왜냐하면 필드 이름은 클래스에 저장되기 때문.
- `tuple` 서브클래스이기 때문에 다음과 같은 연산을 지원한다.
  - 동등 연산
  - 비교 연산
- `tuple`에서 지원하지 않는 다음 기능을 지원한다.
  - `_make(iterable)` 클래스 메소드
  - `_asdict()` 인스턴스 메소드
```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates')
>>> City._fields
('name', 'country', 'population', 'coordinates')
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, Coordinate(28.61, 77.20))
>>> delhi = City._make(delhi_data) # City(*delhi_data)
>>> delhi._asdict()
{'name': 'Delhi NCR', 'country': 'IN', 'population': 21.935, 'coordinates': Coordinate(lat=28.61, lon=77.2)}
>>> import json
>>> json.dumps(delhi._asdict())
'{"name": "Delhi NCR", "country": "IN", "population": 21.935, "coordinates": [28.61, 77.2]}'
```
- `namedtuple`에 기본값을 설정할 수 있다. 이는 오른쪽 요소부터 적용된다.
  - 오른쪽 요소부터 적용되는 이유는 기본값이 있는 필드의 오른쪽 필드들은 모두 기본값을 가지고 있어야하기 때문.
  - `defualts`는 keyward-argument only이다.
```python
>>> Coordinate = namedtuple('Coordinate', 'lat lon reference', defaults=['WGS94'])
>>> Coordinate(0, 0)
Coordinate(lat=0, lon=0, reference='WGS94')
>>> Coordinate._field_defaults
{'reference': 'WGS94'}
```
- `namedtuple에 메소드를 추가할 수도 있다.
  - 가능하다는 것만 알아두자.

## Typed Named Tuple
```python
from typing import NamedTuple
class Coordinate(NamedTuple):
    lat: float
    lon: float
    reference: str = 'WGS84'
```
- `namedtuple`보다 많은 메소드를 가지고 있지는 않다.
  - 다른 점: `__annotations__` 클래스 속성을 가지고 있다.
- 역시 `tuple`의 서브클래스이다.

## 타입 힌트 팁들
- 타입 힌트 `None`은 `NoneType` 싱글톤이 아니라 `NoneType`의 alias이다.

### 변수 어노테이션의 가능성
- 다음은 일반 클래스이다.
```python
class DemoPlainClass:
    a: int
    b: float = 1.1
    c = 'spam'
```
- `a`, `b`는 `__annotations__`에 들어있을 것이다.
- `a`는 클래스 속성이 되지 않는다.
- `b`는 클래스 속성이 된다.
- `c`는 단순한 클래스 변수이다.
- 셋 모두 인스턴스 상에는 존재하지 않는다.

#### `typing.NamedTuple`의 경우,
```python
from typing import NamedTuple

class DemoNTClass(NamedTuple):
    a: int
    b: float = 1.1
    c = 'spam'
```
- `a`, `b`는 어노테이션이 되고 인스턴스 속성이 된다.
  - 클래스에서는 `_collections._tuplegetter` 객체로 존재한다.
- `c`는 일반 클래스 변수이다. (immutable)
- 불변이기 때문에 당연히 인스턴스에 새로운 인스턴스 속성을 추가할 수 없다.
```
>>> DemoNTClass.a
_tuplegetter(0, 'Alias for field number 0')
>>> DemoNTClass.b
_tuplegetter(1, 'Alias for field number 1')
>>> DemoNTClass.c
'spam'
```

### `dataclass`의 경우
```python
from dataclasses import dataclass

@dataclass
class DemoDataClass:
    a: int
    b: float = 1.1
    c = 'spam'
```
- `a`, `b`는 어노테이션에 존재하며 디스크립터가 있는 인스턴스 속성이 된다.
- `c`는 일반 클래스 변수이다.
```
>>> DemoDataClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoDataClass.__doc__
'DemoDataClass(a: int, b: float = 1.1)'
>>> DemoDataClass.a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'DemoDataClass' has no attribute 'a'
>>> DemoDataClass.b
1.1
>>> DemoDataClass.c
'spam'
```

## @dataclass 자세히 보기
- 여러 키워드 인자들이 있다.
```python
@dataclass(*, init=True, repr=True, eq=True, order=False,
              unsafe_hash=False, frozen=False)
```
- `init`: `False`이면 `__init__`을 만들지 않는다.
- `repr`: `False`이면 `__repr__`을 만들지 않는다.
- `eq`: `False`이면 `__eq__`를 만들지 않는다.
- `order`: `True`이면 데이터클래스의 정렬을 가능하게 한다.
- `unsafe_hash`: `__hash__`함수를 만든다.
  - `eq`와 `frozen`이 `True`이면 safe hash 함수가 만들어진다.
- `frozen`: !

### 필드 옵션 (가변성 제거하기)
- 다음 구현은 클래스와 모든 인스턴스가 하나의 리스트를 공유하게 되기 때문에 금지된다.
  - `list`, `dict`, `set`만이 금지되어있다. 나머지 잘못된 가변성 구현은 알아서 잘 피해야한다.
```python
>>> @dataclass
... class ClubMember:
...     name: str
...     guests: list[str] = []
... 
Traceback (most recent call last):
    ...
    raise ValueError(f'mutable default {type(f.default)} for field '
ValueError: mutable default <class 'list'> for field guests is not allowed: use default_factory
```
- 이 경우 오류 메시지에 나온대로 `dataclasses.field`의 `default_factory`를 사용해야한다.
```python
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list[str] = field(default_factory=list)
```
- `default_factory` 키워드 인자는 인자 없는 callable을 인자로 받는다.
  - default 값이 필요한 인스턴스가 만들어질 때마다 `list()`가 실행될 것이다.

### `dataclasses.field`의 키워드 인자
- `default`
  - 필드의 기본값을 설정한다.
  - 기본값은 `_MISSING_TYPE`. 이는 `None`을 기본값으로 설정할 수 있게 해준다.
- `default_factory`
  - 필드의 기본값을 설정하는데 사용되는 0 인자 함수.
  - 기본값은 `_MISSING_TYPE`
- `init`
  - `False`일 경우 필드를 `__init__`의 인자에 포함시키지 않는다..
  - 기본값은 `True`
- `repr`
  - `False`일 경우 필드를 `__repr__`에 포함시키지 않는다.
  - 기본값은 `True`
- `compare`
  - 필드를 비교 연산에 사용한다.
  - 기본값은 `True`
- `hash`
  - 필드를 해시 연산에 사용한다.
  - 기본값은 `None`
    - 이는 필드가 `compare=True`인 경우에만 해시 연산에 사용될 것을 의미한다.
- `metadata`
  - 사용자 정의 클래스와의 매핑
  - `@dataclass`에서 무시된다.
  - 기본값은 `None`

### post-init
- `__post_init__` 함수를 추가할 수 있다.
  - 예를 들어 `__init__` 함수가 실행되고 나서, validation과 다른 필드에서 계산된 필드가 필요할 때.
```python
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list[str] = field(default_factory=list)
    athlete: bool = field(default=False, repr=False)

@dataclass
class HackerClubMember(ClubMember):
    all_handles = set()
    handle: str = ''

    def __post_init__(self):
        cls = self.__class__
        if self.handle == '':
            self.handle = self.name.split()[0]
        if self.handle in cls.all_handles:
            msg = f'handle {self.handle!r} already exists.'
            raise ValueError(msg)
        cls.allhandles.add(self.handle)
```
- dataclass를 상속에 이용하는 것은 좋은 생각은 아니다.  
  
- `@dataclass`는 두 경우를 제외하고 변수 타입에 신경쓰지 않는다.
  - `ClassVar`, `InitVar`이 그 예외.
### 클래스 속성에 타입 힌트 주기
- 위 예시를 Mypy로 체크하면 all_handles에 타입 어노테이션이 없다는 에러가 뜬다.
- `all_handles: set[str] = set()`은 all_handles를 인스턴스 변수로 만들어버린다.
  - set()을 인스턴스 변수로 사용할 수 없다는 오류도 뜬다.
- 다음과 같이 작성하여야 한다.
```python
from typing import ClassVar
...
    all_handles: ClassVar[set[str]] = set()
```

### 필드가 아닌 값을 초기 인자로 만들기
```python
from dataclasses import dataclass
from typing import InitVar

@dataclass
class C:
    i: int
    j: int = None
    database: InitVar[DatabaseType] = None

    def __post_init__(self, database):
        ...
```
- `database`는 `C`의 생성자 인자에 포함된다.
- `__post_init__`의 인자로 들어온다.

## 코드 냄새
- 데이터 클래스는 코드 냄새가 난다.  
- 코드 냄새란
  - 뭔가 잘못될 수 있다는 것을 쉽게 알아차릴 수 있는 패턴
  - 하지만 항상 나쁘다는 것을 말하는 것은 아니다.
- 클래스가 무언가를 '하지 않고', 데이터만을 보관하는 것은 OOP에 위배된다.
  - 인스턴스가 중복될 수도 있고, 산개할 수도 있다.  
- 좋은 사용법은 두 가지가 있다.
1. Scaffolding에 사용하는 방법
  - 초기 구현으로 사용하여 발전시켜 나갎 때.
2. 중간 매개체로 이용할 때
  - JSON과 같은 포맷으로 변환할 때 편하다.
  - 이때에, 데이터 클래스의 인스턴스는 불변이어야 한다.

## 클래스 인스턴스 패턴 매칭하기

### 단순 클래스 패턴 매칭 
- remind:
  - `case [str(name), _, _, (float(lat), float(lon))]:`
- new:
  - `case float():`
  - `case float(x):`
  - 주의: `case float`는 모든 경우에 매칭되며 float라는 이름의 변수를 받는다.
- 다음 9개 기본 타입이 단순 클래스 패턴 매칭에 이용될 수 있다.
  - `bytes`, `dict`, `float`, `frozenset`, `int`, `list`, `set`, `str`, `tuple`

### 키워드 클래스 패턴 매칭
- 퍼블릭 인스턴스 속성이 있는 클래스의 인스턴스에 사용될 수 있다.
- `case City(continent='Asia')`
  - `continent` 속성이 `'Asia'`인 모든 City 인스턴스와 매칭된다.
- `case City(continent='Asia', country=cc)`
  - `continent` 속성이 `'Asia'`인 모든 City 인스턴스와 매칭되고, `country` 속성을 `cc`라는 변수로 받는다.
- `case City(continent='Asia', country=country)`
  - 위의 경우와 같다. `country` 속성을 `country`라는 변수로 받는다.

### 위치 클래스 패턴 매칭
- `namedtuple`, `NamedTuple`, `dataclass` 클래스는 `__match_args__`를 가지고 있다.
- `__match_args__`를 정의할 수도 있는데, 나중에 다룬다.
```python
>>> City.__match_args__`
('continent', 'name', 'country')
```
- `__match_args__`에 나온 순서대로 패턴 매칭이 된다.
- `case City('Asia')`
- `case City('Asia', _, country)`
- 모든 인스턴스 속성이 `__match_args__`에 존재하는 것은 아니다.
  - 그 경우에 키워드 인자를 이용하여 패턴 매칭을 해야한다.
