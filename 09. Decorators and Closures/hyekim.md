## **Decorators 101**

- A decorator is a function or another callable.
- A decorator may replace the decorated function with a different one.
- Decorators are executed immediately when a module is loaded.

## **When Python Executes Decorators**

**function decorators are executed as soon as the module is imported, but the decorated functions only run when they are explicitly invoked.**

데코레이터는 모듈이 임포트 되었을 때 실행되고, 장식된 함수(inner function)은 함수가 호출되었을 때 실행된다.

이것은 임포트 타임, 런타임 간의 차이를 강조한다

## **Variable Scope Rules**

```python
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
..?
```

***global scope***

***local scope***

nonlocal scope

## **functools.wraps**

데코레이터에서 kwargs 까지 다루려면 functools.wraps를 써야함

## 유용한 데코레이터

**`@lru_cache(), @cache` 인메모리 캐시 구현**

**`@singledispatch` 함수 오버로딩에 쓰임**

## 함수형 데코레이터, 클래스형 데코레이터 비교

```python
import time

DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

## 함수형
def clock(fmt=DEFAULT_FMT):  1
    def decorate(func):      2
        def clocked(*_args): 3
            t0 = time.perf_counter()
            _result = func(*_args)  4
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)  5
            result = repr(_result)  6
            print(fmt.format(**locals()))  7
            return _result  8
        return clocked  9
    return decorate  10

## 클래스형
class clock:  1

    def __init__(self, fmt=DEFAULT_FMT):  2
        self.fmt = fmt

    def __call__(self, func):  3
        def clocked(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)  4
            elapsed = time.perf_counter() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(self.fmt.format(**locals()))
            return _result
        return clocked
```

무엇이 더 좋은가?
