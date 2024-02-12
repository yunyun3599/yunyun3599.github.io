# 파이썬으로 알아보는 디자인패턴  

## 디자인 패턴
디자인 패턴은 개발 중에 자주 발생하는 일반적인 문제들을 어떻게 추상화하여 해결할 수 있는 지에 대한 개념입니다.   
이번 포스트에서는 디자인 패턴을 적용한 코드와 그렇지 않은 코드를 파이썬을 이용해 비교해보도록 하겠습니다.  
그 과정에서 동적인 언어인 파이썬의 특성으로 인해 일반적인 정적인 언어에서는 유용한 디자인 패턴이 파이썬에서는 오히려 좋지 못한 결과를 내는 경우도 함께 살펴보도록 하겠습니다.  

### 디자인 패턴의 종류   
디자인 패턴은 크게 생성 패턴, 구조 패턴, 행동 패턴으로 나뉘어집니다.   
이번 포스트에서는 생성 패턴에 대해 살펴보도록 하겠습니다.  

> **생성(creational) 패턴**
>  - 생성 패턴은 객체를 인스턴스화 할 때의 복잡성을 최대한 추상화하기 위한 것입니다.  
>  - 객체 초기화를 위한 파라미터를 결정하거나 초기화에 필요한 객체를 준비하는 것 등의 모든 관련 작업을 단순화하는 데 사용됩니다.  
>  - 생성 패턴을 통해 간단한 인터페이스를 제공할 수 있게 되며, 더 안전하게 객체를 생성할 수 있게 됩니다.  
>  - 이번 포스트에서는 파이썬에서 많이 사용되는 Borg 패턴으로 싱글턴 패턴을 대체하는 방법에 대해 알아보도록 하겠습니다.  


## 생성 패턴
### 팩토리
파이썬은 언어의 특성상 모든 것이 객체입니다.  
따라서 클래스, 함수 혹은 사용자 정의 객체 각각의 역할이 특별히 구분되어 있지 않으며 모두 똑같이 취급될 수 있습니다.  
모든 것이 객체이므로 모두 할당이나 파라미터에 사용 가능하므로 파이썬에서는 팩토리 패턴이 별로 필요하지 않습니다.  
객체를 생성할 수 있는 함수를 만들거나 생성하려는 클래스를 파라미터로 전달할 수 있기 때문입니다.  

### 싱글턴과 공유 상태  
싱글턴 패턴은 다르게 표현하면 전역 변수의 한 형태라고 할 수 있습니다.  
이는 싱글턴 패턴을 사용하면 어떤 객체에 의해서 언제든지 해당 싱글턴 객체가 수정될 수 있다는 것을 의미하며, 코드를 예측하기 어렵게 만듭니다.  
일반적으로 싱글턴은 사용하지 않는 것이 좋은데, 그럼에도 불구하고 꼭 필요한 경우가 있다면 `모듈`을 사용하는 것이 대안이 될 수 있습니다.  
파이썬에서는 모듈에 객체를 생성할 수 있으며, 모듈을 임포트한 모든 곳에서 사용이 가능합니다.  
객체를 여러번 임포트 하더라도 sys.modules에 로딩되는 것은 항상 한 개이므로 모듈은 이미 싱글턴이라는 것을 의미합니다.  

#### 공유 상태  
하나의 인스턴스로 표현되는 싱글턴을 사용하는 것보다 여러 인스턴스에서 사용할 수 있도록 데이터를 복제해 사용하는 것이 더 좋은 방법입니다.  
`모노 스테이트 패턴`에서는 싱글턴인지 아닌지에 상관 없이 일반 객체처럼 많은 인스턴스를 만들 수 있어야 합니다.  
`모노 스테이트 패턴`을 사용하면 완전히 투명한 방법으로 정보를 동기화하므로 사용자는 객체가 내부에서 로직이 어떻게 동작하는지 신경쓰지 않아도 된다는 장점이 있습니다.  

다음 코드는 `모노 스테이트 패턴`을 이용해 어떻게 정보를 동기화하는 지 확인해보기 위한 코드입니다.  
```py
class SharedAttribute:
    # 공유 속성
    _shared_attribute = None

    def __init__(self, attribute):
        self.shared_attribute = attribute

    @property
    def shared_attribute(self):
        if self._shared_attribute is None:
            raise AttributeError("attribute가 초기화되지 않음")
        return self._shared_attribute

    @shared_attribute.setter
    def shared_attribute(self, new_attribute):
        self.__class__._shared_attribute = new_attribute

    def get_attribute(self):
        return self.shared_attribute


if __name__ == "__main__":
    print("### c1 인스턴스 공유속성 'one'으로 초기화 ###")
    c1 = SharedAttribute(attribute="one")
    print(f"c1의 공유 속성 값: {c1.get_attribute()}")   # c1의 공유 속성 값: one

    print("\n### c2 인스턴스 공유속성 'two'로 초기화 ###")
    c2 = SharedAttribute(attribute="two")
    print(f"c1의 공유 속성 값: {c1.get_attribute()}")   # c1의 공유 속성 값: two
    print(f"c2의 공유 속성 값: {c2.get_attribute()}")   # c2의 공유 속성 값: two

    print("\n### c2 인스턴스 공유속성 'three'로 변경 ###")
    c2.shared_attribute = "three"
    print(f"c1의 공유 속성 값: {c1.get_attribute()}")   # c1의 공유 속성 값: thr
    print(f"c2의 공유 속성 값: {c2.get_attribute()}")   # c2의 공유 속성 값: three
```

공유 속성을 더 캡슐화하고 싶다거나 더 많은 속성에 대한 공유가 필요하다면 디스크립터를 사용할 수 있습니다.    
디스크립터를 활용하면 코드가 분리되어 더 응집력있는 코드를 짤 수 있게 되어 단일 책임 원칙을 준수할 수 있다는 장점이 있습니다.   
다만 디스크립터를 사용하면 더 많은 코드가 필요하다는 문제가 발생하므로 자주 재사용될 것 같은 코드 위주로 활용하는 것이 좋습니다.   
디스크립터를 활용한 코드는 다음과 같습니다.  
```py
class DescriptorAttribute:
    def __init__(self, initial_value=None):
        self.value = initial_value
        self._name = None

    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.value is None:
            raise AttributeError(f"{self._name} was never set")
        return self.value

    def __set__(self, instance, new_value):
        self.value = new_value

    def __set_name__(self, owner, name):
        self._name = name


class SharedAttribute:
    shared_attribute1 = DescriptorAttribute()
    shared_attribute2 = DescriptorAttribute()

    def __init__(self, attr1, attr2):
        self.shared_attribute1 = attr1
        self.shared_attribute2 = attr2

    def get_attribute1(self):
        return self.shared_attribute1

    def get_attribute2(self):
        return self.shared_attribute2


if __name__ == "__main__":
    print("### c1 인스턴스 공유속성 각각 'a', 'b'로 초기화 ###")
    c1 = SharedAttribute(attr1="a", attr2="b")
    print(f"c1의 공유 속성 1 값: {c1.get_attribute1()}")    # c1의 공유 속성 1 값: a
    print(f"c1의 공유 속성 2 값: {c1.get_attribute2()}")    # c1의 공유 속성 2 값: b

    print("\n### c2 인스턴스 공유속성 각각 'A', 'B'로 초기화 ###")
    c2 = SharedAttribute(attr1="A", attr2="B")
    print(f"c1의 공유 속성 1, 2 값: ({c1.get_attribute1()}, {c1.get_attribute2()})")    # c1의 공유 속성 1, 2 값: (A, B)
    print(f"c2의 공유 속성 1, 2 값: ({c2.get_attribute1()}, {c2.get_attribute2()})")    # c2의 공유 속성 1, 2 값: (A, B)

    print("\n### c2 인스턴스 공유속성 각각 'C', 'D'로 변경 ###")
    c2.shared_attribute1 = "C"
    c2.shared_attribute2 = "D"
    print(f"c1의 공유 속성 1, 2 값: ({c1.get_attribute1()}, {c1.get_attribute2()})")    # c1의 공유 속성 1, 2 값: (C, D)
    print(f"c2의 공유 속성 1, 2 값: ({c2.get_attribute1()}, {c2.get_attribute2()})")    # c2의 공유 속성 1, 2 값: (C, D)
```

#### borg 패턴
borg 패턴의 경우 싱글턴을 꼭 사용해야 하는 경우 취할 수 있는 대안 중 하나입니다.  
borg 패턴 역시 모노 스테이트 패턴인데요, 주요 개념은 같은 클래스의 모든 인스턴스가 모든 속성을 복제하도록 객체를 만드는 것입니다.  
borg 패턴을 구현한 예시 코드는 아래와 같습니다.  
```py
class BorgClass:
    _attributes = {}

    def __init__(self):
        self.__dict__ = self.__class__._attributes

    def get_attributes(self):
        return f"self._dict__ = {self.__dict__}"


if __name__ == "__main__":
    cl1 = BorgClass()
    cl1.common_attribute = "common1"
    print(cl1.get_attributes())     # self._dict__ = {'common_attribute': 'common1'}

    print("\ncreate c2 class")
    cl2 = BorgClass()
    print(cl1.get_attributes())     # self._dict__ = {'common_attribute': 'common1'}
    print(cl2.get_attributes())     # self._dict__ = {'common_attribute': 'common1'}

    print("\nchange value of common attribute")
    cl1.common_attribute = "new common value"
    print(cl1.get_attributes())     # self._dict__ = {'common_attribute': 'new common value'}
    print(cl2.get_attributes())     # self._dict__ = {'common_attribute': 'new common value'}
```

이와 같이 borg 패턴을 사용하면 각 인스턴스가 초기화 될 때 클래스 변수를 사용하게 되므로 특정 속성들을 동일하게 갖게 됩니다.  
그러나 위의 코드에서는 한 가지 문제점을 발견할 수 있는데요, 바로 `BorgClass` 함수에서 공용속성으로 사용하기 위한 `dictionary` 객체를 직접 할당하고 있다는 것입니다.  
기본 클래스에 `dictionary`와 관련된 로직을 뺴기위해선 `Mixin`을 사용할 수 있습니다.  
```py
class SharingMixin:
    def __init__(self, *args, **kwargs):
        try:
            self.__class__._attributes
        except AttributeError:
            self.__class__._attributes = {}
        self.__dict__ = self.__class__._attributes
        super().__init__(*args, **kwargs)


class ParentBorgClass:
    def __init__(self, common_attribute):
        self.common_attribute = common_attribute


class BorgClass(SharingMixin, ParentBorgClass):
    def get_attributes(self):
        return f"common_attribute = {self.common_attribute}"

if __name__ == "__main__":
    print("#### BorgClass1 Example ####")
    c1 = BorgClass(common_attribute="c1_common_attribute")
    print(f"c1's {c1.get_attributes()}")          # c1's common_attribute = c1_common_attribute

    print("\ncreate c2 class")
    c2 = BorgClass("c2_common_attribute")
    print(f"c1's {c1.get_attributes()}")          # c1's common_attribute = c2_common_attribute
    print(f"c2's {c2.get_attributes()}")          # c2's common_attribute = c2_common_attribute

    print("\nchange value of common attribute")
    c1.common_attribute = "c1 new common value"
    print(f"c1's {c1.get_attributes()}")          # c1's common_attribute = c1 new common value
    print(f"c2's {c2.get_attributes()}")          # c2's common_attribute = c1 new common value
```
이처럼 borg 패턴을 사용하면 공통 속성을 갖는 인스턴스들을 생성해서 활용할 수 있게 됩니다.  

#### 빌더
빌더 패턴은 객체의 복잡한 초기화를 추상화하기 위한 패턴입니다.  
빌더 패턴도 다른 패턴 및 구문과 동일하게 여러 사용자가 반복 사용하게 되는 부분에 구현하는 것이 좋습니다.   
빌더 패턴은 필요로하는 객체를 직접 생성해주는 하나의 객체를 만드는 패턴입니다.  
즉 사용자가 특정 객체 생성에 필요한 제 3의 객체들을 직접 생성해서 전달하는 것이 아니라, 이러한 과정들을 포함하여 모든 것을 처리해주는 추상화된 객체를 이용해야 한다는 것입니다.  
빌더 객체는 필요한 모든 것들을 어떻게 생성하고 연결하는 지 알고 있습니다.  
궁극적으로 인스턴스를 얻는 것이 목적이므로 빌더 객체는 클래스 메서드 형태의 사용자 인터페이스를 제공하며, 사용자는 최종 객체 생성에 필요한 모든 정보를 해당 인터페이스에 파라미터로 전달하여 원하는 인스턴스를 생성하게 됩니다.  
빌더 패턴의 간단한 예제 코드는 아래와 같습니다.   
```py
class Student:
    def __init__(self):
        self.id = None
        self.name = None
        self.age = None
        self.grade = None
        self.teacher = None


class StudentBuilder:
    def __init__(self):
        self.student = Student()

    def set_id(self, id):
        self.student.id = id
        return self

    def set_name(self, name):
        self.student.name = name
        return self

    def set_age(self, age):
        self.student.age = age
        return self

    def set_grade(self, grade):
        self.student.grade = grade
        return self

    def set_teacher(self, teacher):
        self.student.teacher = teacher
        return self

    def build(self):
        return self.student


if __name__ == "__main__":
    John = StudentBuilder()\
        .set_id("30321")\
        .set_name("John")\
        .set_age(16)\
        .set_grade(3)\
        .set_teacher("Colin")\
        .build()
    print(John.__dict__)

```
