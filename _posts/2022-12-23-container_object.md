---
title:  "파이썬의 컨테이너 객체"
excerpt: "파이썬의 컨테이너 객체에 대해 알아보고 컨테이너 객체를 만들어봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-23
last_modified_at: 2022-12-23
---

# 컨테이너 객체
컨테이너는 `__contains__` 메서드를 구현한 객체로, `__contains__` 메서드는 일반적으로 boolean 값을 반환합니다.  
컨테이너 객체가 사용되는 구문은 `element in elements` 형태의 구문입니다.  
위의 코드는 파이썬에서 `elements.__contains__(element)` 형태로 해석됩니다.  

## 컨테이너 객체 생성 예제
컨테이너 객체를 직접 생성해볼 수 있도록 예제를 하나 만들어보겠습니다.  
작성할 예제에는 아래 클래스들이 사용됩니다.  
- AmusementPark
    - 놀이공원 객체
    - 개장시간, 폐장시간 속성을 토대로 \_\_contains\_\_ 함수에서는 특정 시간에 놀이공원이 개장중인지를 반환
- OpeningHours
    - 개장 시간 범위에 대한 객체
    - AmusementPark의 속성으로 들어가는 객체로, \_\_contains\_\_의 세부 동작 내용 구현을 위임받은 객체

위의 두 가지 클래스는 모두 \_\_contains\_\_ 함수를 포함하고 있는 컨테이너 객체입니다.  

```python
class OpeningHours:
    def __init__(self, open_at, close_at):
        self.open_at = open_at
        self.close_at = close_at

    def __contains__(self, hour):
        return self.open_at <= hour <= self.close_at


class AmusementPark:
    def __init__(self, open_at, close_at):
        self.open_at = open_at
        self.close_at = close_at
        self.opening_hours = OpeningHours(self.open_at, self.close_at)

    def __contains__(self, hour):
        return hour in self.opening_hours


if __name__ == "__main__":
    amusement_park = AmusementPark(10, 21)
    time_list = [9, 10, 15, 20, 23]
    for time in time_list:
        print(f"The amusement is open at {time}? - {time in amusement_park}:")
```

amusement_park 객체를 생성 후에 time_list 내에 있는 시간마다 해당 시간에 amusement_park가 열려있는 상태인지 확인하는 예제입니다.  
AmusementPark 객체 내에서 OpeningHours 객체에 \_\_contains\_\_ 함수의 세부 구현 사항을 위임하였습니다.  

코드를 실행한 결과는 다음과 같습니다.  
![](/assets/img/2022-12/2022-12-23-container_object/example_result.png)
10시 ~ 21시까지가 개장 시간이므로 그 외의 시간은 False가 출력되는 것을 확인할 수 있습니다.  