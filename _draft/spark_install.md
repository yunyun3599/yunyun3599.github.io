# Pyspark install on mac

# 설치
```sh
$ pip install pyspark
```

아무데서나 `pyspark` 치면 실행 가능해야함  
그런데 나는 안됨

문제 1. SPARK_HOME 변수 설정 안됨
pyspark의 위치를 찾아보겠음
```sh
$ which pyspark
```
mac이고 pip로 install 했는데 `/opt/homebrew/bin/pyspark`가 나옴
`/opt/homebrew/bin/`는 PATH에 잡혀있으므로 아무데서나 pyspark를 실행해보면 문제가 없을 시에 정상 동작해야함.  
근데 나는 아래와 같은 오류가 뜸  
![](/assets/img/2022-11/2022-11-25-spark_install/spark_install_env_setting.png)

### 해결법
zsh를 쓰고 있으므로 ~/.zshrc에 아래 내용 추가
```sh
$ export SPARK_HOME=/opt/homebrew
```

이제 잘 되어야 할 것 같으나 아래와 같은 문제가 발생함  
![](/assets/img/2022-11/2022-11-25-spark_install/spark_install_scala_problem.png)
뭔가 찾아야하는 directory를 제대로 못찾은 것 같음  

뿐만 아니라 pyspark 파일 내용을 보면 더 이상함
![](/assets/img/2022-11/2022-11-25-spark_install/spark_install_pyspark_content.png)
저기서 참고하는 위치라면 앞에서 설정한 `SPARK_HOME`은 `/opt/homebrew` 이므로 `/opt/homebrew/python/pyspark.shell.py` 등의 파일이 있어야하는데 없음.  

따라서 경로 설정이 잘못되었다고 볼 수 있음  

pip로 깔았으므로 site-packages쪽에 가서 라이브러리를 더 뜯어보겠음  
```sh
$ cd /opt/homebrew/lib/python3.8/site-packages/pyspark
```
나는 python도 homebrew로 깔았고, 3.8 버전을 쓰고있어서 저 위치에 pyspark가 깔림.  
만약에 파이썬 버전이 다르면 중간에 `python3.8` 경로를 본인 버전에 맞게 바꾸면 됨  

이 위치에 있는 파일들을 보면 앞에서 pyspark 파일에 언급된 파일들이 다 있는 것을 알 수 있음.  
따라서 SPARK_HOME 경로를 여기로 다시 잡겠음  

~/.zshrc 파일 수정
```sh
vi ~/.zshrc
```
```
export SPARK_HOME="/opt/homebrew/lib/python3.8/site-packages/pyspark"
```
후에 `source ~/.zshrc`로 터미널 재시작 후 `pyspark` 쳐보기  

만약 아래와 같은 오류가 나온다면 
> java.net.BindException: Can't assign requested address: Service 'sparkDriver' failed after 16 retries

다음 파일을 수정해주기
```sh
$ vi $SPARK_HOME/bin/load-spark-env.sh
```

마지막 줄에 아래 내용 추가
```
export SPARK_LOCAL_IP="127.0.0.1"
```

후에 다시 pyspark 실행했더니 나는 드디어 된다.