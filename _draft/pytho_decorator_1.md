# 파이썬의 데코레이터

파이썬의 데코레이터는 함수, 메서드의 기능을 쉽게 수정하기 위해서 사용되는 방법입니다.  
즉 origin 함수가 있다면 해당 함수에 기능을 추가한 함수를 다시 원래 함수를 정의한 이름인 origin에 재할당 하는 것이 데코레이터가 하는 일입니다.  

이 내용을 코드로 보면 아래와 같습니다.  
```python
def origin():
    print("do something original")

def modifier(func):
    print("do something more")
    return func

origin()    # do something original 출력
origin = modifier(origin)
orogin()    # do something original과 do something more 출력
```

위의 코드는 조금 더 간단히 아래와 같이 나타낼 수도 있습니다.  
```python
@modifier
def origin():
    ...
```
여기에서 modifier는 `데코레이터`라고 하며, origin 함수를 `decorated`되었다 혹은 `wrapped`되었다고 합니다.  

또한 데코레이터는 함수나 메서드뿐만 아니라 제너레이터, 클래스 등의 다른 종류의 객체에도 적용이 가능합니다.   
가장 먼저 함수에 적용된 데코레이터에 대해 알아보도록 하겠습니다.  

## 함수 데코레이터
함수에 데코레이터를 적용하면 추가적인 로직을 함수에 손쉽게 적용할 수 있습니다.  
데코레이터를 사용해서 할 수 있는 작업으로는 파라미터의 유효성이나 조건을 검사할 수도 있고, 메인 비즈니스 로직이 아닌 부가 기능을 정의해 둘 수도 있는 등 활용 방안이 매우 많습니다.  

함수형 데코레이터를 정의하는 방식은 다음과 같습니다.  
```py
from functools import wraps

def decorator_func(operation):
    @wraps(operation)
    def wrapped(*args, **kwargs):
        """데코레이터에서 수행할 기능 작성"""
        return operation(*args, **kwargs)
    return wrapped
```
데코레이터로 사용할 함수를 만든 후 함수 내부에 `wrapped`라는 이름의 함수를 두었습니다.  
`wrapped` 함수 내에서 원하는 데코레이터 함수의 로직을 작성 후에 데코레이터를 적용할 함수인 `operation` 함수를 반환하여 수행시킵니다.  
그리고 최종적으로 wrapped 함수를 반환하여 데코레이터의 로직과 기존 함수의 로직을 합하여 반환하게 되는 것입니다.  


추가적으로 함수 데코레이터를 이용해서 특정 함수의 수행 시에 함수가 호출된 시간을 로그로 남기는 간단한 예제 코드를 작성해보도록 하겠습니다.  
```py
import time
import logging
from functools import wraps


# 로거 설정
log = logging.getLogger("decorator")
log.setLevel(logging.INFO)
log.addHandler(logging.StreamHandler())


def log_start_time(operation):
    @wraps(operation)
    def wrapped(*args, **kwargs):
        log.info(f"function starts at {time.strftime('%c', time.localtime())}")
        return operation(*args, **kwargs)
    return wrapped


@log_start_time
def origin(n: int):
    for _ in range(n):
        time.sleep(1)


if __name__ == "__main__":
    origin(3)
```
origin 함수에서는 전달받은 횟수만큼 반복하며 1초씩 쉬는 간단한 로직이 들어있습니다.  
그리고 origin 함수는 log_start_time이라는 함수에 decorated 되어있습니다.   
log_start_time 함수는 수행 시작 시점을 log로 남겨주는 역할을 수행합니다.  

<br/>

## 클래스 데코레이터
클래스에도 데코레이터를 적용할 수 있습니다.  
함수형과의 차이점은 데코레이터 함수의 파라미터로 함수가 아닌 클래스를 받는다는 점입니다.  

클래스 데코레이터를 구현한 간단한 예제를 한 번 살펴보도록 하겠습니다.  

아래 예제는 판매 품목 각각에 대한 할인율을 제공받아 할인된 가격을 반환하도록 하는 데코레이터 입니다.  
```py
class PriceDiscounter:
    def __init__(self, discount_rates: dict):
        self.discount_rates = discount_rates

    def discount(self, discount_items):
        return {
            item: getattr(discount_items, item) * (1 - discount_rate)
            for item, discount_rate in self.discount_rates.items()
        }


class Discount:
    def __init__(self, **discount_rates):
        self.discounter = PriceDiscounter(discount_rates)

    def __call__(self, discount_items):
        discount_items.discount = self.discounter.discount
        return discount_items
```
Discount 클래스는 초기화될 때 할인율을 적용한 가격을 반환하도록 하는 PriceDiscounter 객체를 변수를 갖습니다.  
그리고 `__call__` 함수를 통해 할일을 적용할 품묵이 있는 클래스를 받아, 할인된 가격을 적용시키는 discount 함수를 가진 객체로 변환하여 반환합니다.  

이 클래스 데코레이터를 사용하는 클래스의 모습은 아래와 같습니다.  
```py
@Discount(
    shirt=0.1,  # 10% 할인
    pants=0.3,  # 30% 할인
    dress=0.5   # 50% 할인
)
class Clothes:
    def __init__(self):
        self.shirt = 30000
        self.pants = 20000
        self.dress = 50000
```

위의 Clothes 객체를 사용하는 코드는 아래와 같습니다.  
```py
clothes = Clothes()
discounted_clothes = clothes.discount(clothes)
print("discounted clothes: ", discounted_clothes)
```
코드를 수행하면 결과로 아래 내용이 출력됩니다.  
```discounted clothes:  {'shirt': 27000.0, 'pants': 14000.0, 'dress': 25000.0}```

데코레이터에 전달한 할인율이 적용된 가격이 출력된 것을 확인할 수 있습니다.  


## 데코레이터에 인자 전달
데코레이터에 필요한 인자를 추가적으로 전달해야 할 경우, 데코레이터를 생성하는 방법에는 두 가지가 있습니다.  
첫 번째는 중첩함수를 만들어 데코레이터를 한 단계 더 깊게 만드는 것입니다.  
두 번째는 데코레이터를 위한 클래스를 만드는 것입니다.  

### 중첩 함수를 통한 인자 전달
먼저 중첩함수를 만들어 데코레이터에 인자를 전달하는 방식의 코드를 살펴보도록 하겠습니다.  
```py
from functools import wraps
import time
import logging

# 로거 설정
log = logging.getLogger("decorator")
log.setLevel(logging.INFO)
log.addHandler(logging.StreamHandler())

def consumed_time(job_name: str, time_limit: int):
    def calc(operation):
        @wraps(operation)
        def wrapped(*args, **kwargs):
            start_time = time.time()
            operation(*args, **kwargs)
            elapsed_time = time.time() - start_time
            if (elapsed_time > time_limit):
                log.warning(f"{job_name} takes too long time: {elapsed_time} (time limit = {time_limit})")
        return wrapped
    return calc


@consumed_time(job_name="sleep_n_seconds", time_limit=3)
def sleep_n_secs(n):
    for _ in range(n):
        time.sleep(1)


if __name__ == "__main__":
    sleep_n_secs(2)
    sleep_n_secs(3)
```
위와 같이 코드를 작성하면 `consumed_time` 함수에서 사용할 파라미터를 받아 해당 데이터를 데코레이터 내에서 활용할 수 있습니다.  
이 때 가장 밖의 첫번째 함수는 파라미터를 받아 내부 함수에 전달하는 역할을 하고. 두 번째 함수는 데코레이터가 될 함수이며. 가장 안쪽의 세 번째 함수는 데코레이팅의 결과를 반환하는 함수입니다.  

### 데코레이터를 위한 클래스 생성을 통한 인자 전달  
중첩된 함수를 통해 데코레이터를 생성하는 것보다 더 간단한 방법으로는 클래스를 이용해 데이터를 정의하는 방법이 있습니다.  
이 때는 `__init__`메서드에 파라미터를 전달한 후에 `__call__` 매직 메서드에서 데코레이터의 로직을 구현하면 됩니다.  

위와 동일한 역할을 하는 데코레이터를 클래스를 이용해 생성한 후 사용해보도록 하겠습니다.  
```py
from functools import wraps
import time
import logging

# 로거 설정
log = logging.getLogger("decorator")
log.setLevel(logging.INFO)
log.addHandler(logging.StreamHandler())


class TimeConsumed:
    def __init__(self, job_name: str, time_limit: int):
        self.job_name = job_name
        self.time_limit = time_limit

    def __call__(self, operation):
        @wraps(operation)
        def wrapped(*args, **kwargs):
            start_time = time.time()
            operation(*args, **kwargs)
            elapsed_time = time.time() - start_time
            if (elapsed_time > self.time_limit):
                log.warning(f"{self.job_name} takes too long time: {elapsed_time} (time limit = {self.time_limit})")
        return wrapped


@TimeConsumed(job_name="sleep_n_seconds", time_limit=3)
def sleep_n_secs(n):
    for _ in range(n):
        time.sleep(1)


if __name__ == "__main__":
    sleep_n_secs(2)
    sleep_n_secs(3)
```

데코레이터를 `TimeConsumed` 클래스를 이용하여 생성하였습니다.  
이 때 `TimeConsumed` 인스턴스가 `__init__`이 수행되며 생성된 후에 @ 연산이 수행되면서 `__call__` 매직 메서드가 수행되어 데코레이터가 정상적으로 함수에 적용됩니다.  

<br>

## 데코레이터 활용 시 유의할 점
데코레이터는 많은 편리한 이점을 주지만, 잘못 사용하면 예상치 못한 부작용을 낳을 수 있습니다.  
많이 발생할 수 있는 실수에 대해 알아보도록 하겠습니다.  
