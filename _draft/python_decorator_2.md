# 파이썬의 데코레이터(2)

## 데코레이터 활용 시 유의할 점
데코레이터는 많은 편리한 이점을 주지만, 잘못 사용하면 예상치 못한 부작용을 만날 수 있습니다.  
많이 발생할 수 있는 실수에 대해 알아보도록 하겠습니다.  

### 원본 객체의 데이터 보존
데코레이터에서 원본 객체의 데이터를 변경하여 원치 않는 부작용을 만나는 경우가 있습니다.  
아래 예시를 확인해보도록 하겠습니다.  

```py
import logging

# 로거 설정
log = logging.getLogger("decorator")
log.setLevel(logging.INFO)
log.addHandler(logging.StreamHandler())


def logger(function):
    def wrapped(*args, **kwargs):
        log.info(f"{function.__qualname__} 실행")   # sample_func 실행
        return function(*args, **kwargs)
    return wrapped


@logger
def sample_func():
    """This is a sample function"""
    pass


if __name__ == "__main__":
    print("sample_func.__qualname__:", sample_func.__qualname__)    # logger.<locals>.wrapped
    sample_func()
    help(sample_func)   # wrapped(*args, **kwargs)
```

위의 예시에서 `@logger` 데코레이터는 단순히 로그를 찍는 역할만 합니다.  
그러나 그 결과 `sample_func.__qualname__` 값은 `logger.<locals>.wrapped`로 변하게 됩니다.  
또한 docstring도 기존에 정의해 둔 값이 아닌 `wrapped(*args, **kwargs)`로 덮어써지게 됩니다.  

이런 일을 방지하기 위해 추가해야 하는 코드는 단순합니다.  
바로 `@wraps(function)`를 래핑하는 함수 위에 붙여주는 것입니다.  

```py
from functools import wraps

def logger(function):
    @wraps(function)
    def wrapped(*args, **kwargs):
        ...
```


### 데코레이터 로직의 위치
데코레이터 함수의 로직은 가장 안에 정의되어야합니다.  
그 이유를 살펴보기 위해 아래 예시를 보겠습니다.  

```py
import time
import logging
from functools import wraps


# 로거 설정
log = logging.getLogger("decorator")
log.setLevel(logging.INFO)
log.addHandler(logging.StreamHandler())


def wrong_time_calc(function):
    log.info(f"{function.__qualname__} starts at {time.strftime('%c', time.localtime())}")
    start_time = time.time()

    @wraps(function)
    def wrapped(*args, **kwargs):
        result = function(*args, **kwargs)
        log.info(f"총 수행 시간: {time.time() - start_time}")
        return result
    return wrapped



@wrong_time_calc
def wrong_sample_func(n: int):
    for _ in range(n):
        time.sleep(1)


if __name__ == "__main__":
    wrong_sample_func(2)
    wrong_sample_func(1)
```

위의 `wrong_time_calc` 데코레이터는 함수가 시작된 시간과, 함수의 수행에 걸린 총 시간을 로그로 남깁니다.  
이 때 밑에서 대략 2초가 걸리는 함수와 1초가 걸리는 함수를 각각 호출했으므로 원하는 결과물은 아래와 같을 것입니다.  
```
wrong_sample_func starts at Fri Feb 10 22:52:01 2023
총 수행 시간: 2.00xxxxxx
wrong_sample_func starts at Fri Feb 10 22:52:01 2023
총 수행 시간: 1.00xxxxxx
```

그러나 실제로 함수를 수행해보면 아래와 같은 결과가 나옵니다.  
```
wrong_sample_func starts at Fri Feb 10 22:52:01 2023
총 수행 시간: 2.007222890853882
총 수행 시간: 3.0081560611724854
```

함수의 실행 시점은 하나만 찍힐 뿐더러, 두번째 함수의 수행시간이 약 3초로 나오는 것을 확인할 수 있습니다.  

이런 결과가 나오는 이유는 시작 시점을 로깅하고 시작 시간을 저장해두는 로직이 가장 안쪽 함수에 있지 않기 때문입니다.  
따라서 함수 바깥쪽에 있는 로직은 데코레이터를 특정 함수에 적용하자마자 수행되며, 실제로 데코레이터가 적용된 함수가 수행되는 시점에는 돌지 않게됩니다.  

그러므로 어떠한 로직을 데코레이터를 적용한 함수가 수행될 떄마다 함께 수행되게 하고 싶다면 위의 코드를 아래와 같이 바꾸어야 합니다.  
```py
def right_time_calc(function):
    @wraps(function)
    def wrapped(*args, **kwargs):
        log.info(f"{function.__qualname__} starts at {time.strftime('%c', time.localtime())}")
        start_time = time.time()
        result = function(*args, **kwargs)
        log.info(f"총 수행 시간: {time.time() - start_time}")
        return result
    return wrapped
```

이렇게 데코레이터 외부에 로직이 위치해있을 때는 데코레이터를 적용한 함수 및 클래스들의 실행시점이 되기 전에 외부 로직이 실행된다는 부작용이 있음을 확인해보았습니다.  
이러한 부작용은 일반적인 상황에서는 피하기 위해 노력해야하지만, 어떤 상황에서는 요긴하게 이용될 수도 있습니다.  

예를 들어 스프링에서 빈을 생성해 제어를 역전시킨 예시를 생각해봅시다.  
이때 생성된 빈들은 싱글톤으로 생성되어 하나의 객체만 존재하게 됩니다.  

이처럼 빈으로 등록할 클래스들을 데코레이터를 이용해서 설정해두면, 실행 시점이 되기 전에 미리 Bean으로 생성할 객체들을 등록해둘 수 있습니다.  
```py
BEAN_OBJECT = {}


def register_bean(cls):
    """하나의 인스턴스만 쓸 클래스 등록"""
    BEAN_OBJECT[cls.__name__] = cls()
    return BEAN_OBJECT[cls.__name__]


@register_bean
class Registry:
    """Bean 등록 Registry"""
    def __init__(self):
        self.name = 'repository'


@register_bean
class Controller:
    """Bean 등록 Controller"""


@register_bean
class Service:
    """Bean 등록 Service"""


if __name__ == "__main__":
    print(BEAN_OBJECT) # {'Registry': <__main__.Registry object at 0x100a5b2e0>, 'Controller': <__main__.Controller object at 0x100abb160>, 'Service': <__main__.Service object at 0x100abb370>}
    print(BEAN_OBJECT['Registry']) # <__main__.Registry object at 0x100a5b2e0>
    print(BEAN_OBJECT['Registry'].name) # repository
```


## 어디서나 동작하는 데코레이터 만들기
데코레이터를 함수형으로 만들면 함수에 해당 데코레이터를 적용할 수 있습니다.  

아래 데코레이터 `check_rule`은 특정 함수를 수행할 권한이 있는 지를 체크하는 역할을 하는 데코레이터입니다.  


```py
from functools import wraps


USER_LIST = ['A', 'B', 'C']


class NoAccessRightException(Exception):
    pass


def check_rule(function):
    @wraps(function)
    def wrapped(user_id):
        if user_id in USER_LIST:
            return function(user_id)
        else:
            raise NoAccessRightException(f"{user_id} does not have right to access credential contents")
    return wrapped


@check_rule
def access_credential(user_id):
    print(f"{user_id} can read credential contents")


if __name__ == "__main__":
    try:
        access_credential('A')  # A can read credential contents
        access_credential('D')  # D does not have right to access credential contents
    except NoAccessRightException as e:
        print(e)
```

위의 코드처럼 해당 데코레이터를 함수에 적용하면 정상적으로 동작합니다.  

그러나 만약 해당 데코레이터를 클래스의 메서드에 적용하면 어떻게될까요?  
```py
class Credential:
    @check_rule
    def access_credential(self, user_id):
        print(f"{user_id} can read credential contents")


credential = Credential()
credential.access_credential('B')
```

클래스 메서드에 데코레이터를 적용한 위의 코드를 실행하면 아래 에러를 만나게 됩니다.  
```
TypeError: wrapped() takes 1 positional argument but 2 were given
```

그 이유는 클래스 내의 메서드는 `self`라는 추가 변수가 있기 때문입니다.  
데코레이터에서 `wrapped` 함수는 `user_id` 파라미터만을 받도록 정의해뒀는데, `self`와 `user_id` 두 개의 파라미터가 넘어왔기 때문에 이런 오류가 난 것입니다.  

이런 경우에는 디스크립터 프로토콜을 구현한 데코레이터 객체를 만들어 메서드와 함수 모두에 대응 가능한 데코레이터를 생성할 수 있습니다.  
```py
class check_rule:
    def __init__(self, function):
        self.function = function
        wraps(self.function)(self)

    def __call__(self, user_id):
        if user_id in USER_LIST:
            return self.function(user_id)
        else:
            raise NoAccessRightException(f"{user_id} does not have right to access credential contents")

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.__class__(MethodType(self.function, instance))  # MethodType(함수, 인스턴스)는 인스턴스에 메서드를 동적으로 추가할 수 있게 해줌
```

이처럼 디스크립터 프로토콜을 이용해 데코레이터를 생성하면 함수와 클래스의 메소드 모두에 적용 가능한 데코레이터를 만들 수 있습니다.  
