---
title:  "Vue Computed와 Watch"
excerpt: "vue의 computed와 watch에 대해 알아봅니다."

categories:
  - Vue.js
tags:
  - [Vue.js, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2022-12-12
last_modified_at: 2022-12-12
---

# computed와 watch의 사용 목적
computed와 watch는 모두 vue 내에서 정의된 데이터 값에 변경이 일어나는 지를 확인하고, 변경될 때마다 정의된 함수를 실행시키기 위해 사용됩니다.  

## computed와 watch를 사용하지 않았을 때
firstname과 lastname이라는 문자열 데이터를 합하여 fullname을 화면에 보여주는 경우를 생각해봅시다.  
코드 내에서 두 개의 데이터를 합할 수도 있고, 두 데이터를 합한 값을 반환하는 함수를 만들어 사용할 수도 있을 것입니다.  
{% raw %}
```vue
<template>
    <div>Concat variable: {{(firstName + ' ' + lastName)}}</div>
    <div>Concat with function: {{getFullName()}}</div>
</template>
<script>
    export default {
        data() { 
            return {
                firstName: 'Andy',
                lastName: 'Park'
            };
        },
        methods: {
            getFullName() {
                return this.firstName + ' ' + this.lastName
            }
        }
    }
</script>
```
위 코드에서 첫번째 줄은 데이터를 코드 내에서 합하여 화면에 띄웠고, 두번째 줄은 데이터를 합하여 반환하는 함수를 이용하여 fullname을 화면에 띄웠습니다.  
코드 수행 결과 아래 사진과 같은 내용이 화면에 보이게 됩니다.  
![](/assets/img/2022-12/2022-12-12-computed_and_watch/plain_example.png)

위의 두 경우에도 데이터는 정확하게 화면에 보여집니다.  
그러나 화면의 여러 부분에 해당 데이터를 보여줘야 한다면, 데이터를 합치는 연산을 여러번 해야한다는 단점이 있습니다.  

## Computed
Computed는 정의된 데이터 값과 연관된 또 다른 데이터를 정의해서 사용할 수 있도록 해줍니다.  
위와 동일하게 동작하도록 computed를 사용하여 코드를 작성하도록 하겠습니다.  
```vue
<template>
    <div>Concat by Computed: {{fullName}}</div>
</template>
<script>
    export default {
        data() { 
            return {
                firstName: 'Andy',
                lastName: 'Park'
            };
        },
        computed: {
            fullName() {
                return this.firstName + ' ' + this.lastName;
            }
        }
    }
</script>
```
computed 내에 정의된 fullName은 `this.firstName + ' ' + this.lastName` 값을 반환하는 함수인 동시에 데이터의 키값이 됩니다.  

computed로 데이터를 정의하면 함수가 실행되어 지정한 로직에 따라 fullName에 값을 할당하게 됩니다.  
그리고 computed내에 사용되는 값 중 하나라도 변경되면 fullName 함수가 자동으로 실행되고, fullName 데이터의 값이 변경됩니다.  

즉 computed에 정의된 fullName은 함수이자 동시에 vue 인스턴스의 데이터 입니다.  

computed에서 정의된 데이터는 화면 내 여러 곳에서 사용되어도 이에 대한 연산은 한 번밖에 일어나지 않습니다.  

## Watch
Watch도 Computed처럼 데이터의 변경이 일어나는 지 감시하고, 변경이 일어나면 지정된 함수를 실행하여 데이터의 값을 업데이트합니다.  

computed와 watch의 차이점으로는 computed는 기존에 정의된 값을 기반으로 새로운 데이터 값을 생성하기 위해 사용되지만 watch는 watch에 정의된 데이터 값 하나만을 감시하기 위한 용도로 사용됩니다.  

또한 watch는 데이터 변경이 일어나야지만 실행되므로, 초기에 지정된 값에 대해서는 함수가 실행되지 않습니다.  

watch를 사용하는 방식은 아래 코드와 같습니다.  
```vue
<template>
    <div>Concat by Watch: {{fullNameByWatch}}</div>
</template>
<script>
    export default {
        data() { 
            return {
                firstName: 'Andy',
                lastName: 'Park',
                fullNameByWatch: ''
            };
        }
        watch: {
            firstName() {
                this.fullNameByWatch = this.firstName + ' ' + this.lastName;
            },
            lastName() {
                this.fullNameByWatch = this.firstName + ' ' + this.lastName;
            }
        }
    }
</script>
```

위에서 사용한 코드를 모두 종합해보고, firstName과 매핑된 input을 주어 이름을 바꿀 수 있도록 해보았습니다.  
```vue
<template>
    <input type="text" v-model="firstName" />
    <div>Concat variable: {{(firstName + ' ' + lastName)}}</div>
    <div>Concat by function: {{getFullName()}}</div>
    <div>Concat by Computed: {{fullName}}</div>
    <div>Concat by Watch: {{fullNameByWatch}}</div>
</template>
<script>
    export default {
        data() { 
            return {
                firstName: 'Andy',
                lastName: 'Park',
                fullNameByWatch: ''
            };
        },
        methods: {
            getFullName() {
                return this.firstName + ' ' + this.lastName;
            }
        },
        computed: {
            fullName() {
                return this.firstName + ' ' + this.lastName;
            }
        },
        watch: {
            firstName() {
                this.fullNameByWatch = this.firstName + ' ' + this.lastName;
            },
            lastName() {
                this.fullNameByWatch = this.firstName + ' ' + this.lastName;
            }
        }
    }
</script>
```
{% endraw %}  
firstName에 변경이 없을 때 나타나는 화면은 아래와 같습니다.  
![](/assets/img/2022-12/2022-12-12-computed_and_watch/all_before_change.png)

Watch를 통해 만든 데이터에는 아직 값이 생성되지 않았음을 확인할 수 있습니다.  

input창 내의 firstName을 변경해보도록 하겠습니다.  
![](/assets/img/2022-12/2022-12-12-computed_and_watch/all_after_change.png)

watch로 인해 생성되는 데이터까지 값이 잘 들어갔음을 확인할 수 있습니다. 