---
title:  "파이썬의 상속"
excerpt: "파이썬에서 상속이 어떻게 사용되는 지와 상속을 사용하기 적절한 경우에 대해 알아봅니다. "

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-01-08
last_modified_at: 2023-01-08
---
# 파이썬의 상속

파이썬에서 상속은 아래와 같은 형태로 이루어집니다.  
```python
class ChildClass(ParentClass):
    ...
```

상속은 부모 클래스를 확장하여 더 특화된 기능을 개발할 일이 있을 때 주로 사용합니다.     

즉 상속을 사용하는 상황은 아래와 같은 경우입니다.   
```
부모 클래스: 일반적인 기능을 포함    
자식 클래스: 부모 클래스를 상속받으며 좀 더 특정한 상황에 사용
```

## 상속을 사용할 때 고려할 점
상속을 사용할 때는 그로 인한 장점과 단점을 모두 고려해야합니다.  
+ 장점) 부모 클래스의 속성과 메서드 등을 사용할 수 있어 중복된 코드를 줄일 수 있습니다.  
- 단점) 부모 클래스의 메소드이 모두 사용 가능해지면서 너무 많은 기능들이 한 번에 추가됩니다.  

위와 같은 사항을 고려해 봤을 때 무분별하게 상속을 사용하면 문제가 발생할 수 있습니다.  

왜냐하면 상속을 통해 부모 클래스를 확장한 자식 클래스는 부모 클래스와 **강하게 결합**되어 있기 때문입니다.  
-> 소프트웨어 설계 시 결합도는 최대한 줄이는 것이 좋습니다.  

따라서 상속을 하고자 할 때는 상속 받은 메서드를 모두 사용하게 되는가를 확인해 보는 것이 좋습니다.  


만약 대부분의 메서드를 미사용하거나 재정의해야 한다면 해당 코드는 아래와 같은 상황으로 인해 상속을 사용하지 않는 것이 더 적합할 수도 있습니다.  
>1. 상위 클래스가 막연하게 정의 되었거나 너무 많은 책임을 가진 것은 아닌 지   
>    (이런 경우 잘 정의된 인터페이스로 대체하는 것이 더 나을 수도 있습니다)
>2. 하위 클래스는 상위 클래스에 대해 적절한 세분화가 아닌 지

**결론적으로, 상속받은 클래스의 기능을 물려받으면서 추가 기능을 더하려고 하거나 특정 기능을 수정하려는 경우에는 상속이 적합합니다.**  


## 잘못된 상속의 예
상속을 잘못 사용하는 예시를 살펴보도록 하겠습니다.  
잘못된 상속의 예시로는 데이터 구조를 활용하여 객체를 생성하지 않고 데이터 구조 자체를 객체로 만들어 도메인을 처리하는 경우가 있습니다.   

즉, Dict라는 자료형을 **이용해** 객체를 만드는 것이 아니라 Dict 데이터 구조를 **상속해** 도메인을 처리하려고 하는 것입니다.  

아래 구현할 EmployeeInfo는 직원 이름을 key로 해당 직원의 정보를 가지고 있습니다.  
이 때 EmployeeInfo 클래스는 UserDict를 상속받아 구현해보도록 하겠습니다.  
```python
class EmployeeInfo(collections.UserDict):
    """데이터 구조를 도메인 객체로 만든 잘못된 상속의 예"""

    def update_employee_info(self, employ_name, **new_employee_info):
        self[employ_name].update(new_employee_info)
```
위의 코드처럼 데이터 구조를 직접 상속받으면 안쓰는 메서드들까지 해당 클래스에 포함되며, 이런 메서드들울 사용자가 사용하게 되면 예기치 못한 문제들이 발생할 수 있습니다.  

실제로 EmployeeInfo 객체를 이용해 정보를 저장 및 업데이트 해보고, 해당 객체에서 사용할 수 있는 메서드들을 조회해보도록 하겠습니다. 
```python
employee_info = EmployeeInfo({
    "Jack": {
        "salary": 50000,
        "department": "marketing"
    }
})
employee_info.update_employee_info("Jack", grade="S") 
# 결과: {'Jack': {'salary': 50000, 'department': 'marketing', 'grade': 'S'}}로 업데이트
print(dir(employee_info))   # 안쓰는 메서드 너무 많음
```
`dir(employee_info)`의 값은 아래와 같습니다.  
```
['_MutableMapping__marker', '__abstractmethods__', '__class__', '__contains__', '__copy__', '__delattr__', '__delitem__' '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__setattr__', '__setitem__', '__sizeof__', '__slots__', '__str__', '__subclasshook__', '__weakref__', '_abc_impl', 'clear', 'copy', 'data', 'fromkeys', 'get', 'items', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'update_employee_info', 'values']
```
사용하고자 하는 기능 이상으로 너무 많은 메서드들이 포함되었음을 확인할 수 있습니다.  

이 뿐만 아니라 EmployeeInfo라는 이름을 보고 해당 객체가 UserDict를 확장한 사전타입임을 알기 어렵다는 단점도 있습니다.  

상속이 적절하게 일어나려면 기본 클래스에 추가/확장하여 조금 더 특화된 것을 만드는 것이 목적이어야 합니다.  

그렇다면 위의 경우에는 어떤 식으로 코드를 짜는 것이 적절할까요?  
이럴 때는 "컴포지션"을 사용하는 것이 더 좋습니다.  

## 컴포지션을 이용한 예
`EmployeeInfo` 클래스를 컴포지션을 이용해 구현하면 코드는 아래와 같이 변경됩니다.  
```python
class EmployeeInfo():
    """도메인 구현을 상속이 아닌 컴포지션으로 대체한 예"""

    def __init__(self, employee_info):
        self._data = employee_info

    def update_employee_info(self, employ_name, **new_employee_info):
        self._data[employ_name].update(new_employee_info)

    def __getitem__(self, employ_name):
        return self._data[employ_name]

    def __len__(self):
        return len(self._data)
```

이처럼 UserDict를 직접 상속받는 대신 컴포지션을 이용하면 데이터 구조를 변경한다고 해도 인터페이스를 유지하면 사용자에게는 영향이 미치지 않는다는 장점이 있습니다.  
