---
title:  "파이썬 객체의 동적인 속성"
excerpt: "파이썬의 객체의 동적인 속성에 대해 공부하고, 예제를 통해 내용을 파악해봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-24
last_modified_at: 2022-12-24
---

# 파이썬 객체의 동적인 속성
파이썬에서 객체의 속성값에 접근할 때는 `<object명>.<attribute명>` 형식으로 접근합니다.  
예를 들어 다음과 같은 `SampleObject`를 생성했다고 합시다.  
```python
class SampleObject:
    def __init__(self, something):
        self.something = something
```
`SampleObject`는 `something`라는 속성을 갖습니다.  
따라서 위의 SampleObject 객체의 something 속성에 접근하기 위해서는 아래와 같이 코드를 작성하게 됩니다.  
```python
sample_object = SampleObject("something")
print(sample_object.something) # something
```

## \_\_getattribute\_\_와  \_\_getattr\_\_ 매직 메서드
### \_\_getattribute\_\_ 메서드
위의 예시에서 `something` 속성에 접근할 때 파이썬에서는 `sample_object`의 `__getattribute__` 매직 메서드를 호출하여 `__dict__ ` 값에 `something`이 있는 지 확인하고, 값을 가져옵니다.  
만약 값이 없다면 AttributeError가 발생합니다.  
```python
sample_object = SampleObject("something")
print(sample_object.something)  # something
print(sample_object.nothing)    # AttributeError
```
위 코드의 결과는 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-24-dynamic_attribute/attribute_error_from_getattribute.png)
`something` 속성은 존재하므로 속성의 값이 출력되었고, `nothing` 속성은 존재하지 않으므로 AttributeError가 발생하였습니다.  

### \_\_getattr\_\_ 메서드
앞에서 구현한 SampleObject에서는 존재하지 않는 속성값에 접근하려고 하면, AttributeError가 발생했습니다.  
이는 \_\_getattribute\_\_ 메서드를 수행한 결과 원하는 속성값을 찾지 못했기 때문입니다.  

만약 \_\_getattribute\_\_ 메서드에서 속성값을 찾지 못한 경우 바로 AttributeError를 발생시키지 않고, 추가적인 일을 수행하고 싶다면 어떻게 하는 게 좋을까요?  
이 때 바로 \_\_getattr\_\_ 매직 메서드를 사용할 수 있습니다.  

\_\_getattr\_\_ 메서드는 \_\_getattribute\_\_의 수행 결과 속성값을 찾지 못했을 때 추가적으로 호출됩니다.  
위의 `SampleObject`에 \_\_getattr\_\_ 메서드를 추가적으로 구현해보도록 하겠습니다.  
```python
class SampleObject:
    def __init__(self, something):
        self.something = something

    def __getattr__(self, item):
        print(f"{item} is an invalid attribute")
```

SampleObject에 없는 속성값에 접근하려고 했을 때 해당 속성값은 없다는 문장을 출력하도록 `__getattr__`를 구현하였습니다.  
```python
sample_object = SampleObject("something")
print(sample_object.something)  # something
print(sample_object.nothing)    # None
```
위의 코드를 다시 동작시키면 다음 결과가 나옵니다.  
![](/assets/img/2022-12/2022-12-24-dynamic_attribute/set_getattr_method.png)
`nothing` 이라는 존재하지 않는 속성에 접근하려 했을 때 `__getattr__` 메서드의 내용이 수행되는 것을 확인할 수 있습니다.  
> <p style="font-size: 0.8rem"> 정리해보면  <br>
<span style="background: yellow;">__getattribute__</span>: 객체의 속성에 접근할 때 호출. 객체 내에 해당 속성이 없으면 AttributeError 발생  <br>
<span style="background: yellow;">__getattr__ 메서드</span>: __getattribute__의 결과 속성이 없을 때 추가적으로 호출<br></p>

## 예제 만들어보기
위의 내용을 토대로 새로운 예제를 하나 생성해보도록 하겠습니다.  
1. 메인코드에서 빵 종류와 각 빵의 가격을 dict로 생성.
2. 메인코드에서 정의해둔 빵의 종류들을 `menu`라는 리스트형의 속성으로 갖는 `Bakery` 객체를 생성
3. bake 메서드를 통해 빵을 Bakery에 속성으로 추가 가능
4. menu에 없는 빵 속성을 가져오려고 하면 AttributeError 발생
5. menu에 있으나 아직 추가되지 않은 빵 속성을 가져오려고 하면 안내 메세지 출력
6. 속성으로 추가된 빵의 경우 가격을 리턴

먼저 Bakery class를 만들어보도록 하겠습니다.  
```python
class Bakery:
    def __init__(self, menu: list):
        self.menu = menu

    def bake(self, bread, price):
        self.__dict__[bread] = price
        print(f"{bread} is baked!!\n")

    def __getattr__(self, order):
        if order in self.menu:
            print(f"{order} is not baked yet")
            return 0
        else:
            raise AttributeError(f"We don't sell {order}.")
```
Bakery 객체를 초기화할 때 menu 리스트를 받아 판매할 빵의 종류를 정의합니다.  

bake 메소드를 통헤 아직 추가되지 않은 빵을 속성으로 추가할 수 있습니다.  

`__getattr__` 메서드를 통해 아직 속성값으로 추가되지 않은 빵 중 menu에 있는 빵은 아직 만들어지지 않았다는 메세지를 출력합니다.  
만약 menu에 없는 빵 속성에 접근하려고 한다면 `__getattr__` 메서드에서 AttributeError를 발생시킵니다.  

아래는 Bakery 객체를 생성/사용해보는 메인 코드입니다.  
```python
if __name__ == "__main__":
    menu_dict = {"croissant": 1.1, "bagel": 1.2, "baguette": 1.5, "brioche": 1.3, "muffin": 1.4}
    budget = 10

    bakery = Bakery(menu_dict.keys())

    ## 크로아상 만들어 지기 전에 주문
    print("##Order Croissant##")
    budget -= bakery.croissant
    print(f"left budget: {budget}\n")

    ## 크로아상 Bake
    print("##Bake Croissant##")
    bakery.bake("croissant", menu_dict["croissant"])

    ## 크로아상 주문
    print("##Order Croissant##")
    budget -= bakery.croissant
    print(f"left budget: {budget}\n")

    ## 메뉴에 없는 스콘 주문
    print("##Order Scon##")
    budget -= bakery.scon
```
croissant 조리 전에 주문을 한 번 해본 후 bake 메서드를 통해 croissant을 조리하고 다시 주문을 합니다.  
croissant 조리 전에 주문했을 때는 budget이 줄지 않고, bake 후에 주문했을 때는 줄어들었음을 확인할 수 있습니다.   
그리고 마지막으로 메뉴에 없는 scon을 주문하여 AttributeError를 발생시켜보도록 합니다.  

위 코드의 수행 결과는 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-24-dynamic_attribute/bakery_example.png)
