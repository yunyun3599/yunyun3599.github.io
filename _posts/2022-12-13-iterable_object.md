---
title:  "파이썬의 이터러블 객체"
excerpt: "파이썬의 이터러블 객체에 대해 알아보고 이터러블 객체와 시퀀스를 만들어봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-13
last_modified_at: 2022-12-13
---

# 이터러블 객체
파이썬에서 이터러블하다는 것은 반복 가능하다는 것을 뜻합니다.  
예를 들어 리스트, 튜플, 딕셔너리 등은 for 문을 통해 원하는 값을 반복적으로 가져올 수 있으므로 이터러블 객체입니다.  

파이썬에서 `for element in elements:` 형태로 객체를 반복하기 위해서는 아래 두가지 사항 중 하나를 만족해야합니다. 
1. 객체가 \_\_next\_\_ 나 \_\_iter\_\_ 메서드 중 하나를 포함
2. 객체가 시퀀스이고 \_\_len\_\_과 \_\_getitem\_\_모두 가짐 

참고로 위에 언급된 매직메서드 중 무엇을 구현했느냐에 따라 객체의 종류를 아래와 같이 구분할 수 있습니다. 
- 이터러블: \_\_iter\_\_ 매직 메서드를 구현한 객체
- 이터레이터: \_\_next\_\_ 매직 메서드를 구현한 객체

<br>

# 이터러블 객체 생성
파이썬에서는 for문 수행 시 가장 먼저 iter() 함수가 \_\_iter\_\_ 메서드가 객체 내에 있는 지를 확인합니다.  

따라서 \_\_iter\_\_ 매직 메서드를 포함하여 이터러블 객체 예시 코드를 작성해보도록 하겠습니다.  
아래에서 생성할 이터러블 객체는 26주 적금에 대한 객체입니다.  
이 객체는 아래와 같은 특성을 가지고 있습니다.  
- 초기 저금액을 받아 인스턴스를 생성한 후 매 주마다 몇 주차인지와 적금에 넣어야 할 돈의 액수를 리턴합니다.  
- 적금에 돈을 저금한 주차가 26주차가 넘어가면 `InstallmentSavingMaturityException`이 발생합니다.  

코드는 아래와 같습니다.  
```python
class InstallmentSaving26:
    def __init__(self, base_price):
        self.base_price = base_price
        self.this_week_price = base_price
        self.week_count = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.week_count >= 26:
            raise InstallmentSavingMaturityException
        self.week_count += 1
        self.this_week_price = self.base_price * self.week_count
        return self.week_count, self.this_week_price


class InstallmentSavingMaturityException(Exception):
    pass


if __name__ == "__main__":
    saving = InstallmentSaving26(1000)
    for week, price in saving:
        print(f"Week {week}: save {price} won")

```

위 코드를 보면 for문이 동작할 때 호출되는 iter() 메서드는 \_\_iter\_\_ 메서드를 호출할 것이고, 해당 메서드는 self를 반환함으로써 26주 적금 객체 자신이 이터러블임을 알려줍니다.  

그리고 루프의 각 단계마다 next()함수가 호출되고 next() 함수는 객체 내의 \_\_next\_\_함수에 동작을 위임하게 됩니다.  

그 결과 위의 객체는 `InstallmentSavingMaturityException` 예외가 발생할 때까지 반복됩니다.  

위 코드의 수행 결과는 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-13-iterable_object/26saving_iterable_object.png)
26주차까지 1000원씩 증액하며 적금이 진행되다가 27주차에 예외가 발생했음을 확인할 수 있습니다.  

위의 코드는 순차적으로 잘 작동하나, 한 번 사용한 인스턴스는 이미 모든 반복을 완료했기 때문에 다시 루프를 돌리려고 해도 돌지 않는다는 단점이 있습니다.  
이터러블 객체를 사용하는 부분 코드를 아래와 같이 변경하고 수행해보도록 하겠습니다.   
```python
if __name__ == "__main__":
    saving = InstallmentSaving26(1000)
    for i in range(1, 4):
        try:
            for week, price in saving:
                print(f"Week {week}: save {price} won")
        except InstallmentSavingMaturityException:
            print(f"{i}. saving is done")
```
그 결과는 다음과 같습니다.  
![](/assets/img/2022-12/2022-12-13-iterable_object/iterable_object_result.png)
1~26주차 저금이 완료된 후 `saving is done` 이라는 문장이 출력되는 전체 프로세스가 3번 일어날 것을 기대했으나 실제로는 1~26주차 저금이 완료된 후에 더 이상의 저금은 이뤄지지 않고 `saving is done` 문장만 3번 연속 출력됨을 확인할 수 있습니다.  

이 이유는 이미 반복이 완료된 인스턴스를 또다시 반복하려 했기 때문입니다.

위의 경우 26주 적금을 3번 반복하려 했으나 실제로 동작한 내용은 다음과 같습니다. 
1. 반복이 수행될 때 이터러블 객체는 이터레이터를 생성하고 이를 통해 반복됨
2. 26주 적금은 반복이 시작될 때 호출되는 \_\_iter\_\_에서 self를 반환함
3. 26주 적금 객체는 \_\_next\_\_를 구현하였기 때문에 이터레이터이므로 \_\_iter\_\_에서 self를 반환한 것은 이터레이터를 반환한 것임
4. 객체 자체가 이터레이터로서 동작하므로 반복이 끝난 후에 새로 for문을 돌려도 self를 반환하여 앞서 이미 완료된 반복문에서 사용한 것과 동일한 이터레이터를 반환함
5. 이미 완료된 상태이므로 반복이 다시 수행되지 않음

위에서 발생한 문제를 해결하기 위해 \_\_iter\_\_에서 계속 동일한 이터레이터를 반환하도록 하지 말고 제너레이터를 반환하도록 하여 위 문제를 해결해보겠습니다.  
제너레이터를 이용하여 구현한 코드는 다음과 같습니다.  
```python
class InstallmentSaving26WithGenerator:
    def __init__(self, base_price):
        self.base_price = base_price
        self.this_week_price = base_price

    def __iter__(self):
        week_count = 1
        while week_count <= 26:
            yield week_count, week_count * self.base_price
            week_count += 1


if __name__ == "__main__":
    saving = InstallmentSaving26WithGenerator(1000)
    for i in range(1, 4):
        for week, price in saving:
            print(f"Week {week}: save {price} won")
        print(f"{i}. saving is done")
```
iter 함수에서 yield를 통해 제너레이터를 반환하였습니다.  
위의 코드는 for문이 한 번 반복되면 끝이 나는 이터레이터가 아니라 제너레이터를 사용하였으므로 적금 전체 프로세스가 3번 반복될 것입니다. 

<br>

# 시퀀스 객체 생성
앞에서 생성한 이터러블 객체는 순차적으로 내부 값들에 접근합니다.  
이렇게 다음 값을 그때그때 계산하여 돌려주는 방식은 메모리 사용량이 적다는 장점이 있으나, 임의의 n번째 원소에 접근하기 위해서는 n번 반복해야한다는 단점이 있습니다.  

이에 반해 시퀀스 객체는 바로 특정 위치의 요소를 가져올 수 있습니다.  
바로 원하는 위치의 요소에 접근하기 위해 미리 사용할 데이터들을 전부 생성해두므로 이터러블 객체보다 메모리 사용량은 많습니다.  

26주 적금의 예를 다시 이용하여 시퀀스 객체를 생성해보도록 하겠습니다.  
시퀀스 객체는 \_\_len\_\_과 \_\_getitem\_\_ 매직 메서드를 포함해야 합니다.  
```python
class InstallmentSaving26Sequence:
    def __init__(self, base_price):
        self.base_price = base_price
        self._price_list = [base_price * week for week in range(1, 27)]

    def __getitem__(self, week):
        return self._price_list[week]

    def __len__(self):
        return len(self._price_list)


if __name__ == "__main__":
    saving = InstallmentSaving26Sequence(1000)
    week10 = 10
    print(f"Saved price on Week {week10} is {saving[week10 - 1]}")
```

객체의 `_price_list` 속성에 26주간 저금해야할 액수를 미리 구해놓은 후 해당 속성을 이용하여 \_\_getitem\_\_과 \_\_len\_\_을 구현하였습니다.  

코드 수행 결과로는 아래 문장이 출력됩니다.  
> Saved price on Week 10 is 10000