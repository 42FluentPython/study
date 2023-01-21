# 1장

파이썬의 가장 좋은 특성중 하나는 일관성이다
→ 하나를 알면 열을 안다.

다른 객체지향 언어와 달리, python은 collection의 길이를 구할 때 `collection.len()` 이 아니라 `len(collection)`을 사용한다.

특수 메서드는 선행, 후행 이중 밑줄로 작성된다 ( `__getitem__` )

내가 만든 객체가 다음과 같은 기본 언어 구조와 연계하길 원할 때 special method를 구현한다.

- Collections
- Attribute access
- Iteration (including asynchronous iteration using `async for`)
- Operator overloading
- Function and method invocation
- String representation and formatting
- Asynchronous programming using `await`

# ****The Python [Data Model](https://docs.python.org/3/reference/datamodel.html)**

- Object creation and destruction
- Managed contexts using the `with` or `async with` statements

# ****A Pythonic Card Deck****

[Named Tuple 사용법](https://zzsza.github.io/development/2020/07/05/python-namedtuple/)

```bash
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits
                                        for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]
```

1. `__len__` 을 구현해서 `len(french_deck)` 사용가능.
2. `__getitem__` 을 구현해서 인덱싱 가능
3. `__len__` , `__getitem__` 둘 다 구현했으므로 `random.choice` 함수 사용 가능

→ 이를 통해 알 수 있는, special method를 통해 Python Data Model을 활용하면 얻을 수 있는 이점

1. 기본적인 동작을 수행하기 위한 구체적인 메서드 이름을, 클래스 사용하는 사람이 알 필요가 없음( 다른 언어의 경우 collection의 길이를 알기 위해 .length()를 사용할지 .size()를 사용할지 )
2. 바퀴의 재발명을 할 필요가 없음 (random.choice)

이외에도 FrenchDeck은 `__getitem__` 을 구현했기 때문에 슬라이싱, 이터레이션, 리버스 기능도 지원한다.
`in` 키워드 도 지원하는데, `__contains__` 가 구현되지 않았기 때문에, 순차적인 탐색을 통해 지원한다.

# ****How Special Methods Are Used****

special method 는 직접 호출해선 안된다. interpreter에 의해서만 호출되어야만 한다.

하지만 list, str, bytearray, numpy array같은 builtin type의 경우 special method 대신 ob_size 같은 필드의 값을 가져온다. 메서드 호출보다 빠름.

special method를 직접호출하는 경우는 생성자에서 부모의 생성자를 호출하는 경우 말곤 없어야한다.

## **Emulating Numeric Types**

`__add__` : + 연산

`__abs__` : abs()

`__mul__` : * 연산