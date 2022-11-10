---
title:  "깃허브 블로그 포스팅하는 방법"
excerpt: "이 부분이 포스트의 소개로 목록에 보여집니다."

categories:
  - Blog
tags:
  - [Blog, jekyll]

toc: true
toc_sticky: true
 
date: 2022-11-10
last_modified_at: 2022-11-10
---

## 1. _posts 폴더 생성
모든 포스트는 _posts 폴더 하에 저장하게 됩니다.  
따라서 github 블로그 디렉토리 *(깃허브아이디.github.io)* 의 최상단에 _post 디렉토리를 생성하고 그 안에 md파일로 포스트 내용들을 작성하면 됩니다.


## 2. 머릿말 정보
Jekyll을 통해 어떻게 블로그를 포스팅하는 지에 대한 문서입니다.  
포스트 정보에 대한 내용은 다음과 같이 작성합니다.  
```
---
title:  "깃허브 블로그 포스팅하는 방법"
excerpt: "이 부분이 포스트의 소개로 목록에 보여집니다."

categories:
  - Blog
tags:
  - [Blog, jekyll]

toc: true
toc_sticky: true
 
date: 2022-11-10
last_modified_at: 2022-11-10
---
```
 

모든 포스트에 대한 정보는 md 파일 상단에 위와 같은 문서를 작성함으로써 표기합니다.  
1. title은 포스트의 제목이고, excerp는 포스트 목록에서 포스트에 대한 소개로 나타낼 문구를 적어주시면 됩니다.  
2. categories는 포스트의 카테고리이고, tags는 해당 포스트의 tag 목록을 적으시면 됩니다.  
3. toc은 포스트 내의 헤더 정보들만 모아서 목차를 사용할 것인지 여부로, true 값을 주면 목차가 생성됩니다.   
  - toc_sticky를 통해 목차가 스크롤을 따라 움직이게 할 지 여부를 지정할 수 있습니다.  
4. date는 글을 처음 작성한 날짜이고, last_modified_at은 글을 최종 수정한 날짜입니다.


## 3.포스트 내용 작성 정보
포스트 내용은 마크다운(md) 혹은 html로 작성 가능합니다.  
머릿말이 --- 끝나면, 그 밑의 내용은 md나 html 형식으로 작성하면 됩니다.  


## 4. 작성한 내용 확인해보기
터미널에서 블로그 포스트 디렉토리로 이동한 후에 다음 명령어를 치면 로컬에서 지킬 서버를 올릴 수 있습니다.
```shell
bundle exec jekyll serve
```

정상적으로 서버가 구동되면 http://localhost:4000 으로 접속하여 블로그의 모습을 로컬에서 확인해볼 수 있습니다.

## 5. github에 push
변경 내용을 github에 push하면 변경 내용이 {깃허브아이디}.github.io 블로그에도 반영되어 공개됩니다.

<br/>
해당 포스트는 [훌륭한 분의 멋있는 블로그](https://ansohxxn.github.io/blog/jekyll-directory-structure/)를 참고하여 작성되었습니다. 