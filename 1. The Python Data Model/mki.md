다른 언어 개발자는 이론적으론 아름답지만 아무도 쓰지 않는 언어를 만들지만, 귀도는 언어학적으로는 부족해도 코딩하는 재미를 느끼게 해주는 언어를 만들었습니다.

파이썬의 가장 좋은 특징 중 하나는 일관성입니다.

OOP를 해봤다면 len(collection)이 이상하다고 생각하죠. 그러나 이게 Pythonic입니다. 그리고 이상한 녀석이 나온 건 Python Data Model때문입니다.

Python은 Framework와 비슷합니다.

Python Data Model을 사용해서  class를 만들 때, special method를 작성합니다. special method는 `__*__`와 같이 생겼습니다. `obj[key] = obj.__getitem__(key)`

magic method는 special method를 부르는 별명입니다. dunder method라고 부르기도 합니다.

# What’s New in This Chapter

이번 장은 변경사항이 적지만, 책 전반에 걸쳐 str.format() 대신 f””로 변경했습니다.

가끔 my_fmt.format()을 사용합니다.

# A Pythonic Card Deck

Example 1-1은 간단하지만 two special methods의 영향력을 보여줍니다.

- `[object.**__getitem__**(*self*, *key*)](https://docs.python.org/3/reference/datamodel.html?highlight=__getitem#object.__getitem__)`
- `[object.**__len__**(*self*)](https://docs.python.org/3/reference/datamodel.html?highlight=__len#object.__len__)`

이 책에서는 class의 member variable규칙은 _name입니다.

- [`collections.**namedtuple**(*typename*, *field_names*, ***, *rename=False*, *defaults=None*, *module=None*)`](https://docs.python.org/3/library/collections.html?highlight=collection#collections.namedtuple)

 nametuple은 database의 record와 비슷합니다.

랜덤하게 카드를 꺼낼 수도 있습니다.

- [`random.**choice**(*seq*)`](https://docs.python.org/3/library/random.html?highlight=choice%20random#random.choice)

special method의 장점은 길이를 구할 때 size인지 length인지 물어볼 필요가 없다는 겁니다.

풍부한 Python standard library를 사용하기 쉽고, 처음부터 하나하나 만들 필요가 없습니다.

`__getitem__`은 `[ ]`에 `self._cards`를 전달해주고, slicing을 자동으로 지원해줍니다.

```python
deck[:3]
```

```python
deck[12::13]
```

`__getitem__`은 iterable합니다.

```python
for card in deck:
  print(card)
```

reverse iterate도 가능합니다.

```python
for card in reversed(deck):
  print(card)
```

- doctest

in operator

```python
Card('Q', 'hearts') in deck
```

sorting

```python
suit_values = dict(spades=3, hearts=2, diamonds=1, clubs=0)
def spades_high(card):
	rank_value = FrenchDeck.ranks.index(card.rank)
	return rank_value * len(suit_values) + suit_values[card.suit]

for card in sorted(deck, key=spades_high): # doctest: +ELLIPSIS
  print(card)
```

우리가 만든 class에 `__len__`과 `__getitem__`만 작성해도 sequence처럼 동작합니다.

### **How About Shuffling?**

FrenchDeck은 immutable이라 shuffling은 안됩니다. Chapter 13에서 `__setitem__`를 배우면 할 수 있습니다.

# How Special Methods Are Used

special methods는 여러분들이 아니라 interpreter에 의해 호출되도록 만들어졌습니다. 사용자는 `my_object.__len__()`대신, `len(my_object)`를 사용합니다.

- `PyVarObject`
- `ob_size`

special method는 추상적으로 호출됩니다.

`for i in x:` → `iter(x)` → `x.__iter__()`, `x.__get__iter()`

special method를 직접 호출하는 것을 지양해야 합니다.

## Emulating Numeric Types

몇몇 special method는 +에 응답합니다. Chapter 16에서 더 자세히 알아봅시다.

2차원 벡터 클래스를 만들어봅시다.

built-in `complex`는 2차원 벡터를 표현할 때 자주 사용됩니다.

Vector class를 만들 때 아래의 special method를 작성할 겁니다.

- `__repr__`
- `__abs__`
- `__bool__`
- `__add__`
- `__mul__`

위 method 아무것도 직접 호출되지 않을 겁니다. Python interpreter가 호출할 겁니다.

`+`, `*`는 새로운 instance를 return합니다.

Vector * 3은 되는데 3 * Vector는 아직 안됩니다. `__rmul__`을 Chapter 16에서 배울겁니다.

## String Representation

`__repr__` 은 built-in `repr` 이 호출합니다.

console, debugger이 repr을 호출합니다.

`__str__`은 built-in `print` function이 호출합니다.

## Boolean Value of a Custom Type

- `__bool__`

위 special method를 작성하지 않으면 user class는 언제나 True입니다.

```python
def __abs__(self):
	return math.hypot(self.x, self.y)
```

```python
def __bool__(self):
	return bool(self.x or self.y)
```

## Collection API

- `ABCs—*abstract base classes*`
- Python에서 가장 중요한 collection interface
- `UML class diagram`
- `Iterable` → for
- `Sized` → len
- `Container` → in

# Overview of Special Methods

Python Language Reference의 Data Model chapter에서는 80개 이상의 special method를 설명합니다.

# Why len Is Not a Method

CPython의 built-in object는 어떤 method도 호출하지 않습니다.

len()은 method로 불리지 않습니다.

# Chapter Summary

여러분의 class에 special methods를 작성하면 built-in type처럼 동작할 수 있습니다.

logging과 debugging을 위해 __repr__, __str__을 작성합시다.

operator overloading을 고려합시다.

# Further Reading

# Soapbox
