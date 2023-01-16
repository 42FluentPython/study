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

