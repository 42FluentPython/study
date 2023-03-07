dict를 사용하는 곳이 너무 많아서 최적화가 잘 되어있고, 안에서는 hash table로 동작

hash table인 type은 set, frozenset

set은 다른 언어 set과 달리 API가 많음. 그래서 algorithm을 만들 때 좀더 명확하게 표현할 수 있음

- Modern syntax to build and handle dicts and mappings, including enhanced
unpacking and pattern matching
- Common methods of mapping types
- Special handling for missing keys
- Variations of dict in the standard library
- The set and frozenset types
- Implications of hash tables in the behavior of sets and dictionaries

# What’s New in This Chapter

# Modern dict Syntax

build, unpack, process mappings

Python 3.9 | Operator

Python 3.10 match/case

## dict Comprehensions

```python
>>> dial_codes = [
... (880, 'Bangladesh'),
... (55, 'Brazil'),
... (86, 'China'),
... (91, 'India'),
... (62, 'Indonesia'),
... (81, 'Japan'),
... (234, 'Nigeria'),
... (92, 'Pakistan'), ... (7, 'Russia'),
... (1, 'United States'),
... ]
>>> country_dial = {country: code for code, country in dial_codes}
>>> country_dial
{'Bangladesh': 880, 'Brazil': 55, 'China': 86, 'India': 91, 'Indonesia': 62, 'Japan': 81, 'Nigeria': 234, 'Pakistan': 92, 'Russia': 7, 'United States': 1}
>>> {code: country.upper()
... for country, code in sorted(country_dial.items())
... if code < 70}
{55: 'BRAZIL', 62: 'INDONESIA', 7: 'RUSSIA', 1: 'UNITED STATES'}
```

## Unpacking Mappings

- **

## Merging Mappings with |

```python
>>> d1 = {'a': 1, 'b': 3}
>>> d2 = {'a': 2, 'b': 4, 'c': 6}
>>> d1 | d2
{'a': 2, 'b': 4, 'c': 6}
>>> d1
{'a': 1, 'b': 3}
>>> d1 |= d2
>>> d1
{'a': 2, 'b': 4, 'c': 6}
```

# Pattern Matching with Mappings

- destructuring

# Standard API of Mapping Types

- **[isinstance**(*object*, *classinfo*)](https://docs.python.org/3/library/functions.html?highlight=isinstance#isinstance)
- collections.abc.Mapping
- collections.abc.MutableMapping

## What Is Hashable

__hash__

## Overview of Common Mapping Methods

## Inserting or Updating Mutable Values

# Automatic Handling of Missing Keys

defaultdict

## defaultdict: Another Take on Missing Keys

## The __missing__ Method

## Inconsistent Usage of __missing__ in the Standard Library

# Variations of dict

## collections.OrderedDict

## collections.ChainMap

## collections.Counter

## shelve.Shelf

## Subclassing UserDict Instead of dict

# Immutable Mappings

# Dictionary Views

# Practical Consequences of How dict Works

# Set Theory

## Set Literals

## Set Comprehensions

# Practical Consequences of How Sets Work

## Set Operations

# Set Operations on dict Views

# Chapter Summary

dict가 생겨서 OrderDict는 거의 사용안하고, 하위호완성을 위해 남겨둠

setdefault, update

# Further Reading
