# Chapter02 - An Array of Sequences
## main topics
* List comprehensions and the basics of generator expressions
* Using tuples as records versus using tuples as immutable lists
* Sequence unpacking and sequence patterns
* Reading from slices and writing to slices
* Specialized sequence types, like arrays and queues

## Pattern Matching With Sequences
### 3.10
[pep-634](https://peps.python.org/pep-0634/)\
[pep-635](https://peps.python.org/pep-0635/)\
[pep-636](https://peps.python.org/pep-0636/)

## Overview of Built-In Sequences
### Container Sequences
	다른 타입의 아이템들을 저장 할 수 있는 sequence.
	list, tuple, collections.deque
### Flat Sequences
	하나의 단순 타입의 아이템들을 저장 할 수 있는 sequence.
	str, bytes, array.array
### example - Container
```python
>>> container = (9.46, 'cat', [2.08, 4.29])
>>> id(container[0])
4341644464
>>> id(container[1])
4342075504
>>> id(container[2])
4342075648
```
### example - Flat
```python
>>> from array import *
>>> flat = array('d', [9,46,2.08,4.29])
>>> id(flat[0])
4342083632
>>> id(flat[1])
4342083632
>>> id(flat[2])
4342083632
```
`Cotainer Sequences`의 경우, object reference를 참조하는것을 확인.\
`Flat Sequences`의 경우 선형적 메모리에 할당 하는것을 확인.

### Python Object in memory
메모리에 적재된 모든 파이썬 오브젝트 메타데이터와 함께 헤더를 가짐.\
가장 단순한 파이썬 오브젝트인 `float`은 하나의 value field와 두 개의 metadata field를 가짐.\
* **ob_refcnt** - 객체의 참조 횟수
* **ob_type** - 객채 타입에 대한 포인터
* **ob_fval** - C의 double형

### Another Way of grouping sequence type is by mutability
* **Mutable Sequences** - list, bytearray, array.array, collections.deque.
* **Immutable Sequences** - tuple, str, bytes
  
**example**
```python
>>> from collections import abc
>>> issubclass(list, abc.Sequence)
True
>>> issubclass(list, abc.MutableSequence)
True
>>> issubclass(list, abc.Collection)
True
>>> issubclass(list, abc.Reversible)
True
```

## List Comprehensions and Generator Expressions
### [Comprehensions Syntax](https://docs.python.org/ko/3/reference/expressions.html?highlight=list%20comprehension#displays-for-lists-sets-and-dictionaries)
```
comprehension ::= assignment_expression comp_for
comp_for      ::= [ "async" ] "for" target_list "in" or_test [ comp_iter ]
comp_iter     ::= comp_for | comp_if
comp_if       ::= "if" or_test [ comp_iter ]
```
### List Comprehensions and Readability
**example.1 - non listcomps**
```python
>>> symbols = '$¢£¥€¤'
>>> codes = []
>>> for symbol in symbols:
...     codes.append(ord(symbol))
...
>>> codes
[36, 162, 163, 165, 8364, 164]
```
**example.2 - listcomps**
```python
>>> symbols='$¢£¥€¤'
>>> codes = [ord(symbol) for symbol in symbols]
>>> codes
[36, 162, 163, 165, 8364, 164]
```

### Local Scope Within Comprehensions and Generator Expressions
[pep-572](https://peps.python.org/pep-0572/)\
**example.1 - non generator expressions**
```python
>>> x = 'ABC'
>>> codes = [ord(x) for x in x]
>>> x
'ABC'
>>> codes
[65, 66, 67]
```
**example.2 - generator expressions**
```python
>>> x = 'ABC'
>>> codes = [last := ord(c) for c in x]
>>> last
67
>>> c
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'c' is not defined
```
Walrus Operator라고 불리는 `:=` 연산자로 할당하면 comprehensions 또는 expressions을 사용하더라도 접근 가능하게 해줌.

### Listcomps Versus map and filter
**example.1 - listcomp vs map/filter**
```python
>>> symbols = '$¢£¥€¤'
>>> beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
>>> beyond_ascii
[162, 163, 165, 8364, 164]
>>> beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols))) >>> beyond_ascii
[162, 163, 165, 8364, 164]
```
map/filter로 하는 모든것은 listcomp로 할 수 있음.

### Cartesian Products
listcomps는 데카르트 곱으로 부터 리스트를 만들 수 있음.

**example.1 - Cartesian product using a list comprehension**
```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color, size) for color in colors for size in sizes]
>>> tshirts
[('black', 'S'), ('black', 'M'), ('black', 'L'), ('white', 'S'), ('white', 'M'), ('white', 'L')]
>>> for color in colors:
...     for size in sizes:
...         print((color, size))
...
('black', 'S')
('black', 'M')
('black', 'L')
('white', 'S')
('white', 'M')
('white', 'L')
>>> tshirts = [(color, size) for size in sizes for color in colors]
>>> tshirts
[('black', 'S'), ('white', 'S'), ('black', 'M'), ('white', 'M'), ('black', 'L'), ('white', 'L')]
```
1. 생성된 리스트의 튜플은 color, size에 의해 정렬.
2. 정렬된 리스트의 결과에 주목, `for loops`의 중첩된 동일한 순서로 `listcomp`는 생성됨.
3. size, 그리고 color 순서로 정렬된 아이템을 얻기 위해 for 구문을 재정렬 하면됨.