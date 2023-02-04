# 5. Data Class Builders

class를 쉽게 만드는 방법 중 하나인 data class에 대해서 배워봅시다. 이번 장에서는 3개의 class builder를 다룹니다.

data class라는 단어를 처음 보았는데, design pattern입니다.

[https://refactoring.guru/ko/smells/data-class](https://refactoring.guru/ko/smells/data-class)

- `collections.namedtuple`
    - 가장 간단한 방법이며, Python 2.6이후 가능
- `typing.NamedTuple`
    - type hint는 3.5, class syntax는 3.6
- `@dataclasses.dataclass`
    - class decorator는 더 많은 커스터마이징을 제공 3.7부터

typing.TypedDict는 class builder처럼 보이지만 아닙니다.

# What’s New in This Chapter

Classic Named Tuple은 1st edition에서도 설명했지만, 나머지는 새로운 내용입니다.

# Overview of Data Class Builders

```python
class Coordinate:

    def __init__(self, lat, lon):
        self.lat = lat
        self.lon = lon
```

```python
>>> from coordinates import Coordinate
>>> moscow = Coordinate(55.76, 37.62)
>>> moscow
<coordinates.Coordinate object at 0x10a30db10>
>>> location = Coordinate(55.76, 37.62)
>>> location == moscow
False
>>> (location.lat, location.lon) == (moscow.lat, moscow.lon)
True
```

__repr__를 object로부터 상속받아서, `<coordinates.Coordinate object at 0x10a30db10>`과 같은 결과를 얻었습니다. 

collections.namedtuple class를 사용하면 `__init__`, `__repr__`, and `__eq__` 을 알아서 생성해줍니다.

```python
>>> from collections import namedtuple
>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True
>>> moscow = Coordinate(55.756, 37.617)
>>> moscow
Coordinate(lat=55.756, lon=37.617)
```

- collections.namedtuple class

```python
>>> import typing
>>> Coordinate = typing.NamedTuple('coordinate', [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True
>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```

- typing.NamedTuple class

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

- dataclasses.dataclass

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

## Main Features

`typing.NamedTuple`, `dataclasses.dataclass`는 `__annotations__`를 가지고 있습니다. fields의 type hint를 알려주는 special method인데, 직접 부르는 것은 선호되지 않습니다. `inspect.get_annotations(MyClass)`나, `typing.get_type_hints(MyClass)`를 사용하세요. 그러면 더 많은 기능을 사용할 수 있습니다.

### **Mutable instances**

`collections.namedtuple`, `typing.NamedTuple`는 tuple subclass를 생성해 immutable입니다. `@dataclass`는 mutable class를 생성하지만 frozen=true일 때는 immutable을 생성합니다.

### **Class statement syntax**

`typing.NamedTuple`과 `dataclass`는 class statement 문법을 지원해서 method과 docstring를 추가하기 쉽습니다.

### **Construct dict**

`collections.namedtuple`과 `typing.NamedTuple`은 class instance의 field로부터 dict를 생성하기 위한 instance method `._asdict`를 제공합니다. dataclasses는 `dataclasses.asdict` function을 제공합니다.

### **Get field names and default values**

위 3개의 class builder로부터 fields 이름과 default value를 얻을 수 있습니다. named tuple classes는 `._fields`와 `._fields_defaults`를 사용하면 되고, dataclasses는 `fields`라는 function을 제공하는데 return이 Field라는 Object이고 name, default를 attributes로 가집니다.

class → fields

object → attributes

### **Get field types**

`typing.NamedTuple`과 `@dataclass`는 field의 type을 얻을 수 있는 `__annotations__`를 제공합니다. 전에 언급했듯이 `typing.get_type_hints`를 사용합시다.

### **New instance with changes**

named tuple instance x가 주어졌을 때, x._replace(**kwargs)는 attribute가 수정된 새로운 instance를 반환합니다. dataclasses.replace(x, **kwargs)도 마찬가지입니다.

### **New class at runtime**

class statement syntax가 읽기 쉽지만… hardcoding입니다. framework가 runtime에 class를 만들어야 한다면 collections.namedtuple나 typing.의 default function call syntax 혹은 

# Classic Named Tuples

`collections.namedtuple`은 tuple subclass를 생성할 수 있습니다. field names, class name, __repr__를 설정할 수 있습니다. 만들어진 class는 tuple이 필요한 곳 어디서든 사용됩니다. Python standard library의 많은 function은 tuple을 return하는데, 사실은 named tuple을 return합니다.

```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinated')
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinated=(35.689722, 139.691667))
>>> tokyo.population
36.933
>>> tokyo.coordinated
(35.689722, 139.691667)
>>> tokyo[1]
'JP'
```

`collections.namedtuple(class_name, fields_name)`  → return Constructor.

fields_name은 space로 구분합니다.

Constructor의 position argument에 fields value를 전달하면 됩니다.

tuple의 subclass라 __eq__, __lt__ 를 상속받아서 사용할 수 있습니다.

게다가 named tupel은 tuple보다 조금 더 많은 기능이 있습니다. `_fields` class attribute, `_make(iterable)` class method, `_asdict()` instance method

_asdict()는 Python 3.7까지는 OrderedDict를 return하고 그 이후에는 Dict를 return합니다. 여전히 OrderedDict를 원하면 _asdict()문서를 참고하세요.

keyword-only argument인 defaults는 instance의 초기값을 지정할 수 있습니다.

```python
>>> Coordinate = namedtuple('Coordinate', 'lat lon reference', defaults=[40, 'WGS84'])
>>> Coordinate(0)
Coordinate(lat=0, lon=40, reference='WGS84')
```

typing.NamedTuple, @dataclass는 method를 정의할 수 있지만, collections.namedtuple에 method를 넣는 것은 가능하기는 한데… 꼼수입니다.

# Typed Named Tuples

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float
    reference: str = 'WGS84'
```

모든 instance field는 typing되어 있습니다. reference는 default가 정해져 있습니다.

# Type Hints 101

Type hint = type annotations

Type hint는 bytecode compiler나 interpreter가 강요하지 않습니다.

## No Runtime Effect

Type Hint는 IDE나 Type Checker를 위한 문서입니다. Python Program에는 영향을 미치지 않습니다.

Mypy, PyCharm IDE

## Variable Annotation Syntax

`typing.NamedTuple`, `@dataclass`는 PEP 526을 따릅니다.

var_name: some_type = a_value

## The Meaning of Variable Annotations

```python
class DemoPlainClass:
    a: int           # <1>
    b: float = 1.1   # <2>
    c = 'spam'       # <3>
```

a는 __annotations__에 등록되지만 class attribute는 아닙니다.

b는 등록도 되고, class attribute입니다.

c는 등록되지 않지만, class attribute입니다.

```python
>>> from demo_plain import DemoPlainClass  
>>> DemoPlainClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoPlainClass.a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: type object 'DemoPlainClass' has no attribute 'a'
>>> DemoPlainClass.b
1.1
>>> DemoPlainClass.c
'spam'
```

__annotations__는 Python interpreter에 의해 생성되며, source code의 type hint를 기록합니다.

### Inspecting a typing.NamedTuple

```python
import typing
class DemoNTClass(typing.NamedTuple):
    a: int
    b: float = 1.1
    c = 'spam'
```

```python
>>> from demo_nt import DemoNTClass
>>> DemoNTClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}
>>> DemoNTClass.a
_tuplegetter(0, 'Alias for field number 0')
>>> DemoNTClass.b
_tuplegetter(1, 'Alias for field number 1')
>>> DemoNTClass.c
'spam'
```

typing.NamedTuple은 type hint가 있는 class attribute를 다르게 처리합니다. `descriptor`라고 불리는데, Chapter 23에서 설명합니다.

```python
>>> DemoNTClass.__doc__
'DemoNTClass(a, b)'
>>> nt = DemoNTClass()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: DemoNTClass.__new__() missing 1 required positional argument: 'a'
>>> nt = DemoNTClass(8)
>>> nt.a
8
>>> nt.b
1.1
>>> nt.c
'spam'
>>> nt
DemoNTClass(a=8, b=1.1)
```

nt를 생성하기 위해서 DemoNTClass Constructor에 a argument를 전달해주어야 합니다.

### Inspecting a class decorated with dataclass

```python
from dataclasses import dataclass
@dataclass
class DemoDataClass:
    a: int
    b: float = 1.1
    c = 'spam'
```

```python
>>> from demo_dc import DemoDataClass
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

typing.NamedTuple과는 다르게 a attribute가 없습니다. dataclass에서 b, c는 class attribute이지만, a는 public attribute라 instance 생성 후 get, set할 수 있습니다. b는 instance attribute를 위해 default value를 가지고 있지만, c는 class attribute입니다.

dataclass instance는 mutable입니다.

Python instance는 자기 자신만의 instance attribute를 생성할 수 있습니다.

```python
>>> dc.c = 'whatever'
>>> dc.z = 'secret stash'
```

# More About @dataclass

```python
@dataclass(*, init=**True**, repr=**True**, eq=**True**, order=**False**, unsafe_hash=**False**, frozen=**False**)
```

* 이후의 parameter는 keyword-only parameter입니다.

- init: __init__을 정의하면 False가 됩니다.
- repr: __repr__을 정의하면 False가 됩니다.
- eq: __eq__를 정의하면 False가 됩니다.
- order: instance를 sorting하고 싶다면 사용합니다.
- unsafe_hash: 어렵습니다. 문서를 참고합시다.
- frozen: immutable → mutable
    - 진정한 의미의  immutable은 아니고, __setattr__, __delattr__를 생성해 raise Error합니다.
    - frozen=True라면 적절한 __hash__ method를 생성합니다. False라면 __hash__=None입니다.

## Field Options

Python은 default parameter이후에 without default parameter를 허용하지 않습니다.

Mutable default values는 초보 파이썬 개발자에게 버그를 유발합니다.

dataclass는 class attribute에 default value로 mutable type을 선언하는 것을 방지합니다.

```python
@dataclass
class ClubMember:
    name: str
    guests: list = []
```

```python
ValueError: mutable default <class 'list'> for field guests is not allowed: use default_factory
```

```python
from dataclasses import dataclass, field
@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)
```

default_factory는 class의 instance가 생성될 때마다 동작해서 모든 instance가 같은 object를 공유하는 일이 없도록 방지합니다. 하지만 list, dict, set만 방지하고 그 이외의 mutable은 방지하지 않습니다.

Python3.9에서는 list 안의 contents type을 명시하기 위해 [ ]를 허용했습니다.

guest: list, guest: list[str]의 차이

### field function keyword argument

- default
- default factory
- init
- repr
- compare
- hash
- metadata

## Post-init Processing

__post_init__

## Typed Class Attributes

## Initialization Variables That Are Not Fields

## @dataclass Example: Dublin Core Resource Record

# Data Class as a Code Smell

## Data Class as Scaffolding

## Data Class as Intermediate Representation

# Pattern Matching Class Instances

## Simple Class Patterns

## Keyword Class Patterns

## Positional Class Patterns

# Chapter Summary

# Further Reading