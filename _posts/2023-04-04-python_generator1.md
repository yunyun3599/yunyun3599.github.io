---
title:  "파이썬 제너레이터(1)"
excerpt: "파이썬 제너레이터가 무엇인지 알아보고 파이썬에서 반복을 구현하는 여러 방식에 대해 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-04-04
last_modified_at: 2023-04-04
---
# 파이썬 제너레이터와 반복  

## 제너레이터
제너레이터는 한 번에 하나씩 요소를 반환하는 이터러블을 생성해주는 객체입니다.  
제너레이터는 필요한 요소를 모두 메모리에 저장해두는 방식이 아니고 필요할 때마다 생성해 리턴할 수 있도록 생성 방법을 가지고 있는 방식으로 동작합니다.  
그렇게 때문에 고성능이면서 메모리를 적게 사용하는 반복을 하려고 할 때 사용되며 무거운 객체를 사용하거나 무한 시퀀스를 생성하는 것도 가능합니다.

### yield
제너레이터를 사용할 때는 `yield`라는 키워드가 포함됩니다.  
`yield` 키워드가 포함된 함수를 호출하면 제너레이터 인스턴스가 생성됩니다.  
제너레이터는 iterable하기 때문에 for 루프와 함께 사용 가능합니다.  

### 제너레이터 표현식
리스트 컴프리헨션(list comprehension)으로 리스트를 생성했던 것처럼 제너레이터 표현식을 이용해서 제너레이터를 생성하는 방법도 있습니다.  
사용 방법은 아래와 같습니다.  
```py
# list comprehension
[x**2 for x in range(10)]

# 제너레이터 표현식
(x**2 for x in range(10))
```
참고로 제너레이터 표현식도 `max()`, `sum()`과 같은 이터러블 연산이 가능한 함수에 직접 전달할 수 있습니다.  


## 제너레이터 사용 예제
다음은 제너레이터를 이용해 반복을 실행하는 예제입니다.  
아래 예제는 `사원id, 연봉`의 정보를 가지고 있는 csv파일을 읽어 최고 연봉값, 최저 연봉값, 연봉 평균을 구하는 예제입니다.  

먼저 iterable한 값인 salary_list를 받아 요소를 하나씩 확인하며 최고연봉, 최저연봉, 연봉 평균을 갱신하는 `SalaryCalculator` 객체를 만들어보도록 하겠습니다.  
참고로 `SalaryCalculator` 객체는 salary_list 내의 값 중 최고, 최소, 평균값을 구하는 역할만 하며 제너레이터는 salary_list를 생성하는 함수에서 구현할 것입니다.  
```python
UPPER_LIMIT = 10 ** 10


class SalaryCalculator:
    def __init__(self, salary_list):
        self.salary_list = salary_list
        self.min_salary = UPPER_LIMIT
        self.max_salary = 0
        self._total_salary = 0
        self._employee_count = 0

    def calculate(self):
        for salary in self.salary_list:
            self.min_salary = min(self.min_salary, salary)
            self.max_salary = max(self.max_salary, salary)
            self._total_salary += salary
            self._employee_count += 1

    @property
    def avg_salary(self):
        return self._total_salary / self._employee_count
```

다음은 csv파일을 읽어와 이터러블한 정보를 리턴해주는 함수를 확인해보도록 하겠습니다.  
기존에 많이 사용하는 이터러블은 List이므로 제너레이터가 아닌 리스트에 정보를 담아 리턴하는 함수를 먼저 보도록 하겠습니다.  
```py
def load_list_data(path):
    salary_list = []
    with open(path) as f:
        for line in f:
            *_, salary = line.partition(",")
            salary_list.append(int(salary))
    return salary_list
```
csv 파일을 한 줄씩 읽고 해당 값을 list에 append한 후 list를 반환하였습니다.  
이 함수를 이용해 `SalaryCalculator`를 동작시키려면 아래와 같은 코드가 나옵니다.  
```py
data_list = load_list_data(path=data_path)

salary_list_calculator = SalaryCalculator(data_list)
salary_list_calculator.calculate()
print("salary_list_calculator.avg_salary", salary_list_calculator.avg_salary)
```

다음으로는 `SalaryCalculator`를 초기화할 때 `salary_list` 파라미터에 제너레이터 객체를 전달해보도록 하겠습니다.  
csv파일에서 값을 읽어와 제너레이터를 반환하는 함수는 다음과 같습니다.  
```py
def load_generator_data(path):
    with open(path) as f:
        for line in f:
            *_, salary = line.partition(",")
            yield int(salary)
```
역시나 파일을 한 줄씩 가져와 값을 리턴하는데, 이전처럼 한 줄씩 읽은 값을 모두 list에 append하고 list 자체를 리턴하는 것이 아니라 `yield` 키워드를 이용해 값을 그때그때 반환합니다.  
위 함수를 이용해 `SalaryCalculator`를 동작시키는 코드는 아래와 같습니다.
```py
data_generator = load_generator_data(path=data_path)
salary_generator_calculator = SalaryCalculator(data_generator)
salary_generator_calculator.calculate()
print("salary_generator_calculator.avg_salary", salary_generator_calculator.avg_salary)
```

## 파이썬의 반복
파이썬에서 관용적으로 반복 코드는 `for element in iterable` 형태로 구현됩니다.  
이 구문을 사용하기 위해서 `iterable` 자리에 오는 객체는 `__iter__` 메서드를 구현해 반복 가능해야합니다.  
또한 `__next__` 메서드를 구현하면 객체는 이터레이터가 됩니다.  

먼저 무한 시퀀스를 만드는 이터러블하지 않은 코드를 살펴보도록 하겠습니다.  
아래 코드는 초기깂과 한 번에 얼마씩 값을 증가시킬지에 대한 값을 받습니다.  
그리고 계속해서 점점 커진 값을 반환합니다.  
```py
class NotIterableSequence:
    def __init__(self, start, step):
        self.current = start
        self.step = step

    def __next__(self):
        value = self.current
        self.current += self.step
        return value


if __name__ == "__main__":
    sequence = NotIterableSequence(0, 10)
    print(next(sequence)) # 0
    print(next(sequence)) # 10
    print(next(sequence)) # 20

    for value in NotIterableSequence(0, 10):    # TypeError: 'NotIterableSequence' object is not iterable
        print(value)
```
코드를 실행시켜보면 `__next__` 메서드를 구현했기 때문에 `next(sequence)`를 호출할 때마다 값이 10씩 증가하는 것을 확인할 수 있습니다.   
그런데 for문을 사용해 반복을 해보려고 하면 객체가 iterable하지 않다는 오류가 발생합니다.  
그 이유는 반복을 하려면 구현해야하는 `__iter__` 함수가 구현되지 않았기 때문입니다.  
> 참고로 `next()` 내장함수는 이터러블을 다음 요소의 값으로 이동시키고 기존 값을 반환하는 역할을 하며, 객체의 `__next__` 함수를 호출합니다.  
> 만약 next() 함수를 호출했는데 더 이상 반환할 값이 없다면 StopIteration이 발생합니다.  
> 이 때 `next(obj, 'default')`로 두번째 인자를 주면 StopIteration이 발생하는 대신 두번째 인자를 반환합니다.  

`__iter__` 함수를 구현해 iterable한 함수를 생성해보도록 하겠습니다.  
무한 시퀀스를 발생시키는 객체이므로 리턴값이 100이 넘어가면 멈추도록 하였습니다.  
```py
class IterableSequence:
    def __init__(self, start, step):
        self.current = start
        self.step = step

    def __next__(self):
        value = self.current
        self.current += self.step
        return value

    def __iter__(self):
        return self


if __name__ == "__main__":
    sequence = IterableSequence(0, 10)
    print(next(sequence))
    print(next(sequence))
    print(next(sequence))

    for value in IterableSequence(0, 10):
        if value > 100:
            break
        print(value)
```
이 코드는 객체를 이터러블하게 만들기 위해 `__iter__` 함수를 구현했으므로 for문을 사용한 부분이 정상적으로 동작합니다.  


## itertools 
itertools는 python의 모듈로, 이터러블과 관련된 여러 가능들을 제공해줍니다.  

### itertools.islice
itertools를 이용해 반복 도중 특정 값들에 대해 필터링을 하는 예제를 살펴보도록 하겠습니다.  
위의 salary 예제에서 outlier를 제거하기 위해 salary가 2000 미만이거나 100000 초과인 사람 중 앞의 100명에 대한 데이터만 계산하려고 합니다.  
앞에서 봤던 코드에서 반복문 안에 다음과 같은 조건문을 추가해서 해당 값들을 제거할 수도 있습니다.  
```py
def calculate(self):
  for salary in self.salary_list:
    if 2000 <= salary <= 10000 and self._employee_count <= 100:  # 조건문 추가
      self.min_salary = min(self.min_salary, salary)
      self.max_salary = max(self.max_salary, salary)
      self._total_salary += salary
      self._employee_count += 1
```
이런식의 코드도 정상 동작하지만 `SalaryCalculator`는 그저 연봉 정보를 받아 계산을 하는 역할인데, 조건을 가지고 있는 것은 객체의 책임을 넘어섭니다.  
또한 조건 세부 값이 변경되면 `SalaryCalculator`의 값을 바꿔야하는데, 수정 사항이 있을 때마다 이런식의 처리를 하는 것은 비효울적입니다.  

이 때는 외부에서 리스트의 값을 적절히 조절해서 SalaryCalculator에 가공된 값의 리스트를 넘겨주는 것이 적당합니다.  
그리고 이런 식의 작업을 위해 itertools의 기능을 활용할 수 있습니다.  
islice는 리스트의 범위를 지정해 해당 범위 내의 요소만 반환하는 역할을 합니다.  
```py
class SalaryCalculator:
    def __init__(self, salary_list):
        self.salary_list = salary_list
        self.min_salary = UPPER_LIMIT
        self.max_salary = 0
        self._total_salary = 0
        self._employee_count = 0

    def calculate(self):
        for salary in self.salary_list:
            self.min_salary = min(self.min_salary, salary)
            self.max_salary = max(self.max_salary, salary)
            self._total_salary += salary
            self._employee_count += 1

    @property
    def avg_salary(self):
        return self._total_salary / self._employee_count


def load_generator_data(path):
    with open(path) as f:
        for line in f:
            *_, salary = line.partition(",")
            yield int(salary)


if __name__ == "__main__":
    data_path = "./salary_data.csv"
    data_generator = load_generator_data(path=data_path)
    salary_filtered_generator = islice(filter(lambda salary: 2000 <= salary <= 1000000, data_generator), 101)
    
    afilter_calculator = SalaryCalculatorWithFilter(salary_filtered_generator)
    filter_calculator.calculate()
    print("filter_calculator.avg_salary", filter_calculator.avg_salary)
```
위 코드에서 `SalaryCalculator`는 앞에서 봤던 코드와 완벽하게 일치합니다.  
다만 바깥단에서 islice와 filter를 이용해 받은 이터러블 객체의 요소를 한 번 필터링한 후 해당 값을 가지고 `SalaryCalculator`를 초기화하였습니다.  
기존 코드의 수정 없이 요구사항이 잘 반영된 것이며, 각 객체의 책임이 명확이 분산되어있습니다.  

또한 generator를 사용했으므로 각 요소에 대한 평가도 게으르게 수행되기 때문에 메모리상의 손해도 전혀 없습니다.  
전체에 대한 필터링을 한 것처럼 보이지만, 실제로는 하나씩 가져왔기 때문에 모든 것을 한 번에 메모리에 올릴 필요가 없는 것입니다.  


### itertools.tee
itertools.tee 는 원래의 이터러블을 새로운 이터러블 여러 개로 분할합니다.  
그리고 똑같은 이터러블을 여러번 반복할 필요 없이 분할된 이터러블을 사용해 필요한 연산을 수행합니다.  

tee 함수는 제너레이터를 사용한 for 문이 여러개 있는 것과 유사하며 가장 앞의 for문을 기준으로 위치가 이동합니다.  
따라서 만약 tee를 이용해 만든 새로운 이터러블이 2개라면 있으면 전체 메모리 사용량이 2배가 되는 것은 아니나 처리 중인 특정 index의 요소에 대해서는 메모리 사용량이 2배가 됩니다.  
그러므로 개별 요소가 크고 이터러블을 여러 개 복사해야하면 tee함수 사용에 주의해야하나, 개별 요소가 작고 어떤 이터러블이 다른 이터러브의 순서를 크게 앞지르지 않으면 tee를 사용하는 것도 좋습니다.  

tee를 사용해서 위의 `SalaryCalculator` 예제를 간단하게 수정하여 그냥 max_salary와 max_salary를 10의자리에서 반올림한 round_max_salary를 구하는 예제로 변경해보도록 하겠습니다.  
```py
import itertools

UPPER_LIMIT = 10 ** 10


class SalaryCalculatorWithFilter:
    def __init__(self, salary_list):
        self.salary_list = salary_list
        self.max_salary = 0
        self.round_max_salary = 0

    def calculate(self):
        _max, _round_max = itertools.tee(self.salary_list, 2)
        self.max_salary = max(_max)
        self.round_max_salary = max(round(salary, -2) for salary in _round_max)
        return self


def load_generator_data(path):
    with open(path) as f:
        for line in f:
            *_, salary = line.partition(",")
            yield int(salary)


if __name__ == "__main__":
    data_path = "./salary_data.csv"
    data_generator = load_generator_data(path=data_path)
    calculator = SalaryCalculatorWithFilter(data_generator).calculate()
    print("filter_calculator.max_salary", calculator.max_salary)
    print("filter_calculator.round_max_salary", calculator.round_max_salary)
```
tee 함수를 이용해 기존 이터러블을 두 개의 이터러블로 바꾼 후 각각 필요한 연산을 수행했습니다.  
이 때 주의할 점은 만약 제너레이터로 전달한 `self.salary_list` 값을 이용해 작업을 하면 tee를 통해 반환된 값들 역시 모두 소진된다는 점입니다.   
제너레이터는 한 번 값을 이용해 한 바퀴를 돌고 난 후에는 해당 제너레이터의 시퀀스에 남은 값이 없어서 재사용이 불가능합니다.  
```py
lst = [i for i in range(10)]
generator = (i for i in range(10))

print("max(lst):", max(lst))              # max(lst): 9
print("max(generator):", max(generator))  # max(generator): 9
print("max(lst):", max(lst))              # max(lst): 9
print("max(generator):", max(generator))  # ValueError: max() arg is an empty sequence -> 재사용 불가
```
tee를 이용해 반환된 값들도 한 번 사용하면 다시 사용이 불가능할 뿐 아니라, `a, b = itertools.tee(iterable, 2)` 식으로 생성된 후 `iterable`이 사용되면 a, b 값에 대한 이터러블도 실행될 수 있으므로 주의해야합니다.  

