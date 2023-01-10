# Chapter01 - The Python Data Model
## French Deck
**sample**
```python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])
```
**result**
```dir(Card)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmul__', '__setattr__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', '_asdict', '_field_defaults', '_fields', '_fields_defaults', '_make', '_replace', 'count', 'index', 'rank', 'suit']
```
## How Special Methods Are Used

python special method는 사용자에 의해 호출되는게 아는 파이썬 인터프리터에 의해 호출.\
예시로 `my_object.__len__()`로 작성하는것이 아닌 `len(my_object)`로 작성.\
즉 유저 정의 클래스의 경우 파이썬은 유저가 구현한 `__len__` 메소드를 호출함.\
파이썬 인터프리터는 빌트인 타입을 다루는 경우 쇼컷을 취함.\
파이썬 변수크기 콜렉션은 `ob_size` 필드를 가지고 있음.\
만약 `my_object`가 빌트인 타입의 인스턴스라면 `len(my_object)` 호출 시 `__len__` 메소드 호출이 아닌 `ob_size`필드를 찾음.

### Emulating Numeric Types
[3.3.8](https://docs.python.org/ko/3/reference/datamodel.html#emulating-numeric-types)

연산자 오버라이딩을 통해 숫자형이 아닌 객체에 대해 마치 숫자형 처럼 핸들링 할 수 있음.

**example1 - add**
```python
>>> v1 = Vector(2, 4)
>>> v2 = Vector(2, 1)
>>> v1 + v2
Vector(4, 5)
```
**example2 - abs**
```python
>>> v = Vector(3, 4)
>>> abs(v)
5. 0
```
**example3 - mul**
```python
>>> v * 3
Vector(9, 12)
>>> abs(v * 3)
15.0
```

```python
import math

class Vector:

	def __init__(self, x=0, y=0):
		self.x = x
		self.y = y

	self __repr__(self):
		return f'Vector({self.x!r}, {self.y!r}'

	def __abs__(self):
		return math.hypot(self.x, self.y)
	
	def __bool__(self):
		return bool(abs(self))

	def __add__(self, other):
		x = self.x + other.x
		y = self.y + other.y
		return Vector(x, y)

	def __mul__(self, scalar):
		return Vector(self.x * scalar, self.y * scalar)
```
위의 예시에서 `__add__`, `__mul__`, `__abs__`는 python document에서 보듯 numeric type처럼 핸들링 하기 위한 `special method`.

### String Representation
[3.3.1](https://docs.python.org/ko/3/reference/datamodel.html#basic-customization)

`__repr__`은 빌트인 함수인 `repr`에 의해 호출되며 검사할 객체의 문자열 표현을 가져옴.\
`__repr__`에 의해 리턴된 문자열은 명확해야하며, 가능하다면 표시된 객체를 다시 만드는데 필요한 소스코드와 일치.\
반면에 `__str__`은 암시적으로 print 함수를 사용함.\
`__str__`은 최종 사용자가 보기 적합한 형태의 스트링을 리턴함.
### Boolean Value of a Custom Type
[3.3.1](https://docs.python.org/ko/3/reference/datamodel.html#basic-customization)

if 또는 while 문을 제어하는 표현식, 피연산자 및 피연산자 또는 피연산자와 같은 `Boolean context` 모든 객체 허용.\

### Collection API
크게 두 가지 데이터 타입이 존재
* collections - 컨테이너 데이터 형
* collections.abc - 컨테이너의 추상 베이스 클래스