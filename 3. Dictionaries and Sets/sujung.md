# Chapter03 - Dictionaries and Sets

파이썬의 `dict` type은 파이썬의 구현에 있어 필수적인 부분이기에 직접/간접적으로 항상 사용하고 있음.\
파이썬 코어 구성요소들은 메모리에 `dictionary`로 나타냄.\
`__builtions__.__dict__`는 모든 빌트인 타입, 오브젝트, 함수를 저장함.

파이썬 `dicts`는 매우 최적화 되어있고, 향상 중이며 파이썬의 고성능 `dict`은 `hash table`에 기반함.

`hash table` 기반의 다른 빌트인 타입으론 `set`, `frozenset`이 존재.\
각 자료구조들은 다른 유명 언어에서 만났던 `set`보다 풍부한 API와 연산자를 제공

## Modern dict syntax
### dict Comprehensions
```python
>>> dial_codes = [
... (880, 'Bangladesh'),
... (55, 'Brazil'),
... (86, 'China'),
... (91, 'India'),
... (62, 'Indonesia'),
... (81, 'Japan'),
... (234, 'Nigeria'),
... (92, 'Pakistan'), ... (7, 'Russia'),
... (1, 'United States'),
... ]
>>> country_dial = {country: code for code, country in dial_codes}
>>> country_dial
{'Bangladesh': 880, 'Brazil': 55, 'China': 86, 'India': 91, 'Indonesia': 62, 'Japan': 81, 'Nigeria': 234, 'Pakistan': 92, 'Russia': 7, 'United States': 1} >>> {code: country.upper()
... for country, code in sorted(country_dial.items())
... if code < 70}
{55: 'BRAZIL', 62: 'INDONESIA', 7: 'RUSSIA', 1: 'UNITED STATES'}
```
`dictcomp`는 이터러블에서 `key, pair`쌍을 얻어 `dict` 인스턴스를 생성.

### Unpacking Mappings
**example.1 - one argument in function call**
```python
>>> def dump(**kwargs):
... return kwargs
...
>>> dump(**{'x': 1}, y=2, **{'z': 3})
{'x': 1, 'y': 2, 'z': 3}
```
위 형식으로 작성하는 경우 중복키가 있으면 안됨.\
위 형식에서 중복키는 금지됨.

**example.2 - dict literal**
```python
>>> {'a': 0, **{'x': 1}, 'y': 2, **{'z': 3, 'x': 4}}
{'a': 0, 'x': 4, 'y': 2, 'z': 3}
```
위 형식으로 작성하는 경우 중복키 허용.\
중복키가 존재하면 가장 마지막에 나오는것을 사용.

### Merging Mappings with |
`python 3.9`버전 부터 `|`, `|=` 연산자를 merge mappings를 위해 사용.

**example.1 - operator |**
```python3
>>> d1={'a':1, 'b':3}
>>> d2={'a':2, 'b':4, 'c':6}
>>> d1 | d2
{'a': 2, 'b': 4, 'c': 6}
>>> d2 | d1
{'a': 1, 'b': 3, 'c': 6}
```
보통 뉴 멥핑은 왼쪽 피연산자의 타입과 같음.\
그러나 두 번째 피 연산자의 타입의 유저 정의라면 오퍼랜드 오버로딩 규칙에 의해 유저 정의로 설정.

**example.2 - operator |=**
```python3
>>> d1
{'a': 1, 'b': 3}
>>> d2
{'a': 2, 'b': 4, 'c': 6}
>>> d1 |= d2
>>> d1
{'a': 2, 'b': 4, 'c': 6}
>>> d1 = {'a':1, 'b':3}
>>> d2 |= d1
{'a': 1, 'b': 3, 'c': 6}
```

## Pattern Matching with Mappings
mapping을 위한 pattern은 `dict literal`처럼 보임, 하지만 Pattern은 실제 인스턴스 또는 `collections.abc.Mapping`의 가상 서브클래스와 매칭 할 수 있음.

## Standard API of Mapping Types
`collections.abc`는 `Mapping`, `MutableMapping`를 제공.\
커스텀한 맵핑을 구현하기 위해, `collections.UserDict`를 확장하는것, `dict`를 랩핑하는것은 쉬움.

## Automatic Handling of Missing **Keys**

## Variations of dict

## Immutable Mappings
## Dictionary Views
## Practical Consequences of How dict Works
## Set Theory
## Practical Consequences of How Sets Work
## Set Operations on dict Views