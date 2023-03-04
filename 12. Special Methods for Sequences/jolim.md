# Special Methods for Sequences

## Vector를 확장하면서 다룰 것
- `__len__`, `__getitem__`
- 많은 요소를 가지고 있는 객체의 안전한 표현(repr)
- 적절한 slicing 지원. slicing이 새로운 벡터를 만들어낸다.
- aggregate hashing. 모든 요소를 이용해 hashing한다.
- custom formatting language extension
- `__getattr__`, `__setattr__`

- [final vector code](https://github.com/fluentpython/example-code-2e/blob/master/12-seq-hacking/vector_v5.py)

## `reprlib.repr`
- `reprlib.repr()`은 repr의 길이를 제한해준다. 너무 많은 요소가 있을 경우 필요하다.

## protocol
- informal protocol은 일부 메소드만 구현해도 된다.
- `Sequence`가 되기 위해서는 `__len__`과 `__getitem__`을 구현해야한다.
  - iteration을 구현하기 위해서 `__getitem__`으로 충분하다. `__len__`까지 구현할 필요는 없다.

## slice
- 인덱스에 슬라이스를 넘기면 다음과 같이 리턴된다.
```python
>>> class MySeq:
...     def __getitem__(self, index):
...             return index
... 
>>> s = MySeq()
>>> s[1]
1
>>> s[1:2]
slice(1, 2, None)
>>> s[1:2:-1]
slice(1, 2, -1)
>>> s[1:2:-1:4]
  File "<stdin>", line 1
    s[1:2:-1:4]
            ^
SyntaxError: invalid syntax
>>> s[1:2:-1, 2:3, 4]
(slice(1, 2, -1), slice(2, 3, None), 4)
```
- `S.indices(len)`은 slice를 '정규화'한다.
```python
>>> slice(None, 10, 2).indices(4)
(0, 4, 2)
>>> slice(-3, None, -1).indices(8)
(5, -1, -1)
```
- slice 시 벡터를 리턴하는 구현은 다음과 같다.
```python
def __getitem__(self, key):
    if isinstance(key, slice):
        cls = type(self)
        return  cls(self._components[key])
    index = operator.index(key)
    return self._components[key]
```
- `operator.index`는 객체를 적절한 인덱스 값으로 바꾸어준다.
  - `__index__` 특수 메소드를 이용한다.
  - 적절하지 않은 값이 들어가면 `TypeError`를 발생시킨다.

## 동적 어트리뷰트 접근
- 첫 네 개의 요소를 `v.x`, `v.y`, `v.z`, `v.t`로 접근하고 싶다.
```python
__match_args__ = ('x', 'y', 'z', 't')

def __getattr__(self, name):
    cls = type(self)
    try:
        pos = cls.__match_args__.index(name)
    except ValueError:
        pos = -1
    if 0 <= pos < len(self._components):
        return self._components[pos]
    msg = f'{cls.__name__!r} object has no attribute {name!r}'
    raise AttributeError(msg)
```
- `__getattr__`가 호출되는 과정은 다음과 같다.
  - 인스턴스 어트리뷰트 `x`가 있는지 찾는다.
  - 찾지 못한다면 클래스 어트리뷰트 `x`가 있는지 찾는다.
  - 찾지 못한다면 `__getattr__`를 찾는다.

- 위의 구현에는 결함이 있다.
```python
>>> v = Vector(range(5))
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0])
>>> v.x
0.0
>>> v.x = 10
>>> v.x
10
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0])
```
- 이 경우 `v.__dict__`에 `x`가 등록이 되고, 인스턴스 어트리뷰트가 되면서 `__getattr__`가 호출되지 않은 것이다.
- `__setattr__`를 구현해야 막을 수 있다.
```python
def __setattr__(self, name, value):
    cls = type(self)
    if len(name) == 1:  # <1>
        if name in cls.__match_args__:  # <2>
            error = 'readonly attribute {attr_name!r}'
        elif name.islower():  # <3>
            error = "can't set attributes 'a' to 'z' in {cls_name!r}"
        else:
            error = ''  # <4>
        if error:  # <5>
            msg = error.format(cls_name=cls.__name__, attr_name=name)
            raise AttributeError(msg)
    super().__setattr__(name, value)
```
- 이 때, `__slots__`를 사용하여 새로운 어트리뷰트의 생성을 막는 것은 하지 말아야한다.
  - 오직 메모리 성능 향상을 위해서만 사용하시오

## hashing
- tuple을 생성하지 않는 hashing 방법
```python
def __hash__(self):
    hashes = (hash(x) for x in self._components)
    return reduce(operator.xor, hashes, 0)
```
- `reduce` 함수는 시작할 때 시퀀스의 첫 번째 값과 두 번째 값을 계산 함수의 첫 번째와 두 번째 파라미터로 넘긴다.
  - 모든 파라미터를 두 번째 인자로 넘기기 위해서는 `reduce`의 세 번째 인자인 initializer를 설정해주는 것이 좋다.

## 빠른 `==`
- 이전에는 tuple을 만들어서 비교했다.
- 모든 요소를 tuple로 만드는 것은 비용이 크다.
```python
def __eq__(self, other):
    if len(self) != len(other):
        return False
    for a, b in zip(self, other):
        if (a != b):
            return False
    return True
```
- 한 줄로 쓰면 다음과 같다.
```python
def __eq__(self, other):
    return len(self) == len(other) and all(a == b for a, b in zip(self, other))
```

## formatting
- 생략

