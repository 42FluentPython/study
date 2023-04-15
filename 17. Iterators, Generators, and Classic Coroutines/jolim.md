# Iterators, Generators, and Classic Coroutines

- iterable은 다음과 같은 연산을 지원한다.

  - `for` 루프
  - comprehensions
  - unpacking
  - collection instance 만들기

- 다음 주제를 다룰 것이다.
  - `iter()` 빌트인
  - 파이썬에서 classic iterator pattern 사용하기
  - iterator pattern을 generator 함수나 genexp로 대체하는 방법
  - generator 함수의 작동 방식
  - 표준 라이브러리의 범용 generator 함수들
  - `yield from`을 이용해서 generator 합치기
  - generator과 classic coroutine의 유사점 및 차이점

## A Sequence of Words

- 다음을 점진적으로 개선할 것이다.

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)  # <1>

    def __getitem__(self, index):
        return self.words[index]  # <2>

    def __len__(self):  # <3>
        return len(self.words)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)  # <4>
```

## Sequence가 iterable인 이유

- 객체 `x`를 순회할 때, `iter(x)`가 호출된다.
- `iter()`의 동작은,
  1. `__iter__`을 호출하여 iterator를 얻는다.
  2. `__iter__`가 구현되어 있지 않다면, `__getitem__`을 찾아서 index 0부터 순회한다.
  3. `__getitem__`이 구현되어 있지 않다면 `TypeError`를 발생시킨다.
- `__getitem__`을 통한 순회는 deprecated 되었다.
  - `__getitem__`만을 구현한 클래스는 `isinstance` 체크 시 `abc.Iterable`의 인스턴스로 확인되지 않는다.
  - `__iter__`을 구현했다면 `abc.Iterable`의 인스턴스로 확인된다.

## Callable을 이용한 `iter` 호출

- `Callable`과 sentinel을 이용해서 `iter(Callable, sentinel)` 방식으로 호출할 수 있다.

```python
>>> def d6():
...     return randint(1, 6)
...
>>> d6_iter = iter(d6, 1)
>>> d6_iter
<callable_iterator object at 0x104e0b790>
>>> for roll in d6_iter:
...     print(roll)
...
5
4
2
```

- `for`문의 순회에서,

  - 매 순회마다 `d6`이 호출되고, 반환값이 sentinel이 아니라면 `roll`에 할당된다.
  - sentinel이라면 순회를 종료한다.

- 다음의 예시에서, EOF가 나올 때까지 `block`을 처리하는 예시이다.

```python
from functools import partial

with open('mydata.db', 'rb') as f:
    read64 = partial(f.read, 64)
    for block in iter(read64, b''):
        process_block(block)
```

- 이 예시에서 `partial` 함수는 반드시 필요한데, `iter`에 들어갈 callable은 인자를 받지 않아야 하기 때문이다.

## Iterable과 Iterator

- iterable이란
  - `iter` 내장 함수를 통해 iterater를 얻을 수 있는 모든 객체
  - `__iter__` 메소드를 구현해서 iterator를 리턴하는 객체는 iterable이다.
  - sequence는 모두 iterable이고, `__getitem__` 메소드를 구현한 객체도 iterable이다.
- python은 iterable에서 iterator를 얻어낸다.

- 다음의 `for`문을 `while`문과 `iter`를 사용하여 교체할 수 있다.

```python
>>> s = 'ABC'
>>> for  char in s:
...     print(char)
...
A
B
C
```

```python
>>> s = 'ABC'
>>> it = iter(s)
>>> while True:
...     try:
...             print(next(it))
...     except StopIteration:
...             del it
...             break
...
A
B
C
```

- `StopIteration`은 iterator가 모두 소모되었을 때 발생하는 신호이다.
  - `for` 문이나 comprehension같은 다른 순회의 경우에 내부적으로 처리된다.
- iteration interface(`collections.abc.Iterabor`)는 다음과 같은 메소드를 가진다.

  - `__next__`: 다음 요소를 리턴한다. 없다면 `StopIteration`을 발생시킨다.
  - `__iter__`: `self`를 리턴한다. iterable이 필요한 자리에 iterator를 넣을 수 있게 한다.
    - 따라서 모든 iterator는 iterable이다.

- `x` 객체가 iterable인지 확인할 때는`isinstance(x, abc.Iterator)`를 사용하면 된다.
- iterator 객체 내부의 요소가 모두 소모되었을 때 재사용하는 방법은 없다.
  - 새로운 iterator 객체를 만들어야 한다.

## Sentence Classes with `__iter__`

### Sentence Take #2: A Classic Iterators

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'

    def __iter__(self):  # <1>
        return SentenceIterator(self.words)  # <2>


class SentenceIterator:

    def __init__(self, words):
        self.words = words  # <3>
        self.index = 0  # <4>

    def __next__(self):
        try:
            word = self.words[self.index]  # <5>
        except IndexError:
            raise StopIteration()  # <6>
        self.index += 1  # <7>
        return word  # <8>

    def __iter__(self):  # <9>
        return self
```

- iterator은 iterable이지만 iterable은 iterator가 아니다.
  - iterable에 `__next__` 함수를 구현해서는 안된다.

### Sentence Take #3: A Generateor Function

```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        for word in self.words:  # <1>
            yield word  # <2>
        # <3>
```

## Generator의 작동 방식

- `yield` 키워드가 본문에 있는 함수는 generator 함수이다.
  - generator 함수가 호출되면 generator 객체가 반환된다.
  - 따라서, generator 함수는 generator 팩토리이다.

```python
>>> def gen_123():
...     yield 1
...     yield 2
...     yield 3
...
>>> gen_123
<function gen_123 at 0x104e3f560>
>>> gen_123()
<generator object gen_123 at 0x104de0d50>
>>> for i in gen_123():
...     print(i)
...
1
2
3
>>> g = gen_123()
>>> next(g)
1
>>> next(g)
2
>>> next(g)
3
>>> next(g)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

```python
>>> def gen_AB():
...     print('start')
...     yield 'A'
...     print('continue')
...     yield 'B'
...     print('end.')
...
>>> for c in gen_AB():
...     print('-->', c)
...
start
--> A
continue
--> B
end.
```

## Lazy Sentence

### Sentence Take#4: Lazy Generator
- Lazy의 반댓말은 Eager이다.
- lazy evaluation과 eager evalution.
- 지금까지의 `Sentence`는 eager evalution으로 값을 가져왔다.
  - eager evalution은 메모리를 더 소모한다.
- lazy evalution으로 값을 가져오는 `Sentence`의 예제
```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text  # <1>

    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'

    def __iter__(self):
        for match in RE_WORD.finditer(self.text):  # <2>
            yield match.group()  # <3>
```
- `finditer`메소드는 `self.text`에서 매칭되는 값을 찾아 `MatchObject` 인스턴스를 내보내는 iterator를 리턴한다.
- `match.group()`는 `MatchObject` 인스턴스에서 매칭된 값을 리턴한다.

## Sentence Take #5: Lazy Generator Expression
```python
import re
import reprlib

RE_WORD = re.compile(r'\w+')


class Sentence:

    def __init__(self, text):
        self.text = text

    def __repr__(self):
        return f'Sentence({reprlib.repr(self.text)})'

    def __iter__(self):
        return (match.group() for match in RE_WORD.finditer(self.text))
```

## 언제 genexp를 쓰는가?
- genexp를 사용했을 때 두 줄 이상이 된다면 generator 함수를 사용하는 것이 좋다.
  - 가독성 면에서 낫다.

## Iterators vs Generators
- iterator는 `__next__` 메소드를 구현한 객체이다.
  - 파이썬에서 대부분의 iterator는 generator이다.
- generator function에는 `__next__` 메소드가 없다.
  - generator function의 반환값인 generator 객체에는 `__next__`가 있다.

## An Arithmetic Progression Generator
- `range` 내장 함수는 정수에 bound된 arithmetic progression이다.
- `range`를 일반화하여 AP를 만들 것.
  - `ArithmeticProgression(begin, step[, end])`
  - `range(start, stop[, step])` 와 순서가 다르다.
```python
class ArithmeticProgression:

    def __init__(self, begin, step, end=None):
        self.begin = begin
        self.step = step
        self.end = end  # None -> "infinite" series

    def __iter__(self):
        result_type = type(self.begin + self.step)
        result = result_type(self.begin)
        forever = self.end is None
        while forever or result < self.end:
            yield result
            result = self.being + self.step * index
```
- 마지막 줄이 `result += self.step`이 아닌 이유는 연속된 float 연산은 오차를 증가시키기 때문이다.
- generator 함수로 더 간단하게 작성할 수 있다.
```python
def aritprog_gen(begin, step, end=None):
    result = type(begin + step)(begin)
    forever = end is None
    index = 0
    while forever or result < end:
        yield result
        index += 1
        result = begin + step * index
```

### Arithmetic Progression with Itertools
- `itertools.count`를 통해 더 간단한 구현이 가능하다.
- `itertools.takewhile`은 다음과 같은 행동을 보여준다.
  - generator를 리턴한다.
```python
>>> gen = itertools.takewhile(lambda n: n < 3, itertools.count(1, .5))
>>> list(gen)
[1, 1.5, 2.0, 2.5]
```
- 다음과 같이 구현한다.
```python
def aritprog_gen(begin, step, end=None):
    first = type(begin + step)(begin)
    ap_gen = itertools.count(first, step)
    if end is None:
        return ap_gen
    return itertools.takewhile(lambda n: n < end, ap_gen)
```

## 표준 라이브러리의 generator 함수
생략...
## Iterable Reducing Functions
생략...

## `yield from`으로 만드는 subgenerator
- 다음의 예시들에서 `gen`은 동일한 결과를 낸다.
```python
>>> def sub_gen():
...     yield 1.1
...     yield 1.2
...
>>> def gen():
...     yield 1
...     for i in sub_gen():
...             yield i
...     yield 2
...
>>> for x in gen():
...     print(x)
...
1
1.1
1.2
2
```
```python
>>> def sub_gen():
...     yield 1.1
...     yield 1.2
...
>>> def gen():
...     yield 1
...     yield from sub_gen()
...     yield 2
...
>>> for x in gen():
...     print(x)
...
1
1.1
1.2
2
```
- `yield from`의 리턴값
```python
>>> def sub_gen():
...     yield 1.1
...     yield 1.2
...     return 'Done!'
...
>>> def gen():
...     yield 1
...     result = yield from sub_gen()
...     print('<--', result)
...     yield 2
...
>>> for x in gen():
...     print(x)
...
1
1.1
1.2
<-- Done!
2
```

## `itertools.chain` 재구현하기
- `for`문을 이용한 구현
```python
>>> def chain(*iterables):
...     for it in iterables:
...             for i in it:
...                     yield i
...
>>> s = 'ABC'
>>> r = range(3)
>>> list(chain(s, r))
['A', 'B', 'C', 0, 1, 2]
```
- `yield from`을 이용한 구현
```python
>>> def chain(*iterables):
...     for i in iterables:
...             yield from i
...
>>> list(chain(s, r))
['A', 'B', 'C', 0, 1, 2]
```
## Traversing Tree
생략

## Generic Iterable Type
- `typing.Iterable[T]`는 `typing.Generator[T]`의 서브타입이다.

## Classic Coroutines
- classic coroutine은 generator의 다른 사용 방식이다.
- tuple이 두 가지 다른 사용 방식과 두 가지 type annotation 방식을 가진 것처럼 generator과 classic coroutine은 다른 맥락에서 사용된다.

- generator은 순회를 위한 데이터를 생산한다.
- coroutine은 데이터의 소비자이다.
- coroutine은 순회와는 관련이 없다.

## Example: Coroutine to Compute a Running Average
```python
from collections.abc import Generator

def averager() -> Generator[float, float, None]:  # <1>
    total = 0.0
    count = 0
    average = 0.0
    while True:  # <2>
        term = yield average  # <3>
        total += term
        count += 1
        average = total/count
```
