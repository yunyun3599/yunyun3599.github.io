---
title:  "Mock Server 만들기"
excerpt: "프로젝트에서 사용할 수 있는 Mock Server를 생성해봅니다."

categories:
  - Vue
tags:
  - [Vue, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2023-01-15
last_modified_at: 2023-01-15
---

# Mock 서버 준비하기

## Mock서버란
Mock 서버란 클라이언트로부터 요청을 받으면 응답하는 가짜 서버를 의미합니다.  

프론트엔드 프로젝트를 진행하다 보면 대부분의 데이터를 서버로부터 받아오게 되는데, 백엔드 개발 여부와 상관 없이 프로젝트를 진행시키기 위해 Mock 서버를 사용할 수 있습니다.  

Mock서버는 Postman이라는 툴을 통해 손쉽게 마련할 수 있습니다.  
Postman은 API 개발을 위해 사용되며 주로 개발된 API가 제대로 동작하는 지를 테스트할 때 사용됩니다.  
그러나 위의 기능 외에도 Postman은 많은 기능을 포함하고 있으며, 그 중 하나가 Mock 서버입니다.  

## Postman 설치 및 Mock 서버 생성
포스트맨은 [공식 홈페이지](https://www.postman.com/downloads/)에서 설치 파일을 다운로드하여 설치할 수 있습니다.  

설치가 완료되었다면 다음 화면으로 이동합니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/create_workspace.png)
해당 화면에서 상단 메뉴의 Create Workspace 메뉴를 통해 workspace를 새로 생성합니다.  

![](/assets/img/2023/01/2023-01-15-vue_mock_server/create_workspace2.png){: width="90%"}

생성된 workspace의 좌측 메뉴에서 Mock Servers를 선택합니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/create_mock_server.png){: width="70%"}

다음으로는 Mock Server에서 제공할 API의 path를 설정해줍니다.  
/test 라는 path에 매핑된 api를 하나 생성해보도록 한 후 Next를 눌러줍니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/create_api.png)

그 후의 화면에서 적당한 이름으로 Mock server의 이름을 설정해 준 후 Create Mock Server 버튼을 누르면 Mock 서버가 생성됩니다.  

서버가 생성된 후에는 아래 화면이 보이는데, 여기에 나온 url을 이용해 api를 호출하게 됩니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/mock_server_url.png)

서버 생성이 완료된 후에는 좌측의 Collections 메뉴를 선택한 후에 앞서 만든 test api를 선택합니다.  
그리고 우측 상단의 메뉴에서 Add Example을 선택합니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/create_example.png)

다음으로는 해당 api가 호출되었을 때 리턴해줄 리스폰스를 정의합니다.  
저는 200 status와 함께 아래 내용을 리턴하도록 작성하였습니다.  
```json
{
    "name": "Alice",
    "age": "16",
    "address" "Wonder Land"
}
```
![](/assets/img/2023/01/2023-01-15-vue_mock_server/response_example.png)

변경 사항을 저장한 후 브라우저에서 위의 api를 호출하도록 하겠습니다.  
호출 형태는 Get요청을 `{앞서 확인한 Mock Server Url}/test` 주소로 보내면 됩니다.  
그럼 앞서 작성한 응답이 돌아오게 됩니다.  
![](/assets/img/2023/01/2023-01-15-vue_mock_server/mock_server_result.png)