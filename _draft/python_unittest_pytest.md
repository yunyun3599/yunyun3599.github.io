# 단위 테스트 - unittest, pytest

## 파이썬 단위 테스트 프레임워크  
파이썬 코드를 단위 테스트할 때는 사용할 수 있는 많은 도구가 있는데요, 거의 모든 시나리오를 다룰 수 있는 두 가지 도구를 대표적으로 살펴보도록 하겠습니다.  
1. unittest: 파이썬 표준 라이브러리에 포함
2. pytest: pip를 통해 설치되는 라이브러라  

### unittest vs pytest
unittest와 pytest 중 어떤 것을 어떤 상황에 사용하는 것이 좋을까요?  
만약 테스트 시나리오만 다루는 것으로 충분한 시스템이라면 다양한 헬퍼 기능을 제공하는 unittest 만으로도 충분히 테스트를 진행할 수 있습니다.  
그러나 코드 내에서 외부 시스템에 연결하는 등의 의존성이 많은 경우에는 테스트 케이스를 파라미터화 할 수 있는 픽스처(fixture)라는 패치 객체가 필요해집니다.  
이처럼 복잡한 옵션이 필요한 경우에는 pytest를 사용하는 것이 더 적절합니다.  

## test코드 작성을 위한 예시 코드  
unittest와 pytest를 이용해 테스트코드를 작성해보기 위해 테스트 수행이 필요한 비즈니스 코드 예시를 살펴보도록 하겠습니다.  
```python
from dataclasses import dataclass
from exceptions import NoItemInCartException, ItemAlreadyInCartException


@dataclass
class Item:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __eq__(self, other):
        return self.name == other.name


class Cart:
    def __init__(self):
        self.item_list = []

    @property
    def price(self):
        return sum([item.price for item in self.item_list])

    def add_item(self, item):
        if item in self.item_list:
            raise ItemAlreadyInCartException(f"{item.name} is already in the cart")
        self.item_list.append(item)

    def delete_item(self, item):
        if item not in self.item_list:
            raise NoItemInCartException(f"{item.name} is not in the cart")
        self.item_list.remove(item)

    def order(self):
        print(f"Order all items in the cart: {[item.name for item in self.item_list]}")
        self.item_list = []
```

위의 코드에서는 두 가지 클래스를 생성합니다.  
먼저 `Item` 객체는 사용자가 구매하고자하는 상품 각각을 나타내며 `name(상품명)`과 `price(상품 가격)`를 속성으로 갖습니다.  
두번째로 `Cart` 객체는 사용자의 장바구니를 의미하며 `item_list`라는 속성을 가져 장바구니에 넣은 Item 인스턴스들을 리스트로 가지고 있습니다.  
또한 `Cart`의 `price` 속성은 장바구니에 넣은 모든 상품의 가격 총합을 나타냅니다.  

Cart 객체에는 간단한 메서드 3개가 구현되어있는데, 각각 다음과 같습니다.  
1. add_item
  - 상품을 `item_list`에 추가하는 함수
  - 이미 존재하는 상품을 넣으려고 하면 `ItemAlreadyInCartException` 예외 발생
2. delete_item
  - `item_list`에 있는 상품을 빼는 함수
  - 없는 상품을 빼려고 하는 경우 `NoItemInCartException` 예외 발생
3. order 함수
  - `item_list`에 있는 상품을 모두 주문 후 `item_list`를 비우는 함수  

## unittest
앞에서 작성한 예시 코드를 활용해 unittest 모듈을 통해 단위테스트를 수행해보도록 하겠습니다.  
unittest는 java JUnit을 기반으로하며 객체지향적이기 때문에 테스트 역시 객체를 이용해 작성되며 클래스의 시나리오별로 테스트를 그룹화하는 것이 일반적입니다.   

unittest를 통해 단위테스트를 하려면 `unittest.TestCase`를 상속하여 테스트 클래스를 만들고, 메서드에 테스트할 조건을 정의하면 됩니다.  
이 때 메서드명은 `test_`로 시작해야하며 `unittest.TestCase`에서 제공하는 메서드를 활용하여 체크하려는 조건이 참인지 확인하면 됩니다.  

### unittest를 이용한 테스트 수행
다음은 앞에서 확인한 `Cart` 객체의 동작을 확인하기 위해 unittest를 이용해 작성한 테스트 코드의 예시입니다.  
```python
import unittest
from item_cart import Cart, Item
from exceptions import ItemAlreadyInCartException, NoItemInCartException


class TestCartItem(unittest.TestCase):
    def test_add_item(self):
        cart = Cart()
        cart.add_item(Item("keyboard", 30000))
        self.assertEqual(cart.item_list[0], Item("keyboard", 30000))

    def test_add_already_exist_item(self):
        cart = Cart()
        item = Item("keyboard", 30000)
        cart.add_item(item)
        self.assertRaises(ItemAlreadyInCartException, cart.add_item, item)

    def test_delete_item(self):
        cart = Cart()
        item = Item("keyboard", 30000)
        cart.add_item(item)
        cart.delete_item(item)
        self.assertEqual(cart.item_list, [])

    def test_delete_not_exist_item(self):
        cart = Cart()
        item = Item("keyboard", 30000)
        self.assertRaisesRegex(NoItemInCartException, f"{item.name} is not in the cart", cart.delete_item, item)
```
작성한 테스트 코드는 `add_item`과 `delete_item` 메서드에 각각에 대해 정상 동작을 확인하는 테스트, 예외 발생 여부를 확인하는 테스트로 구성되어있습니다.  
정상 동작을 확인하는 경우에는 테스트를 하고자 하는 메서드를 호출한 후 기대한 결과값과 실제 수행 값이 동일한 지 확인하는 `assertEqual` 함수를 사용해 검증을 진행했습니다.  
예외 상황에 지정한 예외가 제대로 발생하는 지 확인하기 위해서는 `assertRaises`와 `assertRaisesRegex` 함수를 사용했는데요, `assertRaises`는 함수 수행 중 특정 예외가 발생했는 지 여부를 확인해 해당 예외가 발생하면 테스트에 성공합니다.  
`assertRaisesRegex`는 특정 예외가 발생했는지와 더불어 예외 메세지가 설정한 정규식에 부합하는지 여부도 함께 확인합니다.  


### 테스트 파라미터화
테스트 시에 추가적으로 사용해볼 수 있는 유용한 메서드는 바로 `setUp()` 메서드입니다.  
`setUp` 메서드는 테스트 수행 전에 동작하도록 설정된 함수로, 모든 테스트 수행 전에 공통적으로 필요한 준비 작업 등을 위치시키면 코드의 중복 없이 원하는 로직을 동작시킬 수 있습니다.  

위 코드에 다음과 같이 `setUp()` 메서드를 추가하여 테스트 전반에 사용될 픽스처를 정의해보도록 하겠습니다.   
```python
def setUp(self) -> None:
    self.test_data = [
        ([Item('A', 10000), Item('B', 20000)], 30000),
        ([], 0)
    ]
```

`setUp()` 메서드에서 정의한 `test_data`를 이용해 `Cart` 객체의 `price` 속성값이 옳게 계산되는 지 여부를 확인하는 메서드를 작성해보도록 하겠습니다.  
```python
def test_price(self):
    cart = Cart()
    for item_list, result in self.test_data:
        with self.subTest(item_list=item_list):
            cart.item_list = item_list
            price = cart.price
            self.assertEqual(price, result)
```
`test_data`에 들은 테스트 케이스를 반복문을 통해 하나씩 확인해보고있는 것을 확인할 수 있습니다.  
이 때 `subTest`를 사용한 부분이 있는데요, subTest는 호출되는 테스트 조건을 표시하는 데 사용되는 헬퍼로 반복 중 하나가 실패하면 unittest는 subTest에 전달된 변수의 값을 보고합니다.  

아래처럼 실패하는 테스트 케이스를 하나 추가한 후 테스트를 수행해보도록 하겠습니다.  
```py
    def test_price_with_error(self):
      cart = Cart()
      error_included_data = self.test_data + [([Item('A', 10000), Item('B', 20000)], 10000)]
      for item_list, result in error_included_data:
          with self.subTest(item_list=item_list):
              cart.item_list = item_list
              price = cart.price
              self.assertEqual(price, result)
```
수행 결과 아래와 같이 문제가 되는 테스트 케이스에 대한 `AssertionError`가 발생하며 문제가 되는 테스트 케이스의 값을 반환합니다.  
![](/assets/img/2023/05/2023-05-28-python_unittest/unittest_subTest_failed.png)

## pytest
pytest를 이용하기 위해서는 `pip install pytest` 커맨드를 통해 패키지를 설치해야합니다.  
pytest를 통해 테스트를 할 때는 unittest처럼 테스트 시나리오를 통해 객체 지향 모델을 생성해 진행할 수도 있지만 필수 사항은 아닙니다.  
pytest를 통한 테스트에서는 단순히 assert 구문을 사용해 조건을 검사하는 것이 가능합니다.  

pytest는 pytest 명령어를 통해 unittest로 작성된 테스트를 포함하여 탐색 가능한 모든 테스트를 한 번에 실행하는 것이 가능합니다.  
이러한 호환성 덕분에 unittest에서 pytest로의 점진적인 전환이 가능합니다.  

### pytest를 이용한 테스트 수행
pytest를 이용해 위에서 unittest를 이용해 수행했던 테스트와 동일한 내용의 테스트 코드를 작성해보도록 하겠습니다.  
```python
import pytest
from item_cart import Cart, Item
from exceptions import ItemAlreadyInCartException, NoItemInCartException


def test_add_item():
    cart = Cart()
    cart.add_item(Item("keyboard", 30000))
    assert cart.item_list[0] == Item("keyboard", 30000)


def test_add_already_exist_item():
    cart = Cart()
    item = Item("keyboard", 30000)
    cart.add_item(item)
    pytest.raises(ItemAlreadyInCartException, cart.add_item, item)


def test_delete_item():
    cart = Cart()
    item = Item("keyboard", 30000)
    cart.add_item(item)
    cart.delete_item(item)
    assert cart.item_list == []


def test_delete_not_exist_item():
    cart = Cart()
    item = Item("keyboard", 30000)
    # 예외 메세지 정규식도 함께 확인  
    with pytest.raises(NoItemInCartException, match=f"{item.name} is not in the cart"):
        cart.delete_item(item)
```
값 검증을 위한 테스트인 `test_add_item`, `test_delete_item` 메서드에서 볼 수 있듯이 pytest 결과값 검증은 `assert value_1 == value_2` 수준으로 충분합니다.  
다만 예외 발생 유무를 검사하려면 일부 함수를 사용해야합니다.  
이 때 사용 가능한 함수는 `pytest.raises`로 특정 예외 발생 여부를 검사할 수 있습니다.  
`pytest.raises` 함수는 `test_add_already_exist_item` 함수에서 사용된 것처럼 메서드 형태와 `test_delete_not_exist_item`에서 사용한 것처럼 컨텍스트 관리자 형태로 사용이 가능합니다.  
또한 생성된 예외 메세지도 확인하려면 똑같이 `pytest.raises` 함수를 활용하되 `match` 파라미터에 확인하려는 표현식을 전달하면 됩니다.  

### 테스트 파라미터화  
pytest를 활용해 파라미터화된 테스트를 수행하면 테스트 조합마다 새로운 테스트 케이스를 생성하기 때문에 더 좋은 테스트르 수행할 수 있습니다.  
파라미터화된 테스트를 활용하기 위해서는 `pytest.mark.parametrize` 데코레이터를 사용해야합니다.  

이 때 데코레이터는 2개의 파라미터를 받게 되는데 각 파라미터는 다음 내용을 받습니다.  
- 데코레이터의 첫번째 파라미터: 테스트 함수에 전달할 파라미터의 이름
- 데코레이터의 두번째 파라미터: 해당 파라미터에 대한 각각의 값으로 반복 가능해야함  

`pytest.mark.parametrize` 데코레이터의 사용 예시 코드는 다음과 같습니다.  
```python
@pytest.mark.parametrize("item_list, total_price", [([Item('A', 10000), Item('B', 20000)], 30000), ([], 0)])
def test_price(item_list, total_price):
    cart = Cart()
    cart.item_list = item_list
    price = cart.price
    assert price == total_price
```

위의 코드를 확인해보면 테스트 함수에 `item_list`와 `total_price`라는 파라미터명으로 데이터를 전달하고있고, 데코레이터의 두번째 파라미터로 `[([아이템1, 아이템2], 아이템 가격 총합), (([아이템3, 아이템4], 아이템 가격 총합))]` 형태의 리스트 데이터를 주어 사용할 테스트 데이터를 정의하고 있습니다.  
테스트 함수에서는 데코레이터의 첫번째 인자로 전달된 파라미터명들을 그대로 받아와 반복 가능한 값들을 하나씩 사용하여 조건에 부합하는 지 확인합니다.  

### fixture
pytest를 활용하면 fixture를 이용해 재사용 가능한 기능을 쉽게 만들 수 있습니다.  
또한 특정 값을 가진 데이터 객체를 만들어두고 여러 테스트에서 이 객체를 사용할 수 있습니다.  

픽스처를 정의하기 위해서는 함수를 만들고 `@pytest.fixture` 데코레이터를 적용하며 이렇게 생성된 픽스처를 사용할 때는 원하는 테스트에 파라미터로 픽스처의 이름을 전달하면 pytest가 이를 활용합니다.  

픽스처를 사용한 예시 테스트 코드를 살펴보도록 하겠습니다.  
```python
# 픽스처를 정의하고 해당 픽스처를 사용할 함수에서 픽스처 이름으로 파라미터 호출해 사용
@pytest.fixture
def price_test_case():
    item_list = [Item('A', 10000), Item('B', 20000)]
    result_price = 30000
    return {"item_list": item_list, "result_price": result_price}


# price_test_case라는 픽스처를 호출해서 테스트 데이터로 이용
def test_price_with_fixture(price_test_case):
    cart = Cart()
    new_item = Item('A', 10000)
    cart.item_list = price_test_case["item_list"] + [new_item]
    assert cart.price == price_test_case["result_price"] + new_item.price


def test_price_after_order(price_test_case):
    cart = Cart()
    cart.item_list = price_test_case["item_list"]
    cart.order()
    assert cart.price == 0
```

코드를 확인하면 `price_test_case`라는 함수를 통해 사용할 데이터를 생성하고 있으며 `@pytest.fixture` 데코레이터를 적용해 해당 함수의 반환값을 픽스처로 사용할 것임을 나타내고 있습니다.  
이렇게 생성된 데이터는 타 함수에서 함수명인 `price_test_case`라는 파라미터를 받도록 하여 활용 가능합니다.  

`test_price_with_fixture`와 `test_price_after_order` 함수는 둘 다 `price_test_case`를 사용하기 위해 해당 함수명을 파라미터로 받고있으며, 사용된 데이터를 활용해 원하는 테스트 시나리오를 수행하고 있는데, 공통적으로 동일한 데이터를 활용해 테스트를 수행하므로 픽스처를 이용해 불필요한 코드 중복을 줄이고 있습니다.  

이렇게 테스트 데이터를 생성해 여러 군데에서 사용하는 것 이외에도 픽스처는 직접 호출되지 않는 함수를 수정하거나 사용할 객체를 미리 설정하는 등의 사전 조건 설정에 사용될 수 있습니다.  
