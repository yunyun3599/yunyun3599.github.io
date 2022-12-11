---
title:  "파이썬의 접근 제어자, 프로퍼티"
excerpt: "파이썬에서 접근 제어자가 어떻게 사용되는 지와 프로퍼티에 대해 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-10
last_modified_at: 2022-12-10
---

# 파이썬의 접근 제어자  
## 밑줄로 시작하는 private 속성
파이썬의 모든 프로퍼티는 public 입니다.  
타 언어처럼 private, protected, default 등은 따로 존재하지 않습니다.  

그럼에도 불구하고 파이썬에서도 private임을 암시하는 방법이 있는데요, 바로 프로퍼티 이름 앞에 \_를 붙여 주는 것입니다.  
```python
class Something:
    def __init__(self, value):
        self_var = value
```
위의 코드 같은 경우에 `_var` 변수는 private으로 취급되며, 외부에서 직접적인 접근을 하지 않기를 기대합니다.  

그러나 앞에서 언급했듯이 파이썬의 모든 프로퍼티는 public입니다.  
따라서 어디까지나 직접적인 접근을 하지 않기를 **기대**하는 것이지 직접적인 접근이 **불가**한 것은 아닙니다.  

public 프로퍼티와 private프로퍼티를 모두 갖는 객체 코드를 한번 살펴보도록 하겠습니다.  
```python
class Something:
    def __init__(self, public_value, private_value):
        self.public = public_value
        self._private = private_value


if __name__ == "__main__":
    sth = Something(10, 20)
    print("_private:", sth._private)
    print(sth.__dict__)

```
앞서 언급한 것처럼 `_private`가 라는 변수 명은 밑줄로 인해 private으로 취급되길 원합니다.  
그러나 접근이 불가능한 것은 아니므로 위 코드를 수행하면 아래와 같은 결과를 얻을 수 있습니다.  
![](/assets/img/2022-12/2022-12-10-python_property/private_property_access.png)

또한 `sth.__dict__` 에 대한 출력에서 알 수 있듯이 `_private` 변수 역시 클래스 내의 멤버 변수로 확인이 가능합니다.  

## 이중 밑줄로 시작하는 속성
앞에서 밑줄로 시작하는 속성은 private으로 처리되기를 원하는 속성이지만, 접근이 불가능하지는 않음을 확인했습니다.  
그런데 파이썬에서 이중 밑줄로 시작하는 속성들은 접근하려고 하면 AttribteError가 발생합니다.  
```python
class Something:
    def __init__(self, value):
        self.__var = value


if __name__ == "__main__":
    sth = Something(30)
    print(sth.__dict__)
    print(sth.__var)

```
위와 같은 코드를 실행하면 아래 결과를 얻게 됩니다.  
![](/assets/img/2022-12/2022-12-10-python_property/double_underscore_access.png)

만약 `__var` 속성이 두개의 밑줄로 인해 private 속성으로 처리되어 접근이 안되는 것이라면 AttributeError가 아닌 접근 불가 오류가 발생해야 할 것 입니다.  
그러나 위의 경우에는 AttributeError가 발생하였습니다.  

또한 `sth.__dict__`로 객체의 속성값을 살펴본 결과 `__var`는 존재하지 않고, 대신 `_Something__var` 이 존재하는 것을 알 수 있습니다.  

이를 종합해보면 밑줄 두개로 시작하는 속성을 사용했을 때 아래 두가지 현상이 발생합니다. 
1. AttributeError가 발생한 것
2. `__var`대신 `_Something__var`가 존재하는 것

이 두가지 일이 일어나느 이유는, 파이썬에서 밑줄 두개로 시작하는 변수에 대해서는 이름 맹글링이 일어나기 때문입니다.  
이름 맹글링이란 변수명을 `_클래스명__변수명`의 형태로 변환한게 되는 것으로, 여러번 확장되는 클래스의 메소드를 이름 충돌 없이 오버라이드 하기 위해 발생합니다.  

예시를 살펴보기 위해 다음 4가지 클래스를 생성해보도록 하겠습니다.  
1. Human 클래스
2. Student 클래스 extends Human
3. UniversityStudent 클래스 extends Student
4. ComputerScienceStudent 클래스 extends UniversityStudent

그리고 각 클래스에는 `__walk`라는 함수와 `talk`라는 함수 두 가지를 생성하도록 하겠습니다. 
밑줄 두 개의 유무에 따라 `__walk`는 이름 맹글링이 일어나고, `talk`는 일어나지 않을 것으로 예상할 수 있습니다.  

**Human 클래스** 
```python
class Human(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __walk(self):
        print(f"{self.name} is walking")

    def talk(self):
        print(f"{self.name} is talking")
```

**Student 클래스**
```python
class Student(Human):
    def __init__(self, name, age):
        super().__init__(name, age)

    def __walk(self):
        print(f"Student {self.name} is walking")

    def talk(self):
        print(f"Student{self.name} is talking")
```

**UniversityStudent 클래스**
```python
class UniversityStudent(Student):
    def __init__(self, name, age):
        super().__init__(name, age)

    def __walk(self):
        print(f"University Student {self.name} is walking")

    def talk(self):
        print(f"University Student {self.name} is talking")
```

***ComputerScienceStudent 클래스**
```python
class ComputerScienceStudent(UniversityStudent):
    def __init__(self, name, age):
        super().__init__(name, age)

    def __walk(self):
        super()._Human__walk()
        super()._Student__walk()
        super()._UniversityStudent__walk()
        print(f"Student {self.name} who is majoring in Computer Science is walking")

    def talk(self):
        super().talk()
        print(f"Student {self.name} who is majoring in Computer Science is talking")
```

위의 클래스들을 보면 `__walk` 와 `talk` 메서드들에서 각자 서로 다른 문장을 출력하고 있습니다.  
그리고 ComputerScienceStudent 클래스에서 상위 클래스들에서 정의된 메서드들을 모두 부르고 있습니다.  
이 때 `__walk`는 이름 맹글링이 일어났기 떄문에 `_클래스명__walk`의 형태로 상위 클래스들의 메서드에 접근이 가능합니다.  
반면 `talk`의 경우 같은 이름으로 함수를 오버라이드 하기 때문에 `super.talk()`를 통해 직접 extends한 클래스인 UniversityStudent의 talk()까지 접근할 수 있게 됩니다.  

아래와 같이 위의 코드를 이용해 ComputerScienceStudent 인스턴스의 `__walk`, `talk` 메서드를 실행시켜보도록 하겠습니다.  
```python
if __name__ == "__main__":
    cs_student = ComputerScienceStudent("CS", 20)
    cs_student._ComputerScienceStudent__walk()
    print("\n######end walking#######\n")
    cs_student.talk()
```

다음과 같은 결과를 확인할 수 있습니다. 
![](/assets/img/2022-12/2022-12-10-python_property/name_mangling_practice.png)

<br>

# 파이썬의 프로퍼티
파이썬에서는 객체의 속성에 접근을 제어할 때 프로퍼티를 사용할 수 있습니다.  
프로퍼티는 타 언어의 getter, setter와 같은 역할을 수행합니다.  

프로퍼티를 이용하면 값을 반환하는 메서드(getter 역할)와 속성값의 상태를 변환하는 메서드(setter 역할)를 분리 가능합니다.  

프로퍼티를 사용하는 예제를 생성해보도록 하겠습니다.  
Human 객체의 속성 중 `_age`를 private 속성으로 두고, `@property` 애너테이션을 통해 속성값을 확인할 수 있게 하였고, `@age.setter`를 통해 속성값을 변경할 수 있게 하였습니다.  
여기에서 setter 로직에 나이의 범위를 확인하여 나이가 너무 적거나 많으면 예외를 발생시키도록 하였습니다.  
```python
class Human(object):
    def __init__(self, name, age):
        self.name = name
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        if age > 200 or age < 0:
            raise NotValidAgeException
        self._age = age


if __name__ == "__main__":
    human = Human("Mary", 20)
    print(f"{human.name} is {human.age} years old")
    
    print("### 1 year later ###")
    human.age = human.age + 1
    print(f"{human.name} is {human.age} years old")
    
    print("### 200 year later ###")
    human.age = human.age + 200
 
```

위 코드의 수행 결과는 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-10-python_property/property_practice.png)