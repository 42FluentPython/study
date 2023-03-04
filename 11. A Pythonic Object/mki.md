Pythonic을 강조하는 Martijin Faassen

Python Data Model을 사용하면 User Define Type이 Built-in Types처럼 동작합니다.

이번 장에서 real Python object처럼 동작하는 user-defined class를 만들겁니다. 실제 application을 만들때는 이번 장에서처럼 많은 special method를 만들지는 않지만, library나 framework를 개발해야 한다면 많은 special method를 만들어야 하며, 우리가 만든 class를 사용하는 python 사용자는 python이 제공하는 class처럼 동작하길 원할겁니다. 이러한 기대를 만족시키는 class를 만드는것이 Pythonic의 한 갈래입니다.

1장에서 몇몇 special method를 작성하는 방법을 보여드렸는데, 이번 장에서 다른 special method도 보여줄 겁니다.

1. 다른 타입으로 객체를 변환하는 내장 함수를 지원하기 위해, **`repr()`**, **`bytes()`**, **`complex()`**와 같은 내장 함수를 구현합니다.
2. 클래스 메소드를 이용하여 대체 생성자를 구현할 수 있습니다. 대체 생성자는 다른 매개 변수나 초기화 방법을 제공하며, 이를 이용하여 객체를 생성할 수 있습니다.
3. f-strings, **`format()`** 내장 함수, **`str.format()`** 메서드에서 사용되는 포맷 미니 언어를 확장하여 사용자 정의 객체를 포맷팅할 수 있도록 지원할 수 있습니다.
4. 속성을 읽기 전용으로 만들어 객체 내부의 상태를 안정적으로 유지할 수 있습니다.
5. **`__hash__()`** 메서드를 구현하여, 객체를 해시 가능한 객체로 만들어, 집합(set)이나 딕셔너리(dict)의 키로 사용할 수 있습니다.
6. **`__slots__`** 특수 속성을 이용하여 메모리를 절약할 수 있습니다. 이를 이용하면 객체가 일반적으로 사용하는 **`__dict__`** 속성 대신에 고정된 속성만을 가지도록 구현할 수 있습니다. 이를 통해 메모리를 절약하고, 객체의 생성과 속성 접근 속도를 빠르게 할 수 있습니다.

이 전에 Vector2d를 만들었는데 Chapter 12에서 N-dimentional vector를 만들겁니다.

또 @classmethod, @staticmethod decorator를 언제 사용할 지도 알아보고, Private와 Protected에 대해서도 알아봅시다.

## Duck Typing

Duck typing은 동적 타이핑 언어에서 사용되는 프로그래밍 개념 중 하나입니다. 이 개념은 "오리처럼 걷고, 오리처럼 울면, 그것은 오리다"라는 유명한 명언에서 비롯되었습니다.

즉, Duck typing은 객체의 타입보다는 객체가 가지고 있는 메서드나 속성의 존재 여부에 따라 객체를 판단하는 방식입니다. 이는 객체의 타입을 미리 선언하지 않아도 되는 동적 타이핑 언어에서 유용하게 사용됩니다.

예를 들어, "walk like a duck, quack like a duck"라는 표현은 오리와 관련된 특징을 가진 객체에 대해 사용됩니다. 즉, 객체가 오리처럼 걷고 울면, 그 객체는 오리로 간주됩니다. 이 개념은 객체의 타입이나 상속 관계보다는 객체의 행동에 더 중점을 둡니다.

따라서 Duck typing을 이용하면 객체의 타입을 명시적으로 선언하지 않아도 되며, 객체의 메서드나 속성이 동적으로 변경될 수 있는 상황에서도 유연하게 대처할 수 있습니다. 이는 Python과 같은 동적 타이핑 언어에서 매우 유용하게 사용됩니다.

# What’s New in This Chapter

Pythonic을 더 강조했습니다.

f-strings를 추가했습니다.

object representation method를 알아봅시다.

# Object Representations

OOP Language는 적어도 한 가지 이상의 string 표현 방법을 제공합니다. Python은 2가지 방식이 있습니다.

- repr(), __repr__
- str(), __str__

추가로 2개 더 있습니다.

- bytes(), __bytes__
- format(), f-string, __format__

# Vector Class Redux

- Built-in eval(): “string”을 실행
- Built-in ord(): 문자 → ASCII
- math.hypot() → 빗변 계산

# An Alternative Constructor

```python
@classmethod
def frombytes(cls, octets):
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(*memv)
```

# classmethod Versus staticmethod

classmethod는 class내부의 class variable을 변경할 수 있고, staticmethod는 변경할 수 없도록 설계되었습니다.

# Formatted Displays

{ field_name : formatting specifier }

format() → str.format() → f-strings

custom __format__ 은 신기하다.

# A Hashable Vector2d

private variable __x

getter가 있으면 __hash__를 만들 수 있다.

# Supporting Positional Pattern Matching

__match_args__는 또 뭐지…

# Complete Listing of Vector2d, Version 3

# Private and “Protected” Attributes in Python

mangling

double underscore는 python에서 의미를 갖지만, underscore는 의미를 갖지 않아서 암묵적으로 protected라고 생각하면 된다.

# Saving Memory with __slots__

__slots__는 정의를 class 바로 다음에 해야 합니다!

## Simple Measure of __slot__ Savings

## Summarizing the Issues with __slots__

# Overriding Class Attributes

# Chapter Summary

# Further Reading