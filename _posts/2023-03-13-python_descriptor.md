---
title:  "파이썬 디스크립터(1)"
excerpt: "파이썬 디스크립터가 무엇인지 알아보고, 구현해야 하는 매직 메서드에 대해 알아봅니다. "

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-03-13
last_modified_at: 2023-03-13
---
# 파이썬 디스크립터

디스크립터는 파이썬의 기능 중 하나로 다음 4가지 매직 메서드 중 최소 한 개 이상을 구현한 클래스를 의미합니다.  
1. \_\_get\_\_
2. \_\_set\_\_
3. \_\_delete\_\_
4. \_\_set_name\_\_

디스크립터 객체는 다른 객체의 속성으로 정의될 수 있으며, 해당 속성에 대한 읽기, 쓰기, 삭제 연산을 할 때 구현된 매직 메서드가 호출됩니다.  

디스크립터 구현을 위해 필요한 클래스로는 2가지가 있습니다.  
1. ClientClass: 디스크립터로 구현된 기능을 사용할 구현체 (class 속성으로 descriptor 인스턴스를 가짐)
2. DescriptorClass: 디스크립터 로직의 구현체

디스크립터 객체를 사용하기 위해서는 사용하는 클래스에서 디스크립터 객체를 class 속성으로 가지고 있어야합니다.   
따라서 아래와 같은 형태는 정상적으로 동작합니다.  
```py
class ClientClass:
    descriptor = DescriptorClass()
```
그러나 아래 코드는 정상적으로 동작하지 않습니다.  
```py
class ClientClass:
    def __init__(self):
        self.descriptor = DescriptorClass() 
```

디스크립터의 사용법을 알았으니, 위에서 언급한 4개의 매직 메소드를 구현해보도록 하겠습니다.  
## 디스크립터 구현 매직 메소드
### \_\_get\_\_(self, instance, owner)
\_\_get\_\_을 구현한 디스크립터 객체와 디스크립터 객체를 사용할 client 객체 코드는 아래와 같습니다.  

```py
class DescriptorClass:
    def __get__(self, instance, owner):
        if instance is None:
            return f"Instance is None: {self.__class__.__name__}, {owner.__name__}"
        return f"Call descriptor from {instance}"
    

class ClientClass:
    descriptor = DescriptorClass()
```

\_\_get\_\_메서드의 파라미터는 `(self, instance, owner)`가 있는데요, 해당 메서드가 객체 내에 있기 때문에 self를 우선 첫번째 파라미터로 받습니다.  
그리고 추가적으로 instance와 owner를 받는데요, 이 두 파라미터는 아래와 같은 데이터를 넘겨주게 됩니다.  
1. instance: 디스크립터를 호출한 인스턴스 (위 코드의 경우에는 ClientClass의 인스턴스)
2. owner: 디스크립터를 호출한 객체의 class (위 코드의 경우에는 ClientClass)

따라서 위 코드에 대해 아래와 같은 호출을 하면 서로 다른 결과를 얻게 됩니다.  
```py
print(ClientClass.descriptor)     # Instance is None: DescriptorClass, ClientClass
print(ClientClass().descriptor)   # Call descriptor from <__main__.ClientClass object at 0x100e54f10>
```

위의 경우로 미루어 보아 instance가 None이 경우 instance.__class__로 owner 값을 구할 수 없기 때문에 해당 경우에 대처하기 위해 파라미터로 owner를 따로 받는다는 것을 알 수 있습니다.  
통상적으로 `instance = None`이고 따로 owner값을 쓸 필요가 없다면 주로 그냥 `self`를 반환합니다. 

### \_\_set\_\_(self, instance, value)
\_\_set\_\_ 메서드는 디스크립터에 값을 할당하려고 할 때 호출합니다.  
디스크립터에 값을 할당하는 것은 \_\_set\_\_ 메서드를 구현한 경우에만 유효합니다.  
만약 \_\_set\_\_ 메서드를 구현하지 않았는데 `client.descriptor = "new_value"` 형식으로 값을 할당하면 descriptor 자체를 덮어쓰게 됩니다.  

\_\_set\_\_ 메서드는 데이터 저장 시에 호출되므로 속성의 유효성을 검사하는 로직 등을 위치시키기에 좋습니다.  
유사한 역할을 하는 것으로는 `@property.setter`가 있습니다.  

\_\_set\_\_메서드를 구현한 예시 코드를 살펴보도록 하겠습니다.  
이 예제에서 Book 객체는 seat이라는 이름의 디스크립터를 갖습니다.  
Seat 객체의 \_\_set\_\_ 메서드는 value로 (row, col) 튜블을 받아 해당 위치의 자리를 예매 완료된 상태로 변경합니다.  
```py
class OccupiedSeatException(Exception):
    pass


class Seat:
    def __init__(self, row, col, name=None):
        self._name = name
        self.row = row
        self.col = col
        self.seat = [[False] * self.col for i in range(self.row)]

    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        formatted_seat = ""
        for i in self.seat:
            formatted_seat += f"{str(i)}\n"
        return formatted_seat

    def check_valid_seat(self, target_row, target_col):
        if target_row < 0 or target_col < 0 or target_row >= self.row or target_col >= self.col:
            raise ValueError(f"올바르지 않은 좌석 번호입니다. (가능한 좌석행: 0행-{self.row - 1}, 가능한 좌석열: 0열-{self.col - 1})")
        if self.seat[target_row][target_col]:
            raise OccupiedSeatException("이미 선점된 좌석입니다.")
        return True

    def __set__(self, instance, value: tuple):
        target_row = value[0]
        target_col = value[1]
        if self.check_valid_seat(target_row, target_col):
            self.seat[target_row][target_col] = True


class Book:
    seat = Seat(5, 5)
```
\_\_set\_\_ 메서드에서는 예매하려는 좌석이 유효한 범위 내에 있는 지와 아직 예매가 되지 않았는 지 여부를 확인한 후 조건이 성립되면 좌석의 해당 위치 상태를 변경합니다.  


위 코드를 이용해 (1, 2) 위치의 좌석을 예매하면 다음과 같은 결과를 확인할 수 있습니다.  
```py
book = Book()
print(book.seat)
"""
결과
[False, False, False, False, False]
[False, False, False, False, False]
[False, False, False, False, False]
[False, False, False, False, False]
[False, False, False, False, False]
"""
book.seat = (1, 2)
print(book.seat)
"""
결과
[False, False, False, False, False]
[False, False, True, False, False]
[False, False, False, False, False]
[False, False, False, False, False]
[False, False, False, False, False]
"""
book.seat = (1, 2)
"""
결과로 아래 오류 발생
__main__.OccupiedSeatException: 이미 선점된 좌석입니다.
"""
```

### \_\_delete\_\_(self, instance)
\_\_delete\_\_ 메서드는 디스크립터 속성인 self와 client인 instance를 받습니다.  
\_\_delete\_\_를 통해 객체에서 속성을 제거하는 역할을 수행할 수 있습니다.  
\_\_delete\_\_ 메서드를 동작시키는 방법은 아래와 같습니다.  
```py
del client.descriptor
```

\_\_delete\_\_ 메서드를 구현한 코드를 살펴보기 위해 위의 \_\_set\_\_ 메서드의 예제를 좀 더 발전시켜 보도록 하겠습니다.  
이 예제에서 디스크립터는 좌석 예매를 취소하려고 할 때 해당 좌석의 예매자가 취소 요청자와 동일한 지를 확인한 후에 예매 취소를 진행합니다.  
```py
class SeatException(Exception):
    pass


class Seat:
    def __init__(self, row, col, name=None):
        self._name = name
        self.row = row
        self.col = col
        self.seat = [[False] * self.col for i in range(self.row)]

    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        formatted_seat = ""
        for i in self.seat:
            formatted_seat += f"{str(i)}\n"
        return formatted_seat

    def __set__(self, instance, value: tuple):  # 1
        user_id = value[0]
        target_row = value[1]
        target_col = value[2]
        if self.check_empty_seat(target_row, target_col) and self.check_seat_range(target_row, target_col):
            self.seat[target_row][target_col] = user_id

    def __delete__(self, instance): # 2
        user_id = instance.user_id
        row = instance.row
        col = instance.col
        if self.check_seat_range(row, col):
            if self.seat[row][col] is user_id:
                self.seat[row][col] = False
            else:
                raise SeatException("본인이 예매한 좌석이 아닙니다!")

    def check_seat_range(self, target_row, target_col):
        if target_row < 0 or target_col < 0 or target_row >= self.row or target_col >= self.col:
            raise ValueError(f"올바르지 않은 좌석 번호입니다. (가능한 좌석행: 0행-{self.row - 1}, 가능한 좌석열: 0열-{self.col - 1})")
        return True

    def check_empty_seat(self, target_row, target_col):
        if self.seat[target_row][target_col]:
            raise SeatException("이미 선점된 좌석입니다.")
        return True


class Book:
    seat = Seat(5, 5)

    def __init__(self): # 3
        self.user_id = None
        self.row = None
        self.col = None

    def book_seat(self, user_id, row, col): # 4
        self.user_id = user_id
        self.row = row
        self.col = col
        self.seat = (self.user_id, self.row, self.col)

    def cancel_seat(self, user_id, row, col): # 5
        self.user_id = user_id
        self.row = row
        self.col = col
        del self.seat
```
위의 예시와 크게 달라진 부분 4군데를 주석으로 넘버링해두었습니다.  
각 번호별로 무엇이 바뀌었는 지 확인해보도록 하겠습니다.  
1. 디스크립터 Seat) \_\_set\_\_ 메서드
    - value 파라미터를 기존 (row, col) 튜플에서 (user_id, row, col) 데이터를 가진 튜플로 변경하였습니다.  
    - 좌석이 예매되었을 때 해당 자리에 user_id를 저장하도록 변경하였습니다.  
2. 디스크립터 Seat) \_\_delete\_\_ 메서드
    - instance에 접근하여 현재 취소를 진행하고 있는 user_id 값과 취소하고자 하는 좌석의 row, col 값을 가져옵니다.  
    - 해당 좌석이 현재 인스턴스의 user_id 속성이 담고있는 값과 동일하다면 취소를 진행합니다.  
3. 클라이언트 객체 Book) \_\_init\_\_ 메서드
    - 초기화 시에 인스턴스 속성으로 `user_id, row, col` 값을 갖도록 하였습니다.  
    - 추후에 \_\_delete\_\_ 메서드에서 `instance.user_id` 형태로 속성에 접근하여 값을 사용하기 때문에 속성을 미리 정의해 둔 것입니다.  
4. 클라이언트 객체 Book) book_seat 메서드
    - 디스크립터의 \_\_set\_\_ 메서드를 호출하는 역할을 합니다.
5. 클라이언트 객체 Book) cancel_seat 메서드
    - 디스크립터의 \_\_delete\_\_ 메서드를 호출하는 역할을 합니다.
    - 인스턴스 속성 `user_id, row, col`을 업데이트하여 디스크립터에서 속성을 알맞게 조회할 수 있도록 합니다.  

위의 코드를 아래와 같이 실행시켜보면 의도한 결과를 얻을 수 있습니다.   
```py
    book = Book()
    print(book.seat)
    """
    초기 상태
    [False, False, False, False, False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    """
    book.book_seat('user_a', 1, 2)
    book.book_seat('user_b', 1, 3)
    print(book.seat)
    """
    좌석 예매
    [False, False, False, False, False]
    [False, False, 'user_a', 'user_b', False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    """
    book.cancel_seat('user_a', 1, 2)
    print(book.seat)
    """
    user_a 좌석 취소
    [False, False, False, False, False]
    [False, False, False, 'user_b', False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    [False, False, False, False, False]
    """
    book.cancel_seat('user_a', 1, 3)
    """
    user_a가 user_b의 좌석을 취소하려고 시도 -> 예외 발생
    __main__.SeatException: 본인이 예매한 좌석이 아닙니다!
    """
```


### \_\_set\_name\_\_(self, owner, name)
\_\_set\_name\_\_ 메서드는 디스크립터가 처리하려는 속성의 이름을 전달하는 역할을 합니다.  
\_\_set\_name\_\_를 구현한 디스크립터와 구현하지 않은 디스크립터를 함께 살펴보며 어떤 점이 다른 지 확인해보도록 하겠습니다.  
```py
class DescriptorWithoutSetName:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        instance.__dict__[self.name] = value


class DescriptorWithSetName:
    def __init__(self, name=None):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self.name]

    def __set_name__(self, owner, name):
        self.name = name

    def __set__(self, instance, value):
        instance.__dict__[self.name] = value


class ClientClass:
    descriptor_without_set_name = DescriptorWithoutSetName('descriptor_without_set_name')
    descriptor_with_set_name = DescriptorWithSetName()
```

코드를 확인해보면 \_\_set\_name\_\_을 구현하지 않은 DescriptorWithoutSetName은 초기화 시에 인자로 디스크립터가 처리하려는 속성명을 함께 전달합니다.  
이에 반해 DescriptorWithSetName은 name을 전달하지 않아도 괜찮습니다.  
따라서 \_\_set\_name\_\_을 구현하면 디스크립터 초기화 시에 속성명을 두 번 쓰지 않아도 된다는 장접이 있습니다.  

위 코드를 아래처럼 동작시켜보면 정상적으로 잘 동작하는 것을 확인할 수 있습니다.  
```py
client = ClientClass()
client.descriptor_without_set_name = 'without value'
client.descriptor_with_set_name = 'with value'

print(client.descriptor_with_set_name)  # with value
print(client.descriptor_without_set_name)   # without value
```

## 디스크립터 유형
앞에서 디스크립터 프로토콜을 작동시키는 여러 매직 메서드에 대해 알아보았습니다.  
디스크립터는 이 매직 메서드 중 무엇을 구현하느냐에 따라 크게 두 가지 종류로 나눌 수 있는데요, 각각 데이터 디스크립터와 비데이터 디스크립터입니다.  
1. 데이터 디스크립터: \_\_set\_\_이나 \_\_delete\_\_ 메서드를 구현한 디스크립터
2. 비데이터 디스크립터: \_\_get\_\_만 구현한 디스크립터

참고로 \_\_set\_name\_\_ 메서드의 구현 여부는 구분에 영향을 주지 않습니다.  

또한 객체에서 동일한 이름의 키를 갖는 경우 호출하는 우선 순위는 아래와 같습니다.  
> 비데이터 디스크립터 < 객체의 사전 < 데이터 디스크립터  

따라서 만약 비데이터 디스크립터의 이름과 동일한 이름을 키로 갖는 속성이 객체의 사전에 등록되어있다면, 비데이터 디스크립터는 영영 호출되지 않게 됩니다.  
동일하게 객체의 사전의 키와 데이터 디스크립터의 이름이 갖다면 역시나 객체의 사전의 값은 영영 호출되지 않게 됩니다.  

비데이터 디스크립터를 구현해보고, 정말로 객체의 사전보다 우선순위가 뒤인지 확인해보도록 하겠습니다.  
```py
class NonDataDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return "descriptor value"


class ClientClass:
    descriptor = NonDataDescriptor()


if __name__ == "__main__":
    client = ClientClass()

    print("###### 1. 초기 상태 ######")
    print(client.descriptor)  # 디스크립터 __get__ 메서드 리턴값 출력 -> 'descriptor value'
    print(vars(client))  # client 객체의 사전 조회 -> {}

    print("\n###### 2. 객체 사전에 동일한 디스크립터와 동일한 이름으로 속성 등록 ######")
    client.descriptor = "new_val"  # 객체의 사전에 속성 등록
    print(vars(client))  # client 객체의 사전 조회 -> {'descriptor': 'new_val'}
    print(client.descriptor)  # 객체의 속성 조회 -> 'new_val'

    print("\n###### 3. 객체 속성 삭제 ######")
    del client.descriptor  # 객체의 속성 삭제
    print(vars(client))  # client 객체의 사전 조회 -> {}
    print(client.descriptor)  # 다시 비데이터 디스크립터 값 조회 -> 'descriptor value'
```

코드를 동작시켜보면 `client.descriptor = "new_val"` 형태로 속성을 세팅하면 기존 비데이터 디스크립터 값이 나오지 않는 것을 알 수 있습니다.  
그 이유는 디스크립터에 \_\_set\_\_ 메서드를 구현하지 않았기 때문에 객체의 속성으로 해당 값이 설정이 되기 때문입니다.  
앞서 말했듯 객체의 사전은 비데이터 디스크립터보다 우선순위가 높기 때문에 객체의 사전에 할당된 값이 출력되게 됩니다.  

이번에는 \_\_set\_\_ 메서드가 구현된 데이터 디스크립터를 살펴보도록 하겠습니다.  
```py
class DataDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return "descriptor value"

    def __set__(self, instance, value):
        instance.__dict__['descriptor'] = value


class ClientClass:
    descriptor = DataDescriptor()


if __name__ == "__main__":
    client = ClientClass()
    print(client.descriptor)    # descriptor value

    client.descriptor = "new_value"
    print(client.descriptor)    # descriptor value
    print(vars(client))         # {'descriptor': 'new_value'}

    del client.descriptor       # Error
```
코드에서 데이터 디스크립터는 \_\_set\_\_ 메서드를 통해 객체의 사전에 값을 할당합니다.  
그러나 여기서 할당한 값은 `client.descriptor` 형태로는 조회할 수 없는데요, 이는 앞서 말했듯이 데이터 디스크립터의 우선순위가 객체의 사전보다 높기 때문입니다.  
따라서 `vars(client)`를 통해 `{'descriptor': 'new_value'}`를 얻어 객체의 사전에 새로운 값이 등록되어 있음을 확인할 수는 있지만 `client.descriptor`를 조회하면 항상 디스크립터의 \_\_get\_\_ 리턴 값인 `descriptor value`를 얻게 됩니다.  