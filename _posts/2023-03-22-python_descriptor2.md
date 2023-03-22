---
title:  "파이썬 디스크립터(2)"
excerpt: "파이썬 디스크립터의 사용법과 디스크립터 사용 시 유의해야하는 몇 가지 사항에 대해 알아본 후에 디스크립터를 사용하면 좋은 경우에 대해 살펴봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-03-22
last_modified_at: 2023-03-22
---
# 파이썬 디스크립터 2

## 디스크립터를 사용하지 않은 코드 vs 사용한 코드
디스크립터는 객체의 속성으로 들어가며 값을 반환 및 설정하기 때문에 `@property`와 유사하게 쓰일 수 있습니다.  

먼저 `@property`를 사용해 택배에 대한 현재 배송 상태와 배송 히스토리를 저장하는 코드를 작성하도록 하겠습니다.  
```py
class Parcel:
    def __init__(self, item, current_status):
        self.item = item
        self._current_status = current_status
        self._delivery_history = [current_status]

    @property
    def current_status(self):
        return self._current_status

    @current_status.setter
    def current_status(self, new_status):
        if new_status != self._current_status:
            self._current_status = new_status
            self._delivery_history.append(new_status)

    @property
    def delivery_history(self):
        return self._delivery_history


if __name__ == "__main__":
    pants = Parcel("pants", "배송 준비중")
    pants.current_status = "출고 완료"
    pants.current_status = "배송중"
    print(pants.delivery_history)   # ['배송 준비중', '출고 완료', '배송중']s
```
위 코드에서 Parcel 객체는 `current_status`와 `delivery_history`라는 property를 가지며, `current_status`에 대한 setter를 구현하여 새로운 값으로 `current_status`가 변경되었을 때 `delivery_history`에 신규 상태를 append합니다.  

이러한 로직이 한 군데에서만 사용된다면, `@property` 사용으로도 충분합니다.  
그러나 이런 상태 추적 로직이 여러 군데에서 필요하다면, 같은 로직의 반복을 줄이기 위해 디스크립터로의 전환을 고려해보는 것이 좋습니다.  

따라서 값이 변경될 때마다 변경 히스토리를 추적하여 리스트에 저장하는 descriptor를 생성해보도록 하겠습니다.  
이 때 디스크립터는 `Parcel`에만 한정되어 사용되는 것이 아니라 범용적으로 동작할 수 있도록 할 것이므로 특정 상황에 국한되지 않는 일반적인 이름을 사용하도록 하겠습니다.  
```py
class HistorySaveAttribute:
    def __init__(self, history_list_name):
        self.history_list_name = history_list_name
        self._name = None

    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]

    def __set__(self, instance, value):
        if not instance.__dict__.get(self._name):       # 속성이 아직 설정 되어 있지 않은 경우
            instance.__dict__[self._name] = value

        if not instance.__dict__.get(self.history_list_name):   # 변경 사항 저장 리스트 초기화가 아직 안된 경우
            instance.__dict__[self.history_list_name] = [instance.__dict__[self._name]]

        if instance.__dict__[self._name] != value:
            instance.__dict__[self._name] = value
            instance.__dict__[self.history_list_name].append(value)


class Parcel:
    current_status = HistorySaveAttribute('delivery_history')

    def __init__(self, item, current_status):
        self.item = item
        self.current_status = current_status


if __name__ == "__main__":
    pants = Parcel('pants', '배송 준비중')
    pants.current_status = "출고 완료"
    pants.current_status = "배송중"
    print("pants.current_status:", pants.current_status)  # pants.current_status: 배송중
    print("pants.delivery_history:", pants.delivery_history)    # pants.delivery_history: ['배송 준비중', '출고 완료', '배송중']
```
이 코드는 앞에서 작성했던 `@property`를 이용한 코드와 동일하게 `current_status`가 바뀔 때마다 바뀐 값을 추적하여 `delivery_history`에 append하는 기능을 수행합니다.  
그러나 값을 추적하는 로직은 더 일반적인 이름으로 생성된 `HistorySaveAttribute`에 들어있고, `Parcel` 클래스의 로직은 몹시 간단해진 것을 확인할 수 있습니다.  

## 디스크립터 전역 상태 공유 이슈
디스크립터는 무조건 클래스 속성으로 설정되어야 합니다.  
그런데 클래스 속성으로 설정되었기 때문에 따라오는 부작용이 있습니다.  
바로 해당 클래스의 모든 인스턴스들에 디스크립터의 상태가 전역적으로 공유된다는 문제입니다.  

### 전역 상태 공유 예시 코드
이 문제를 확인해 보기 위해 아래 코드를 살펴보도록 하겠습니다.  
```py
class Descriptor:
    def __init__(self, value):
        self.value = value

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.value

    def __set__(self, instance, value):
        self.value = value


class ClientClass:
    descriptor = Descriptor("value 1")


if __name__ == "__main__":
    client1 = ClientClass()
    client2 = ClientClass()

    print("client1.descriptor:", client1.descriptor)    # client1.descriptor: value 1
    print("client2.descriptor:", client2.descriptor)    # client2.descriptor: value 1

    client2.descriptor = "client 2 value"

    print("\n####After changing value of client2.descriptor####")
    print("client1.descriptor:", client1.descriptor)    # client1.descriptor: client 2 value
    print("client2.descriptor:", client2.descriptor)    # client2.descriptor: client 2 value
```
여기에서 `client2.descriptor`로 분명히 `client2` 인스턴스에 대한 descriptor의 값을 변경했는데, 변경 이후 값을 확인해본 결과 `client1` 인스턴스의 descriptor 값도 변경되어 있음을 확인할 수 있습니다.  
디스크립터가 클래스 속성이므로 각각의 인스턴스들에게 전역적으로 공유되고 있기 때문에 벌어지는 현상입니다.  

따라서 이 문제를 해결하기 위해서는 디스크립터는 각 인스턴스의 값을 보관했다가 반환해야합니다.  
이 전의 예시 코드에서는 각 인스턴스의 `__dict__` 사전에 값을 설정하고 조회하는 방법을 이용했습니다.  

`__get__`의 경우 아래 코드와 같이 instance의 `__dict__`에서 디스크립터의 값을 조회해왔습니다.  
```py
def __get__(self, instance, owner):
    if instance is None:
        return self
    return instance.__dict__[self._name]
```

`__set__`의 경우에도 아래 코드처럼 instance의 `__dict__`에 값을 저장했습니다.  
```py
def __set__(self, instance, value):
    instance.__dict__[self._name] = value
```


### 약한 참조 사용
인스턴스의 `__dict__`에 속성값을 저장하고 해당 내용을 조회하는 것으로 전역 상태 공유 문제는 충분히 해결이 가능하지만, 다른 방법도 존재합니다.  
바로 약한 참조(weakref)를 사용하는 방법입니다.  

만약 디스크립터에서 어떤 인스턴스에 어떤 값을 할당해야하는 지 기억해두고 있기 위해 각 인스턴스를 직접 참조한다면 어떻게 될까요?  
인스턴스는 클래스 속성으로 디스크립터를 이미 참조하고 있는 상황이기 때문에 순환 참조 문제가 나타날 것입니다.  
이렇게 되면 순환 종속성으로 인해 가비지 컬렉션이 되지 않는 문제 등이 발생합니다.  

따라서 weakref 모듈을 이용하여 약한 키를 가진 참조를 이용해야합니다.  
약한 참조를 사용하여 코드를 작성하면 아래와 같은 모양이 됩니다.  
```py
from weakref import WeakKeyDictionary


class Descriptor:
    def __init__(self, value):
        self.value = value
        self.mapping = WeakKeyDictionary()

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.mapping.get(instance, self.value)   # 해당 인스턴스 찾을 수 없으면 기본값 self.value 반환

    def __set__(self, instance, value):
        self.mapping[instance] = value


class ClientClass:
    descriptor = Descriptor("value 1")


if __name__ == "__main__":
    client1 = ClientClass()
    client2 = ClientClass()

    print("client1.descriptor:", client1.descriptor)    # client1.descriptor: value 1
    print("client2.descriptor:", client2.descriptor)    # client2.descriptor: value 1

    client2.descriptor = "client 2 value"

    print("\n####After changing value of client2.descriptor####")
    print("client1.descriptor:", client1.descriptor)    # client1.descriptor: value 1
    print("client2.descriptor:", client2.descriptor)    # client2.descriptor: client 2 value
```
이처럼 약한 참조를 통해 각 인스턴스별로 어떤 값이 매핑해있는 지를 디스크립터 내에서 관리할 수 있습니다.  

그러나 약한 참조를 사용하는 경우 아래와 같은 단점이 있습니다.  
1. 데이터를 객체가 아닌 디스크립터가 결과적으로 소유하게 됩니다.  
    - 이에 따라 객체의 사전에서 완전한 데이터를 반환받을 수 없게 됩니다.  
2. 각 객체는 `__hash__` 메서드를 구현해 해싱이 가능해야합니다.
    - 해싱이 불가능한 경우 `WeakKeyDictionary`의 key에 매핑이 불가능합니다.  

## 디스크립터를 사용하면 좋은 경우
디스크립터를 구현하고 사용하는 법에 대해 알아보았으니, 어떤 경우에 디스크립터를 사용하는 것이 적절한 지 알아보도록 하겠습니다.  
디스크립터는 재사용이 많이 되는 부분에 코드 중복을 줄이기 위해 사용될 수 있고, 더 높은 수준의 추상화를 제공하기 위해 구현될 수 있습니다.  

### 1. 프로퍼티가 필요한 구조가 반복되는 경우  
프로퍼티가 필요한 구조가 반복되는 경우는 디스크립터를 사용하기 적합합니다.  
`@property` 데코레이터는 `__get__`, `__set__`, `__delete__`를 구현한 디스크립터입니다.  
따라서 디스크립터는 `@property`보다 응용되어 사용될 수 있습니다.  

### 2. 클래스 메서드에도 동작하는 데코레이터를 만드는 경우  
데코레이터의 경우 함수에 동작하도록 데코레이터를 생성하고 추가적인 구현을 하지 않으면 클래스의 메서드에서는 해당 데코레이터를 사용할 수 없습니다.  
따라서 데코레이터로 동작할 클래스를 `__get__` 메서드를 구현한 디스크립터로 구현하여 함수와 클래스의 메서드 모두에서 동작하도록 구현하는 것이 좋습니다.  
클래스 메서드에서도 동작하는 데코레이터를 만드는 예제는 [이전 포스트](https://yunyun3599.github.io/python/python_descriptor/)에서 확인할 수 있습니다.  

### 3. 클라이언트가 사용하는 내부 API에 대해 디스크립터 사용
디스크립터에는 비즈니스 로직이 없고 구현 코드만 있는 것이 적합합니다.  
따라서 비즈니스 로직에서 사용되는 객체나 데이터 구조를 정의할 때 디스크립터를 유용하게 사용할 수 있습니다.  

## 파이썬에서 사용되는 디스크립터 살펴보기
마지막으로 디스크립트의 훌륭한 사용 예제를 알아보기 위해 파이썬 자체에서 구현되고 사용되는 디스크립터에 대해 알아보도록 하겠습니다.  
파이썬에서 사용되는 디스크립터의 좋은 예시 중 하나는 바로 `함수`입니다.  

파이썬의 함수는 `__get__` 메서드를 구현했기 때문에 클래스 안에서 메서드처럼 동작이 가능합니다.  
메서드는 첫번째 파라미터로 `self`라는 추가 파라미터를 가진 함수인데, 이 떄 `self`는 메서드를 소유하고 있는 클래스의 인스턴스를 의미합니다.  

따라서 `메서드(self, ...)`의 형태는 `함수(instance(), ...)`와 동일합니다.  

### 함수와 객체의 바인딩
파이썬에서 함수를 메서드로 변환한다는 것은 함수를 객체에 바인딩한다는 것입니다.  
그리고 파이썬에서는 이 변환 작업을 디스크립터를 이용해 수행합니다.  

메서드가 객체에 바인딩된 함수임을 보여주는 예시 코드를 확인해보도록 하겠습니다.  
```py
class Example:
    def __init__(self, x):
        self.x = x

    def method(self):
        self.x = self.x + 1
        return self.x


if __name__ == "__main__":
    example = Example(1)
    print(example.method())         # 결과 2
    print(Example.method(example))  # 결과 3
```
이 코드에서 `example.method()`와 `Example.method(example)`는 동일하게 동작합니다.  
그 이유는 위의 두 구문은 사실 디스크립터의 도움을 받아 내부적으로 변환된 구문일 뿐 하는 일이 동일하기 때문입니다.   

Example 클래스 내의 `method`는 클래스의 속성으로 정의된 객체인데, 함수이므로 `__get__`메서드가 구현되어 있으므로 `__get__`메서드가 먼저 호출되어 함수를 메서드로 변환하게 됩니다.   
즉 디스크립터는 여기에서 함수를 객체의 인스턴스에 바인딩하는 역할을 하는 것입니다.  

### 디스크립터를 이용해 호출 가능한 객체 생성하기
앞서서 살펴본 것처럼 파이썬의 함수는 디스크립터를 이용해 구현되었기 때문에 메서드로도 동작 가능합니다.  
함수가 어떤 식으로 구현되었기에 이런 일이 가능한지를 알아보기 위해 직접 callable한 객체를 디스크립터를 이용해 구현하여 객체 내외부에서 모두 사용 가능함을 살펴보도록 하겠습니다.  
```py
from types import MethodType
"""
<MethodType>
MethodType(callable, instance) 형태로 쓰임
- 첫번째 파라미터는 호출 가능한 객체여야 함 (이 예제에서 Method 객체는 __call__을 구현하였으므로 호출 가능한 객체임)
- 두번쨰 파라미터는 함수를 바인딩 할 객체
"""


class Method:
    def __init__(self, name):
        self.name = name

    def __call__(self, instance, arg1, arg2):
        print(f"{self.name}: {instance} 호출, arg1 = {arg1}, arg2 = {arg2}")

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return MethodType(self, instance)   # MethodType을 이용해 함수 -> 메서드 변환


class SampleClass:
    method = Method("객체 내부에서 초기화")


if __name__ == "__main__":
    instance = SampleClass()
    Method("객체 외부에서 초기화")(instance, '인자 1', '인자 2')
    instance.method('인자 1', '인자 2')     # Method 객체에 __get__을 구현하지 않으면 오류남.
```
위 코드에서 Method 객체는 `__get__` 메서드를 구현한 디스크립터입니다.  
이 때 `__get__` 메서드는 객체 외부에서 함수 형태로 호출되었을 때는 그냥 객체 자체를 반환합니다.  
이와 다르게 객체 내부에서 메서드 형태로 호출되었을 때는 self(=Method라는 호출 가능한 객체)를 인스턴스에 바인딩하여 반환합니다.  
파이썬에서 함수를 구현한 것처럼 그 자체로도 동작하고, 인스턴스의 메서드 형태로도 동작하는 객체를 만든 것입니다.  

만약 `__get__` 메서드를 구현하지 않았다면 `instance.method('인자 1', '인자 2')` 은 아래 오류 메세지와 함께 동작하지 않을 것입니다.  
>TypeError: \_\_call\_\_() missing 1 required positional argument: 'arg2'   
`__call__` 메서드는 첫번쨰 인자로 `self`를 받는데 따로 바인딩 된 인스턴스 없이 그냥 호출했기 때문에 `self` 자리에 `instance`가, `instance`자리에 `인자 1`이, `arg1`자리에 `인자 2`가 전달되고 `arg2`에는 전달된 값이 없어 오류가 발생한 것입니다.  

따라서 `__get__` 메서드를 구현해 디스크립터로 해당 객체를 생성하고, 인스턴스 내부에서 메서드 형태로 호출되었을 때는 인스턴스와 Method 객체를 바인딩해 반환해주는 로직이 필요한 것입니다.  
