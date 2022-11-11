---
title:  "파이썬 - Docstring, 어노테이션"
excerpt: "파이썬의 docstring과 annotation에 대해 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-11-11
last_modified_at: 2022-11-11
---

# Docstring
docstring은 코드를 설명하는 문서라고 볼 수 있습니다.  
docstring을 잘 작성해 놓으면 코드의 이해도를 높일 수 있고, 이는 더 나은 협업, 유지보수 등으로 이어질 수 있습니다.  

## 그렇다면 docstring과 주석의 차이는 무엇일까요?  
  
|📝|docstring|주석|
|---|:---:|:---:|
|권장 여부|O|X|
|종류|문서|코멘트|
|사용 이유|코드에 대한 문서화를 위해|코드만으로 내용 설명이 불충분해서|
|코드 변경 시|docstring을 기반으로 코드 파악|주석의 내용과 실제 동작을 비교해야 함|  

즉, docstring은 코드를 문서화하기 위해 전반적인 틀이나 모듈에 대한 내용을 기입해 두는 것입니다.  
반면 주석은 코드 만으로 내용 설명이 불충분하기 때문에 내용에 대한 부가설명을 위해 달아두는 것입니다.  

## docstring의 예  
표준 라이브러리에서도 docstring은 활발하게 사용되고 있습니다.  
예를 들어 dict 라이브러리의 docstring을 살펴보겠습니다. 

```python
dict.update??
```
파이썬을 대화형으로 실행하거나 주피터 노트북 같은 곳에서 위의 코드를 치고 실행시키면 아래와 같은 결과를 얻을 수 있습니다.
>'D.update([E, ]**F) -> None.  Update D from dict/iterable E and F.\nIf E is present and has a .keys() method, then does:  for k in E: D[k] = E[k]\nIf E is present and lacks a .keys() method, then does:  for k, v in E: D[k] = v\nIn either case, this is followed by: for k in F:  D[k] = F[k]'

결과로 나온 내용이 dict.update 함수의 docstring 내용이 됩니다.
![](/assets//img//2022-11-11-docstring_and_annotation_1.png)

위 코드는 dict 문서에서 update 함수 부분인데, 위에서 출력된 내용과 동일한 내용의 docstring이 작성되어 있음을 확인할 수 있습니다. 

dict.update의 docstring의 내용은 다음과 같습니다.  
1. .keys() 메소드를 가진 파라미터가 들어오면 이 내용을 dict에 업데이트 해 준다
3. .keys() 메소드가 없는 경우에는 쌍으로 된 이터러블이 들어오면 이를 풀어서 dict에 추가해 준다  

## docstring이 포함된 함수 만들어보기
docstring이 정의되어 있다면 객체의 .\_\_doc\_\_ 속성을 통해 docstring의 내용을 확인할 수 있습니다.  
아래는 time 타입으로 시간을 입력하면 [morning, afternoon, evening, night] 중 어디에 속하는 지를 문자열로 반환하는 함수입니다.

```python
from enum import Enum
from datetime import time


class TimeZone(Enum):
    morning = "morning"
    afternoon = "afternoon"
    evening = "evening"
    night = "night"


def get_time_zone(input_time: time) -> str:
    """
    시간을 받으면 5시 ~ 12까지는 morning, 
    12시 ~ 17시까지는 afternoon, 
    17시 ~ 21시까지는 evening, 
    21시 ~ 5시 까지는 night을 반환한다.

    :param input_time: 시간대 문자열로 변경을 원하는 시간
    :return: 시간대 문자열
    """
    if type(input_time) != time:
        raise TypeError("input type must be time")

    if input_time.hour < 5:
        return TimeZone.night.value
    elif input_time.hour < 12:
        return TimeZone.morning.value
    elif input_time.hour < 17:
        return TimeZone.afternoon.value
    elif input_time.hour < 21:
        return TimeZone.evening.value
    else:
        return TimeZone.night.value


print(get_time_zone.__doc__)
print("9:00 is " + get_time_zone(time(9, 00, 00)))
print("14:00 is " + get_time_zone(time(15, 00, 00)))
print("20:00 is " + get_time_zone(time(20, 00, 00)))
print("3:00 is " + get_time_zone(time(3, 00, 00)))
```

>"""   
\~\~\~   
"""  
 
로 묶여있는 부분이 docstring인데요, 위의 파일을 실행시켜보면 아래와 같은 결과를 얻을 수 있습니다.  



>시간을 받으면 5시 ~ 12까지는 morning,  
12시 ~ 17시까지는 afternoon,  
17시 ~ 21시까지는 evening,  
21시 ~ 5시 까지는 night을 반환한다.  
<br/>
:param input_time: 시간대 문자열로 변경을 원하는 시간   
:return: 시간대 문자열 
<br/>  
9:00 is morning 
14:00 is afternoon 
20:00 is evening 
3:00 is night 

밑의 4줄은 프린트문으로 함수를 수행시켜본 결과이고, 위의 6줄이 docstring의 내용을 프린트 한 부분입니다.  
<br/>
<hr/> 

# 어노테이션 
어노테이션은 코드 사용자에게 함수의 파라미터로 어떤 값이 와야하는 지에 대하여 명시하는 부분입니다.   

아래와 같이 학기에 열리는 강의 목록과 최소 수강해야 하는 학점을 받는 클래스를 만들어 보았습니다.  
```python
from typing import List


class Semester:
    lecture_list: List
    minimum: int

    def __init__(self, lecture_list: List, minimum: int):
        self.lecture_list = lecture_list
        self.minimum = minimum

    def can_register(self, lecture: str, credit: int) -> bool:
        """강의 수강이 가능한 지 bool 리턴"""
        if lecture in self.lecture_list and credit >= self.minimum:
            return True
        return False
```

`can_register` 함수의 경우 파라미터로 lecture와 credit을 받는데, 이 때 lecture는 `str`타입, credit 은 `int` 타입이라고 명시해 두었습니다.  
이렇게 명시해두면, 해당 함수가 어떠한 자료형을 파라미터로 받는 지 알기 쉬워, 코드를 파악하기가 용이해집니다.  

하지만 이 때 주의점은, 어노테이션을 작성하더라도 파이썬에서 <span style="background-color:#fff0ba">타입을 강제하고나 검사하지는 않는다는 점입니다.</span>  

어노테이션을 사용하면 \_\_annotations\_\_라는 특수한 속성이 생깁니다.  
이 속성은 어노테이션의 이름과 타입을 매핑한 dict 값입니다. 

```python
semester = Semester(['algorithm', 'network', 'database', 'software engineering', 'artificial intelligence'], 3)
print(semester.can_register.__annotations__)
```
위의 코드처럼 Semester 객체를 만든 후 can_register 함수의 \_\_anntations\_\_ 속성을 조회하면 아래와 같은 결과를 얻을 수 있습니다.  
>{'lecture': <class 'str'>, 'credit': <class 'int'>, 'return': <class 'bool'>}

또한 함수의 파라미터 뿐만 아니라 변수에도 직접 어노테이션을 달 수 있다.  
```python
class Semester:
    lecture_list: List
    minimum: int
```
이처럼 변수에 달은 주석도 아래와 같은 코드로 \_\_anntations\_\_ 속성을 확인할 수 있다.
```python
print(Semester.__annotations__)
``` 
>결과  
{'lecture_list': typing.List, 'minimum': <class 'int'>}
