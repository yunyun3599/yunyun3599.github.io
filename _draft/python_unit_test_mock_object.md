# 파이썬 단위 테스트 - Mock 객체  

## 모의 객체와 단위 테스트
### 모의 객체를 사용하는 상황
시스템이 실제로 구동될 때는 데이터베이스, 스토리지 서비스, 외부 API 등 각종 외부 서비스와 연결됩니다.  
외부 서비스를 이용하면 필연적으로 부작용이 발생하며, 이런 부작용을 최소화하기 위해 각종 인터페이스를 통해 추상화하겠지만, 이러한 부분 역시 테스트에 포함되게 됩니다.  
이와 같은 상황에서 모의 객체는 원치 않는 부작용으로부터 테스트 코드를 보호하는 가장 좋은 방법 중 하나입니다.  

### 단위 테스트와 통합테스트의 범위
단위 테스트는 데이터베이스 연결, HTTP 요청을 비롯한 외부 서비스들을 테스트하지 않습니다.  
외부 서비스에 대한 테스트는 작성한 로직에서 벗어날 뿐 아니라 단위 테스트는 빠르게 실행되어야하는데 외부 연결에 대한 대기시간을 기다려야하는 것은 단위 테스트의 취지와 맞지 않습니다.  
그러므로 단위 테스트에서는 외부 서비스 호출에 대한 내부 코드가 정상 동작하는 지 정도만 테스트하며 외부 서비스가 정상 동작하여 올바른 결과를 돌려받는 지에 댁해서는 확인하지 않습니다.  

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
패치를 하면 런타임 중에 참조되는 코드가 바뀌고 원래 코드와의 연결이 끊어지므로 테스트가 조금 더 어려워지는 단점이 있습니다.  
또한 런타임 시 인터프리터에서 객체를 수정하는 오버헤드도 있기 때문에 성능상의 이슈도 발생할 수 있습니다.  
> 참고로 몽키패치 또는 모의를 사용하는 것 자체가 문제가 되는 것은 아니나 몽키패치를 남용하게 된다면 코드 개선의 여지가 있다고 해석할 수 있습니다.  


## 테스트 더블
테스트 더블이란 테스트 스위트에서 실제 코드를 대신해 실제인 것처럼 동작하는 코드입니다.  
테스트 더블을 사용하게 되는 상황은 다음과 같습니다.  
1. 실제 상용 코드가 필요하지 않은 경우
2. 특정 서비스에 접근해야하는데 권한이 없는 경우
3. 부작용이 있기 때문에 단위테스트에서 실행하고 싶지 않은 경우  

테스트 더블의 종류로는 더미(dummy), 스텁(stub), 스파이(spy), 모의(mock) 등이 있습니다. 여기에서 mock은 가장 일ㅇ반적인 형태의 객체로 모든 경우에 적합합니다.  
또한 표준 라이브러리에도 모의 객체가 포맏외어 있기 때문에 `unittest.mock.Mock`을 Import하여 모의 객체를 사용 가능합니다.  

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
이처럼 매직 메서드를 테스트하려는 상황이 아니라면 일반 `Mock`을 사용해도 동작에 문제가 없습니다.  