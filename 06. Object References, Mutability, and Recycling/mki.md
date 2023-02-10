# 6. Object References, Mutability, and Recycling

Object와 Name은 서로 다릅니다. Python에서 Variable은 Box가 아니라 Label입니다.

Object Identity, value, alias

Tuple의 trait이 드러날 때입니다. Tuple은 immutable이지만, value는 mutable입니다. 이제 swallow copy와 deep copy에 대해서 배울겁니다.

Reference와 Mutable parameter에 대해서도 다룹니다. mutable parameter defaults 문제와 다루는 방법을 알아봅시다.

garbege collection에 대해서도 알아봅시다. del command와 selection of tricks에 대해서 살펴봅시다.

이번 편은 딱딱하지만, Python bug의 메인 주제이기도 합니다.

# What’s New in This Chapter

이번 편은 1st Edition과 다르지 않습니다. 다만 ==대신 is keyword를 사용했습니다.

# Variables Are Not Boxes

variables as box metaphor는 Python의 이해를 방해합니다. OOP의 reference variable을 이해하기도 어렵습니다.

```python
>>> a = [1, 2, 3]
>>> b = a
>>> a.append(4)
>>> b
[1, 2, 3, 4]
```

b = a는 copy가 아니고, [1, 2, 3]에 b라는 label을 추가한 것입니다.

예를 들어, 시소를 a에 할당했다고 한다면, 시소는 이미 a에 할당하기 전에 생성되었다는 것을 기억하시길 바랍니다.

assign이라기보다 bind라고 생각합시다.

# Identity, Equality, and Aliases

```python
>>> charles = {'name': 'Charles L. Dodgson', 'born': 1832}
>>> lewis = charles
>>> lewis is charles
True
>>> id(charles), id(lewis)
(4357394752, 4357394752)
>>> lewis['balance'] = 950
>>> charles
{'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}
```

```python
>>> alex = {'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}
>>> alex == charles
True
>>> alex is not charles
True
```

'Charles L. Dodgson'라는 사람이 있다고 해봅시다. 그의 별명은 lewis입니다. 같은 이름과 같은 해에 태어난 똑같은 사람이 있지만, 그는 alex라고 불립니다. alex와 lewis는 다릅니다.

== → 값은 같습니다.

is → 하지만 둘은 같은 존재는 아닙니다.

Object의 identity는 생성 후 절대로 변하지 않습니다. is operator는 Object의 identity를 비교합니다.

## Choosing Between == and is

== operator는 value를 비교합니다. is operator는 identity를 비교합니다. 그래서인지 == operator를 더 자주 사용합니다.

is operator는 singleton pattern에 사용되며, None에도 사용됩니다. None은 대표적인 singleton입니다.

```python
x is None
x is not None
```

is operator는 == operator보다 빠릅니다. 그 이유는 is operator는 overload하지 않기 때문입니다. special method를 호출하지 않고 오직 id만 비교합니다. 반면에 == operator는 단지 a.__eq__(b)의 syntactic sugar입니다. object로부터 상속받은 __eq__는 is operator와 같이 id를 비교합니다. 하지만 많은 built-in types는 의미있는 값을 비교하기 위해 __eq__를 override합니다. 따라서 large collection이나 deeply nested struecture에 == operator를 사용하면 많은 처리를 거칩니다.

대부분은 == operator를 사용하며 None은 거의 is operator에 사용됩니다. None을 == 에 사용하는 코드들은 대부분 잘못된 것이었습니다.

## The Relative Immutability of Tuples

Tuple과 같은 Python Data structure는 Object의 Reference를 저장합니다. tuple은 immutable이지만, reference의 원본은 mutable일 수 있습니다.

```python
>>> t1 = (1, 2, [30, 40])
>>> t2 = (1, 2, [30, 40])
>>> t1 == t2
True
>>> id(t1[-1])
4510683648
>>> t1[-1].append(99)
>>> t1
(1, 2, [30, 40, 99])
>>> id(t1[-1])
4510683648
>>> t1 == t2
False
```

Copy는 Object를 내용은 같지만 id가 다른 것을 생성하는 것입니다. 그러면 Object안의 reference의 Object도 복사해야 할까요? 아니면 reference만 복사해야 할까요?

# Copies Are Shallow by Default

```python
>>> l1 = [3, [55, 44], (7, 8, 9)]
>>> l2 = list(l1)
>>> l2
[3, [55, 44], (7, 8, 9)]
>>> l2 == l1
True
>>> l2 is l1
False
```

list를 copy하는 방법은 list constructor를 이용하는 것입니다.

또 다른 방법은 l2 = l1[:] shallow copy입니다. 모든 mutable type에 사용할 수 있습니다. mutable안의 내용물이 immutable이면 문제가 없고, 메모리도 절약할 수 있습니다. 그런데 안의 내용이 mutable이라면 문제가 발생합니다.

```python
>>> l1 = [3, [66, 55, 44], (7, 8, 9)]
>>> l2 = list(l1)
>>> l1.append(100)
>>> l1[1].remove(55)
>>> print('l1:', l1)
l1: [3, [66, 44], (7, 8, 9), 100]
>>> print('l2:', l2)
l2: [3, [66, 44], (7, 8, 9)]
>>> l2[1] += [33, 22]
>>> l2[2] += (10, 11)
>>> print('l1:', l1)
l1: [3, [66, 44, 33, 22], (7, 8, 9), 100]
>>> print('l2:', l2)
l2: [3, [66, 44, 33, 22], (7, 8, 9, 10, 11)]
```

list, mutable type의 +=은 내부에 content를 추가합니다.

그러나 tuple, immutable type의 +=은 내부에 content를 추가하는 것이 아니라, 새로운 tuple을 생성해 교체합니다.

## Deep and Shallow Copies of Arbitrary Objects

Shallow copy가 항상 문제가 되는 것은 아닙니다. 그러나 deep copy가 필요할 때도 있습니다. copy module은 copy()와 deepcopy()를 제공합니다.

```python
>>> from bus import Bus
>>> import copy
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)
>>> bus3 = copy.deepcopy(bus1)
>>> id(bus1), id(bus2), id(bus3)
(4521088464, 4520801808, 4521095248)
>>> bus1.drop('Bill')
>>> bus2.passengers
['Alice', 'Claire', 'David']
>>> id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
(4521217856, 4521217856, 4520943552)
>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David']
```

deepcopy와 순환참조

```python
>>> a = [10, 20]
>>> b = [a, 30]
>>> a.append(b)
>>> a
[10, 20, [[...], 30]]
>>> from copy import deepcopy
>>> c = deepcopy(a)
>>> c
[10, 20, [[...], 30]]
```

deepcopy가 copy되면 안되는 외부 소스나, 싱글톤 객체를 copy할 수 도 있습니다. 그때 __copy__, __deepcopy__ special method를 overriding하면 됩니다.

# Function Parameters as References

function parameter 전달은 call by sharing입니다. 즉, reference가 전달됩니다.

```python
>>> def f(a, b):
...     a += b
...     return a
... 
>>> x = 1
>>> y = 2
>>> f(x, y)
3
>>> x, y
(1, 2)
>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b
([1, 2, 3, 4], [3, 4])
>>> t = (10, 20)
>>> u = (30, 40)
>>> f(t, u)
(10, 20, 30, 40)
>>> t, u
((10, 20), (30, 40))
```

## Mutable Types as Parameter Defaults: Bad Idea

parameter default는 python의 유용한 기능 중 하나입니다. 하지만 default를 mutable type을 하게 되면 어지럽습니다.

```python
>>> from haunted_bus import HauntedBus
>>> bus1 = HauntedBus(['Alice', 'Bill'])
>>> bus1.passengers
['Alice', 'Bill']
>>> bus1.pick('Charlie')
>>> bus1.drop('Alice')
>>> bus1.passengers
['Bill', 'Charlie']
>>> bus2 = HauntedBus()
>>> bus2.pick('Carrie')
>>> bus2.passengers
['Carrie']
>>> bus3 = HauntedBus()
>>> bus3.passengers
['Carrie']
>>> bus3.pick('Dave')
>>> bus2.passengers
['Carrie', 'Dave']
>>> bus2.passengers is bus3.passengers
True
>>> bus1.passengers
['Bill', 'Charlie']
```

bus3을 만들 때 더 이상 empty list가 아닙니다.

mutable value를 받는 parameter의 default가 보통 None인 이유입니다.

## Defensive Programming with Mutable Parameters

mutable argument를 받는 function을 사용할 때 caller가 전달하는 mutable value가 변한다는 것을 인지해야 한다는 것을 고려해야 합니다.

```python
>>> from twilight_bus import TwilightBus
>>> basketball_team = ['Sue', 'Tina', 'Maya', 'Diana', 'Pat']
>>> bus = TwilightBus(basketball_team)
>>> bus.drop('Tina')
>>> bus.drop('Pat')
>>> basketball_team
['Sue', 'Maya', 'Diana']
```

```python
class TwilightBus:
    """A bus model that makes passengers vanish"""

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []  # <1>
        else:
            self.passengers = passengers  #<2>

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)  # <3>
```

```python
def __init__(self, passengers=None):
    if passengers is None:
        self.passengers = [] else:
        self.passengers = list(passengers)
```

# del and Garbage Collection

del은 function이 아니라 statement입니다.

del은 object를 지우는 것이 아니라 reference를 지웁니다.

```python
>>> a = [1, 2]
>>> b = a
>>> del a
>>> b
[1, 2]
>>> b = [3]
```

garbage collector는 object의 reference가 없을 때 지웁니다.

__del__은 어렵습니다. refcount가 0이 될 때 object는 destructor를 실행하고 free됩니다.

Python의 garbage collector의 핵심 알고리즘은 reference count입니다.

CPython이 아닌 다른 Python의 garbage collector는 refcount에만 의존하지 않습니다.

weakref.finalize

```python
>>> import weakref
>>> s1 = {1, 2, 3}
>>> s2 = s1
>>> def bye():
...     print('...like tears in the rain.')
... 
>>> ender = weakref.finalize(s1, bye)
>>> ender.alive
True
>>> del s1
>>> ender.alive
True
>>> s2 = 'spam'
...like tears in the rain.
>>> ender.alive
False
```

# Tricks Python Plays with Immutables

```python
>>> t1 = (1, 2, 3)
>>> t2 = tuple(t1)
>>> t2 is t1
True
>>> t3 = t1[:]
>>> t3 is t1
True
```

# Chapter Summary

# Further Reading