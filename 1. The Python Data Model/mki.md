**파이썬의 가장 좋은 특징 중 하나는 일관성입니다.**

`obj[key] = obj.__getitem__(key)`

# What’s New in This Chapter

- asynchronous programming을 지원하는 special methods가 추가되었습니다.

# A Pythonic Card Deck

two special methods

- `__getitem__`
- `__len__`

# How Special Methods Are Used

special methods는 interpreter에 의해 호출되도록 만들어졌습니다.

사용자는 `my_object.__len__()`대신, `len(my_object)`를 사용합니다.

## Emulating Numeric Types

## String Representation

- `__repr__`

## Boolean Value of a Custom Type

- __bool__

## Collection API

- ABCs—*abstract base classes*

# Overview of Special Methods

# Why len Is Not a Method

# Chapter Summary

# Further Reading

- “Data Model” chapter of *The Python Language Reference*
- *Python in a Nutshell*, 3rd ed. by Alex Martelli, Anna Ravenscroft, and Steve Holden
(O’Reilly)

# Soapbox