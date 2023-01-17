# 2. An Array of Sequences

## Overview of Built-in Sequences
- C로 구현된 여러 sequence type이 있다.
  - Container sequences: 여러 다른 타입을 담는다.
    - list, tuple, colections.deque ...
  - Flat Sequences: 하나의 타입만을 담는다.
    - str, bytes, array.array

### Container Sequence와 Flat Sequence의 비교
- Container Sequence는 각 아이템의 레퍼런스를 메모리에 담고 있다.
- Flat Sequence는 각 아이템을 메모리에 담고있다.
- Flat Sequence가 메모리를 훨씬 효율적으로 사용한다.
- Flat Sequence는 원시 기계 값(bytes, integers, floats)밖에 담지 못한다.

### Mutable Sequence vs Immutable Sequence
- Mutable Sequences: list, bytearray, array.array, collections.deque
- Immutable Sequences: tuple, str, bytes

n.b. tuple은 abc.Sequence의 subclass는 아니지만 virtual subclass이다. 따라서 `issubclass(tuple, abc.Sequence)`는 `True`이다. list와 abc.MutableSequence 역시 그러하다.

## List Comprehension (listcomp)
- `beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]`와 같은 형태로 쓴다.
- 아래와 같은 형태의 중첩도 가능하다. (cartesian product)
```
  tshirts = [(color, size) for color in colors
                            for size in sizes]
```

### Local Scope Within Listcomp and Genexps
- `codes = [ord(x) in for x in x]`에서 첫 번째와 세 번째 변수는 다른 변수이다.
- walus operator를 사용하면 outer scope로 꺼낼 수 있다.
  - `codes = [last := ord(c) for c in x]`

### Listcomp vs `map` and `filter`
- 다음 두 대입문은 같은 결과를 만들어낸다.
  - `beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]`
  - `beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))`
- listcomp 쪽이 속도도 더 빠르고 이해하기도 쉽다.

### Listcomp의 목적
- Listcomp의 목적은 하나다. list를 만드는(build) 것.
- for문이나 map을 이용해서 list를 만들 수도 있다. 하지만 Listcomp가 list를 만든다는 의도를 파악하기 쉽다.

### Generator Expressions (genexp)
- Listcomp에서 bracket을 parenthesis로 치환하면 genexp가 된다.
  - genexp는 generator를 만들어냄.
  - `beyond_ascii = (ord(s) for s in symbols if ord(s) > 127)`
  - single argument for call일 경우 괄호가 필요 없음.
    - `beyond_ascii = tuple(ord(s) for s in symbols if ord(s) > 127)`
- Genexp는 tuple, array, 그 밖에 다른 sequence를 만드는데 사용된다.
- Listcomp를 거쳐서 sequence를 만들 수도 있지만 genexp가 메모리를 더 절약한다.
  - `beyond_ascii = tuple([ord(s) for s in symbols if ord(s) > 127])`
  vs
  - `beyond_ascii = tuple(ord(s) for s in symbols if ord(s) > 127)`
  - Listcomp는 리스트 전체를 만든 후 sequence로 변환한다.
  - Genexp를 쓰면 요소 하나 하나를 sequence에 집어넣는다.

## Tuple은 Immutable List의 기능만을 가지고 있는 것이 아니다.

### Record로써의 tuple
- tuple의 이름에 맞는 기능.
- 좌표를 표현할 때 tuple을 사용할 수 있다.
  - cartesian coordinate: `(1, 2)`, `(2, 3)`
  - `[0]`은 x 좌표를 나타내고 `[1]`은 y 좌표를 나타낸다.
- tuple이 mutable하다면, 그래서 sort되거나 element가 삽입된다면 의미를 잃는다. 따라서 tuple은 immutable하다.
- named tuple은 chapter 5에서 다룬다.
- field 이름을 짓기위해서 class를 만드는 것보다 tuple을 사용하여 record를 쓰는게 더 간편할 때가 있다.

### Immutable List로써의 tuple
- tuple은 list에 비해 다음 두 가지 장점을 가진다.
  - clarity: 길이가 변하지 않는다.
  - performance: 같은 길이의 list에 비해 메모리 사용량이 적다.
- tuple의 내용물에 mutable object가 들어있으면 오류의 가능성이 높아진다.
  - hashable한 object는 value가 변하지 않는다.
  - unhashable tuple은 dict key나 set의 요소가 될 수 없다.
- tuple이 고정된 값을 가지는지 아닌지 알기 위해서 다음 함수를 사용할 수 있다.
```python
def fixed(o):
    try:
        hash(o)
    except TypeError:
      return false
    return True
```

### tuple의 메소드와 list의 메소드 비교
- tuple은
  - augmented operator 함수들 내장되어있지 않음. 사용할 수는 있음. (`__add__`를 통해서)
  - `copy`(shallow copy) 메소드 없음.
  - `__getnewargs__` special method 있음: pickle에서 최적화를 위한 것
    - pickle: 데이터를 파일 형태로 저장하고 싶을 때 사용한다.
  - `reverse()` 불가

## Sequence와 Iterable의 Unpacking

### 단순 unpacking
```python
lax_coordinates = (33.9425, -118.408056)
latitude, longitude = lax_coordinates
```

### Function Call에서의 unpacking
```python
>>> t = (20, 8)
>>> divmod(*t) # divmod(20, 8)
(2, 4)
>>> quotient, remainder = divmod(*t)
>>> quotient, remainder # 이 표기를 쓰면 tuple이 된다.
(2, 4)
>>> result = quotient, remainder
>>> result
(2, 4)
```

### \*를 이용해서 나머지를 취하기
- 나머지는 리스트로 묶인다.
```python
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
```
- 언패킹 인자가 부족하면 ValueError가 난다.
```python
>>> a, b, *rest = range(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: not enough values to unpack (expected at least 2, got 1)
```
- 맨 앞과 중간에도 묶을 수 있다.
```python
>>> a, *body, c, d = range(6)
>>> a, body, c, d
(0, [1, 2, 3], 4, 5)
>>> *head, b, c, d = range(3)
>>> head, b, c, d
([], 0, 1, 2)
```

### 함수에서 *를 이용해서 언패킹하기
- 함수에서 다음과 같이 *를 이용한 언패킹을 여러 번 사용할 수 있다.
```python
>>> def func(a, b, c, d, *rest)
...     return a, b, c, d, rest
...
>>> func(*[1, 2], 3, *range(4, 7))
(1, 2, 3, 4, [5, 6])
```
- 함수에서의 *rest인자는 중간에는 쓸 수 없다. 뒤의 인자는 keyword argument로 인식된다.
```python
>>> def func(a, b, *rest, d):
...     return a, b, rest, d
... 
>>> func(*range(6))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: func() missing 1 required keyword-only argument: 'd'
```

### 언패킹을 이용해 Sequent Literal 만들기
```python
>>> *range(4), 4                  # tuple
(0, 1, 2, 3, 4)
>>> [*range(4), 4]                # list
[0, 1, 2, 3, 4]
>>> {*range(4), 4, *(5, 6, 7)}    # set
{0, 1, 2, 3, 4, 5, 6, 7}
```

### 중첩된 언패킹
```python
>>> a, b, (c, d) = [1, 2, [3, 4]]
>>> a, b, c, d
(1, 2, 3, 4)
>>> a, b, [c, d] = (1, 2, (3, 4))
>>> a, b, c, d
(1, 2, 3, 4)
```
- 첫 번째 요소를 튜플 형식으로 뽑아낼 때는 주의해야한다.
```python
>>> (a,) = [1]
>>> a
1
>>> ((a,),) = [[1]]
>>> a
1
```

## Sequence를 이용한 패턴 매칭
- case문은 다음과 같은 순서로 패턴을 매칭한다.
  1. Sequence item의 개수가 맞는지
  2. 요소의 내용이 일치하는지
  3. if문을 만족하는지
```python
def handle_command(self, message):
    match message:
        case ['BEEPER', frequency, times]:
            self.beep(times, frequency)
        case ['NECK', angle]:
            self.rotate_neck(angle)
        case ['LED', ident, intensity]:
            self.leds[ident].set_brightness(ident, intensity)
        case ['LED', ident, red, green, blue]:
            self.leds[ident].set_color(ident, red, green, blue)
        case _:
            raise InvalidCommand(message)
```
- if문은 nested unpacking이 진행되고 평가된다.
```python
for record in metro_areas:
    match record:
        case [name, _, _, (lat, lon)] if lon <= 0:
            print(f'{name:15} | {lat:9.4f} | {lon:9.4f}')
```

- caution.
  - str, bytes, bytearray는 match/case문에서 Sequence로 다뤄지지 않는다.
  - 패턴 매칭에 이용하려면 다음과 같은 테크닉이 필요하다.
    - `phone`이 숫자로 이루어진 str일 때
```python
match tuple(phone):
    case ['1', *rest]:
        ...
    case ['2', *rest]:
        ...
    case ['3' | '4', *rest]:
        ...
```

- 다음 Sequence 타입들이 sequence pattern과 호환된다.
  - list, tuple, range, collection.deque memoryview, array.array

?? Unlike unpacking, patterns don't destructure iterables that are not sequences (such as iterators). ??

- `_`는 특수하게 취급된다.
  - 사용하지 않는 변수 자리에 사용된다.
  - 패턴에 두 번 이상 등장할 수 있는 유일한 변수 이름이다.

- 패턴의 일부를 변수로 묶을 수 있다.
```python
    case [name, _, _, (lat, lon) as coord]:
        ...
```

- 패턴에 타입을 지정할 수 있다.
  - name이 str이어야하고 lat, lon이 float이어야 할 때
```python
    case [str(name), _, _, (float(lat), float(lon))]:
```
- 패턴 중간에 asterisk를 사용할 수도 있다.
  - str인 name으로 시작하고 float인 lat, lon으로 이루어진 tuple로 끝나는 Sequence를 매칭할 경우
```python
    case [str(name), *_, (float(lat), float(lon))]:
```

- Sequence를 매칭시키고 싶을 때 다음과 같은 방법을 쓴다.
  - 다음 방법은 parms가 sequence가 아니더라도 parms에 인자가 들어온다.
```python
    case ['lambda', parms, *body]:
        ...
```
  - 다음 방법을 쓰면 parms가 Sequence임을 보장할 수 있다.
```python
    case ['lambda', [*parms], *body]:
        ...
```
- 타입만을 테스트하고 변수를 사용하지 않을 때 다음과 같이 사용할 수 있다.
```python
match test:
    case [str()]:
        return 'string'
    ...
```

## Slicing

### 왜 마지막 요소가 제외되는지
생략

### Slice Object
- slicing은 `seq[start:stop:step]`의 형태로 이루어진다.
  - `seq.__gettiem__(slice(start, stop, step))`이 호출된다.
- slice object를 이용하여 범위에 이름을 붙일 수 있다.
```python
SCOPE = slice(0, 6, 2)
new_list = some_list[SCOPE]
```

### 다차원 Slice와 Ellipsis
- `[]`연산자는 여러 개의 index나 slice object를 쉼표로 나누어서 받을 수 있다.
- `a[i, j]`는 `a.__get_item((i, j))`로 호출된다.
  - 따라서 NumPy 패키지나 사용자 정의 오브젝트에서 다차원 슬라이스를 사용할 수 있게 해준다.
  - `a[m:n, k:l]`
- memoryview를 제외하고, built-in sequence는 1차원이다.
  - tuple의 형태로 인자를 받을 수 없다.

- `...`는 Ellipsis object의 alias이다. (Ellipsis object는 ellipsis class의 싱글톤이다.)
  - 함수 파라미터 등으로 넣을 때 특별한 의미를 가진 것처럼 보이지만 Ellipsis object가 넘어가는 것 뿐이다.
  - `func(1, ..., 3)` == `func(1, Ellipsis, 3)`
  - 사용자 정의 클래스 등에서 이용할 수는 있겠다.
- NumPy에서 slicing할 때 `...`를 이용할 수 있다.
  - x가 4차원 배열일 때 다음과 같다.
  - `x[i: ...]` == `x[i, :, :, :]`
n.b. `...`는 작성되지 않은 코드의 placeholder로도 이용된다. 인터프리터는 아무것도 실행하지 않고 넘어간다.
n.b. `Ellipsis`는 오브젝트의 이름이고 `ellipsis`는 클래스의 이름이다. 컨벤션과는 반대이다.
  - 이는 `bool` 클래스의 인스턴스들인 `True`, `False`의 경우와 비슷하다.

### Slice에 대입하기
- mutable sequcne는 slice 표기를 이용하여 삽입되거나, 제거되거나, 수정될 수 있다.
```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[2::2] = [11, 22]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 2 to extended slice of size 3
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
>>> l[2:3] = []
>>> l
[0, 1, 22, 9]
```
- slice에 대입할 경우 반드시 iterable을 대입하여야 한다.
- step이 있을 경우 slice된 길이와 대입할 iterable의 길이는 동일해야한다.

### \+와 \*, Sequence와 함께 쓰기
```python
>>> l = [1, 2, 3]
>>> l * 5
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
>>> 5 * 'abcd'
'abcdabcdabcdabcdabcd'
```

caution.
- reference가 들어있는 sequence에 * 연산자를 사용할 경우 원하지 않는 결과가 나올 수 있다.
```python
>>> l = [['a']] * 3
>>> l
[['a'], ['a'], ['a']]
>>> l[0][0] = 'b'
>>> l
[['b'], ['b'], ['b']]
```

### list의 list 만들기
생략 (위의 cuation이 적용된 내용)

### augmented assignment와 sequence
- `+=`와 `*=`는 첫 번째 피연산자가 무엇인지에 따라 행동이 달라진다.
- `+=`는 `__iadd__`를 호출하지만 `__iadd`가 구현되어있지 않을 경우 `__add__`를 호출한다.
  - iadd는 in-place addition이라는 뜻

```
>>> a += b
```
- a가 `__iadd__`를 구현한 경우
  - a가 mutable sequence일 때 a가 바뀐다.
  - 이는 `a.extend(b)`와 비슷하게 동작한다.
- a가 `__iadd__`를 구현하지 않은 경우
  - `a += b` == `a = a + b`
  - 새로운 object가 만들어져 a에 들어간다.
- iadd의 구현 여부에 따라 a가 바뀔지 아닐지가 정해진다.
- immutable sequence의 repeated concatenation은 비효율적이다.

### A += A assignment의 특이 케이스
```
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> t
(1, 2, [30, 40, 50, 60])
```
-  `s[a] += b`를 실행할 경우 bytecode에서 다음과 같은 일이 일어난다.
  1. `s[a]`를 TOS(Top Of Stack)에 넣는다.
  2. `TOS += b`를 실행한다. TOS의 reference가 mutable object인 경우 성공한다.
  3. `s[a] = TOS`를 실행한다. s가 immutable일 경우 실패한다.

- 따라서
  - mutable object를 immutable object에 넣지 말아라.
  - augmented assignment은 atomic operation이 아니다.
  - bytecode를 살펴보는 것은 어려운 일이 아니다. 한 번 볼 때가 좋을 때도 있다.

### `list.sort` vs `sorted` built-in
- `list.sort`는 in-place sort이다. copy를 만들지 않는다.
  - 이를 명확히 하기 위해 return값은 `None`이다.
  - in-place operation/method 등은 return 값이 `None`인 것이 컨벤션이다.
  - `random.shuffle(s)`도 `None`을 리턴한다.
  - 이는 단점도 있는데, cascade call이 불가능하다.
- built-in `sorted`는 새로운 list를 리턴한다.
  - iterable object(immutable sequence, generator도 포함)를 인자로 받아 리스트를 리턴한다.
- `list.sort`와 built-in `sorted`는 두 가기 keyword-only argument를 받는다.
  - `reverse=True`인 경우 내림차순으로 정렬. 기본 값은 `False`
  - `key`는 단일 argument 함수로 그 리턴 값을 기준으로 정렬이 된다.
    - `key=str.lower`인 경우 case-insensitive하게 정렬된다.
    - `key=len`일 경우 길이를 기준으로 정렬된다.
    - 기본 함수는 `identity function`(항등 함수)이다.
    - `min()`, `max()` built-in 등을 사용할 수 있다.
    - 라이브러리 함수를 사용할 수도 있다. e.g. `itertools.groupby()`, `heapq.nlargest()`

caution.
- ASCII 기준으로 정렬된다.
- non-ASCII의 경우 엄밀히 정렬되지 않을 수 있다.

#### 정렬되고 나서...
- 정렬된 sequence는 검색이 쉽다.
  - `bisect` 모듈을 이용해서 binary search를 할 수 있다.
  - `bisect.insort`를 사용하면 정렬된 상태로 요소를 삽입할 수 있다.
- !읽어보기 [Managing Ordered Sequences with Bisect](https://fpy.li/bisect)

## List가 답이 아닐 때 (list 외의 mutable sequence)
- list는 가장 많이 쓰이는 sequence이다.
- 수 만 개의 float를 담고있는 sequence 여러 개를 다룰 때는 array를 사용하는 게 적합하다.
- sequence 양 끝에서 삽입과 삭제가 자주 일어날 때는 deque를 쓰는 것이 좋다. (FIFO)
- 이 단원에서는 list나 tuple이 아닌 다른 sequence를 써야할 만한 상황과 그런 sequence에 대해 다룬다.

### Array
- sequence가 숫자 만을 다룰 때는 array.array가 더 효율적이다.
- array는 모든 mutable sequence operation을 지원한다.
  - `.pop`, `.insert`, `.extend` 등...
- 빠른 로딩과 저장을 위한 `.frombytes`, `.tofile` 등도 지원한다.
- python array는 C array만큼이나 컴팩트하다.
  - array에는 요소 각각의 객체가 담기는 것이 아니라 c type으로 담긴다.
- array 생성 시 대응되는 C type을 의미하는 typecode를 인자로 넘겨주어야한다.
  - `'b'`의 경우는 signed char C 타입이다.
    - 이 경우 interpreter는 `number` type으로 인식한다.
  - signed char이 아닌 경우 에러가 발생한다.
```python
>>> from array import array
>>> sints = array('b', [1, 2, 3])
>>> sints
array('b', [1, 2, 3])
>>> sints[0] = 127
>>> sints
array('b', [127, 2, 3])
>>> sints[0] = 128
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
OverflowError: signed char is greater than maximum
```

- 대량의 float array 생성, 저장, 로딩
```python
>>> from array import array
>>> from random import random
>>> floats = array('d', (random() for i in range(10**7)))
>>> floats[-1]
0.08340685773523371
>>> fp = open('floats.bin', 'wb')
>>> floats.tofile(fp)
>>> floats2 = array('d')
>>> floats2
array('d')
>>> fp = open('floats.bin', 'rb')
>>> floats2.fromfile(fp, 10**7)
>>> fp.close()
>>> floats2[-1]
0.08340685773523371
>>> floats2 == floats
True
```
- 각 줄이 float(text)로 된 textfile에서 float를 불러오는 것보다 60배 빠르다.
  - float built-in에서 parsing이 일어난다.
- `array.tofile`은 각 줄에 text 형태의 float를 저장하는 것보다 7배 빠르다.
- 1000만 개의 float는 binary file로는 80,000,000 byte이고 text file로는 181,515,739 byte이다.

- 특정 수학적 array를 위해서는 bytes와 bytearray 타입이 있다.
  - 예를 들자면 raster image

#### list와 array의 비교
- list와 동일한 점
  - (repeated) concatenation(`*`, `+`)과 in-place repected concatenation(`*=`, `+=`)를 지원한다.
  - `in` operator를 지원한다. (`.__contains__(e)`)
  - `.append(e)`, `.count(e)`, `.extend(it)`, `.index(e)`, `.insert(p, e)`, `.pop([p])`, `.remove(e)`, `reverse()` 메소드가 있다.

- list에는 있지만 array에는 없는 기능
  - `del` 연산자를 지원하지 않는다. (`.__delitem__(p)`)
    - 책에는 있다고 되어있는데 실제로 찾아보니까 없다.
  - `reversed()`를 사용할 수 없다. (`.__reversed__()`)
    - 뒤집힌 generator를 만들어낸다.
  - `.clear()`, `.copy()`, `.sort([key], [reverse]`가 없다.

- list에는 없지만 array에는 있는 기능
  - `.byteswap()`: endian을 바꾸는 기능.
  - `copy.copy`를 지원한다. (`.__copy__()`)
  - `copy.deepcopy`의 최적화를 지원한다. (`.__deepcopy__()`)
  - `.frombytes(b)`: sequence를 packed machine value로 해석하여 append한다.
  - `.fromfile(f. n)`: 파일 f에서 n 개의 요소를 packed machine value로 해석하여 append한다.
  - `.fromlist(l)`: 리스트에서 append한다. `TypeError`가 생기면 append하지 않는다.
  - `.itemsize`: attribute이다. array 각 요소의 byte 길이이다.
  - `.tobytes()`: bytes object로 리턴한다.
  - `.tofile(f)`: binary file f에 저장한다.
  - `.tolist()`: list로 리턴한다.
  - `.typecode`: attribute이다. C type을 나타내는 1자의 string code이다.

### Memory Views
- 공유 메모리 sequence type
  - byte를 복사하지 않고 array의 slice를 다룰 수 있다.
- `memoryview.cast`는	복사나 bit 변화 없이 byte를 읽고 쓰는 방법을 바꾸게 해준다.
- 6 byte의 메모리를 1\*6, 2\*3, 3\*2의 view로 사용하기
```python
>>> from array import array
>>> octets = array('B', range(6))
>>> m1 = memoryview(octets)
>>> m1.tolist()
[0, 1, 2, 3, 4, 5]
>>> m2 = m1.cast('B', [2, 3])
>>> m2.tolist()
[[0, 1, 2], [3, 4, 5]]
>>> m3 = m1.cast('B', [3, 2])
>>> m3.tolist()
[[0, 1], [2, 3], [4, 5]]
>>> m2[1,1] = 22
>>> m3[1,1] = 33
>>> octets
array('B', [0, 1, 2, 33, 22, 5])
```

- memoryview의 특성상 자료형이 손상될 수도 있다.
```python
>>> numbers = array('h', [-2, -1, 0, 1, 2])
>>> memv = memoryview(numbers)
>>> len(memv)
5
>>> memv[0]
-2
>>> memv_oct = memv.cast('B')
>>> memv_oct.tolist()
[254, 255, 255, 255, 0, 0, 1, 0, 2, 0]
>>> memv_oct[5] = 4
>>> numbers
array('h', [-2, -1, 1024, 1, 2])
```
- [struct와 memoryview에 관해서](https://fpy.li/2-18)

### NumPy
- NumPy는 파이썬이 과학적 계산 응용에 잘 쓰이는 이유이다.
- NumPy에는 다차원 단일 타입 array와 matrix 타입 등이 구현되어 있다.
  - 사용자 정의 record 등도 담을 수 있다.
- SciPy는 NumPy를 응용하여 만들어졌다.
  - 과학적 계산: 선형대수, 미적분, 통계 등.
  - python의 interactive prompt, api 등을 활용할 수 있으면서 C나 Portran으로 최적화된 계산을 제공한다.
- Pandas는 NumPy와 SciPy를 이용해 만들어졌다.
- `pip install numpy`를 통해 설치해야 한다.

- NumPy로 구현된 2차원 array의 간단한 계산
```python
>>> import numpy as np
>>> a = np.arange(12)
>>> a
array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11])
>>> type(a)
<class 'numpy.ndarray'>
>>> a.shape
(12,)
>>> a.shape = 3, 4
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
>>> a[2]
array([ 8,  9, 10, 11])
>>> a[2, 1]
9
>>> a[:, 1]
array([1, 5, 9])
>>> a.transpose()
array([[ 0,  4,  8],
       [ 1,  5,  9],
       [ 2,  6, 10],
       [ 3,  7, 11]])
```
- NumPy는 `numpy.ndarray`의 요소에 대해서 고수준의 로딩, 저장, 연산을 지원한다.
```python
>>> import numpy
>>> floats = numpy.loadtxt('floats-10M-lines.txt')
>>> floats[-3:]
array([0.20763122, 0.99883902, 0.99485856])
>>> floats *= .5  # floats의 모든 수에 0.5를 곱한.
>>> floats[-3:]
array([0.10381561, 0.49941951, 0.49742928])
>>> from time import perf_counter as pc
>>> t0 = pc(); floats /= 3; pc() - t0                   # performance_check
0.02679770899703726
>>> numpy.save('floats-10M', floats)
>>> floats2 = numpy.load('floats-10M.npy', 'r+')
>>> floats2 == floats
array([ True,  True,  True, ...,  True,  True,  True])
>>> floats2 *= 6
>>> floats2[-3:]
memmap([0.20763122, 0.99883902, 0.99485856])
```

- array를 text로 저장하기
```python
>>> from array import array
>>> arr = array('f', (random() for _ in range(10**7)))
>>> with open('float-10M-lines.txt', 'w') as f:
...     for d in arr.tolist():
...         f.write(f"{d}\n")
...
```

### Deque
- `.append`와 `.pop`을 사용하면 list를 stack이나 queue처럼 사용할 수 있다.
  - `.append`와 `.pop(-1)`을 사용하면 stack을 만들 수 있다.
  - `.append`와 `.pop(0)`를 사용하면 queue를 만들 수 있다.
- `collections.deque`는 빠르게 insert와 pop이 가능한 thread-safe deque이다.
- `deque`는 제한될(bounded) 수 있다.
  - 길이를 제한하여 고정된 최대 길이를 가지게 만들 수 있다.
  - 제한된 `deque`는 최대 길이에 도달했을 때 한 쪽에 요소가 추가되면 다른 쪽의 요소를 버린다.
  - 제한된 길이를 나타내는 `maxlen` attribute는 read-only이다.
```python
>>> from collections import deque
>>> dq = deque(range(10), maxlen=10)
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.rotate(3)
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4)
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.appendleft(-1)
>>> dq
deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.extend([11, 22, 33])
>>> dq
deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)
>>> dq.extendleft([10, 20, 30, 40])
>>> dq
deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
```

#### list와 deque의 비교
- list와 동일한 점
  - (repeated) concatenation(`*`, `+`)과 in-place (repeated) concatenation(`*=`, `+=`)를 지원한다.
    - 책에는 `.__add__(sq)`, `.__mul__(n)`, `.__imul__(n)`, `.__rmul__(n)`가 구현되지 않은 걸로 나오나 호출할 수 있음.
  - `.append(e)`, `.clear()`, `.copy()`, `.count(e)`, `.extend(i)`, `.remove(e)`, `.reverse()`, `.index(e)` 메소드가 있다.
  - `in` 연산자를 지원한다. (`.__contains__(e)`)
    - `.__contains__(e)`가 없다고 되어있는데 있다.
  - `del` 연산자를 지원한다. (`.__delitem__(e)`)
  - `[]`연산자를 지원한다. 슬라이스도 지원. (`.__getitem__(p)`, `.__setitem__(p, e)`)
    - `s[p]`, `s[p] = e`
  - `iter(s)` 함수를 지원한다. (`.__iter__()`)
  - `len(s)` 함수를 지원한다. (`.__len__()`)
  - `reversed(s)` 함수를 지원한다. (`.__reversed__()`)
    - `__copy__`가 구현되지 않은 걸로 나오지만 호출할 수 있음.

- list에는 있지만 deque에는 없는 점
  - `.sort([key], [reverse])` 메소드가 있다.

- list에는 없지만 deque에는 있는 점
  - `.appendleft(e)`, `.extendleft(i)`, `.popleft()`, `.rotate(n)`, `.popleft()` 메소드가 있다.

- 그 외 차이점
  - `.insert(p, e)`
    - deque에서는 최대 크기에 도달하면 `IndexError`를 띄운다.
  - `.pop`
    - `l.pop(p)`는 `p`자리의 요소를 제거한다.
    - `dq.pop()`은 마지막 요소를 제거한다.

### 그 외의 Queue
- `queue`
  - 동기화된(i.e., thread-safe) 클래스들을 제공한다. 스레드 간의 안전한 통신에 이용될 수 있다.
    - `SimpleQueue`: `maxsize`로 한정될 수 없다.
    - `Queue`, `LifoQueue`, `PriorityQueue`
      - 원소의 개수가 maxsize에 도달하면 새 요소의 삽입을 block한다; 다른 스레드가 가져갈 때까지 기다린다.

- `multiprocessing`
  - process 간 통신을 위해 설계되어있다.
    - `SimpleQueue`(한정될 수 없음), `Queue`
    - `JoinableQueue`는 task 관리를 위한 특수한 queue이다.

- `asyncio`
  - 비동기 프로그램에서의 task 관리를 위해 설계되었다.
    - `Queue`, `LifoQueue`, `PriorityQueue`, `JoinableQueue`

- `heapq`
  - queue 클래스를 가지고 있지 않다.
  - mutable sequence를 heap queue나 priority queue로 사용할 수 있게 해준다.
    - `heappush`, `heappop` 등

## 읽을 거리
- [Sorting HOW TO](https://fpy.li/2-22): `sorted`와 `list.sort`
- [PEP 3132 - Extended Iterable Unpacking](https://fpy.li/2-2)
- [Missing *-unpacking generaliation](https://fpy.li/2-24)
- [PEP 448 - Additional Unpacking Generalization](https://fpy.li/pep448)
- [Structural Pattern Matching](https://fpy.li/2-6)
- [PEP 636 - Structural Pattern Matching: Tutorial](https://fpy.li/pep636)
  - [Appendix A - Quick Intro](https://fpy.li/2-27)
- [PEP 635 - Structural Pattern Matching: Motivation and Rationale](https://fpy.li/pep635)
- [Less copies in Python with the buffer protocol and memoryviews](https://fpy.li/2-28): memoryview에 대한 tutorial
- [__Python Data Science Handbook__](https://fpy.li/2-29): Numpy
- [From Python to Numpy](https://fpy.li/2-31): "NumPy is all about vectorization"
- [Container datatypes](https://fpy.li/collec): deque에 익숙해지기