---
title:  "파이썬의 호출형(callable) 객체"
excerpt: "파이썬의 호출형 객체에 대해 알아보고, 예제를 작성해 봅니다.  "

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-24
last_modified_at: 2022-12-24
---

# 파이썬의 Callable 객체
파이썬에서의 호출형 객체는 함수처럼 동작하는 객체를 의미합니다.  

함수처럼 동작하는 객체를 사용하면 객체에는 **상태**가 있기 때문에 함수 호출 사이에 정보를 저장할 수 있다는 장점이 있습니다.  

호출형 객체를 생성하는 방법에는 두 가지가 있습니다.  
1. 데코레이터 생성
2. 매직메서드 \_\_call\_\_ 구현  

두 가지 방법 중 이번 포스팅에서는 두번째 *`매직메서드 __call__ 구현`* 방법을 통해 호출형 객체를 생성해보도록 하겠습니다.  

아래 `IncrementMachine` 객체는 `counter`라는 속성을 갖습니다.  
그리고 `__call__` 매직 메서드를 구현하여, 객체 호출 시에 `counter` 값을 1씩 증가시킵니다.  
```python
class IncrementMachine:
    def __init__(self):
        self.counter = 0

    def __call__(self):
        self.counter += 1
        return self.counter
```

## 호출형 객체 예시 코드
호출형 객체를 사용하는 조금 더 심화된 예제를 확인해보도록 하겠습니다.  

>[예제 코드 상세 내용]
><p style="font-size: 0.8rem">
1. Gift 객체는 버전 a, b, c 속성을 가지고 있으며 각 속성의 값은 주문 수량이다.<br>
2. Gift 객체는  `__call__` 매직 메서드를 구현하였으며, 주문할 버전을 파라미터로 받아 해당 버전의 주문 수량을 증가시킨다.  <br>
3. 버전 a, b, c 이외의 버전이 주문으로 들어오면 해당 버전은 주문 불가능하다는 안내 메세지를 띄운다.  <br>
4. 각 주문 이후에는 현재 주문 수량 상태를 프린트한다.</p>

Gift 클래스 코드는 다음과 같습니다.  
```python
class Gift:
    def __init__(self):
        self.version_a = 0
        self.version_b = 0
        self.version_c = 0

    def __call__(self, *orders):
        for order in orders:
            if order == "version_a":
                self.version_a += 1
            elif order == "version_b":
                self.version_b += 1
            elif order == "version_c":
                self.version_c += 1
            else:
                print(f"{order} is not available")

        print(f"<Order Status>\n "
              f"- version_a: {self.version_a}\n "
              f"- version_b: {self.version_b}\n "
              f"- version_c: {self.version_c}\n")
```

gift 인스턴스를 생성하고 몇 가지 버전을 주문해보도록 하겠습니다.  
그리고 마지막에는 존재하지 않는 버전인 `version_d`를 주문해보도록 하겠습니다.  
```python
gift = Gift()
gift("version_a")
gift("version_b", "version_b", "version_c")
gift("version_d")
```

위 코드를 수행하면 아래와 같은 내용이 출력됩니다.  
![](/assets/img/2022-12/2022-12-24-python_callable_object/%08gift_example_result.png)
주문한 대로 수량이 올라가며, `version_d`를 주문했을 때는 해당 주문은 불가능하다는 안내 메세지가 출력됩니다.  

이처럼 호출형 객체를 사용하면 함수처럼 객체를 사용하되, 객체의 속성을 이용해 상태를 저장해둘 수 있어 유용합니다.  