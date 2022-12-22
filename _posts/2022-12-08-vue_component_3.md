---
title:  "Vue Component(3)"
excerpt: "리스트 랜더링 (v-for), 조건에 따른 랜더링(v-if, v-show), 이벤트에 따른 로직(v-on)에 대해 알아봅니다. "

categories:
  - Vue
tags:
  - [Vue, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2022-12-08
last_modified_at: 2022-12-08
---

# v-for
v-for는 리스트 랜더링 시에 사용되는 구문입니다.  
사용할 때는 `v-for=(item, index) in list` 형식으로 사용됩니다.  
v-for를 이용하면 리스트 내에 있는 원소들 하나하나에 대해 렌더링을 진행합니다.  

힉생 리스트 내에 있는 학생 정보를 하나씩 표에 렌더링하는 코드를 살펴보도록 하겠습니다.  
{% raw %}
```vue
<template>
    <div>
        <table>
            <thead>
                <tr>
                    <th>번호</th>
                    <th>이름</th>
                    <th>나이</th>
                    <th>전공</th>
                </tr>
            </thead>
            <tbody>
                <tr :key="i" v-for="(student, i) in studentList">
                    <td>{{(i+1)}}</td>
                    <td>{{student.name}}</td>
                    <td>{{student.age}}</td>
                    <td>{{student.major}}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                studentList: [
                    {"name": "Tom", age: 21, major: "Computer Science"},
                    {"name": "Emma", age: 22, major: "Medicine"},
                    {"name": "Jorge", age: 20, major: "Business"}
                ]
            }
        }
    }
</script>
```
{% endraw %}  
결과로 다음 화면이 나타납니다.  
![](/assets/img/2022-12/2022-12-08-vue_component_3/v-for.png)

<br>

# v-if, v-else, v-show
## v-if와 v-else
v-if는 특정 변수의 true/false 값에 따라 렌더링 여부를 바꾸고 싶을 때 사용합니다.  
v-if는 `<p v-if="isTrue">참입니다</p>` 의 형식으로 사용할 수 있습니다.  

v-if와 v-else를 같이 쓸 때는 다음과 같이 쓸 수 있습니다.  
```vue
<p v-if="isTrue">참입니다</p>
<p v-else>거짓입니다</p>
```

## v-show
v-show는 v-if와 유사하게 특정 값이 참일 때만 보여주길 원하는 화면이있을 때 사용합니다.  
v-show는 `<p v-show="isTrue">참입니다</p>` 형식으로 사용할 수 있습니다.  

### v-show와 v-if의 차이점
v-show와 v-if는 특정 값의 참 거짓 여부에 따라 컴포넌트를 보여줄 지 말지를 결정하는 로직으로, 언뜻 보면 유사해 보입니다.  

그러나 v-if와 v-show에는 다음과 같은 차이점이 있습니다.  
- v-if는 조건을 만족하는 순간 html 블록을 생성하고 조건을 만족하지 않으면 html을 삭제하게 됩니다.   
- v-show는 조건 만족 여부와상관 없이 무조건 html 블록이 생성됩니다.
    - v-show는 조건 만족 시에는 css의 display를 이용해 화면에 보이도록 하며, 조건을 만족하지 못하면 화면에서 숨기도록 처리됩니다.    

따라서 조건값의 변경이 많이 일어나는 경우에는 v-show를 사용하는 것이 낫고, 조건 값의 변경이 거의 일어나지 않는 경우에는 v-if를 사용하는 것이 낫습니다.  

## v-if, v-else, v-show 예시 코드
아래 코드는 라디오 버튼을 이용해 마케팅 수신 동의 여부를 선택하게 하고, 동의 여부에 따라 다른 문구를 보여주는 예시입니다.  

또한 마케팅 수신에 동의하였을 때는 v-show를 이용해 수신할 매체를 선택하는 컴포넌트도 화면에 보이도록 하였습니다. 
{% raw %}
```vue
<template>
    <div>
        <span>마케팅 수신 동의</span>
        <label><input type="radio" v-bind:value="true" v-model="marketing">agree</label>
        <label><input type="radio" v-bind:value="false" v-model="marketing">disagree</label>
        <h3 v-if="marketing">마케팅 수신 동의에 감사드립니다. </h3>
        <h3 v-else>마케팅 수신에 동의하시면 다양한 혜택을 받으실 수 있습니다. </h3>
        <div v-show="marketing">
            <label><input type="checkbox" v-model="recieve_list" value="e-mail">이메일</label>
            <label><input type="checkbox" v-model="recieve_list" value="message">문자 메세지</label>
            <label><input type="checkbox" v-model="recieve_list" value="kakaotalk">카카오톡</label>
            <p>동의 항목: {{recieve_list}}</p>
        </div>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                marketing: false,
                recieve_list: []
            }
        }
    }
</script>
```
{% endraw %}  
마케팅 수신 동의를 하지 않는 경우 화면은 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-08-vue_component_3/v_if_marketing_disagree.png)

마케팅 수신 동의를 한 경우 화면은 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-08-vue_component_3/v_if_marketing_agree.png)

# v-on
## 클릭 이벤트
클릭 이벤트 발생 시에 특정 로직을 수행하고 싶을 때는 `v-on:click="수행 함수"` 형태 또는 `@click="수행 함수"` 형식으로 코드를 작성합니다.  
이 때 수행할 함수는 `<script>` 태그 내에 `methods` 안에 정의하면 됩니다.  
```vue
<template>
    ...
</template>
<script>
    export default {
        data() {
            return {
                ...
            }
        },
        // 이벤트 발생 시 수행할 함수 정의 위치 
        methods: {
            수행할_함수() {
                ...
            } 
        }
    }
</script>
```

만약 클릭 이벤트가 발생했을 때 여러가지 함수를 수행시키고 싶다면 다음과 같이 작성할 수 있습니다.  
```vue
<button type="button" @click="(func_a(), func_b())">Click</button>
```
<br>
다음과 같은 3개의 클릭 이벤트가 있는 예제를 작성해보도록 하겠습니다.  
- 사용자에게 interval값을 입력받는다.
- Add버튼을 누르면 interval 값만큼 counter를 증가시킨다.  
- Sub버튼을 누르면 interval 값만큼 counter를 감소시킨다.
- reset 버튼을 누르면 counter를 0으로 리셋하고 alert 팝업을 띄운다.  
{% raw %}
```vue
<template>
    <div>
        <div>
            <span>interval 값 세팅: </span>
            <input type="text" v-model.number="interval" />
        </div>
        <button type="button" v-on:click="increaseCounter(interval)">Add</button> &nbsp;
        <button type="button" @click="decreaseCounter(interval)">Sub</button> &nbsp;
        <button type="button" @click="(resetCounter(), alertReset())">Reset</button>
        <p>counter 값: {{counter}}</p>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                counter: 0,
                interval: 1
            };
        },
        methods: {
            increaseCounter(interval) {
                this.counter = this.counter + interval;
            },
            decreaseCounter(interval) {
                this.counter = this.counter - interval;
            },
            resetCounter() {
                this.counter = 0;
            },
            alertReset() {
                alert('Counter가 리셋되었습니다.')
            }
        }
    }
</script>
```
{% endraw %}

아래 사진과 같은 화면을 통해 버튼을 눌러 counter 값을 조정할 수 있게 됩니다.  
![](/assets/img/2022-12/2022-12-08-vue_component_3/on_click_event.png)


## Change 이벤트
Change는 바인딩된 데이터의 값이 바뀔 때 사용됩니다.  
change 를 사용할 때는  
`<select v-model="변경_여부를_체크할_변수" @change="호출할_함수>"`  
형식으로 사용합니다.  
select를 사용하여 선택된 옵션이 바뀔 때마다 alert 팝업을 띄우는 예시 코드를 작성해보겠습니다.  
```vue
<template>
    <div>
        <select v-model="selectedOption" @change="alertChange">
            <option value="First">First</option>
            <option value="Business">Business</option>
            <option value="Economy">Economy</option>
        </select>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                selectedOption: 'First'
            };
        },
        methods: {
            alertChange() {
                alert(this.selectedOption+"가 선택되었습니다.")
            }
        }
    }
</script>
```

## Key 이벤트
key 이벤트는 키보드에서 특정 자판이 입력되었을 때 수행할 로직을 연결할 때 사용됩니다.  
사용되는 형태는 다음과 같습니다.  
`<input @keyup.enter="submit" />`  

위의 경우에는 enter키가 눌렸을 때와 매핑된 경우이고, 이 외에도 자주 사용되는 키로는 아래 목록이 있습니다.  
- enter
- tab
- delete
- esc
- space
- down
- up
- left
- right