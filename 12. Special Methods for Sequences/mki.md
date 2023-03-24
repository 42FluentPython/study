Object가 어떤 타입인지 신경쓰기보단, 어떻게 행동하는지 신경쓰세요 - Alex Martelli

이번 장에서는 11장에서 만들었던 Vector2d를 확장해서 immutable flat sequence처럼 동작하게 만듭니다. element는 float이 될 겁니다. 잘 따라오면 아래의 조건을 만족합니다.

- __len__, __getitem__을 만들겁니다.
- 많은 item의 safe representation
- 적절한 slicing을 제공
- hashing
- custom formatting

또 dynamic attribute access를 만들겁니다.

protocol과 duck typing의 관계

# What’s New in This Chapter

typing.Protocol에 대해서 새롭게 다룹니다.

__getitem__이 좀 더 견고해졌습니다. duck typing을 적용했기 때문이죠. operator.index도 적용했답니다.

# Vector: A User-Defined Sequence Type

vector를 만들 때 inheritence가 아니라 composition을 사용할 겁니다.

## **Vector Applications Beyond Three Dimensions**

N-dimension Vector가 어디에 쓰일까? 정보 검색 기술에서 엄청 쓰이고, 각 단어마다 1개의 차원이 있다고 생각합니다. 이렇게 생각하는 것을 Vector space model이라고 합니다.

- NumPy
- SciPy
- gensim

# Vector Take #1: Vector2d Compatible

Vector를 만들건데, Vector2d와 거의 비슷하지만 constructor가 조금 다릅니다. Vector(1, 2, 3)이렇게 해도 되지만, 그냥 iterable을 argument로 받읍시다.

만약 iterable argument가 1000개면 다 출력하나요? 아니죠… reprlib이 해결해줍니다.

repr만들 때 이친구는 디버깅 용도로 사용해서 에러나면 감지를 못합니다 그래서 잘 만들어야 합니다

두 가지 이유때문에 inheritance를 안했습니다. 생성자, sequence protocol

# Protocols and Duck Typing

sequency type을 만들기위해 상속받을 필요 없이 몇개의 method 만 만들면 sequence처럼 동작하는 것을 1장에서 확인했습니다. 단지 sequence protocol을 만족하기만 하면 됩니다.

informal interface 명시적인 인터페이스가 아님. 단지 요구하는 속성과 메소드만 만들면 됨

protocol은 약속이기에 무언가 강요하지 않는다. 단지 len과 getitem이 있다면 sequence라는 것이다

protocol에는 두 가지가 있다. static과 dynamic. static은 명시된 것을 모두 작성해야 한다

# Vector Take #2: A Sliceable Sequence

slicing하니 vector가 아니라 array가 나온다

## How Slicing Works

slice()

indices

stride

## A Slice-Aware __getitem__

operator.index()

# Vector Take #3: Dynamic Attribute Access

getattr을 이용한 x y z

getattr을 이용해 v.x = 10은 아무 영향을 안줌 setattr

# Vector Take #4: Hashing and a Faster ==

map과 reduce 그림

zip()

# Vector Take #5: Formatting

# Chapter Summary

# Further Reading