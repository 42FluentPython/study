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