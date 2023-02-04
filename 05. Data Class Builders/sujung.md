# Chapter05 - Data Class Builders

파이썬은은 간단한 class를 만들 수 있는 몇가지 방법을 제공.\
이번 챕터에서는 3가지의 class builder를 학습.
* **collections.namedtuple**: 가장 단순한 방법
* **typing.NamedTuple**: 필드 타입에 대한 설명이 필요한 경우 사용.
* **@dataclasses.dataclass**: class decorator는 더 많은 사용자 설정을 가능.

class builder를 학습한 이후 왜 `Data Class`를 `code smell`이라고 불리는지 논의함.

## Overview of Data Class Builders
**example.1 - coordinates**
```python
>>> class Coordinate:
...     def __init__(self, lat, lon):
...         self.lat = lat
...         self.lon = lon
...
>>> moscow = Coordinate(55.76, 37.62)
>>> moscow	# class __repr__로 호출된 정보가 유익하지 않음
<__main__.Coordinate object at 0x104fc91c0>
>>> location = Coordinate(55.76, 37.62)
>>> location == moscow # __eq__ object의 아이디 비교를 상속
False
>>> (location.lat, location.lon) == (moscow.lat, moscow.lon)
True
```
`__init__` `boilerplate`를 작성하는것은 구식임.\
특히 class가 두 가지 이상의 속성을 가진다면, 각각의 속성은 3번 언급되어야함.\
`boilerplate`를 우리가 파이썬 오브젝트로 기대하는 기본적 기능을 주지 않음.

**example.2 - namedtuple**
```python
>>> from collections import namedtuple
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True
>>> moscow = Coordinate(55.756, 37.617)
>>> moscow # __repr__ 위에 만든것보다 훨씬 유용한 정보 제공
Coordinate(lat=55.756, lon=37.617)
>>> moscow == Coordinate(lat=55.756, lon=37.617) # __eq__ 내부 속성 비교
True
```
동일한 로직을 `class builder`인 `nametuple`을 이용해 구현해봄.

**example.3 - typing.NamedTuple**
```python
>>> import typing
>>> Coordinate = typing.NamedTuple('Coordinate', [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```
동일한 기능을 `typing.NamedTuple`도 제공하며, 각 필드에 대한 타입 어노테이선도 추가 할 수 있음.\
python 3.6 버전 이상부터 `typing.NamedTuple`은 `class` 구문으로 사용 할 수 있음.

**example.4 - typing.NamedTuple Coordinate**
```python
from typing import NamedTuple

class Coordinate(NamedTuple):
	lat: float
	lon: float

	def __str__(self):
		ns = 'N' if self.lat >= 0 else 'S'
		we = 'E' if self.lon >= 0 else 'W'
		return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'

>>> issubclass(Coordinate, typing.NamedTuple)
False
>>> issubclass(Coordinate, tuple)
True
```

**example.5 - coordinate.py**
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
	lat: float
	lon: float

	def __str__(self):
		ns = 'N' if self.lat >= 0 else 'S'
		we = 'E' if self.lon >= 0 else 'W'
		return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

### Main Features
|                        | namedtuple   | NamedTuple   | dataclass   |
|------------------------|--------------|--------------|-------------|
| mutable instance       | NO           | NO           | YES         |
| class statement syntax | NO           | YES          | YES         |
| construct dict         | x._asdict() | x._asdict() | dataclasses.asdict(x) |
| get field names        | x._fields   | x._fields   | [f.name for f in dataclasses.fields(x)] |
| get defaults           | x._field_defaults | x._field_defaults | [f.default for f in dataclasses.fields(x)] |
| get field types | N/A | x.\_\_annotations\_\_ | x.\_\_annotations\_\_ |
| new instance with changes | x._replace(...) | x._replace(...) | dataclasses.replace(x, ...) |
| new class at runtime | namedtuple(...) | NamedTuple(...) | dataclasses.make_dataclass(...) |

## Classic Named Tuples
`collections.namedtuple`의 기능은 `tuple`의 서브 클래스를 생성하는 공장임.\
`collections.namedtuple`의 서브 클래스는 필드 이름, 클래스 이름, 유익한 정보를 주는 `__repr__`이 강화된 `tuple`의 서브 클래스임.\
파이썬의 표준 라이브러리는 `tuple`을 반환해왔지만, 지금은 편위를 위하여 `named tuple`을 리턴함.

**example.1 - Defining and using a named tuple type**
```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates') # 1
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667)) # 2
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722, 139.691667))
>>> tokyo.population # 3
36.933
>>> tokyo.coordinates
(35.689722, 139.691667)
>>> tokyo[1]
'JP'
```
1. namedtuple을 만들기 위해선 클래스명과, 필드 명이 있는 리스트 2개의 인자가 필요함.
2. 필드의 값은 반드시 입력해야하며, 생성자에 분리된 상태로 전달되야 함. (반면 튜플의 경우 단일 이터레이블 전달)
3. namedtuple의 필드명 또는 위치로 접근 가능

**example.2 -Named tuple attributes and methods**
```python
>>> City._fields # 1
('name', 'country', 'population', 'coordinates')
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, Coordinate(28.613889, 77.208889))
>>> delhi = City._make(delhi_data) # 2
>>> delhi._asdict() # 3
{'name': 'Delhi NCR', 'country': 'IN', 'population': 21.935, 'coordinates': Coordinate(lat=28.613889, lon=77.208889)}
>>> import json
>>> json.dumps(delhi._asdict()) # 4
'{"name": "Delhi NCR", "country": "IN", "population": 21.935, "coordinates": [28.613889, 77.208889]}'
```
1. `_fields`는 클래스의 필드 이름을 가지는 튜플.
2. `_make()`는 `iterable`로 부터 객체를 생성, City._make(delhi_data)는 City(*delhi_data)와 같음.
3. `_asdict()`는 `named tuple` 객체로 부터 만들어진 딕셔너리를 리턴.
4. `_asdict()`는 JSON format 직렬화에 유용함.


## Typed Named Tuples
**example.1 - typing_named/coordinates2.py**
```python
from typing import NamedTuple

class Coordinate(NamedTuple):
	lat: float
	lon: float
	reference: str = 'WGS84'
```

**example.2 - 

## Type Hints 101
`type annotations`으로 알려진 `type hints`는 함수의 인자, 반환 값, 변수, 속성의 기대되는 타입을 선언.\
`type hints`는 파이썬 바이트 코드 컴파일러와 인터프리터에 의해 전혀 강요되지 않음.

### No Runtime Effect
`type hints`는 파이썬 프로그램의 런타임에 아무런 영향이 없음.

**example.1 - Python does not enforce type hints at runtime**
```python
>>> import typing
>>> class Coordinate(typing.NamedTuple):
...     lat: float
...     lon: float
...
>>> trash = Coordinate("Ni!", None) 
>>> print(trash)
Coordinate(lat='Ni!', lon=None) # 1
```
1. 파이썬은 런타임에 타입체크를 하지 않음.

`type hints`는 서드파티의 도움을 받아 코드 실행없이 정적분석을 통해 검사함.

### Variable Annotation Syntax
`typing.NamedTuple`, `@dataclass`는 `variable annotations`라는 구문을 사용함.\
해당 구문은 클래스 구문에서 속성을 정의하는 컨텍스트 구문에 대한 소개. \
또한 값을 이용한 초기화도 가능함, 해당 값은 default로 이용됨.

### The Meaning of Variable Annotations
```python
>>> class DemoPlainClass:
...     a: int
...     b: float = 1.1
...     c: 'spam'
...
>>> DemoPlainClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>, 'c': 'spam'}
```

## More About @dataclass
`@dataclass` decorator는 몇 가지 키워드를 사용할 수 있음.

### Field Options
### Post-init Processing
### Typed Class Attributes
### Initialization Variables That Are Not Fields
### @dataclass Example: Dublin Core Resouce Record

## Data Class as a Code Smell
직접 데이터 클래스를 작성하던, 클래스 빌더를 이용하던 데이터 클래스는 설계에 문제가 있음을 나타냄.\
OOP의 주요 아이디어는 행위와 데이터를 같은 코드 유닛에 위치하는것임.\

### Data Class as Scaffolding

### Data Class as Intermediate Representation 

## Pattern Matching Class Instances
