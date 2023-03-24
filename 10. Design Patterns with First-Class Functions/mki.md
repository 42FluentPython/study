소프트웨어 공학에서 디자인 패턴은 일반적인 레시피입니다.

디자인 패턴은 GoF에 의해서 인기가 많아졌습니다.

디자인 패턴이 프로그래밍 언어와 독립적이지만, 모든 디자인 패턴이 모든 언어에 적용되는 것은 아닙니다.

예로 Iterator Pattern은 Python에 내장되어 있습니다.

GoF 저자들은 이렇게 말했습니다. “우리 책은 SmallTalk와 C++을 생각하면서 작성했습니다.” 만약 Procedural Programming Language를 생각하며 책을 썼다면, Inheritance, Encapsulation, Polymorphism Pattern을 설명했을 겁니다.

이번 장에서는 특정 상황에서 Function이 어떻게 Class처럼 동작하는지 설명할겁니다.

# What’s New in This Chapter

# Case Study: Refactoring Strategy

## Classic Strategy

일반적인 Strategy 디자인 패턴을 설명합니다. Class를 사용합니다.

## Function-Oriented Strategy

Strategy는 좋은 디자인 패턴의 예입니다. 공통 Strategy를 추상 클래스로 만들어서 공유해 사용하면 메모리 이득을 봅니다. 추상 클래스를 Flyweight라고 부릅니다.

Strategy에 내부 상태를 가진다면 Class방법을 사용해야하지만, 내부 상태 변화 없이 Context만 변화시키는 기능만 한다면 굳이 Strategy를 Class로 만들 필요 없고, Function으로 만들면 됩니다.

## Choosing the Best Strategy: Simple Approach

만약 위 예시에서 정의한 Promotion중 가장 할인율이 큰 함수를 예제처럼 만들면 에러가 발생할 확률이 있습니다.

## Finding Strategies in a Module

Python에서 Module도 First-Class Object입니다. Standard Library가 얘네들을 다루는 함수들을 제공합니다.

global() Built-in Function을 사용해서 Strategy를 가져올 수도 있고, inspect.getmembers()를 사용해서 가져올 수 도 있습니다.

# Decorator-Enhanced Strategy Pattern

예제의 Concrete Strategy는 surfix로 _promo가 계속 붙는데, 이때 단점은 다른 개발자가 _promo를 빼먹고 작업하면 에러가 발생할 수 있다는 겁니다. 이때 Decorator를 사용합시다.

- 프로모션 전략 기능에는 _promo 접미사가 필요하지 않으므로 특수 이름을 사용할 필요가 없습니다.
- @promotion decorator는 장식된 기능의 목적을 강조하고, 일시적으로 프로모션을 비활성화하기 쉽게 합니다: 장식자에 대한 코멘트만 남깁니다.
- 프로모션 할인 전략은 @promotion decorator가 적용되는 한 다른 모듈, 시스템 어디에서나 정의할 수 있습니다.

# The Command Pattern

Command Pattern에 대한 설명

# Chapter Summary

Python에서 많은 경우에 Strategy, Command Pattern을 모방하기보다는 Callback으로 구현하는게 더 낫습니다.

# Further Reading