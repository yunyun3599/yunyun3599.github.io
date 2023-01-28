# 부모 컴포넌트와 자식 컴포넌트 간의 이벤트, 함수, 데이터 값 사용

## 부모 -> 자식 컴포넌트 이벤트 트리거
부모 컴포넌트에서 자식 컴포넌트를 import하여 사용할 때는 `ref`로 자식 컴포넌트의 아이디를 지정하여 자식 컴포넌트의 각종 요소에 접근할 수 있습니다.  
```vue
<ChildComponent1 ref="child_component" />
```
위처럼 자식 컴포넌트에 `ref="child_component"` 값을 주어 접근이 필요할 때 이 값을 사용하며, ref는 HTML 태그에서 사용되는 id와 유사하다고 이해할 수 있습니다.  

부모 컴포넌트에서 자식 컴포넌트를 import한 후 자식 컴포넌트의 이벤트를 발생시키는 예시 코드를 살펴보겠습니다.  
아래 코드는 자식 컴포넌트인 ChildComponent1.vue 코드입니다. 
```vue
<template>
    <button type="button" @click="childFunc(this.msg)" ref="btn">child button</button>
</template>
<script>
    export default {
        data() {
            return {
                msg: "Child"
            }
        },
        methods: {
            childFunc(msg) {
                alert(msg + " 컴포넌트에서 발생시킨 이벤트")
                this.msg = "Child"
            }
        }
    }
</script>

```
버튼이 하나 있으며, 해당 버튼이 눌릴 때마다 `childFunc` 함수가 호출되어 alert 창을 띄웁니다.  
파라미터로 받은 msg를 토대로 어느 컴포넌트에서 발생시킨 이벤트인지 문구를 보여줍니다.  

다음은 위의 ChildComponent1을 import하여 사용하는 ParentComponent1 예시 코드입니다.  
```vue
<template>
    <h2>자식 컴포넌트의 이벤트 트리거</h2>
    <button type="button" @click="triggerChildButtonClick" ref="btn">parent button</button>
    <ChildComponent1 ref="child_component" />
</template>
<script>
    import ChildComponent1 from '../components/ChildComponent1.vue';
    export default {
        components: {ChildComponent1},
        methods: {
            triggerChildButtonClick() {
                this.$refs.child_component.msg = "Parent"
                this.$refs.child_component.$refs.btn.click();
            }
        }
    }
</script>
```
부모 컴포넌트에서는 parent button을 가지며 해당 버튼을 눌렀을 때 `triggerChildButtonClick` 함수를 호출합니다.  
이 함수는 ChildComponent1의 `msg` 값을 "Parent"로 바꾸고 ChildComponent1에 있는 버튼 클릭 이벤트를 트리거하는 역할을 수행합니다.  

위의 코드를 잘 보시면 ChildComponent에 `ref=child_component` 라는 값을 주었습니다.  
덕분에 해당 컴포넌트 내부 요소에 `this.$refs.child_component.msg = "Parent"`와 같은 코드를 통해 접근할 수 있음을 확인할 수 있습니다.  

## 부모 -> 자식 컴포넌트 함수 호출
부모 컴포넌트에서 자식 컴포넌트에 정의된 함수를 직접 호출할 수도 있습니다.  
위와 동일하게 적용되는 부분이 많으니 예시 코드를 먼저 살펴보도록 하겠습니다.  

다음은 ChildComponent2.vue 코드입니다.  
```vue
<script>
    export default{
        methods: {
            callFromParent() {
                alert('부모 컴포넌트에서 직접 호출한 함수')
            }
        }
    }
</script>
```
`callFromParent`라는 함수를 자식 컴포넌트에서 정의해두었습니다.  

아래는 ParentComponent2.vue 코드입니다.  
```vue
<template>
    <h2>자식 컴포넌트의 함수 호출</h2>
    <ChildComponent2 ref="child_component" />
    <button type="button" @click="callChildFunc">call child func</button>
</template>
<script>
    import ChildComponent2 from '../components/ChildComponent2.vue';
    export default {
        components: {ChildComponent2},
        methods: {
            callChildFunc() {
                this.$refs.child_component.callFromParent();
            }
        }
    }
</script>
```

많은 부분이 위와 동일합니다.  
부모 컴포넌트에 있는 버튼을 눌렀을 때 부모 컴포넌트에 있는 `callChildFunc`라는 함수가 호출되고, 해당 함수는 자식 컴포넌트에 있는 `callFromParent`를 다시 호출함을 확인할 수 있습니다.  

## 자식 -> 부모 컴포넌트로 이벤트와 데이터 전달 (커스텀 이벤트)
자식 컴포넌트에서 커스텀 이벤트를 생성한 후에 부모 컴포넌트로 전달할 수 있습니다.  
이런 경우에는 `$emit`을 사용해서 이벤트를 전달합니다.  

자식 컴포넌트인 ChildComponent3.vue의 코드는 다음과 같습니다.  
```vue
<template>
    <button type="button" @click="sendMessageToParent">trigger custom event of Parent Component</button>
</template>
<script>
    export default {
        data() {
            return {
                msg: 'message from Child Component'
            };
        },
        methods: {
            sendMessageToParent() {
                this.$emit('send-message', this.msg)    // custom event의 이름은 카멜케이스 X
            }
        }
    }
</script>
```
여기에서 버튼 클릭 시 `sendMessageToParent` 라는 함수를 호출합니다.  
이 함수는 `send-message`라는 이벤트를 부모 컴포넌트로 전달하는 역할을 수행합니다.  
이 이벤트와 함께 두번째 인자로 전하고자 하는 데이터를 함께 전달할 수 있습니다.  

이 커스텀 이벤트가 적용된 부모 컴포넌트의 코드는 아래와 같습니다.  
```vue
<template>
    <div>
        <h2>자식 컴포넌트에서 부모 컴포넌트로 이벤트 / 데이터 전달</h2>
        <ChildComponent3 @send-message="sendMessage"/>
    </div>
</template>
<script>
    import ChildComponent3 from '../components/ChildComponent3.vue';
    export default {
        components: {ChildComponent3},
        methods: {
            sendMessage(data) {
                alert(data);
            }
        }
    }
</script>
```
코드에서 `<ChildComponent3 @send-message="sendMessage"/>` 부분을 통해 커스텀 이벤트가 트리거 되었을 때 sendMessage라는 함수를 호출할 것임을 알 수 있습니다.  

