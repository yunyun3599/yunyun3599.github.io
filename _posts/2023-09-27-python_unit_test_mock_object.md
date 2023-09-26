---
title:  "파이썬 단위테스트 - Mock"
excerpt: "파이썬 단위테스트 시 Mock 객체를 사용하는 방법에 대해 알아봅니다. "

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2023-09-27
last_modified_at: 2023-09-27
---
# 파이썬 단위 테스트 - Mock 객체  

## 모의 객체와 단위 테스트
### 모의 객체를 사용하는 상황
시스템이 실제로 구동될 때는 데이터베이스, 스토리지 서비스, 외부 API 등 각종 외부 서비스와 연결됩니다.  
외부 서비스를 이용하면 필연적으로 부작용이 발생하며, 이런 부작용을 최소화하기 위해 각종 인터페이스를 통해 추상화하겠지만, 이러한 부분 역시 테스트에 포함되게 됩니다.  
이와 같은 상황에서 모의 객체는 원치 않는 부작용으로부터 테스트 코드를 보호하는 가장 좋은 방법 중 하나입니다.  

### 단위 테스트와 통합테스트의 범위
단위 테스트는 데이터베이스 연결, HTTP 요청을 비롯한 외부 서비스들을 테스트하지 않습니다.  
외부 서비스에 대한 테스트는 작성한 로직에서 벗어날 뿐 아니라 단위 테스트는 빠르게 실행되어야하는데 외부 연결에 대한 대기시간을 기다려야하는 것은 단위 테스트의 취지와 맞지 않습니다.  
그러므로 단위 테스트에서는 외부 서비스 호출에 대한 내부 코드가 정상 동작하는 지 정도만 테스트하며 외부 서비스가 정상 동작하여 올바른 결과를 돌려받는 지에 대해서는 확인하지 않습니다.  

그렇다면 외부 서비스까지 포함하여 코드 전반을 테스트하는 것은 언제 실행될까요?  
이러한 전체 로직에 걸친 테스트는 통합 테스트에서 수행합니다.  
통합테스트는 아래와 같은 특징을 갖습니다.  
- 통합테스트는 실제 사용자의 행동을 모방하여 더 넓은 관점에서 기능을 테스트합니다.  
- 통합 테스트는 외부 시스템과 서버에 직접 연결하기 때문에 시간이 오래 걸리고 비용이 많이 듭니다.   
- 일반적으로 단위테스트는 많이 & 항상 실행하는 것에 반해 통합 테스트는 덜 자주 실행하며 주로 특별한 이벤트가 발생한 경우 수행합니다.  

### 코드와 단위 테스트의 관련성  
단위 테스트는 보다 나은 코드를 작성하는 데 도움이 되는데, 단위 테스트가 잘 작성된다는 것은 코드가 응집력이 뛰어나고 세분화되어있으며 작게 잘 짜여져있다는 것을 의미합니다.   
또한 테스트를 통해 작성된 코드가 잘못 작성되었는지 여부를 추론 가능합니다.  
만약 간단한 테스트 케이스 작성을 위해 다양한 몽키패치를 해야한다면 코드가 잘못 설계되었다는 신호입니다.  
unittest 모듈은 `unittest.mock.patch`에서 객체를 패치하기 위한 도구를 제공합니다.  
> 패치: import 중에 경로를 지정했던 원본 코드를 모의 객체같은 다른 것으로 대체하는 것  

>패치를 하면 런타임 중에 참조되는 코드가 바뀌고 원래 코드와의 연결이 끊어지므로 테스트가 조금 더 어려워지는 단점이 있습니다.  
또한 런타임 시 인터프리터에서 객체를 수정하는 오버헤드도 있기 때문에 성능상의 이슈도 발생할 수 있습니다.  

> 참고로 몽키패치 또는 모의를 사용하는 것 자체가 문제가 되는 것은 아니나 몽키패치를 남용하게 된다면 코드 개선의 여지가 있다고 해석할 수 있습니다.  


## 테스트 더블
테스트 더블이란 테스트 스위트에서 실제 코드를 대신해 실제인 것처럼 동작하는 코드입니다.  
테스트 더블을 사용하게 되는 상황은 다음과 같습니다.  
1. 실제 상용 코드가 필요하지 않은 경우
2. 특정 서비스에 접근해야하는데 권한이 없는 경우
3. 부작용이 있기 때문에 단위테스트에서 실행하고 싶지 않은 경우  

테스트 더블의 종류로는 더미(dummy), 스텁(stub), 스파이(spy), 모의(mock) 등이 있습니다. 여기에서 mock은 가장 일반적인 형태의 객체로 모든 경우에 적합합니다.  
또한 표준 라이브러리에도 모의 객체가 포함되어 있기 때문에 `unittest.mock.Mock`을 Import하여 모의 객체를 사용 가능합니다.  

### Mock 객체
Mock 객체는 스펙을 따르는 객체 타입으로 응답값을 직접 수정할 수 있습니다.  
특정 메서드가 호출되었을 때 응답해야하는 값이나 행동을 특정할 수 있는 것입니다.  
Mock 객체는 내부에 호출 방법을 기록하고 이 정보를 사용해 애플리케이션의 동작을 검증합니다.  
파이썬 표준 라이브러리의 Mock 객체 역시 호출 횟수, 사용된 파라미터 등 모든 종류의 검증을 할 수 있는 API를 제공하고 있습니다.  

### Python Mock 객체의 종류  
Mock 객체는 파이썬의 `unittest.mock` 모듈에 위치해있습니다.  

Mock 객체의 종류로는 Mock과 MagicMock이 있는데, 전반적으로 유사하나 아래와 같은 케이스에서 다르게 동작합니다.   
- Mock: 모든 값을 반환하도록 설정할 수 있는 테스트 더블로 모든 호출을 추적 가능합니다.  
- MagicMock: Mock과 동일하나 매직 메서드를 지원합니다. (Magic Method 사용 시에는 Mock이 아닌 MagicMock을 사용해 테스트를 진행해야합니다.)  

### Mock & MagicMock을 사용한 테스트 코드 예제
위에서 확인한 사항들을 검증해보기 위해 Mock과 MagicMock을 사용해 테스트를 진행하는 간단한 코드를 살펴보도록 하겠습니다.  

아래 `Voting` 객체는 투표를 진행하고 현재 투표 현황을 리턴해주는 객체입니다.  
```python
from unittest.mock import Mock, MagicMock

class Voting:
    def __init__(self, *candidates: str):
        self._result = {candidate: 0 for candidate in candidates}

    @property
    def result(self):
        return self._result

    def vote(self, candidate: str):
        if candidate in self._result.keys():
            self._result[candidate] = self._result[candidate] + 1
            print(f"{candidate}: {self._result[candidate]}")
            return self._result[candidate]
        else:
            raise UnregisteredCandidateException(f"{candidate}는 등록되지 않은 후보입니다.")

    def __getitem__(self, candidate: str):
        return self._result[candidate]

    def __len__(self):
        return len(self._result)
```

`result` property를 통해 현재 투표 현황 전체를 리턴받을 수 있고, `vote` 함수를 통해 특정 후보에게 투표할 수 있으며 `__getitem__` 함수를 구현했기 때문에 `voting` 객체에 대해 `dict`처럼 특정 candidate의 투표수를 리턴받을 수 있습니다.  

매직메서드를 사용하는 경우 `MagicMock`을 활용해야함을 확인하기 위해 `__getitem__` 함수를 테스트해보도록 하겠습니다.  
먼저 Mock 객체를 사용하는 경우입니다.  
```py
from unittest.mock import Mock


def get_vote_num(vote: Voting, candidate: str):
    return vote[candidate]


def test_get_vote_num_with_mock():
    vote = Mock()
    vote.__getitem__.return_value = 0
    assert get_vote_num(vote, "candidate1") == 0
```

테스트를 수행해본 결과 다음과 같은 에러가 발생합니다.  
![](/assets/img/2023/06/2023-06-25-python_unit_test_mock_object/error_occurs_when_test_magic_method_by_plain_mock.png)   
매직 메서드를 그냥 `Mock`으로 테스트했기 때문에 발생한 에러입니다.  

그러므로 동일한 내용을 `MagicMock`을 이용해 테스트해보도록 하겠습니다.  
```py
from unittest.mock import MagicMock


def test_get_vote_num_with_magic_mock():
    vote = MagicMock()
    vote.__getitem__.return_value = 0
    assert get_vote_num(vote, "candidate1") == 0
```

위의 코드는 문제 없이 테스트에 성공하는 것을 확인할 수 있습니다.  

그렇다면 정말 magic method가 아닌 경우에는 그냥 `Mock`객체를 이용해도 테스트에 성공할까요?  
알아보기 위해 `vote` 함수를 이용해 테스트를 아래와 같이 진행해보도록 하겠습니다.  

```py
def vote_to_candidate(vote: Voting, candidate: str):
    return vote.vote(candidate=candidate)


def test_vote_with_mock():
    vote = Mock()
    vote.vote.return_value = 10
    assert vote_to_candidate(vote, "candidate") == 10


def test_vote_with_magic_mock():
    vote = MagicMock()
    vote.vote.return_value = 10
    assert vote_to_candidate(vote, "candidate") == 10
```
실행 결과 두 테스트 모두 성공함을 확인할 수 있습니다.  
이처럼 매직 메서드를 테스트하려는 상황이 아니라면 일반 `Mock`을 사용해도 동작에는 문제가 없습니다.  

### mock.patch를 사용한 테스트 더블
외부 모듈에 대한 의존성이 큰 경우처럼 테스트를 단순히 수행하기 어려운 경우에는 mock.patch를 시용해 해당 부분을 대체할 수 있습니다.  
예를 들어 데이터베이스에서 특정 데이터를 가져와서 api를 통해 해당 내용을 전송하는 코드가 있다고 생각해봅시다.  
이 코드는 데이터베이스 연결과 HTTP 연결이 모두 필요한 코드로, 테스트 중 외부 모듈로 인해 실패할 가능성이 높습니다.  

이렇게 동작하는 코드 예시를 보도록 하겠습니다.  
```py
import requests
import pymysql

ENDPOINT = "url.of.endpoint"


class DBConnection:
    def __init__(self):
        self.conn = pymysql.connect(host="localhost",
                                    user="root",
                                    password="1234",
                                    db="test",
                                    port=3306,
                                    use_unicode=True,
                                    charset="utf8")

    def get_data(self, query):
        with self.conn.cursor() as cur:
            cur.execute(query=query)
            result = cur.fetchone()
        return result


class Delivery:
    @staticmethod
    def get_status(item_id: str):
        delivery_info = DBConnection().get_data(
            f"""
            select status, update_time 
              from test.delivery 
             where id={item_id}
            """)
        return {"status": delivery_info[0], "update_time": delivery_info[1].strftime('%Y-%m-%d %H:%M:%S')}

    @classmethod
    def send_status(cls, item_id):
        delivery_info = cls.get_status(item_id=item_id)
        delivery_status = {
            "id": item_id,
            "status": delivery_info["status"],
            "update_time": delivery_info["update_time"]
        }
        response = requests.post(ENDPOINT, json=delivery_status)
        response.raise_for_status()
        return response
```
위 코드는 DBConection 객체를 통해 조회를 원하는 택배의 배송 상태와 마지막 상태 업데이트 시간을 가져옵니다.  
그리고 이 정보를 통해 Payload를 구성해 requests 모듈을 통한 HTTP 호출을 수행합니다.  

테스트를 통해 확인하고 싶은 부분은 DBConnection이 잘 이뤄졌는지나 HTTP 연결이 안정적으로 이루어졌는지 보다는 적절한 정보가 API에 잘 전달되었는 지 여부입니다.  
따라서 requests 부분은 실제 호출 없이 mock.patch를 통해 대체할 수 있으며, DB 연결부분 역시 patch를 통해 실제로 수행되지 않도록 테스트 코드를 작성해보도록 하겠습니다.  
```python
from unittest import mock


@mock.patch("mock_patch_delivery_example.requests")
def test_delivery_status_send(mock_request):
    status = 'on_delivery'
    update_time = '2023-07-15 00:00:00'
    mock_status = {"status": status, "update_time": update_time}
    with mock.patch('mock_patch_delivery_example.Delivery.get_status', return_value=mock_status):
        Delivery.send_status(1)

    expected_payload = {"id": 1, "status": status, "update_time": update_time}
    mock_request.post.assert_called_with(ENDPOINT, json=expected_payload)
```
`@mock.patch("mock_patch_delivery_example.requests")` 코드를 통해 requests를 patch하여 해당 코드가 실제로 호출되지 않도록 하였습니다.  
또한 데이터베이스에서 데이터를 가져오는 부분도 `mock.patch` 함수를 컨텍스트 매니저로 사용하여 데이터베이스 연결이 있는 함수의 리턴값을 미리 생성해둔 값으로 설정하였습니다.  

이러한 작업을 통해 requests mock 객체의 post에 특정 데이터의 파라미터가 전달될 경우 상태가 200이 될 것이라는 지정을 한 것입니다.  
따라서 `mock_request.post`에 동일한 파라미터를 사용해 호출하면 `assert_called_with`는 성공하게 됩니다.  


### 리팩토링
patch를 적절히 사용하면 테스트 수행이 용이해지지만 특정 부분을 반복적으로 패치해야한다면 추상화가 잘못된 것이라고 추정할 수 있습니다.  

또한 patch의 경우 모의하려는 객체의 경로를 문자열로 제공해야하기 때문에 파일의 이름을 바꾸거나 다른 위치로 이동시킨다고 가정하면 해당 파일에 대하여 패치를 한 모든 부분의 코드를 변경해야합니다.  

이러한 부분은 리팩토링을 통해 개선할 수 있습니다.  

리팩토링은 구조를 개선하여 보다 나은 코드로 변경하려고 할 때나 좀 더 일반적인 코드로 수정하여 가독성을 높이고자 할 때 수행합니다.  

리팩토링을 할 때 주의해아 할 점은 리팩토링 전후가 완전히 동일한 기능을 유지하여 컴포넌트의 고객 입장에서는 아무 일도 일어나지 않은 것처럼 느끼도록 수정되어야한다는 것입니다.  

따라서 수정된 코드에 대해 테스트를 수행해야하므로 테스트를 자동화해야하며, 테스트를 자동화하는 효율적인 방안으로는 단위 테스트가 있습니다.  

앞서 `patch`를 통해 단위 테스트를 진행한 Delivery 코드를 patch를 시용하지 않고도 효과적으로 단위테스트를 수행할 수 있도록 변경해보도록 하겠습니다.  
주로 변경할 부분은 patch를 했던 `send_status` 함수입니다.  
이전 코드에서는 `send_status` 함수에서 외부 모듈인 `requests`를 직접 호출하여 외부로 HTTP 요청을 보냈습니다.  
이 부분을 의존성을 주입받고 메서드를 더 작은 부분으로 쪼개는 방법으로 리팩토링하도록 하겠습니다.  
```python
import requests

ENDPOINT = "url.of.endpoint"


class Transport:
    def __init__(self, endpoint):
        self.endpoint = endpoint

    def post(self, payload):
        return requests.post(self.endpoint, json=payload)


class Delivery:
    def __init__(self, transport):
        self.transport = transport

    @staticmethod
    def get_status(item_id: str):
        delivery_info = DBConnection().get_data(
            f"""
            select status, update_time 
              from test.delivery 
             where id={item_id}
            """)
        return {"status": delivery_info[0], "update_time": delivery_info[1].strftime('%Y-%m-%d %H:%M:%S')}

    @classmethod
    def make_payload(cls, item_id):
        delivery_info = cls.get_status(item_id=item_id)
        delivery_status = {
            "id": item_id,
            "status": delivery_info["status"],
            "update_time": delivery_info["update_time"]
        }
        return delivery_status

    def send_request(self, payload):
        response = self.transport.post(ENDPOINT, json=payload)
        response.raise_for_status()
        return response

    def send_status(self, item_id):
        return self.send_request(self.make_payload(item_id=item_id))
```
앞에서와 다르네 객체 초기화 시에 Transport 객체를 주입받아 해당 객체에 `requests` 모듈에 대한 부분을 위임하였습니다.  
따라서 Delivery 객체를 테스트 할 때는 더이상 `requests` 모듈을 `patch`할 필요 없습니다.  

`pytest.fixture`를 이용해 필요한 부분을 Mock으로 변경한 Delivery 객체를 만들고 테스트를 수행해보도록 하겠습니다.  
```python
@pytest.fixture
def delivery():
    delivery = Delivery(Mock())
    status = 'on_delivery'
    update_time = '2023-07-15 00:00:00'
    mock_status = {"status": status, "update_time": update_time}
    delivery.get_status = Mock(return_value=mock_status)
    return delivery


def test_delivery_status_send(delivery):
    delivery.send_status(item_id="1")
    expected_payload = delivery.make_payload(item_id="1")
    delivery.transport.post.assert_called_with(ENDPOINT, json=expected_payload)
```

`fixture`를 통해 `delivery` 객체의 `get_status` 함수를 호출하면 미리 준비해둔 객체를 반환하도록 하였습니다.  
또한 `Transport` 객체 또한 Mock 객체로 변경해두어 `Transport` 객체 내의 `requests` 모듈에 대한 별 다른 조치를 하지 않아도 되도록 설정해두었습니다.  

이처럼 더 수월한 단위테스트가 가능하도록 리팩토링 하여 더 좋은 코드를 만들어낼 수 있습니다.  