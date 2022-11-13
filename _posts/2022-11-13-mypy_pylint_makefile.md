---
title:  "Mypy, Pylint, 자동 검사 설정을 통한 파이썬 코드 검사"
excerpt: "Mypy, Pylint와 자동 검사 설정을 이용한 파이썬 코드 품질 향상을 위한 방법을 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-11-13
last_modified_at: 2022-11-13
---

# Mypy

### mypy란?
mypy는 정적 타입 검사 도구입니다.
> 정적 타입 검사: 코드를 실행해보지 않고 타입 분석을 진행  
동적 타입 검사: 코드를 실행시키고 런타임 시에 타입 분석을 진행

mypy를 이용하면 모든 파일을 분석하여 타입 불일치를 검사해줍니다.   

### mypy 설치
mypy는 pip를 통해 설치 가능합니다.  
```shell
pip install mypy
```

mypy를 설치 후에 mypy [파일명]을 입력하면 타입 검사 결과를 제공합니다.  

### mypy 사용 예시
mypy.py라는 파일을 아래와 같이 생성해보았습니다.
```python
def check_mypy(int_var: int):
    print(int_var)


float_var = 0.1
check_mypy(float_var)
```
그리고 해당 파일에 mypy를 이용해 타입 검사를 해보겠습니다.
```shell
 mypy mypy.py
```

그럼 아래와 같은 결과를 얻을 수 있습니다.

![](/assets/img/2022-11-13-mypy_pylint_makefile/2022-11-13-mypy_pylint_makefile_1.png)

int 형의 값을 파라미터로 받는 함수에 float 형의 변수를 보냈더니 타입 검사를 통해 문제를 찾아냈습니다.

### mypy에서 특정 부분 스킵하도록 하기
만약 찾아낸 문제가 잘못 탐지된 경우라면 탐지된 소스코드 옆에 주석을 추가하여 mypy가 해당 부분을 스킵하도록 할 수 있습니다.  
```python
def check_mypy(int_var: int):
    print(int_var)


float_var = 0.1
check_mypy(int_var=float_var)  # type: ignore
```  
<br>  

# Pylint 

### pylint란? & pylint 설치
Pylint는 코드의 구조를 검사해주는 도구로, PEP-8을 준수했는지 여부를 검사합니다.

pylint 설치도 pip를 통해 할 수 있습니다.
```shell
pip install pylint
```

### pylint 사용
pylint는 터미널에서 그냥 pylint를 입력하여 실행시킬 수 있습니다.
pylint를 적용해보기 위해 PEP-8을 거의 지키지 않은 코드를 만들어 보았습니다.   
```python
def func(var1,var2):
    a=var1
    b = var2
```
아래 명령어로 해당 파일에 대한 검사를 실행하보겠습니다.  
```python
pylint pylint.py 
```

결과는 아래와 같습니다. 
![](/assets/img/2022-11-13-mypy_pylint_makefile/2022-11-13-mypy_pylint_makefile_2.png)

pylint에서 발견한 위 파일의 문제점은 다음과 같습니다. 
1. 마지막 빈 라인이 없습니다.
2. 모듈에 대한 docstring이 없습니다.
3. 함수 / 메소드에 대한 docstring이 없습니다.
4. 변수 a가 snake_case 네이밍 규칙을 따르지 않았습니다.
4. 변수 a가 snake_case 네이밍 규칙을 따르지 않았습니다.

그.. 사실 pylint가 위에서 \"=\" 앞 뒤에 띄어쓰기 없는 부분 같은 걸 잡을 줄 알았는데 좀 의외입니다.  
어쨌든 알려주는 대로 코드를 고쳐보았습니다.  
```python
"""
pylint 확인을 위한 파일입니다.
"""


def func(var1, var2):
    """
    :param var1:
    :param var2:
    :return:
    """
    a_a = var1
    b_b = var2
    print(a_a + b_b)
```

![](/assets/img/2022-11-13-mypy_pylint_makefile/2022-11-13-mypy_pylint_makefile_3.png)

10점 만점에 10점 코드라는데요? 감사합니다 pylint.

### .pylintrc 파일틀 통해 설정값 조정하기
pylint를 이용해 코드 검사를 할 때 어떠한 기본 설정은 빼고싶을 수도 있고, 또 다른 설정은 추가하고 싶을 수도 있습니다.  
이 때 .pylintrc라는 파일을 통해 설정값을 바꿀 수 있습니다.  
```shell
pylint --generate-rcfile > .pylintrc
```
터미널에 위 명령어를 수행시키면 .pylintrc라는 파일이 생깁니다.  

아무래도 위의 코드에서 snake_case를 지킨다고 의미없이 a_a, b_b를 변수명으로 잡은 것이 걸려서 해당 설정을 .pylintrc에서 빼보도록 하겠습니다. 

```shell
vi .pylintrc
```   
파일을 살펴보면 variable-naming-style을 설정하는 부분이 있는데, 이 값을 snake_case가 아닌 any로 바꿔보도록 하겠습니다.
>\# Naming style matching correct variable names.  
variable-naming-style=any

설정을 바꾼 후 변수명을 다시 a, b 같이 snake_case를 따르지 않는 것으로 하고 pylint를 통한 검사를 다시 진행하면 검사에 통과하는 것을 확인하실 수 있습니다. 

<br>

# Makefile을 통한 자동 검사 설정
리눅스 환경에서 빌드를 자동화하는 가장 일반적인 방법은 makefile을 이용하는 것입니다. 

프로젝트에 Makefile이라는 파일명으로 아래 내용의 파일을 만들어 보았습니다. 
```makefile
typehint:
	mypy mypy.py

test:
	pytest test.py

lint:
	pylint pylint.py

checklist: lint	typehint test

.PHONY: typehint test lint checklist

```
이 파일에 따르면 다음 단계를 통해 코드를 검사합니다. 
1. 코딩 가이드라인 검사 (e.g. PEP-8)
2. 타입 검사
3. 테스트 실행

파일을 작성한 후 다음 명령어를 수행시키면 타겟으로 명시된 것들을 수행합니다.  
```sh
make checklist  
```
여기서 타겟은 typehint, test, lint로 각 타겟별로 mypy, pytest, pylint 검사를 뒤에 적어준 파일에 대해 진행합니다. 
![](/assets/img/2022-11-13-mypy_pylint_makefile/2022-11-13-mypy_pylint_makefile_4.png)

참고로 수행해본 결과 checklist에 명시된 순서대로 수행이 되는데, 앞 단계에서 오류가 나면 뒷 단계는 돌지 않는 것 같습니다.  

또한 .PHONY의 경우 실제 파일명과 target의 이름 충돌을 해결하기 위해 적어주는 부분입니다. 
즉 make 명령 시에 실제 파일과 target의 이름이 동일하면 충돌이 발생하는데, .PHONY에 이를 적어줌으로써 문제를 피할 수 있다고 합니다. 