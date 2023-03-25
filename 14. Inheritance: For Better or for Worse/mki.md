상속이 왜 좋아질 수도 있고 나빠질 수도 있지?

Smalltalk 초기에 상속에 대해 전반적인 수정이 필요했다는 Alan Kay

예습 필요. 아래 4개 알아보자.

- super()
- built-in types 상속의 pitfall, 위험성
- 다중상속과 MRO
- Mixin Classes

상속 남용에 반대하는 반발이 2021년에 있었다?!

# What’s New in This Chapter

상속에 대해 Python이 바뀐 것은 없지만, 내용은 엄청 바뀌었음, super(), Mixin Classes, 상속에 대해 계속 경고함.

# The super() Function

super()의 사용 방법

```python
def __setitem__(self, key, value):
    super().__setitem__(key, value)
    self.move_to_end(key)
```

Java는 생성자 오버라이딩할 때 자동으로 슈퍼클래스 생성자를 호출하지만 파이썬은 그런거 없다.

```python
def __init__(self, a, b) :
    super().__init__(a, b)
    ... # more initialization code
```

아래 코드는 super()를 사용하지 않고 superclass에서 바로 메소드 호출해버리기

```python
class NotRecommended(OrderedDict):
    """This is a counter example!"""

    def __setitem__(self, key, value):
        OrderedDict.__setitem__(self, key, value)
        self.move_to_end(key)
```

첫 번째 단점, 이건 하드코딩이라 별로임. 그리고 상속하는 클래스를 누가 변경하면 난리남.

두 번째 단점, 다중 상속 때 뭔가 발생.

super()에 argument가 두개 있음. type, object_or_type

```python
class LastUpdatedOrderedDict(OrderedDict):
    """This code works in Python 2 and Python 3"""

    def __setitem__(self, key, value):
        super(LastUpdatedOrderedDict, self).__setitem__(key, value)
        self.move_to_end(key)
```

타입을 지정해서 다중 상속 때 어떤 super를 부를지 선택하는구만.

# Subclassing Built-In Types Is Tricky

Python 2.2 전에는 Built-in types를 상속할 수 없었음. 이후에는 가능한데 경고가 출력

```python
class DoppelDict(dict):
    def __setitem__(self, key, value):
        super().__setitem__(key, [value] * 2)
```

위 같은 녀석은 built-in 행동 약속을 위반한거야

- late binding?

dict같은 built-in을 상속받지 말고 그냥 collections의 UserDict, UserList를 사용해서 써라 좀

UserDict를 선언하니까… dict가 UserDict로 동작하네?! __setitem__만 overriding했는데, __init__, update()도 바뀌네?

# Multiple Inheritance and Method Resolution Order

diamond problem… superclass의 method가 같은 이름일 때 발생

호출 순서는 어떻게 결정? → 모든 class는 __mro__가 있음.

```python
>>> Leaf.__mro__ # doctest:+NORMALIZE_WHITESPACE
(<class 'diamond1.Leaf'>, <class 'diamond1.A'>, <class 'diamond1.B'>,
         <class 'diamond1.Root'>, <class 'object'>)
```

BFS알고리즘을 쓰는 것 같지만, C3라는 알고리즘을 사용함.

super()는 __mro__에 있는 순서대로 호출함.

어떤 method가 super()를 호출하면 그 method는 cooperative method라고 불림. 이녀석은 cooperative multiple inheritance를 가능하게 함.

super()를 호출하지 않는 noncooperative method는 버그의 원인이 될 가능성이 있음.

상속 순서도 관계가 있음.

```python
from diamond import A

class U():
    def ping(self):
        print(f'{self}.ping() in U')
        super().ping()

class LeafUA(U, A):
    def ping(self):
        print(f'{self}.ping() in LeafUA')
        super().ping()

class LeafAU(A, U):
    def ping(self):
        print(f'{self}.ping() in LeafUA')
        super().ping()
```

LeafUA.ping()은 U → A → Root

LeafAU.pint()은 A → Root

U가 mixin class 임.

Tkinter의 Text class는 다중상속이 복잡한데, Figure 14-2에 __mro__의 흐름이 잘 나타나 있다.

# Mixin Classes

혼자 상속될 수 없는 class임.

## Case-Insensitive Mappings

super()만 잔뜩 있음.

get() overriding여부가 동작에 영향을 미침… 쉣.

# Multiple Inheritance in the Real World

Design Pattern 책의 대부분은 C++

multiple inheritance는 Adapter pattern임.

## ABCs Are Mixins Too

## ThreadingMixIn and ForkingMixIn

http.server.HTTPServer

http.server.ThreadingHTTPServer

http.server.ThreadingMixIn

## Django Generic Views Mixins

## Multiple Inheritance in Tkinter

# Coping with Inheritance

inheritance에 대한 일반적인 이론이 없다!

그냥 최선의 예제, 디자인 패턴, 관습밖에 없다.

그래서 스파게티 코드가 안되도록 팁을 알려주겠다!

## Favor Object Composition over Class Inheritance

상속하지 말고, 합성! 위임!

## Understand Why Inheritance Is Used in Each Case

의도를 명확히 하자!

## Make Interfaces Explicit with ABCs

ABC를 상속해서 구체적으로 만드셈. 근데 ABC 멀티 상속은 좀…

## Use Explicit Mixins for Code Reuse

코드 재사용을 위해 구체적으로 믹스인을 사용해라.

믹스인은 instantiated되면 안됨.

## Provide Aggregate Classes to Users

유저를 위해 Mixin, Base를 상속하는 비어있는 class를 제공하셈.

## Subclass Only Classes Designed for Subclassing

subclass는 상속때만 사용하도록 만들어졌음.

상속의 용도로 만들어진 class만 상속해라 좀. 아무거나 막 상속하지 말고. 이른뜻?

@final

## Avoid Subclassing from Concrete Classes

잘 만들어진 클래스를 상속하는 걸 하지마

## Tkinter: The Good, the Bad, and the Ugly

Tkinter는 Python 1.1보다 전에 나옴

# Chapter Summary

# Further Reading

상속보다는 조합.

상속을 피하는게 대세

# Soapbox

## **Think about the Classes You Really Need**

KISS principle? "Keep It Simple, Stupid” 간단하게 해 병신아

## **Misbehaving Built-Ins: Bug or Feature?**

dict, list상속해서 쓸 수 있지만 최적화 잘해놨는데 굳이? 진짜? 상속할꺼야? UserDict 써

## Inheritance Across Languages

Go는 상속개념이 없음.

요즘 트랜드는 trait임.