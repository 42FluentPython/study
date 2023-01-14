# The Python Data Model

## Special Methods의 목적
- special method = dunder method는 파이썬 built-in 기능과 operation overloading을 이용할 수 있게 해준다.
  - Numeric Type 구현하기
  - 오브젝트의 문자열 representation
  - 오브젝트의 불리언 값
  - collection 구현
 
### Numeric Type 구현하기
- operation overloading을 통해서 수학적 타입을 구현할 수 있다.

### 오브젝트의 문자열 representation
- `__repr__`은 콘솔과 디버거에서 쓰이는 오브젝트의 문자열 representation을 구현한다.
  - `repr`함수를 통해서 호출한다.
  - `"Representation of an object is %r" % object`
  - `"Represetation of an object is {obj!r}".format(obj=object)`
  - `f"Representation of and object is {object!r}"`
- `__str__`은 `str()` built-in 함수에 의해 호출된다. `print()`함수에 의해 쓰인다.
  - `__str__`이 구현되어있지 않으면 `__repr__` 함수가 호출된다.

### 오브젝트의 불리언 값
- 어떤 값 `x`가 참 값인지 거짓 값인지 평가할 때 파이썬은 `bool(x)`함수를 호출한다.
- `bool`은 `__bool__`이 구현되어 있으면 `__bool__`을, 아니라면 `__len__`을 호출한다.'
  - `__len__`도 구현되어있지 않으면 `bool(x)`의 값은 언제나 `True`이다.

### Collection 구현
- Iterable은 `for`, unpacking, iteration을 지원한다.
- Sized는 `len` built-in fuction을 지원한다.
- Container는 in 연산자를 지원한다.

- Sequence 타입은 str, list 등이 있다.
- Mapping 타입은 dict, collections.defaultdict 등이 있다.
- Set 타입은 set, frozenset 등이 있다.

## Overview of Special Methods
- [...SpecialMethods]

### len이 메소드가 아닌 이유.
- len은 built-in type에서 c struct의 길이를 얻어온다.
  - len이 메소드일 경우 이렇게 할 수 없다.
- non-built-in type에서는 `__len__`을 호출한다.
  - 일관성을 지키기 위함.
