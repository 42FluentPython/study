# What’s New in This Chapter

Python 3.10에서 나온 Pattern Matching이 추가되었습니다.

# Overview of Built-In Sequences

표준 라이브러리는 C로 개발된 다양한 sequence type 선택지를 제공합니다. sequence는 메모리의 앞 부분에 Python object에 대한 정보가 담겨있습니다. 이를 metadata라고 합니다.

## Container sequences

list, tuple, collections.deque …

objects의 reference를 저장합니다.

## Flat sequences

str, bytes, array.array

자기 자신의 메모리 공간을 가집니다. C의 array와 같습니다. 그래서 좀 더 효율적입니다.

## This element signifies a general note.

float은 1개의 value field와 2개의 metadata field를 가집니다.

container vs flat과는 다른 방법으로 sequence를 분류합니다.

## Mutable sequences

list, bytearray, array.array, collections.deque

Immutable sequence로부터 모든 method를 상속받습니다.

## Immutable sequences

tuple, str, bytes

중요한 것은 container vs flat, mutable vs immutable입니다.

가장 기본이 되는 sequence type은 list입니다. mutable, container입니다.

바로 list comprehension에 대해 배워봅시다.

# List Comprehensions and Generator Expressions

List Comprehensions은 listcomps로 불립니다. Generator Expressions는 genexps로 불립니다.

listcomps는 매우 readable합니다.

## List Comprehensions and Readability

```python
[func(obj) for obj in seq]
```

listcomps의 목적은 오직 new list를 만드는 것입니다. sideeffect로 list로 무언가 연산하려고 하지 말고, 길게 쓰지 마세요. listcomps가 두 줄 이상이면 나누거나 for를 쓰세요.

line breaks는 [], {}, ()에서 무시됩니다. 라인을 나누고 싶다면 마지막 item에 ,를 넣으세요.

“Walrus operator” :=는 for의 local variable의 마지막 값을 저장합니다.

```python
[last := func(obj) for obj in seq]
```

## Listcomps Versus map and filter

listcomps는 map, filter가 할 수 있는 것을 if, lambda를 사용해 똑같이 할 수 있습니다. filter와 map은 Chapter 7!

## Cartesian Products

Cartesian product = 곱집합

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color, size) for color in colors for size in sizes]
```

size가 먼저 실행됨.

```python
self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]
```

## Generator Expressions

```python
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols)
(36, 162, 163, 165, 8364, 164)
```

```python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in (f'{c} {s}' for c in colors for s in sizes): ... print(tshirt)
```

# Tuples Are Not Just Immutable Lists

tuple은 immutable 뿐만 아니라 이름 없는 records로도 사용됩니다.

## Tuples as Records

Python에서 _는 dummy variable입니다. match/case에서는 wildcard로 사용됩니다. Python console은 실행 결과를 _에 저장합니다.

tuple unpacking

## Tuples as Immutable Lists

- Clarity: 길이가 절때 변하지 않음
- Performance: list보다 메모리를 덜 사용해서 최적화에 좋음

tuple에 mutable sequence를 넣을 수 있긴 한데… 버그를 만들 수 있습니다.

[“Are tuples more efficient than lists in Python?”](https://stackoverflow.com/questions/68630/are-tuples-more-efficient-than-lists-in-python/22140115#22140115)

tuple(t)는 reference를 반환하지만, list(l)은 new copy를 합니다.

tuple안에 reference는 array에 저장되지만, list는 dynamic memory allocate합니다.

## Comparing Tuple and List Methods

tuple.reverse() - X

reversed(my_tuple) - O

# Unpacking Sequences and Iterables

Unpacking은 중요합니다. index를 안써도됩니다.

```python
>>> lax_coordinates = (33.9425, -118.408056)
>>> latitude, longitude = lax_coordinates # unpacking
```

## Using * to Grab Excess Items

```python
>>> a, b, *rest = range(5) >>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(3) >>> a, b, rest
(0, 1, [2])
>>> a, b, *rest = range(2) >>> a, b, rest
(0, 1, [])
```

## Unpacking with * in Function Calls and Sequence Literals

```python
>>> def fun(a, b, c, d, *rest): ... return a, b, c, d, rest ...
>>> fun(*[1, 2], 3, *range(4, 7)) (1, 2, 3, 4, (5, 6))
```

```python
>>> *range(4), 4
(0, 1, 2, 3, 4)
>>> [*range(4), 4]
[0, 1, 2, 3, 4]
>>> {*range(4), 4, *(5, 6, 7)} {0, 1, 2, 3, 4, 5, 6, 7}
```

## Nested Unpacking

```python
metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
def main():
		print(f'{"":15} | {"latitude":>9} | {"longitude":>9}') for name, _, _, (lat, lon) in metro_areas:
				if lon<=0:
						print(f'{name:15} | {lat:9.4f} | {lon:9.4f}')
if __name__ == '__main__':
		main()
```

# Pattern Matching with Sequences

Python 3.10의 새 기능 match/case pattern matching

```python
def handle_command(self, message): match message:
		case ['BEEPER', frequency, times]:
		    self.beep(times, frequency)
		case ['NECK', angle]:
		    self.rotate_neck(angle)
		case ['LED', ident, intensity]:
		    self.leds[ident].set_brightness(ident, intensity)
		case ['LED', ident, red, green, blue]:
		    self.leds[ident].set_color(ident, red, green, blue)
		case _:
				raise InvalidCommand(message)
```

C의 switch/case와 비슷하지만 더 기능이 많습니다.

위 예제가 아래처럼 바뀝니다.

```python
metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
def main():
		print(f'{"":15} | {"latitude":>9} | {"longitude":>9}') for record in metro_areas:
		match record:
				case [name, _, _, (lat, lon)] if lon <= 0:
						print(f'{name:15} | {lat:9.4f} | {lon:9.4f}')
```

## Pattern Matching Sequences in an Interpreter

압박감에 못이겨 제대로 읽지 않음.

- parse
- evaluate

# Slicing

sequence는 slicing을 지원하는데, python의 강력함이 엿보입니다.

## Why Slices and Ranges Exclude the Last Item

- my_list[:3]은 3개를 잘라냈다고 바로 인지가능합니다.
- my_list[3:4]는 4 - 3으로 길이가 1이라고 바로 인식할 수 있습니다.
- split하기 쉽습니다. l[:2], l[2:]

## Slice Objects

s[a:b:c]를 보니 이제야 s[::3]이 이해가 됩니다.

```python
>>> s = 'bicycle' >>> s[::3]
'bye'
>>> s[::-1] 'elcycib'
>>> s[::-2] 'eccb'
```

a:b:c는 오직 [ ] 에서만 동작합니다.

seq[start:stop:step]

slice(start, stop, step)

## Multidimensional Slicing and Ellipsis

a[m:n, k:l]

three full stops …

## Assigning to Slices

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9] >>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
File "<stdin>", line 1, in <module> TypeError: can only assign an iterable >>> l[2:5] = [100]
>>> l
[0, 1, 100, 22, 9]
```

# Using + and * with Sequences

+, *는 new object

## Building Lists of Lists

*는 얕은 복사?!

## Augmented Assignment with Sequences

+= 는 special method __iadd__를 호출, 없으면 a = a + b

*= 는 __imul__

## A += Assignment Puzzler

```python
>>> t = (1,2,[30,40])
>>> t[2] += [50, 60]
```

??!?!?! 이게 된다고?!

# list.sort Versus the sorted Built-In

list.sort()는 return None, list 원본에 직접 영향을 줌

sorted는 new object return, 원본에 영향을 안줌

# When a List Is Not the Answer

list가 만능은 아니다~, array는 메모리 절약, deque는 FIFO에 더 좋다. 아이템이 collection에 있는지 자주 확인한다면 set

## Arrays

list가 숫자만 저장하면 array.array로 바꿔요

## Memory Views

## NumPy

## Deques and Other Queues

# Chapter Summary

# Further Reading