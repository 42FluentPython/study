# A Pythonic Object

파이썬 데이터 모델 덕에 사용자 정의 타입이 빌트인 타입처럼 동작하는것이 가능.\
일반적인 어플리케이션 클래스는 많은 스페셜 메소드를 구현할 필요가 없지만, 라이브러리 또는 프레임워크를 개발한다면 그것들은 파이썬이 제공하는 클래스처럼 동작하길 원함.\
위의 기대를 충족시키는 한 가지 방법은 `Pythonic`임.

## Object Representations
모든 객체지향언어는 적어도 하나의 표준 문자열 표현 방법을 가지고 있음.\
파이썬은 두개를 보유.
* **repr()**
* **str()**
위의 두 기능 말고 추가로 두 개의 스폐셜 메소드를 가짐.
* **__bytes__**
* **__format__**

## classmethod Versus staticmethod
### classmethod
`classmethod`는 메소드가 인스턴스에서 동작하는게 아닌 클래스에서 동작하게 해줌.\
`classmethod`는 메소드 호출 방법을 변경.\
그러므로 첫번째 인자로 인스턴스 대신 클래스를 받음.\
주된 공통 사용 용도론 생성자 대안으로 사용함.(ex) frombytes
```python
@classmethod
def frombytes(cls, octests):
	typecode=chr(octets[0])
	memv = memoryview(octets[1:]).cast(typecode)
```

### staticmethod
`staticmethod`는 메소드가 인스턴스에서 동작하는게 아닌 클래스에서 동작하게 해줌.\
`staticmethod`는 `classmethod`와는 다르게 class 같은 특수한 첫 번째 인자를 받지 않을때 사용함.

### 두 데코레이터의 차이
* **Comparing behaviors of classmethod and staticmethod**
```python
class Demo:
	@classmethod
	def klassmeth(*args):
		return args
	@staticmethod
	def statmeth(*args):
		return args
```
* **result**
```python
>>> Demo.klassmeth()
(<class '__main__.Demo'>,) >>> Demo.klassmeth('spam')
(<class '__main__.Demo'>, 'spam') >>> Demo.statmeth()
()
>>> Demo.statmeth('spam') ('spam',)
```

## Formatted Displays
`format()`은 내장 함수, `str.format()`메소드는 각 타입에 대한 실질적인 포맷팅을 `__format__(format_spec)`에 위임함.\

## A Hashable Vector2d
`hasable`이 되기 위해서는 반드시 `__hash__`와 `__eq__`가 구현되야함.
* **example**
```python
# inside class Vector2d:

def __hash__(self):
	return hash((self.x, self.y))
```

## Supporting Postional Pattern Matching

## Complete Listing of Vector2d, Version3

## Private and "Protected" Attributes in Python
파이썬은 `private` 변수를 만드는 방법이 없음.\
대신 서브 클래스의 속성에 덮어쓰기를 막는 단순한 메커니즘이 존재.\
속성 앞에 `__`를 붙이면 클래스 내부 `__dict__`에 클래스 이름과 같이 저장.

#### example
> * Dog 클래스 내의 `__mood`는 `_Dog__mood`로 저장.
> * Dog을 상속한 Beagle 클래스 내의 `__mood`는 `_Beagle__mood`로 저장

## Saving Memory with \_\_slots\_\_
기본적으로 파이썬은 각 인스턴스의 속성을 `dict`에 저장함.
