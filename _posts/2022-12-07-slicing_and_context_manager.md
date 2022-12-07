---
title:  "파이썬의 인덱싱, 슬라이싱, 컨텍스트 관리자"
excerpt: "파이썬의 인덱스와 슬라이싱, 컨텍스트 관리자에 대해 개념과 예제를 통해 알아봅니다."

categories:
  - Python
tags:
  - [Python, CleanCode]

toc: true
toc_sticky: true
 
date: 2022-12-07
last_modified_at: 2022-12-07
---

# 인덱스와 슬라이스
## 인덱스
파이썬의 리스트에 대해서 인덱싱과 슬라이싱이 가능합니다.  
인덱싱은 원하는 위치의 요소를 가져오는 방식입니다.  
파이썬의 인덱싱의 특이한 점은 음수 인덱스를 통해 배열의 뒤에서부터 접근이 가능하다는 것입니다.  

```python
sample_list = [1, 2, 3]
sample_list[0]      # 1
sample_list[-1]     # 3
```

## 슬라이스
파이썬에서는 슬라이싱을 통해 원하는 구간의 요소들을 가져올 수 있습니다.  

이 때 알아야 할 점은 슬라이싱의 결과는 기존 객체와 같은 타입의 인스턴스라는 점입니다.  
이것이 무슨 뜻이냐면, 리스트에 대해 슬라이싱을 적용하면 슬라이싱의 결과 역시 리스트일 것이고, 튜플에 대해 적용하면 결과가 튜플로 도출되어야한다는 뜻입니다.

슬라이싱은 `list[a:b:c]` 형태로 할 수 있는데, 
- a는 가져올 시작 인덱스 (생략하면 0번부터)
- b는 마지막 인덱스 (생략하면 마지막 요소까지, 마지막 인덱스로 명시된 부분의 바로 앞 요소까지 가져옴)
- c는 interval (생략하면 interval은 1)

을 의미합니다.  
 
```python
sample_list = [1, 2, 3, 4, 5]
sample_list[0:3]        # [1, 2, 3] 
sample_list[0:5:2]      # [1, 3, 5] 
sample_list[:-2]        # [1, 2, 3] 
```

참고로 위의 경우 `sample_list[a:b:c]` 형태로 간격을 전달할 때 실제로는 슬라이스 객체를 전달하는 것과 같습니다.  
이게 무슨 뜻인지 아래 코드를 확인해보도록 하겠습니다.  
```python
sample_list = list(range(1, 11))    # [1, 2, 3, ..., 10]

if sample_list[1:10:2] == sample_list[slice(1, 10, 2)]:
    print("Result is the same")
else:
    print("Result is not the same")
```
위의 경우 `sample_list[slice(1, 10, 2)]`와 `sample_list[1:10:2]`는 같으므로 조건절에서는 `True`를 결과로 반환할 것입니다.  

<br>

# 자체 시퀀스 생성
앞에서 인덱싱을 이용해 `list[key]` 형식으로 리스트의 특정 요소에 접근할 수 있음을 알게 되었습니다.  
이를 가능하게 하는 것은 파이썬의 \_\_get\_item\_\_이라고 하는 매직 메소드 때문입니다.  

시퀀스 객체는 이 \_\_get\_item\_\_ 매직 메서드와 \_\_len\_\_ 매직 메서드를 모두 구현한 객체입니다.  

시퀀스 객체에 대해서는 슬라이싱과 인덱싱이 가능합니다.  
\_\_get\_item\_\_와 \_\_len\_\_ 를 모두 구현한 객체를 생성하고 동작을 확인해보도록 하겠습니다.  


```python
class Classroom:
    def __init__(self, student_list: list):
        self._student_names = student_list

    def __len__(self):
        return len(self._student_names)

    def __getitem__(self, student_number):
        return self._student_names.__getitem__(student_number)


if __name__ == "__main__":
    classroom = Classroom(["Kang", "Kim", "Park", "Song", "Lee", "Jeon", "Choi", "Hwang"])
    print(len(classroom))   # 8
    print(classroom[0])     # Kang
    print(classroom[6:])    # ['Choi', 'Hwang']
```

인덱싱과 슬라이싱이 모두 잘 동작하는 것을 확인할 수 있습니다.  

<br>

# 컨텍스트 관리자
## 컨텍스트 관리자란
컨텍스트 관리자는 특정 로직 앞 뒤로 처리해야 하는 로직이 있을 때 유용합니다.  

예를 들면 파일을 읽어 무언가를 처리해야 하는 로직이 있을 때 파일이 제대로 열렸는지, 오류가 나진 않았는 지를 메인 로직에서 처리하는 것은 적합하지 않습니다.  
파이썬에서 파일을 여닫을 때는 `with open(file_name) as f:` 구문을 많이 쓰는데요, 이 구문이 가장 대표적인 컨텍스트 관리자의 예입니다.  

컨텍스트 관리자는 메인 로직 이전에 실행되는 <span style="background-color: yellow;">\_\_enter\_\_</span> 매직 메서드와 메인 로직 종료 후에 수행되는 <span style="background-color: yellow;">\_\_exit\_\_</span> 매직 메서드를 갖습니다.  

위의 `with open(file_name) as f:` 를 가지고 설명을 해보자면, \_\_enter\_\_는 `as f:` 의 f 변수에 들어갈 값을 반환합니다.  
\_\_exit\_\_는 로직이 정상종료되든 예외가 발생하든 호출되며 예외 사항을 파라미터로 받습니다.  
```python
# __exit__ 매직 메서드의 형태
def __exit__(self, exc_type, exc_value, exc_traceback):
```

## 컨텍스트 관리자 생성 예제
예제로 컨텍스트 관리자를 하나 생성해보도록 하겠습니다.  
만들 컨텍스트 관리자는 파일을 저장하기 이전에 파일 시스템이 잔여 용량이 충분한지 확인하는 역할을 합니다.
-  FileDownloader: 컨텍스트 관리자. 파일 다운로드 이전에 용량이 충분한지 확인
- FileSystem: 파일 시스템. 파일이 다운로드되면 총 용량이 줄어듦

**사용 함수**
```python
def check_remain_capacity(file_system, file_size):
    if file_system.total_size >= file_size:
        file_system.decrease_total_size(file_size)
        print("##### DOWNLOAD success #####")
        print(f"downloaded file size: {file_size}")
        print(f"remain file system: {file_system.total_size}")
        return True
    return False


def print_error_log(file_system, file_size):
    print("##### DOWNLOAD failed #####")
    print("lack of file system capacity")
    print(f"remain file system: {file_system.total_size}, download file size: {file_size}")
```
- `check_remain_capacity`: 파일 시스템에 용량이 층분한지 확인하는 함수. 용량이 충분하면 True 반환, 부족하면 False 반환
- `print_error_log`: 파일 시스템에 용량이 부족할 때 에러 로그를 프린트하는 함수

**FileSystem 클래스**
```python
class FileSystem:
    def __init__(self, total_size: int):
        self.total_size = total_size

    def decrease_total_size(self, size: int):
        self.total_size = self.total_size - size
```
- total_size를 가지고 있으며 새로운 파일이 다운로드 되면 total_size를 파일의 크기만큼 줄입니다.  

**FileDownloader 클래스**
```python
class FileDownloader:
    def __init__(self, file_system: FileSystem, file_size: int):
        self.file_system = file_system
        self.file_size = file_size

    def __enter__(self):
        return check_remain_capacity(self.file_system, self.file_size)

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            print_error_log(self.file_system, self.file_size)
        print("##########end#############\n")
```
- 초기화 시에 사용할 파일 시스템과 다운로드 받을 파일 용량을 받습니다.  
- \_\_enter\_\_ 메서드에서 용량이 충분한지 확인하고 충분하면 True, 부족하면 False를 반환합니다.  
- \_\_exit\_\_ 메서드에서는 에러가 발생한 경우 로그를 남깁니다  

**LackOfCapacityException 예외 클래스**
```python
class LackOfCapacityException(Exception):
    pass
```
- 파일 시스템 용량이 부족할 때 발생하는 예외입니다.  

**메인 코드**
```python
if __name__ == "__main__":
    fs = FileSystem(100)
    download_list = [10, 20, 50, 30]

    for file in download_list:
        with FileDownloader(fs, file) as success:
            if success:
                print("Do something with downloaded file")
            else:
                raise LackOfCapacityException

```
- 다운로드 받을 파일의 용량 리스트를 가지고 있습니다.  
- 사용할 파일시스템의 총 용량은 100으로 설정하여 초기화합니다.  
- 파일 하나하나마다 컨텍스트 매니저를 이용하여 파일을 다운로드 받습니다.  
- 파일 다운로드에 성공했을 때는 파일을 이용해서 할 일을 하고 파일 다운로드에 실패하면 LackOfCapacityException을 발생시킵니다.  

위의 코드를 실행하면 `download_list = [10, 20, 50, 30]` 내의 원소들 중 30에 해당하는 원소를 실행할 때 예외가 발생하는 것을 확인할 수 있습니다.  


## contextlib 모듈을 이용한 컨텍스트 관리자 생성 예제
컨텍스트 관리자 객체를 굳이 생성할 필요가 없을 때 contextlib에서 제공하는 데코레이터를 이용해 간편하게 컨텍스트 관리자를 생성할 수 있습니다.  

위의 예제와 동일하게 작동하는 컨텍스트 관리자를 contextlib를 이용해서 생성한 코드는 다음과 같습니다.  
```python
import contextlib


@contextlib.contextmanager
def file_downloader(file_system, file_size):
    result = check_remain_capacity(file_system, file_size)
    try:
        yield result
    except LackOfCapacityException:
        print_error_log(file_system, file_size)
    print("##########end#############\n")
```
yield 문을 기준으로 yield문 앞 부분이 \_\_enter\_\_ 로직, 뒷 부분이 \_\_exit\_\_로직으로 작동하게 됩니다.  

이렇게 생성한 컨텍스트 관리자를 이용하는 코드는 아래와 같습니다.  
```python
if __name__ == "__main__":
    fs = FileSystem(100)
    download_list = [10, 20, 50, 30]

    for file in download_list:
        with file_downloader(fs, file) as success:
            if success:
                print("Do something with downloaded file")
            else:
                raise LackOfCapacityException

```

코드 수행의 결과는 앞의 예제와 동일합니다.  