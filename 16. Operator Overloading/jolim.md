# Operator Overloading

## 연산자 오버로딩 101

- 함수 호출 (`()`), 속성 접근(`.`), 요소 접근/슬라이스(`[]`)도 연산자지만 여기서 다루지 않음.
- 내장 타입의 연산자는 오버로딩할 수 없다.
- 새로운 연산자를 만들 수 없다.
- 몇 가지 연산자는 오버로딩할 수 없다.
  - `is`, `and`, `or`, `not`

## 단항연산자

- `-`: `__neg__`에 의해 정의됨
- `+`: `__pos__`에 의해 정의됨
  - 대개의 경우 `x == +x`이지만 예외적인 경우가 있음.
    - 새로운 인스턴스를 만드는 과정 때문에 예외적인 경우가 생긴다.
- `~`: `__invert__`에 의해 정의됨
  - 비트 연산자 not.
- `abs`를 단항연산자라고 보기도 한다.
- 단항연산자는 `self`만을 인자로 받는다.

## `+` 오버로딩하기

- `a + b` 연산은 `a.__add__`을 호출한다.
  - `a.__add__`가 정의되어 있지 않거나 `a.__add__`가 NotImplemented를 반환하면 `b.__radd__`를 호출한다.
    - `b.__radd__`가 없거나 `NotImplemented`를 반환하면 `TypeError`가 발생한다.
- 의도되지 않은 연산 시도가 있다면 `NotImplemented`를 반환해야 올바른 fallback 함수가 실행되고 TypeError가 반환된다.
  - `NotImplemented`가 아니라 그냥 예외가 던져지게 놔둔다면 에러가 왜 났는지 파악하기 힘들다.
- `__radd__`는 보통 다음과 같이 정의되어 `__add__`를 호출하게 된다..

```python
def __radd__(self, other):
    return self + other
```

- 같은 타입이나 제한된 타입만 연산되게 하고싶다면 다음과 같이 한다.

```python
def __matmul__(self, other):
    if (isinstance(other, abc.Sized) and
        isinstance(other, abc.Iterable)):
        if len(self) == len(other):
            return sum(a * b for a, b in zip(self, other))
        else:
            raise ValueError('@ requires vectors of equal length')
    else:
        return NotImplemented

def __rmatmul(self, other):
    return self @ other
```

## Augmented Assignment Operators
- `__iadd__`가 구현되어 있지 않다면, `a += b` 는 `a = a + b`와 같다.
- in-place 연산이 필요하다면 in-place operator를 오버로딩 해야한다.
  - immutable에서 구현되어서는 안 된다.

## Soapbox
- 연산자 오버로딩은 많이 남용된다. 잘 생각해보고 쓰자.


### 유의점
- 단항연산자와 이항연산자를 오버로딩하는 경우, self를 변경해서는 안된다.
- augmented assignment의 경우에만 self가 변경될 수 있다.
- `NotImplemented`는 싱글톤 객체이다. `NotImplementedError`와는 다르다.
```
