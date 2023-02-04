# Object References, Mutability, and Recycling

## 대입하다(assign)
- 값이 변수에 대입되어 있다고 말하지 변수가 값에 대입되어 있다고 하지는 않는다.
- 그러나 값이 먼저고 변수가 나중에 생긴다.
- 따라서 변수가 값에 '붙는다'(bind)라고 표현할 것이다.
```python
>>> class Gizmo:
...     def __init__(self):
...             print(f'Gizmo id: {id(self)}')
... 
>>> x = Gizmo()
Gizmo id: 4315762256
>>> y = Gizmo() * 10
Gizmo id: 4314353328
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for *: 'Gizmo' and 'int'
>>> dir()
['Gizmo', '__annotations__', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'x']
```
- `Gizmo`가 생기고, * 연산에서 오류가 발생하고, y 변수는 생성되지 않는다.

## Identity, Equlity(동등), Aliases
```python
>>> charles = {'name': 'Charlse L. Dodgson', 'born': 1832}
>>> lewis = charles
>>> lewis is charles
True
>>> id(charles), id(lewis)
(4315811392, 4315811392)
>>> lewis['balance'] = 950
>>> charles
{'name': 'Charlse L. Dodgson', 'born': 1832, 'balance': 950}
>>> alex = {'name': 'Charlse L. Dodgson', 'born': 1832, 'balance': 950}
>>> alex == charles
True
>>> alex is not charles
True
```
- `charles`와 `lewis`는 같은 객체를 가리킨다.
- `alex`는 또 다르지만 데이터가 같다.
- `==` 연산은 `.__eq__()`의 alias이다.
- `is` 연산은 `id()`를 비교한다.

### `id()`를 사용하는 경우
- 디버깅 시, 두 객체가 다른 컨텍스트에 존재하여 `is` 연산을 사용할 수 없을 때가 있다.
- 그런 경우 `id()`를 사용하여 디버깅할 수 있다.

## `==`과 `is` 중 어떤 것을 써야하나?
- 대부분 `==`를 사용하면 된다.
- 싱글톤 객체와 비교할 때는 `is`를 사용해도 된다.
  - 가장 흔한 예는 `None`이다.
  - 센티널 싱글톤으로 비교할 수도 있다.
  - `==`를 사용해도 되지만 약간 느리다.
  ```python
  END_OF_DATA = object()
  ...
  if node is END_OF_DATA:
      return
  ...
  ```
- `is` 연산자는 단순히 id만을 비교하기 때문에 `__eq__`를 호출하는 `==`보다 빠르다.
  - `__eq__`가 오버라이딩 되지 않은 객체는 `object`에서 상속된 `id()`를 비교하는 `__eq__`를 사용한다.
  - 대부분의 클래스가 `__eq__` 오버라이딩을 사용한다.

## 튜플의 상대적 immutability
...

## 기본 복사는 얕은 복사이다.
- list를 복사할 때 `[:]`를 이용할 수도 있다.
...

## 함수 매개변수는 레퍼런스이다.
- immutable 객체는 레퍼런스로 건네지지만 변하지 않는다.
  - mutable 객체는 레퍼런스로 건네지고 변할 수 있다.
```python
>>> def f(a, b):
...     a += b
...     return a
... 
>>> x = 1
>>> y = 2
>>> f(x, y)
3
>>> x, y
(1, 2)
>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b
([1, 2, 3, 4], [3, 4])
>>> m = (10, 20)
>>> n = (30, 40)
>>> f(m, n)
(10, 20, 30, 40)
>>> m, n
((10, 20), (30, 40))
>>> 
```

## 매개변수의 기본값을 mutable로 사용하면 안된다.
```python
>>> class HauntedBus:
...     def __init__(self, passengers=[]):
...         self.passengers = passengers
... 
>>> h1 = HauntedBus(['paul', 'john'])
>>> h1.passengers.append('locke')
>>> h1.passengers
['paul', 'john', 'locke']
>>> h2 = HauntedBus()
>>> h2.passengers
[]
>>> h2.passengers.append('megan')
>>> h2.passengers
['megan']
>>> h3 = HauntedBus()
>>> h3.passengers
['megan']
```
...

## mutable 매개변수에 대해 안전한 프로그래밍 하기
```python
>>> class TwilightBus:
...     def __init__(self, passengers=None):
...             if passengers is None:
...                     self.passengers = []
...             else:
...                     self.passengers = passengers
... 
>>> basketball_team = ['sue', 'tina', 'maya', 'diana', 'pat']
>>> t = TwilightBus(basketball_team)
>>> t.passengers.remove('sue')
>>> t.passengers
['tina', 'maya', 'diana', 'pat']
>>> basketball_team
['tina', 'maya', 'diana', 'pat']
```
- `basketball_team`이 망가져버렸다.
- 다음과 같이 해야 안전하다.
```python
>>> class TwilightBus:
...     def __init__(self, passengers=None):
...             if passengers is None:
...                     self.passengers = []
...             else:
...                     self.passengers = list(passengers)
... 
>>> basketball_team = ['sue', 'tina', 'maya', 'diana', 'pat']
>>> t = TwilightBus(basketball_team)
>>> t.passengers.remove('sue')
>>> t.passengers
['tina', 'maya', 'diana', 'pat']
>>> basketball_team
['sue', 'tina', 'maya', 'diana', 'pat']
```
- 이럴 경우, 인자로 iterable을 넘겨줄 수 있고 `passengers`에 list의 모든 메소드를 사용할 수 있다.

## `del`과 가비지 콜렉션
- `del(x)`는 `del x`와 같은 뜻이지만, 이것은 `(x)`와 `x`가 같기 때문.
  - `del`은 함수가 아니다.
- 다음의 경우, 마지막 줄에서 a가 가리키던 값의 메모리는 회수될 것이다.
```python
>>> a = [1, 2]
>>> b = a
>>> del a
>>> b
[1, 2]
>>> b = [3]
```
- `__del__` 매직 메소드가 있지만 잘 사용하지 않는다.
  - 가비지 콜렉터에 의해 사라지기 직전에 실행된다.
- CPython에서 가비지 콜렉터는 레퍼런스 카운팅에 의해 구현된다.
- Jython, IronPython같이 다른 플랫폼에 의존하는 파이썬 구현은 레퍼런스 카운팅이 0이 될 때 `__del__`이 실행되지 않는다.
  - 따라서 CPython 이외에서 `__del__`은 일관적인 실행을 보장하지 않는다.
- pypy의 가비지 콜렉터. [PyPy, Garbage Collection, and a Deadlock](https://fpy.li/6-7)
- `weakref`를 사용하면 객체가 죽었는지 살았는지 알 수 있다.
```python
>>> import weakref
>>> s1 = {1, 2, 3}
>>> s2 = s1
>>> def bye():
...     print('bye')
... 
>>> ender = weakref.finalize(s1, bye)
>>> ender.alive
True
>>> del s1
>>> ender.alive
True
>>> s2 = 'spam'
bye
>>> ender.alive
False
```