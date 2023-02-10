# 8. Type Hints in Functions

Type Hint는 Python 2.2의 Type Unification이후로 가장 큰 변화입니다. 하지만 여전히 Python은 Type Hint를 Optional로 두고 있습니다.

PEP 484에서 syntax와 semantic을 설명하고 있습니다. Type Hint의 목적은 Developer들이 Debugging을 더 잘 하기 위해서 입니다.

Python을 사용하는 사용자들의 범주는 매우 넓기 때문에 Type Hint를 강제하지 않습니다. static type, subtyping, generics를 배우는 시간이 너무 오래 걸리기 때문입니다.

- A hands-on introduction to gradual typing with Mypy
- The complementary perspectives of duck typing and nominal typing
- Overview of the main categories of types that can appear in annotations—this is
about 60% of the chapter
- Type hinting variadic parameters (*args, **kwargs)
- Limitations and downsides of type hints and static typing

# What’s New in This Chapter

이번 Chapter는 1st Edition에는 없었습니다. 왜냐하면 Type Hint가 Python 3.5에서 등장했기 때문입니다.

static type system의 한계에서 PEP 484에서는 점진적 type system에 대한 이야기가 나오기 시작했습니다.

# About Gradual Typing

PEP 484에서 gradual(점진적) type system에 대해 이야기하기 시작했는데, MicroSoft의 TypeScript와 같은 역할을 Mypy가 수행하고 있습니다.

gradual type system은 아래와 같은 특징을 지니고 있습니다.

## Is optional

## Does not catch type errors at runtime

## Does not enhance performance

# Gradual Typing in Practice

세상에는 많은 Python Type Checker가 있습니다. Google의 pytype, Microsoft의 pyright, Facebook의 Pyre, IDE에 내장된 것들도 있습니다. 하지만 이 책에서는 Mypy를 사용합니다.

## Starting with Mypy

```python
pip install mypy
pip install pytest
```

## Making Mypy More Strict

```python
mypy --disallow-untyped-defs messages_test.py 

messages.py:14: error: Function is missing a type annotation  [no-untyped-def]
messages_test.py:9: error: Function is missing a type annotation  [no-untyped-def]
messages_test.py:13: error: Function is missing a return type annotation  [no-untyped-def]
messages_test.py:13: note: Use "-> None" if function does not return a value
Found 3 errors in 2 files (checked 1 source file)
```

options

```python
mypy --disallow-untyped-defs
mypy --disallow-incomplete-defs
```

options을 일일히 넣지 말고, Mypy configuration file을 사용하면 편리합니다.

function의 return type을 지정해줘야 Mypy가 검사합니다!!!

## A Default Parameter Value

```python
def hex2rgb(color=str) -> tuple[int, int, int]:
```

type hint와 parameter default가 비슷해서 실수하기 쉽습니다. 따라서 규칙을 정해야 합니다.

- code style checker: flake8
- code formatter: blue, black
- type checker: mypy
- tester: pytest, doctest

## Using None as a Default

```python
from typing import Optional

def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
```

typing.Optional은 단지 argument가 str or None이라는 뜻입니다.

# Types Are Defined by Supported Operations

Type은 Type이 수행할 수 있는 Operations에 의해 정해집니다.

```python
def double(x):
    return x * 2
```

## Duck typing

Smalltalk에서 채택된 Duck typing은 오직 runtime에만 관심있습니다. 또한, Object의 type이 무엇인지는 관심이 없고, 어떤 operator을 지원하는지만 관심있습니다. Object만 type이 있고, variable은 type이 없습니다.

## Nominal typing

C++, Java, C#의 관점에서의 typing은 runtime이 아닌, source code에만 관심있는 typing입니다. Object와 Variable 모두 Type이 있습니다. type checker는 source code를 실행하지 않고 읽기만 합니다.

간단한 예를 통해서 duck typing은 flexible하지만, 예상치 못한 문제를 야기합니다.

# Types Usable in Annotations

## The Any Type

Any는 모든 Type을 포함합니다.

Liskov Substitution Principle—LSP

## Simple Types and Classes

Simple Type이나, ABC

consistent-with, subtype-of

int → float → complex

## Optional and Union Types

```python
>>> from typing import Optional
>>> def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
...     pass
```

Optional은 사실 Union의 shortcut입니다.

```python
>>> from typing import Union
>>> def show_count(count: int, singular: str, plural: Union[str, None]) -> str:
...     pass
```

Python 3.10부터 | operator를 사용할 수 있습니다.

```python
>>> def show_count(count: int, singular: str, plural: str | None = None) -> str:
...    pass
```

Union의 간단한 예는 built-in function중에 ord입니다. ord는 Unicode를 int로 변환해줍니다.

```python
def ord(c: Union[str, bytes]) -> int:
```

return type에 Union을 사용하는 것은 피해주세요.

Nested Union은 Flat Union과 같습니다.

## Generic Collections

## Tuple Types

## Generic Mappings

## Abstract Base Classes

## Iterable

## Parameterized Generics and TypeVar

## Static Protocols Callable

## NoReturn

# Annotating Positional Only and Variadic Parameters

# Imperfect Typing and Strong Testing

# Chapter Summary

# Further Reading