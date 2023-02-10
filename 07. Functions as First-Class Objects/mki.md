# 7. Functions as First-Class Objects

Python에서 Function은 First-class object입니다. Computer Science에서 First-class objects는 다음과 같이 정의합니다.

- Created at runtime
- Assigned to a variable or element in a data structure
- Passed as an argument to a function
- Returned as the result of a function

Functional Programming에서는 Function이 반드시 First-class objects입니다.

이번 Chapter와 Part III에서 function을 object처럼 다루는 법을 배울 겁니다.

# What’s New in This Chapter

1st Edition에서는 The Seven Flavoers of Callable Objects 였는데, The Seven Flavoers of Callable Objects로 변경되었습니다. 그 이유는 Python 3.5에서 Native coroutines, Python 3.6에서 Asyncronous generators가 추가되었기 때문입니다.

Python 3.8에서 Positional-only Parameters가 추가되었습니다.

# Treating a Function Like an Object

예제를 통해 function object는 function class의 instance라는 것을 확인해봅시다.

```python
>>> def factorial(n):
...     """returns n!"""
...     return 1 if n < 2 else n * factorial(n - 1)
... 
>>> factorial(42)
1405006117752879898543142606244511569936384000000000    
>>> factorial.__doc__
'returns n!'
>>> type(factorial)
<class 'function'>
```

Example 7-2에서는 function을 variable에 할당하는 것과, argument로 전달하는 것을 보여드리겠습니다.

```python
>>> fact = factorial
>>> fact
<function factorial at 0x100c531a0>
>>> fact(5)
120
>>> map(factorial, range(11))
<map object at 0x100d5a980>
>>> list(map(factorial, range(11)))
[1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 3628800]
```

# Higher-Order Functions

Function이 다른 Function의 argument가 될 수 있고, Function의 return이 Function이 될 수 있다면, Function은 Higher-Order Function입니다. Example 7-2가 그 예입니다.

Example 7-3

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=len)
['fig', 'apple', 'cherry', 'banana', 'raspberry', 'strawberry']
>>>
```

Example 7-4

```python
>>> def reverse(word):
...     return word[::-1]
... 
>>> reverse('testing')
'gnitset'
>>> sorted(fruits, key=reverse)
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
>>>
```

## Modern Replacements for map, filter, and reduce

map, filter, reduce, apply는 Higher-Order function의 대표적인 예입니다. apply는 Python 3에서 제거되었습니다. 그리고 map, filter, reduce는 listcomp, genexp로 대체될 수 있습니다.

Example 7-5

```python
>>> list(map(factorial, range(6)))
[1, 1, 2, 6, 24, 120]
>>> [factorial(n) for n in range(6)]
[1, 1, 2, 6, 24, 120]
>>> list(map(factorial, filter(lambda n: n % 2, range(6))))
[1, 6, 120]
>>> [factorial(n) for n in range(6) if n % 2]
[1, 6, 120]
>>>
```

reduce는 Python 2에서는 Built-in function이었지만, Python 3에서는 functools Module로 변경되었습니다.

Example 7-6

```python
>>> from functools import reduce
>>> from operator import add
>>> reduce(add, range(100))
4950
>>> sum(range(100))
4950
```

- any(iterable) - 하나라도 True이면 return True
- all(iterable) - 모두 True이면 return True

Higher-Order Function을 위해 Anonymous Function을 사용할 수 있습니다.

# Anonymous Functions

lambda keyword는 Anonymous Function을 생성합니다.

lambda를 사용할 때 body에 statement를 사용할 수 없고, expression만 사용할 수 있습니다. assignment = 또한 statement이므로 사용할 수 없습니다. 다만 assigning expression인 :=는 lambda에 사용할 수 있지만, 권장되지 않습니다.

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=lambda word: word[::-1])
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
>>>
```

lambda가 읽기 어렵다면 다음의 refactoring을 권장합니다.

## **Fredrik Lundh’s lambda Refactoring Recipe**

1. lambda가 뭘 하는지 comment를 적자
2. comment의 핵심 단어를 찾자
3. lambda를 def로 바꾸고 이름을 핵심 단어로 하자
4. comment를 지운다

lambda syntax는 syntax sugar일 뿐입니다.

# The Nine Flavors of Callable Objects

callable operator ()

callable built-in function은 Object가 callable인지 아닌지 확인해줍니다.

Python 3.9에는 9가지 Callable types가 있습니다.

## User-defined functions

def, lambda로 만들어진 functions

## Built-in functions

CPython에서 C로 개발된 functions

## Built-in methods

dict.get과 같이 C로 개발된 methods

## Methods

class의 body에 작성된 functions

## Classes

Class를 call하면 __new__는 instance를 생성하고, __init__은 instance를 초기화합니다.

## Class instances

Class에 __call__ special method가 정의되어 있다면 instance는 callable입니다.

## Generator functions

yield keyword와 관련있습니다. Function이나 Method의 body에 yield가 있다면 generator object를 return합니다. generator는 실행 될 때마다 yield를 한 줄 씩 반환합니다.

## Native coroutine functions

async def로 정의된 Functions나 Methods는 coroutine objects를 return합니다. Python 3.5에서 추가되었습니다.

## Asynchronous generator functions

async def에서 yield를 하면 asynchronous generator objects를 return합니다. async for를 사용해야 합니다.

generator와 asynchronous는 Chapter 17과 21에서 자세히 살펴보겠습니다.

```python
>>> abs, str, 'Ni!'
(<built-in function abs>, <class 'str'>, 'Ni!')
>>> [callable(obj) for obj in (abs, str, 'Ni!')]
[True, True, False]
>>>
```

# User-Defined Callable Types

임의의 Python objects도 __call__ special method만 만들면 function처럼 동작할 수 있습니다.

```python
>>> from bingocall import BingoCage
>>> bingo = BingoCage(range(3))
>>> bingo.pick()
1
>>> bingo()
2
>>> bingo()
0
>>> callable(bingo)
True
>>>
```

decorator는 반드시 callable이어야 하는데, decorator를 만들 때 __call__을 정의해야 합니다. decorator는 caching이나, method를 분할해서 개발할 때 좋습니다.

Closures도 decorator와 비슷한데 Chapter 9에서 봅시다.

# From Positional to Keyword-Only Parameters

*는 iterable을 unpack

**는 mapping separated argments

Example 7-9

```python
def tag(name, *content, class_=None, **attrs):
    """Generate one or more HTML tags"""
    if class_ is not None:
        attrs['class'] = class_
    attr_pairs = (f' {attr}="{value}"' for attr, value
                    in sorted(attrs.items()))
    attr_str = ''.join(attr_pairs)
    if content:
        elements = (f'<{name}{attr_str}>{c}</{name}>'
                    for c in content)
        return '\n'.join(elements)
    else:
        return f'<{name}{attr_str} />'
```

Python 3에서 Keyword-only argument가 도입되었습니다.

```python
>>> def f(a, *, b):
...     return a, b
... 
>>> f(1, b=2)
(1, 2)
>>> f(1, 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: f() takes 1 positional argument but 2 were given
```

*뒤에 오는 parameter는 모두 Keyword-only parameter입니다.

## Positional-Only Parameters

/왼쪽에 정의된 parameters는 모두 Positional-Only Parameters입니다.

```python
>>> def divmod(a, b, /):
...     return (a // b, a % b)
...
```

# Packages for Functional Programming

Python을 만든 귀도는 의도하지 않았지만, Functional Programming으로 Python은 유용합니다. lambda 대신에 operator를 적극 사용하는 것을 Functional Programming이라고 하는 것 같습니다.

## The operator Module

함수형 프로그래밍을 할 때 arithmetic operator를 사용하는 것이 유용합니다.

```python
from functools import reduce

def factorial(n):
    return reduce(lambda a, b: a*b, range(1, n+1))
```

```python
from functools import reduce
from operator import mul

def factorial(n):
    return reduce(mul, range(1, n+1))
```

itemgetter, attrgetter도 유용하게 사용할 수 있습니다.

```python
>>> metro_data = [
...     ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
...     ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
...     ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
...     ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
...     ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)), ... ]
>>>
>>> from operator import itemgetter
>>> for city in sorted(metro_data, key=itemgetter(1)):
...     print(city)
...
('São Paulo', 'BR', 19.649, (-23.547778, -46.635833))
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))
```

마지막으로 볼 것은 methodcaller입니다. methodcaller(instance)는 instacne에 정의되어 있는 method를 호출합니다.

```python
>>> from operator import methodcaller
>>> s = 'The time has come'
>>> upcase = methodcaller('upper')
>>> upcase(s)
'THE TIME HAS COME'
>>> hyphenate = methodcaller('replace', ' ', '-')
>>> hyphenate(s)
'The-time-has-come'
```

## Freezing Arguments with functools.partial

functools는 몇 가지 Higher-Order Function을 제공합니다. 이전에 reduce를 살펴봤는데, 이번에는 partial을 알아봅시다.

```python
>>> from operator import mul
>>> from functools import partial
>>> triple = partial(mul, 3)
>>> triple(7)
21
>>> list(map(triple, range(1, 10)))
[3, 6, 9, 12, 15, 18, 21, 24, 27]
```

partial을 사용하면 func의 argument를 미리 지정할 수 있습니다.

```python
>>> import unicodedata, functools
>>> nfc = functools.partial(unicodedata.normalize, 'NFC')
>>> s1 = 'café'
>>> s2 = 'cafe\u0301'
>>> s1, s2
('café', 'café')
>>>s1==s2
False
>>> nfc(s1) == nfc(s2)
True
```

위에서 해봤던 tag function을 이용해봅시다.

```python
>>> from tagger import tag
>>> tag
<function tag at 0x109ad3420>
>>> from functools import partial
>>> picture = partial(tag, 'img', class_='pic-frame')
>>> picture(src='wumpus.jpeg')
'<img class="pic-frame" src="wumpus.jpeg" />'
>>> picture
functools.partial(<function tag at 0x109ad3420>, 'img', class_='pic-frame')
>>> picture.func
<function tag at 0x109ad3420>
>>> picture.args
('img',)
>>> picture.keywords
{'class_': 'pic-frame'}
>>>|
```

partialmethod란 것도 있습니다.

functools에는 유용한 고차함수가 더 있습니다.

- cache
- singledispatch

# Chapter Summary

# Further Reading