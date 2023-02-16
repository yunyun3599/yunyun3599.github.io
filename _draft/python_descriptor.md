# 파이썬 디스크립터

디스크립터는 파이썬의 기능 중 하나로 다음 4가지 매직 메서드 중 최소 한 개 이상을 구현한 클래스를 의미합니다.  
1. \_\_get\_\_
2. \_\_set\_\_
3. \_\_delete\_\_
4. \_\_set_name\_\_

디스크립터 객체는 다른 객체의 속성으로 정의될 수 있으며, 해당 속성에 대한 읽기, 쓰기, 삭제 연산을 할 때 구현된 매직 메서드가 호출됩니다.  

디스크립터 구현을 위해 필요한 클래스로는 2가지가 있습니다.  
1. ClientClass: 디스크립터로 구현된 기능을 사용할 구현체 (class 속성으로 descriptor 인스턴스를 가짐)
2. DescriptorClass: 

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