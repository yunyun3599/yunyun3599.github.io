# 데이터 바인딩
## Axios
Axios는 서버와 데이터를 송수신할 수 있는 HTTP 비동기 통신 라이브러리입니다.  
또한 FE와 BE 개발 시 서로간에 통신을 하기 위해 가장 많이 사용되고 있는 라이브러리이기도 합니다.  

Axios를 통해 리턴되는 값은 Promise 객체 형태입니다.  
Axios는 다양한 브라우저를 지원하고, 응답 시간을 설정할 수 있어 네트워크에 에러가 발생했을 때 정해진 응답 시간으로 초과하면 해당 요청을 종료시킬 수 있다는 장점이 있습니다.  

### Axios에서 제공하는 request
Axios에서는 아래 request methods를 제공합니다.  
- axios.request(config)
- axios.options(url[, config])
- axios.post(url[, data[, config]])
- axios.delete(url[, config])
- axios.put(url[, data[, config]])
- axios.head(url[, config])
- axios.patch(url[, data[, config]])

이러한 다양한 메서드를 통해 서버와 통신 시 목적을 더 분명하게 하고, 요청 유형에 따라 다른 응답을 받을 수 있도록 개발이 가능합니다.  

### Axios 설치
Axios 설치 명령어는 아래와 같습니다.  
```sh
$ npm install axios --save
```

## Mixins 파일 생성
서버 API 호출 로직을 한 곳에서 관리할 수 있도록 Mixin 파일을 생성해보도록 하겠습니다.  
다수의 컴포넌트에서 공통으로 사용되는 API 호출같은 함수를 각각의 컴포넌트에서 구현하면 에러를 수정하거나 변경사항을 반영할 때 고쳐야 할 부분이 많아집니다.  
따라서 한 군데에서 이 모든 것을 관리할 수 있도록 공통 함수를 개발하는 것입니다.  

이번에 작업할 Mixin 파일에는 Mock 서버의 API를 호출하는 함수를 구현하도록 하겠습니다.  
mixins.js 파일은 src/ 하에 위치시켰으며 코드는 아래와 같습니다.  
```js
import axios from 'axios';

export default {
    methods: {
        async $api(url, method, data) {
            return (await axios({
                method: method,
                url,
                data
            }).catch(e => {
                console.log(e);
            })).data;
        }
    }
}
```

작업한 mixins 파일을 Vue 컴포넌트에서 사용하기 위해서는 main.js에 등록해주어야 합니다.  
main.js 파일을 다음과 같이 변경합니다.  
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import mixins from './mixins'

const app = createApp(App)
app.use(router)
app.mixin(mixins)
app.mount('#app')
```

## 서버 데이터 렌더링
### Mock 서버에 api 등록
이제 서버에서 axios를 통해 데이터를 가져온 후 렌더링해보도록 하겠습니다.  
데이터를 가져올 Mock 서버를 생성하는 법은 [이 포스팅](https://yunyun3599.github.io/vue/vue_mock_server/)에서 확인할 수 있습니다.  

리스트 형의 데이터를 리턴하는 `/list` api를 생성해보도록 하겠습니다.  
리턴할 데이터는 다음과 같습니다.  
```json
[
        {"product_name": "기계식 키보드", "price": 25000, "category": "노트북/태블릿", "delivery_price": 5000},
        {"product_name": "무선 마우스", "price": 12000, "category": "노트북/태블릿", "delivery_price": 5000},
        {"product_name": "아이패드", "price": 725000, "category": "노트북/태블릿", "delivery_price": 5000},
        {"product_name": "태블릿 거치대", "price": 32000, "category": "노트북/태블릿", "delivery_price": 5000},
        {"product_name": "무선 충전기", "price": 42000, "category": "노트북/태블릿", "delivery_price": 5000}
]
```
![](/assets/img/2023/01/2023-01-15-server_data_binding/mock_server_api.png)

### api 응답 데이터를 이용한 렌더링
위에서 리턴해주는 데이터를 가지고 화면에 렌더링해주는 코드를 작성해보도록 하겠습니다.  
{% raw %}
```vue
<template>
    <div>
        <table>
            <thead>
                <tr>
                    <th>제품명</th>
                    <th>가격</th>
                    <th>카테고리</th>
                    <th>배송료</th>
                </tr>
            </thead>
            <tbody>
                <tr :key="i" v-for="(product, i) in productList">
                    <td>{{ product.product_name }}</td>
                    <td>{{ product.price }}</td>
                    <td>{{ product.category }}</td>
                    <td>{{ product.delivery_price }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                productList: []
            };
        },
        created() {
            this.getList();
        },
        methods:{
            async getList() {
                this.productList = await this.$api("https://1168783f-6ec2-4497-a0de-75c47cc93718.mock.pstmn.io/list", "get");
            }
        }
    }
</script>
<style scoped>
    table {
        font-family: Arial, sans-serif;
        border-collapse: collapse;
        width: 100%;
    }
    td, th {
        border: 1px solid #dddddd;
        text-align: left;
        padding: 8px;
    }
</style>
```
{% endraw %}

이 코드에서는 vue 라이프사이클 훅에 의해 컴포넌트가 생성된 후에 created 함수가 실행됩니다.  
created 함수에서 methods내의 getList 함수를 호출하여 mock 서버로부터 데이터를 받아와 데이터를 productList 내에 할당합니다.  
그 후에는 v-for를 이용해 데이터들을 보여줍니다.  

![](/assets/img/2023/01/2023-01-15-server_data_binding/vue_use_mock_server_result.png)

가져온 데이터들을 화면에 잘 보여주는 것을 확인할 수 있습니다.  
