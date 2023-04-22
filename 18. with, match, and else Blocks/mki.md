Context managers

end up ~ing

with statement

control flow features

context manager protocol

pattern matching with match/case

else clause, in for, while, and try statements

with statement prevent error and reduces boilerplater code

with, file closing

big deal

convey intention

tend to

overlooked

underused

tear it down

# What’s New in This Chapter

contextlib

parenthesized

# Context Managers and with Blocks

for - iterators

with - context manager

with, __enter__, __exit__

fp, file pointer

subtle

crucial

as fp is retuened by __enter__

invoked

frivolous

as is optional in with statements

```python
def __exit__(self, exc_type, exc_value, traceback):
    """
    exc_type: The exception class
    exc_value: The exception instance
    traceback: A traceback object
    """
    pass
```

## The contextlib Utilities

context manager를 다루기 위한 standard library인데, 머리에 잘 안들어옴. 다시 읽어보니, 아래에 있는 @contextmanager decorator가 contextlib에 있다. @contextmanager로 context manager를 생성하고, 이외의 method로 조작하는 것 같다.

## Using @contextmanager

context manager를 생성하도록 도와주는 decorator

머리에 안들어옵니다. 아직 무쓸모.

# Pattern Matching in [lis.py](https://github.com/norvig/pytudes/blob/main/py/lis.py): A Case Study

Peter Norvig의 어마어마한 코드

Python에서 LISP를?!

## Scheme Syntax

## Imports and Types

## The Parser

## The Environment

## The REPL

## The Evaluator

## Procedure: A Class Implementing a Closure

closure를 안봐서 머리에 안들어옴.

## Using OR-patterns

그나마 이해가능함. match case

# Do This, Then That: else Blocks Beyond if

```python
>>> for a in range(3):
...   print(a)
... else:
...   print("done")
... 
0
1
2
done
>>> a = 3
>>> while a > 0:
...   a -= 1
...   print(a)
... else:
...   print("done")
... 
2
1
0
done
```

# Chapter Summary

복습

# Further Reading

# Soap box
