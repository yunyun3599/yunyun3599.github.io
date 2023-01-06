---
title:  "파이썬의 에러 핸들링"
excerpt: "파이썬에서 적절하게 에러를 처리하는 방법에 대해 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-01-05
last_modified_at: 2023-01-05
---
# 파이썬에서의 에러 핸들링
파이썬에서 발생한 에러를 처리하기 위해서 몇 가지 방법을 사용할 수 있습니다.  
이번 포스팅에서는 살펴볼 방법은 다음 두 가지 입니다.  
- 값 대체
- 예외 처리

## 값 대체  
값 대체는 말 그대로 결과 값을 안전한 다른 값으로 대체하는 방법입니다.  

예를 들면 dict 자료형의 경우, `<dict변수>.get('없는_key', '원하는_value')` 형태로 호출하면, 두번째 파라미터로 제공된 '원하는_value'를 반환합니다.  
만약 두 번째 파라미터를 주지 않는다면, dict 객체는 None을 반환합니다.  

아래 예시를 보도록하겠습니다.  
```python
>> sample_dict = {'a': 'A', 'b': 'B'}
>> print(sample_dict.get('b'))             # B
>> print(sample_dict.get('b', 'BB'))       # B
>> print(sample_dict.get('c', 'C'))        # C
>> print(sample_dict.get('c'))             # None
```
`sample_dict` 변수 내에는 키값으로 'a', 'b'가 있습니다.  
존재하는 key인 `"b"`에 대해 get을 수행했을 때는 두번째 파라미터가 있든 없든 dict 내에 저장되어있던 값인 `"B"`를 반환합니다.  
그러나 존재하지 않는 key인 `"c"` 대해 get을 수행하면 두번째 파라미터가 있는 경우 두번째 파라미터로 전댈된 값인 `"C"`를 반환하고, 없는 경우 None을 반환합니다.  

값 대체의 경우 프로그램이 실패하지 않는다는 장점이 있으나, 오류가 있는 값을 다른 값으로 대체하는 경우 오류를 숨긴다는 문제가 있습니다.  

## 예외 처리
예외 처리를 통해 에러를 핸들링하는 경우에는 예외 상황을 적절히 알려주면서 비즈니스 로직의 흐름을 유지합니다.  

그러나 이 때 예외는 **캡슐화를 약화**시키므로 신중하게 사용해야한다는 주의점이 있습니다.  
그렇다면 예외를 사용하면 왜 캡슐화가 약화될까요??  
예외는 호출자에게 문제를 알려주는 것으로, 예외의 개수가 많아지면 호출자가 호출한 함수에 대해 알아야 하는 정보도 증가하게 됩니다.  
따라서 캡슐화가 약화되는 것입니다.  

예외를 사용해서 문제를 처리할 때는 예외가 코드의 어느 부분에 속해야 하는 지도 고려해야합니다.  
예외는 한 가지 일을 하는 함수의 한 부분입니다.  
따라서 함수가 발생시키거나 처리하는 예외는 캡슐화된 로직과 일치해야합니다.  

예외가 적당하지 못한 위치에서 처리되는 상황과 적절한 위치에서 처리되는 상황을 예제를 통해 알아보겠습니다.  

### 예외 처리 나쁜 예
예제로 간단한 티겟팅 상황을 코딩해보도록 하겠습니다.  
구현할 객체는 결제 수단인 Payment 객체와 티켓팅 로직을 구현하는 Ticketing 객체 두 가지입니다.  

**Payment** 객체는 잔액 값을 속성으로 가지고 있습니다  
```python
class Payment:
    def __init__(self, balance):
        self._balance = balance

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, balance):
        self._balance = balance
```

**Ticketing** 객체는 좌석표를 가지고 있으며, 좌석 가격을 파라미터로 받아 초기화됩니다.  
reserve 메서드를 통해 예매를 진행하며, 선택 좌석의 예약 가능 여부를 가져온 뒤에 결제를 진행합니다.  
이 때 예약 불가능한 좌석을 선택하면 `NotValidSeatException`이 발생하고, 잔액이 부족하면 `NotEnoughBalanceException`이 발생합니다. 

```python
class Ticketing:
    def __init__(self, price):
        self.seat = [[True] * 5 for _ in range(5)]
        self.price = price

    def reserve(self, row: int, col: int, payment: Payment):
        try:
            self.get_seat(row, col)
            self.pay(payment)
            print(f"[{row}행 {col}열] 예매가 완료되었습니다. ")
        except NotValidSeatException:
            logger.info("예매 불가능한 좌석입니다.")
            raise
        except NotEnoughBalanceException:
            logger.info("잔액이 부족합니다.")
            raise

    def get_seat(self, row, col):
        if 0 < row < len(self.seat) and 0 < col < len(self.seat[row]) and self.seat[row][col]:
            self.seat[row][col] = False
        else:
            raise NotValidSeatException

    def pay(self, payment):
        if payment.balance < self.price:
            raise NotEnoughBalanceException
        payment.balance -= self.price
```

reserve 메서드를 보면 해당 메서드 내에서 `NotValidSeatException`와 `NotEnoughBalanceException`를 모두 처리하고 있습니다.  
그러나 저 두 메서드는 사실 별로 관계가 없는 적절하지 못한 위치에서 처리되고 있습니다.  

따라서 각 예외가 발생하는 위치를 조금 더 책임이 명확한 곳으로 옮길 필요가 있습니다.  

### 예외 처리 좋은 예
각 예외를 적합한 위치에서 발생시키기 위해 위와 동일한 기능을 하는 코드를 조금 수정해보도록 하겠습니다.  
좌석과 관련된 Seat 객체, 결제와 관련된 Payment 객체, 티켓팅을 수행하는 Ticketing 객체를 생성하도록 하겠습니다.  
앞에서는 예외를 모두 Ticketing 객체에서 발생시켰지만, 이번 예제에서는 `NotValidSeatException` 예외는 Seat 객체에서 발생시키고, `NotEnoughBalanceException`객체는 Payment 객체에서 발생시키도록 하겠습니다.  

Seat 객체 코드는 아래와 같습니다.  
```python
class Seat:
    def __init__(self):
        self.seat = [[True] * 5 for _ in range(5)]

    def get_seat(self, row, col):
        if 0 < row < len(self.seat) and 0 < col < len(self.seat[row]) and self.seat[row][col]:
            self.seat[row][col] = False
        else:
            logger.info("예매 불가능한 좌석입니다.")
            raise NotValidSeatException
```

Payment 객체 코드는 아래와 같습니다.  
```python
class Payment:
    def __init__(self, balance):
        self._balance = balance

    def pay(self, price):
        if self._balance >= price:
            self._balance -= price
        else:
            logger.info("잔액이 부족합니다.")
            raise NotEnoughBalanceException
```

Ticketing 객체 코드는 아래와 같습니다.  
```python
class Ticketing:
    def __init__(self, price):
        self.seat = Seat()
        self.price = price

    def reserve(self, row, col, payment):
        self.seat.get_seat(row, col)
        payment.pay(self.price)
```

이처럼 각 오류를 적합한 곳에서 발생시키는 것이 에러를 핸들링하는 더 나은 방법입니다.  

## 원본 예외 포함
오류 처리 중에 발생한 오류 대신 다른 오류를 발생시키며 메시지를 변경하고 싶을 수 있습니다.  
이 경우에는 원래 발생했던 예외를 포함하는 것이 좋습니다.  

원본 예외를 포함하려면 아래와 같은 구문을 사용하면 됩니다.  
```python
raise <new exception> from <original exception>
```
이 구문을 사용하면 원본 예외가 새로운 예외의 traceback에 포함됩니다.  

위 예제의 Seat 객체를 조금 변형해 원본 예외를 포함하도록 예외를 발생시켜보도록 하겠습니다.  
제공되는 좌석 범위 외의 좌석을 선택하려고 하면 seat 정보를 저장하는 배열에서 IndexError가 발생하게 되는데요, 이 에러를 받아 새로운 에러를 발생시키나 원본 예외를 포함하여 발생시키도록 하겠습니다.  
```py
class Seat:
    def __init__(self):
        self.seat = [[True] * 5 for _ in range(5)]

    def get_seat(self, row, col):
        try:
            if not self.seat[row][col]:
                raise AlreadyOccupiedSeatException("이미 선택된 좌석입니다.")
            self.seat[row][col] = False
            print(f"{row}행 {col}열 좌석 선택 완료")
        except IndexError as e:
            raise WrongSeatNumberException("좌석 번호 오류") from e
```

위와 같이 에러를 발생시키면 최종적으로 발생한 예외는 `WrongSeatNumberException`이지만 해당 예외를 raise할 때 from에 `IndexError`를 주었기 때문에 원본 예외가 포함됩니다.   
![](/assets/img/2023/01/include_original_exception.png)