---
title:  "Vue Component(2)"
excerpt: "Vue 컴포넌트 내의 각 요소의 사용법을 알아봅니다."

categories:
  - Vue.js
tags:
  - [Vue.js, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2022-11-22
last_modified_at: 2022-11-22
---

# 데이터바인딩  
[이전 포스트](https://yunyun3599.github.io/vue.js/vue_component_1/)에서는 HTML, input(type=text), input(type=number), Textarea를 vue에서 사용할 때 어떤 식으로 databinding을 하게 되는 지 알아보았습니다.  

이 포스트에서는 앞 내용에 이어서 Select, 체크박스, 라디오버튼, 이미지, 클래스바인딩 등에 대해 알아보도록 하겠습니다.  

## Select
Select의 경우 v-model을 이용해 데이터 바인딩을 합니다.  
{% raw %}
```vue
<template>
    <div>
        <select v-model="hexcode">
            <option value="#000000">Black</option > 
            <option value="#008000">Green</option > 
            <option value="#800080">Purple</option>
            <option value="#FF0000">Red</option>
        </select>
        <p>선택된 색상 코드: {{hexcode}}</p>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                hexcode: "#000000"
            }
        }
    }
</script>

```
각 option별로 할당된 value값이 hexcode 데이터의 값이 됩니다.  
![](/assets/img/2022-11/2022-11-22-vue_component_2/2022-11-21-vue_component_2_1.png){: width="80%"}


## Checkbox
체크박스는 input 태그를 사용하기는 하지만 앞에서와 다르게 체크박스의 checked 속성을 이용합니다.  
v-model이 이용하는 값은 체크박스의 value 속성이 아니라 checked 속성을 사용하므로 value 속성에 데이터를 바인딩 하려면 v-model이 아닌 `v-bind:value`를 사용해야합니다.  
```vue
<template>
    <label><input type="checkbox" v-model="color" value="#000000">Black</label>
    <label><input type="checkbox" v-model="color" value="#008000">Green</label>
    <label><input type="checkbox" v-model="color" value="#800080">Purple</label>
    <label><input type="checkbox" v-model="color" value="#FF0000">Red</label>
    <p>Color code is {{color}}</p>
</template>
<script>
    export default {
        data() {
            return {
                color: []
            };
        }
    }
</script>

```
![](/assets/img/2022-11/2022-11-22-vue_component_2/2022-11-21-vue_component_2_2.png){: width="80%"}


## 라디오
라디오도 위의 checkbox처럼 내부 checked 속성과 바인딩됩니다.  
라디오에서는 v-model이 라디오의 value 속성이 아닌 checked 속성을 사용하기 때문에 value 속성에 데이터 바인딩을 하려면 v-model이 아닌 v-bind:value를 사용해야 합니다. 
```vue
<template>
    <div>
        <label><input type="radio" v-bind:value="blackCode" v-model="picked">Black</label>
        <label><input type="radio" v-bind:value="greenCode" v-model="picked">Green</label>
        <label><input type="radio" v-bind:value="purpleCode" v-model="picked">Purple</label>
        <label><input type="radio" v-bind:value="redCode" v-model="picked">Red</label>
        <p>Color code is {{picked}}</p>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                picked: '',
                blackCode: "#000000",
                greenCode: "#008000",
                purpleCode: "#800080",
                redCode: "#FF0000"
            };
        }
    }
</script>
```
![](/assets/img/2022-11/2022-11-22-vue_component_2/2022-11-21-vue_component_2_3.png){: width="80%"}

## Img
img의 경우에는 img의 주소를 객체의 src에 바인딩하는 경우를 살펴보도록 하겠습니다.  
이 때 src 값은 `v-bind:src="데이터키값"` 형태로 바인딩합니다.  
```vue
<template>
    <div>
        <img v-bind:src="imgSrc"/>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                imgSrc: "https://vuejs.org/images/logo.png"
            };
        }
    }
</script>
```
결과로 화면에 vue 로고 이미지가 보이면 됩니다.  

## 클래스 바인딩  
클래스를 바인딩 할 때는 class 속성에 사용할 클래스명을 입력합니다.  
또한 조건에 따라 바인딩할 클래스는 v-bind:class를 이용해 추가적으로 정의할 수 있습니다.  
아래는 라디오버튼을 통해 선택된 색상을 background-color로 갖는 클래스를 바인딩한 코드입니다.  
```vue
<template>
    <div>
        <label><input type="radio" v-bind:value="yellow" v-model="picked">Yellow</label>
        <label><input type="radio" v-bind:value="green" v-model="picked">Green</label>
        <label><input type="radio" v-bind:value="purple" v-model="picked">Purple</label>
    </div>
    <div class="container" 
    v-bind:class="{'yellow': yellow==picked, 'green': green==picked, 'purple': purple==picked}">
        배경색 바꾸기
    </div>
    <br/><br/>
</template>
<script>
    export default {
        data() {
            return {
                picked: '',
                yellow: "yellow",
                green: "green",
                purple: "purple"
            }
        }
    }
</script>
<style scoped>
    container {
        width: 100%;
        height: 50rem;
    }
    .yellow {
        background-color: yellow;
        font-weight: bold;
    }
    .green {
        background-color: green;
        font-weight: bold;
    }
    .purple {
        background-color: purple;
        font-weight: bold;
    }
</style>
```
![](/assets/img/2022-11/2022-11-22-vue_component_2/2022-11-21-vue_component_2_5.png){: width="80%"}

## 인라인 스타일  
인라인 스타일은 데이터를 오브젝트로 선언해서 바인딩할 수 있습니다.  
```vue
<template>
    <div v-bind:style="styleObject">인라인 스타일 바인딩</div>
</template>
<script>
    export default {
        data() {
            return {
                styleObject: {
                    color: 'blue',
                    fontSize: '2rem',
                    fontWeight: 'bold'
                }
            };
        }
    }
</script>
```
위 코드를 적용한 페이지에서는 인라인 스타일이 적용된 파란색 볼드체의 글씨가 보이게 될 것입니다.  
{% endraw %}