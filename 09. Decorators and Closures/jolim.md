# Decorators and Closures
- 데코레이터는 함수에 '표시'를 함으로써 함수의 행동을 어떤 방향으로든 향상시킨다.
- 데코레이터의 작동을 이해하기 위해서는 클로저를 이해하는 것이 필수이다.
- 클로저는 데코레이터, 콜백 프로그래밍, 특정한 함수형 프로그래밍에 필요하다.
- `nonlocal` 키워드를 이해하는 것도 필요하다.
- 데코레이터를 배우기 전에 먼저 배울 것은
  - 파이썬 인터프리터가 어떻게 데코레이터 문법을 처리하는지
  - 변수가 지역 변수인지 아닌지 인터프리터가 어떻게 판단하는지
  - 클로저가 존재하는 이유와 그 동작
  - `nonlocal`로 해결할 수 있는 문제
- 데코레이터에 관해서는
  - 제대로 작동하는 데코레이터 구현하기
  - 표준 라이브러리에 존재하는 강력한 데코레이터들, `@cache`, `@lru_cache`, `@singledispatch`
  - 매개변수화된 데코레이터 구현하기

## Decorator 101
- 데코레이터는 함수(decorated function)를 인자로 받는 callable이다.
- 데코레이터는 함수에 처리를 한 후 그대로 리턴하거나 다른 함수, 또는 어떤 callable 객체를 리턴한다.
- `decorate`라는 데코레이터에 대해 다음은 동일하게 결과를 가진다.
```python
@decorate
def target():
    print('running target()')
```
```python
def target():
    print('running target()')

target = decorate(target)
```
- `target`은 `decorate(target)`에 의해 대체된다.
- 데코레이터는 문법적 설탕일 뿐이다.
- 데코레이터를 함수처럼 호출하는 것은 가능하다. 또한 메타프로그래밍을 할 때에 편리하다.

## 파이썬은 데코레이터를 언제 실행하는가?
- 데코레이터 함수는 대상 함수가 정의된 직후에 실행된다.
  - 일반적으로 임포트 타임 == 파이썬이 모듈을 적재할 때 실행된다.
```python
registry = []


def register(func):
    print(f"running register({func})")
    registry.append(func)
    return func


@register
def f1():
    print("running f1()")


@register
def f2():
    print("running f2()")


def f3():
    print("running f3()")


def main():
    print("running main()")
    print("registry ->", registry)
    f1()
    f2()
    f3()


if __name__ == "__main__":
    main()
```
- 위의 파이썬 파일(registration.py)을 실행하면 다음과 같은 결과를 얻는다.
```
running register(<function f1 at 0x105541bd0>)
running register(<function f2 at 0x105541c60>)
running main()
registry -> [<function f1 at 0x105541bd0>, <function f2 at 0x105541c60>]
running f1()
running f2()
running f3()
```
- 임포트 했을 시 다음과 같은 결과를 얻는다.
```python
>>> import registration
running register(<function f1 at 0x1018ec940>)
running register(<function f2 at 0x1018ec9d0>)
```
- 즉, 데코레이터는 임포트 즉시 실행되지만, 대상 함수는 호출될 때까지 실행되지 않는다.
- 임포트 타임과 런타임의 차이를 잘 알아두도록 하자.

## 등록 데코레이터
- 데코레이터는 일반적으로 대상 함수와 같은 모듈에서 정의되지 않는다.
- `register` 데코레이터는 대상 함수를 그대로 리턴하였다. 일반적으로는 내부 함수를 정의하고 리턴한다.
- 그러나 `register` 데코레이터같은 기술은 여러 프레임워크에서 함수를 중심 기록부에 추가하는데 이용된다.
  - Chapter 10에서 다룰 것.

## 변수 스코프 규칙
```python
>>> def f1(a):
...     print(a)
...     print(b)
... 
>>> f1(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f1
NameError: name 'b' is not defined
```
- `b`가 지역에도 정의되어있지 않고 전역에도 정의되어있지 않기 때문에 오류가 났다.
```python
>>> b = 6
>>> f1(3)
3
6
```
- 전역에 정의해주면 오류가 생기지 않는다.
```python
>>> b=6
>>> f1(3)
3
6
>>> b = 6
>>> def f2(a):
...     print(a)
...     print(b)
...     b = 9
... 
>>> f2(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f2
UnboundLocalError: local variable 'b' referenced before assignment
```
- 지역에 `b`를 선언해주었는데, 호출 이후에 선언했는데도 `UnboundLocalError`가 발생하였다.
- 파이썬이 함수 본문을 컴파일 할 때 `b`가 함수 내부에서 할당되었기 때문에 `b`를 지역 변수로 인식한다.
  - 이는 의도된 것이다.
  - 파이썬은 변수를 선언할 것을 요구하지 않는다. 다만 함수 내부에서 할당된 변수를 지역으로 본다.
    - 자바스크립트 역시 변수 선언을 요구하지 않지만 함수 내부에서 실수로 변수 선언을 하지 않으면 전역 변수가 되어버린다.
- `global` 키워드를 이용하면 다음과 같이 사용할 수 있다.
```python
>>> def f3(a):
...     global b
...     print(a)
...     print(b)
...     b = 9
... 
>>> b = 6
>>> f3(3)
3
6
>>> b
9
```
- 참고로, `global` 키워드는 변수를 '선언'하지 않는다.
```python
>>> def f1(a):
...     global b
...     print(a)
...     print(b)
...     b = 9
... 
>>> f1(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 4, in f1
NameError: name 'b' is not defined
```
- 위의 예시는 두 범위를 보여준다.
  - 모듈 전역 범위
  - 함수 지역 범위
- 또다른 변수 범위가 하나 더 있다.
  - 비지역 범위라고 부른다.
  - 클로저에 중요한 개념이다.

## 클로저
- 많은 블로거들이 익명 함수와 클로저를 헷갈린다.
  - 익명 함수없이 함수 내부에서 함수를 정의하는 것은 흔하지 않고 불편하다.
  - 클로저는 함수 내부에서 함수를 정의할 때만 중요하다.
- 클로저란,
  - 전역 변수도 아니고 클로저의 지역 변수도 아닌 확장된 범위를 포괄하는 함수를 말한다.
- 다음과 같은 함수(또는 callable)이 있다고 해보자.
```python
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11
```
- 함수에 값이 추가될 때마다 누적하여 평균을 계산하는 함수이다.
- 클래스로 다음과 같이 구현할 수 있다.
```python
class Averager:
    def __init__(self):
        self.series = []

    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return total / len(self.series)
```
- 인스턴스 속성에 값이 저장된다.
```python
>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```
- 함수로는 다음과 같이 구현할 수 있다.
```python
def make_averager():
    series = []
    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total / len(series)
    return averager
```
```python
>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```
- 이 함수는 `series`를 `make_averager`의 지역 범위에서 찾아온다.
  - `avg`가 호출될 때, `make_averager`는 이미 리턴되었고, 지역 범위는 종료되었다.
  - 이 `series`를 자유 변수(free variable)이라고 부른다.
    - 지역 범위에 묶이지 않은 변수를 뜻한다.
- `__code__` 속성은 컴파일된 본문을 반영한다.
  - 지역 변수와 자유 변수가 속해있다.
```python
>>> avg.__code__.co_varnames
('new_value', 'total')
>>> avg.__code__.co_freevars
('series',)
```
- `__closure__` 속성에 `series`가 담겨있다.
  - `__closure__` 안의 각 항목은 `avg.__code__.co_freevars`안에 담긴 이름에 해당한다.
  - 이 항목들은 `cells`라고 불리며, `cell_contents`라는 속성이 있어서 그곳에서 값을 직접 찾을 수 있다.
```python
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__
(<cell at 0x103632ef0: list object at 0x103733f40>,)
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
>>> avg.__closure__[0].cell_contents.append(13)
>>> avg(14)
12.0
```
- 위의 closure는 `nonlocal`을 사용하지 않아도 되는 경우이다.`

### 비지역 선언
- `make_average`는 비효율적인 면이 있다.
  - 평균을 구하기 위해 모든 값을 저장해야하는 것은 아니기 때문.
- 다음은 틀린 구현이다.
```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        count += 1
        total += new_value
        return total / count

    return averager
```
```python
>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/jolim/study/python/fluent_python/playground/broken_averager.py", line 6, in averager
    count += 1
UnboundLocalError: local variable 'count' referenced before assignment
```
- 이는 `count += 1`에서, `count`에 담긴 값이 immutable이기 때문에 `count = count + 1`로 평가된다.
  - 즉, 지역 변수가 된다.
  - `total`에도 똑같은 문제가 적용된다.
- 앞서 `series`를 사용했던 함수에 문제가 없었던 이유는 지역 범위에서 `series`에 할당을 하지 않았기 때문이다.
- `nonlocal` 키워드를 사용하면 함수를 자유 변수로 만들 수가 있다.
- 다음은 정상 작동한다.
```python
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count

    return averager
```

### 변수 찾기 로직
- 파이썬 인터프리터는 다음과 같은 순서로 변수를 찾는다.
  - `global x` 선언이 있다면, x를 모듈 전역에서 찾는다.
    - 파이썬은 프로그램 전역이 없다.
  - `nonlocal x` 선언이 있다면, 가장 가까이 둘러싼 함수의 지역에서 x가 정의된 곳을 찾는다.
    - 전역에서는 찾지 않는다.
  - `x`가 매개변수이거나 함수 본문에서 할당되었다면 `x`는 지역 변수이다.
  - `x`가 참조되었으나 함수의 매개변수도 아니고 본문에서 할당되지 않았다면
    - 둘러싼 함수들의 본문에서 차례로 `x`를 찾는다.
    - 함수들의 본문에서 `x`를 찾지 못했다면 모듈 전역에서 찾는다.
    - 모듈 전역에서 찾지 못했다면 `__builtins__.__dict__`에서 찾는다.

## 간단한 데코레이터 구현하기
- 다음은 함수의 실행 시간을 측정하는 데코레이터이다.
```python
import time


def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ", ".join(repr(arg) for arg in args)
        print(f"[{elapsed:0.8f}s] {name}({arg_str}) -> {result}")
        return result

    return clocked
```
- `result = func(*args)`에서 `func`는 자유 변수이다.
```python
import time
from clockdeco0 import clock


@clock
def snooze(seconds):
    time.sleep(seconds)


@clock
def factorial(n):
    return 1 if n < 2 else n * factorial(n - 1)


if __name__ == "__main__":
    print("*" * 40, "Calling snooze(.123)")
    snooze(0.123)
    print("*" * 40, "Calling factorial(6)")
    print("6! =", factorial(6))

```
```
**************************************** Calling snooze(.123)
[0.12808525s] snooze(0.123) -> None
**************************************** Calling factorial(6)
[0.00000033s] factorial(1) -> 1
[0.00000842s] factorial(2) -> 2
[0.00001254s] factorial(3) -> 6
[0.00001621s] factorial(4) -> 24
[0.00002029s] factorial(5) -> 120
[0.00002650s] factorial(6) -> 720
6! = 720
```

### 키워드 인자 포함시키기, 그리고 `functools.wraps`
- 위의 `clock` 함수에는 문제가 있다.
  - 키워드 인자를 넣을 수 없다
  - `__name__`과 `__doc__`이 덮어씌워진다.
- `functools.wraps`로 두 번째 문제를 해결할 수 있다.
```python
import time
import functools


def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_list = []
        if args:
            arg_list.append(", ".join(repr(arg) for arg in args))
        if kwargs:
            pairs = ["%s=%r" % (k, w) for k, w in sorted(kwargs.items())]
            arg_list.append(", ".join(pairs))
        arg_str = ", ".join(arg_list)
        print(f"[{elapsed:0.8f}s] {name}({arg_str}) -> {result}")
        return result

    return clocked
```

## 표준 라이브러리의 데코레이터
- 파이썬에는 3개의 내장 데코레이터가 있다.
  - `property`, `classmethod`, `staticmethod`
- 여기서는 `functools`의 데코레이터를 다룬다.

### `functools.cache`
- 일반적인 경우
```python
from clockdeco import clock
@clock
def fibonacci(n):
  if n < 2:
      return n
  return fibonacci(n - 2) + fibonacci(n - 1)

if __name__ == "__main__":
  print(fibonacci(5))
```
- 실행 결과
```
[0.00000025s] fibonacci(1) -> 1
[0.00000017s] fibonacci(0) -> 0
[0.00000013s] fibonacci(1) -> 1
[0.00000625s] fibonacci(2) -> 1
[0.00002337s] fibonacci(3) -> 2
[0.00000012s] fibonacci(0) -> 0
[0.00000017s] fibonacci(1) -> 1
[0.00000438s] fibonacci(2) -> 1
[0.00000012s] fibonacci(1) -> 1
[0.00000017s] fibonacci(0) -> 0
[0.00000017s] fibonacci(1) -> 1
[0.00000408s] fibonacci(2) -> 1
[0.00000804s] fibonacci(3) -> 2
[0.00001629s] fibonacci(4) -> 3
[0.00004379s] fibonacci(5) -> 5
5
```
- 너무 많이 반복 호출된다.
```python
import functools

from clockdeco import clock

@functools.cache
@clock
def fibonacci(n):
  if n < 2:
      return n
  return fibonacci(n - 2) + fibonacci(n - 1)

if __name__ == "__main__":
  print(fibonacci(5))
```
- `functools.cache`를 쓴 예시
```python
from clockdeco import clock
import functools


@functools.cache
@clock
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)


if __name__ == "__main__":
    print(fibonacci(5))
```
```
[0.00000029s] fibonacci(1) -> 1
[0.00000021s] fibonacci(0) -> 0
[0.00000417s] fibonacci(2) -> 1
[0.00002129s] fibonacci(3) -> 2
[0.00000033s] fibonacci(4) -> 3
[0.00002638s] fibonacci(5) -> 5
5
```
- 실행 결과가 메모이제이션 되면서 호출 횟수가 줄었다.

### 겹쳐진 데코레이터
- 다음 둘은 동등하다.
```python
@alpha
@beta
def my_func():
    ...
```
```python
def my_func():
    ...
my_func = alpha(beta(my_func))
```
- `functools.cache`는 원격 API 호출의 경우나 
- 캐시할 양이 많아질 경우 메모리 소모가 심할 수 있다.
  - 짧은 프로그램의 경우 `functools.cache`를 사용할 수 있다.
  - 긴 프로그램의 경우 적절한 `maxsize` 매개변수와 함께 `functools.lru_cache`를 사용하는 것이 낫다.

## `functools.lru_cache`
- `functools.cache`는 더 오래된 `functools.lru_cache`의 단순한 버전이다.
- LRU는 Least Recently Used를 뜻한다.
- `functools.lru_cache`를 사용하는 방법은 두 가지가 있다.
```python
@lru_cache
def costly_function(a, b):
    ...
```
```python
@lru_cache()
def costly_function(a, b):
    ...
```
- 두 경우 기본 인자가 사용된다.
  - `maxsize=128`
    - `maxsize`는 저장될 함수값의 개수이다.
    - `maxsize`에 도달하면, 가장 오래 전에 사용된 값부터 삭제된다.
    - 2의 제곱으로 설정하는 것이 최적으로 사용할 수 있다.
    - `maxsize=None`으로 설정하면 제한이 사라진다.
      - 이는 `functools.cache`와 동일하다.
      - 모든 메모리를 사용할 때까지 저장할 것이다.
  - `typed=False`
    - 다른 타입의 인자를 받았을 때 다른 결과로 저장할 것인지
    - `f(1)`과 `f(1.0)`을 같게 인식하느냐 다르게 인식하느냐의 차이

## 단일 디스패치 제너릭 함수
- `singledistpach`를 이용하는 것은 함수 오버로딩과 비슷하다.
- 첫 번째 인자의 타입에 따라 다른 함수가 사용되게 할 수 있다.
- `singledispatch`를 붙인 함수는 기본 함수가 되고, 제너릭 함수의 진입점이 된다.
- 'single'의 의미는 하나의 인자의 타입에 따라 여러 함수가 사용된다는 의미이다.
  - 여러 인자의 타입에 따라 함수가 결정될 때는 'mutiple dispatch'라고 한다.
```python
from functools import singledispatch
from collections import abc
import fractions
import decimal
import html
import numbers


@singledispatch  # <1>
def htmlize(obj: object) -> str:
    content = html.escape(repr(obj))
    return f"<pre>{content}</pre>"


@htmlize.register  # <2>
def _(text: str) -> str:  # <3>
    content = html.escape(text).replace("\n", "<br/>\n")
    return f"<p>{content}</p>"


@htmlize.register  # <4>
def _(seq: abc.Sequence) -> str:
    inner = "</li>\n<li>".join(htmlize(item) for item in seq)
    return "<ul>\n<li>" + inner + "</li>\n</ul>"


@htmlize.register  # <5>
def _(n: numbers.Integral) -> str:
    return f"<pre>{n} (0x{n:x})</pre>"


@htmlize.register  # <6>
def _(n: bool) -> str:
    return f"<pre>{n}</pre>"


@htmlize.register(fractions.Fraction)  # <7>
def _(x) -> str:
    frac = fractions.Fraction(x)
    return f"<pre>{frac.numerator}/{frac.denominator}</pre>"


@htmlize.register(decimal.Decimal)  # <8>
@htmlize.register(float)
def _(x) -> str:
    frac = fractions.Fraction(x).limit_denominator()
    return f"<pre>{x} ({frac.numerator}/{frac.denominator})</pre>"
```

```python
@singledispatch  # <1>
def htmlize(obj: object) -> str:
    content = html.escape(repr(obj))
    return f"<pre>{content}</pre>"
```
- <1>의 경우
- 기반 함수를 `@singledispatch`로 감싸서 `object` 타입을 처리하게 한다.
```python
@htmlize.register  # <2>
def _(text: str) -> str:  # <3>
    content = html.escape(text).replace("\n", "<br/>\n")
    return f"<p>{content}</p>"
```
- <2>의 경우
  - 각 특이적 함수는 `@<base_name>.register`로 처리된다.
- <3>의 경우
  - 첫 번째 인자의 런타임 타입이 어떤 함수가 불릴지를 결정한다.
  - 이 특이적 함수의 이름은 상관 없다. 따라서 `_`를 쓰는 것이 적절하다.
    - `Mypy`는 여러 함수가 같은 이름을 가지면 경고를 띄운다. 무시하자.

```python
@htmlize.register  # <4>
def _(seq: abc.Sequence) -> str:
    inner = "</li>\n<li>".join(htmlize(item) for item in seq)
    return "<ul>\n<li>" + inner + "</li>\n</ul>"
```
- <4>의 경우
  - 또다른 타입에 특이적 처리를 하려면 새로운 함수를 맞는 타입으로 등록해주면 된다.

```python
@htmlize.register  # <5>
def _(n: numbers.Integral) -> str:
    return f"<pre>{n} (0x{n:x})</pre>"
```
- <5>의 경우
  - `numbers` ABC는 다른데 사용하는 것이 적절하지 않지만, single dispatch에서 유용하다.
  - 여러 라이브러리에 구현된 숫자 타입을 매칭할 수 있기 때문.

```python
@htmlize.register  # <6>
def _(n: bool) -> str:
    return f"<pre>{n}</pre>"
```
- <6>의 경우
  - `bool`은 `Numbers.Integral`의 하위 타입이지만, `singledispatch`는 가장 구체적인 타입을 찾아서 실행해준다.

```python
@htmlize.register(fractions.Fraction)  # <7>
def _(x) -> str:
    frac = fractions.Fraction(x)
    return f"<pre>{frac.numerator}/{frac.denominator}</pre>"
```
- <7>의 경우에
  - 대상 함수에 타입 힌트를 사용할 수 없거나 사용하기 싫을 때 `@<base_name>.register`의 인자로 타입을 넣어줄 수 있다.
  - 3.4 이후에 적용된다.

```python
@htmlize.register(decimal.Decimal)  # <8>
@htmlize.register(float)
def _(x) -> str:
    frac = fractions.Fraction(x).limit_denominator()
    return f"<pre>{x} ({frac.numerator}/{frac.denominator})</pre>"
```
- <8>의 경우에
  - `@<base_name>.register` 데코레이터는 대상 함수를 그대로 리턴하기 때문에 데코레이터를 여러 개 적용할 수 있다.

- `singledispatch`의 장점은
  - 모듈에 정의된 함수에 유저 정의 타입에 해당하는 함수를 추가할 수 있다는 것이다.
  - 내가 정의하지 않은 클래스에 대해서도 작성할 수 있다.
  - Java 스타일의 오버로딩이나 함수 내의 `if/elif/elif`로는 불가능한 것이다.
- [PEP 443 - Single-dispatch generic functions](https://fpy.li/pep443)


## 매개변수화된 데코레이터
- 매개변수화된 데코레이터는 데코레이터 팩토리를 뜻한다.
  - 데코레이터 팩토리는 매개변수를 받아 데코레이터를 반환한다.
  - 다음은 동일하다.
  ```python
  @deco(param1)
  def my_fn():
      ...
  ```
  ```python
  def my_fn():
      ...
  deco(param1)(my_fn)
  ```
### 매개변수화된 `register` 데코레이터
- 위위위의 예제 `register` 데코레이터를 매개변수화된 데코레이터로 변경시키자.
```python
registry = set()


def register(active=True):
    def decorate(func):
        print(f"running register(active={active}) -> decorate({func})")
        if active:
            registry.add(func)
        else:
            registry.discard(func)

        return func

    return decorate


@register(active=False)
def f1():
    print("running f1()")


@register()
def f2():
    print("running f2()")


def f3():
    print("running f3()")


def main():
    print("running main()")
    print("registry ->", registry)
    f1()
    f2()
    f3()


if __name__ == "__main__":
    main()
```
```
running register(active=False) -> decorate(<function f1 at 0x103399c60>)
running register(active=True) -> decorate(<function f2 at 0x103399cf0>)
running main()
registry -> {<function f2 at 0x103399cf0>}
running f1()
running f2()
running f3()
```

## 매개변수화된 `clock` 데코레이터
```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):  # <1>
    def decorate(func):      # <2>
        def clocked(*_args): # <3>
            t0 = time.perf_counter()
            _result = func(*_args)  # <4>
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)  # <5>
            result = repr(_result)  # <6>
            print(fmt.format(**locals()))  # <7>
            return _result  # <8>
        return clocked  # <9>
    return decorate  # <10>

if __name__ == '__main__':

    @clock()  # <11>
    def snooze(seconds):
        time.sleep(seconds)

    for i in range(3):
        snooze(.123)
```
```
[0.12804362s] snooze(0.123) -> None
[0.12478604s] snooze(0.123) -> None
[0.12621654s] snooze(0.123) -> None
```
- `locals` 내장 함수는 지역 변수와 그 값을 `dict`로 리턴한다.
- 데코레이터 팩토리는 클래스로 구현하는 것이 더 나을 때가 많다.
```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

class clock:  # <1>

    def __init__(self, fmt=DEFAULT_FMT):  # <2>
        self.fmt = fmt

    def __call__(self, func):  # <3>
        def clocked(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)  # <4>
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(self.fmt.format(**locals()))
            return _result
        return clocked
# end::CLOCKDECO_CLS[]

if __name__ == '__main__':

    @clock()
    def snooze(seconds):
        time.sleep(seconds)

    for i in range(3):
        snooze(.123)
```
```python
import time

DEFAULT_FMT = "[{elapsed:0.8f}s] {name}({args}) -> {result}"


class clock:  # <1>
    def __init__(self, fmt=DEFAULT_FMT):  # <2>
        self.fmt = fmt

    def __call__(self, func):  # <3>
        def clocked(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)  # <4>
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ", ".join(repr(arg) for arg in _args)
            result = repr(_result)
            print(self.fmt.format(**locals()))
            return _result

        return clocked


if __name__ == "__main__":

    @clock()
    def snooze(seconds):
        time.sleep(seconds)

    for i in range(3):
        snooze(0.123)
```
- decorate 함수를 만드는 부분과 내부 클로저를 다루는 부분을 나눌 수 있다.

## 더 읽기
- `functools.wrap`함수를 항상 쓰자