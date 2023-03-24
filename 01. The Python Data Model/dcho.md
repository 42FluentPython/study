# Chapter 1. The Python Data Model

파이썬의 가장 좋은 특징 중 하나는 일관성이다. Python으로 잠시 작업한 후에는 새로운 기능에 대한 정보를 바탕으로 정확한 추측을 할 수 있다.

기본적인 객체 연산을 수행할 때, 파이썬 모델이라는 것이 제공하는 API 를 통해 파이썬 상용구를 적용할 수 있다.

이는 파이썬스러움(pythonic)의 핵심이다.

이 책에서 소개하는 `obj[key]`을 `__getitem__`으로 표현 하거나  `my_collection[key]`를 `my_collection.__getitem__(key)`으로 표현 하는 것을 Python의 special method, magic method, dunder method라고 하며 이를 명확하게 이해하면, 구현한 객체를 좀 더 다양하게 활용이 가능하다.

## Magic Method
- python이 내부적으로 구현된 (빌트인) 메소드를 구현함 (이미 만들어짐)
### 용도
- 수치형 연산자 모방
    
        연산자 오버로딩을 통해 
- 객체의 문자열 표현
        
        __repr__ 을 이용한다. 이는 repr() 내장 메서드에 의해 호출, 구현하지 않으면 객체의 정보가 출력이 된다.
        나는 django에서 __str__을 썻었는데 이 책에선 둘중 하나를 구현해야 한다면 __repr__을 구현하라고 한다. 그 이유는 python 인터프리터가 __str__ 가 구현되어 있지 않을 때, 자동으로 __repr__을 호출하기 때문이라고 한다.
        정리하자면 __repr__은 객체 자체를 표현하기 위한 목적이고 __str__는 문자화라는데 목적.
- 객체의 Boolean 값

        __bool__이 구현되어 있으면 __bool__을 따르지만 그렇지 않은 경우에는 __len__을 호출하여 0이 반환되면 False, 아닐 경우 True를 반환한다.
        __bool__이나 __len__이 구현되지 않은 경우, 기본적으로 사용자 정의 클래스의 객체는 참으로 간주한다. 
- collections 구현

        모든 컬렉션이 구현해야 하는 세 가지 필수 인터페이스를 통합한다.
        - Iterable은 for, unpacking 및 반복을 지원한다.
        - Sized는 len built-in function 지원한다.
        - Container는 in 연산자를 지원한다.

        컬렉션의 매우 중요한 전문 분야이다.
        - Sequence, list 및 str과 같은 built-ins 인터페이스 형식 지정.
        - dict, collections.default dict 등에 의해 구현된 매핑.
        - Set, 집합 및 고정 집합 built-ins 유형의 인터페이스.


## len이 메서드가 아닌 이유

특별히 len() 함수는 `__len__` 메서드를 통해 호출되지만, 자주 사용되는 연산이므로 내장 객체에 대해서 메서드를 호출하지 않고 C 구조체의 필드 값을 반환하는 함수로 작용한다. 따라서 len() 은 메서드가 아닌 함수로 특별 취급 받는다. 내장형 객체의 효율성을 위해 함수로 작용하지만 `__len__` 메서드를 통해 직접 정의할 수 있도록 함으로써 효율성과 일관성의 타협점을 찾은 것으로 볼 수 있다.



