# Vue 컴포넌트 - Slot, Provide/Inject

## Vue 컴포넌트 재사용 - Slot
Slot은 컴포넌트 재사용 측면에서 사용되는 것으로, 대부분의 화면 구성은 유사하나 아주 일부분만 바꿔서 여러 군데에서 사용하고 싶을 때 유용합니다.  
slot을 통해 하위 컴포넌트의 마크업을 재정의하거나 확장할 수 있습니다.  

주로 기본 틀에 해당하는 컴포넌트를 slot을 통해 만들고, 추후 해당 slot을 포함하여 실질적인 컨텐츠에 해당하는 부분만을 작업하는 식으로 작업하게 됩니다.  

예시 코드를 통해 slot의 사용 방법에 대해 더 자세히 알아보도록 하겠습니다.  
아래는 slot으로 사용할 SlotLayout.vue 코드입니다.  
```vue
<template>
    <div class="slot-container">
        <header>
            <slot name="header"></slot>
        </header>
        <main>
            <slot></slot>
        </main>
        <footer>
            <slot name="footer"></slot>
        </footer>
    </div>
</template>
```
Slot에 name을 지정해서 사용하도록 하였습니다.  
이 slot을 활용하여 작업을 할 때는 각 slot에 해당하는 부분만 작업을 하면 되므로, slot에 들어가는 부분을 제외하면 전반적으로 비슷한 디자인을 가지게 됩니다.  

slot을 사용하는 컴포넌트에서는 삽입한 컴포넌트 안에서 다음과 같이 template 태그를 사용해 코드를 작성할 수 있습니다.  
이 때 코드가 들어갈 slot의 위치를 명시하기 위해 `v-slot` 디렉티브를 사용합니다.  
만약 slot에 name이 없다면, `v-slot:default`로 지정해주면 됩니다.  

이제 위의 slot을 이용하는 컴포넌트 코드를 작성해보도록 하겠습니다.  
```vue
<template>
    <slot-layout>
        <template v-slot:header>
            <h1>Title in Header</h1>
        </template>
        <template v-slot:default>
            <p> Content </p>
        </template>
        <template v-slot:footer>
            <p> footer </p>
        </template>
    </slot-layout>
</template>
<script>
    import SlotLayout from '../components/SlotLayout.vue';
    export default {
        components: {SlotLayout}
    }
</script>
```
컴포넌트 안에서 slot 내부에 들어갈 코드를 작성하고 싶으면 <template> 태그를 열고, `v-slot:{원하는 slot의 이름}`을 통해 slot을 지정한 후 내용을 작성합니다.  
참고로, `v-slot:{원하는 slot의 이름}` 형태는 `#{원하는 slot 이름}` 으로 단축어를 이용해 좀 더 간단히 사용할 수도 있습니다.  

위 코드의 결과는 아래 화면처럼 나타납니다.  
![](/assets/img/2023/01/2023-01-21-vue_slot/plain_slot_use.png)

slot 코드를 아래와 같이 바꿔 부모 컴포넌트에서 작업된 코드와 slot을 통해 따로 작업하지 않고 재사용한 부분을 더 명확히 확인해보도록 하겠습니다.  
```vue
<template>
    <div class="slot-container">
        <header style="background: yellow">
            <p>header start</p>
            <slot name="header"></slot>
            <p>header end</p>
        </header>
        <main style="background: skyblue">
            <p>main start</p>
            <slot></slot>
            <p>main end</p>
        </main>
        <footer style="background: pink">
            <p>footer start</p>
            <slot name="footer"></slot>
            <p>footer end</p>
        </footer>
    </div>
</template>
```
slot 내의 header, main, footer 태그에 각각 background color를 주었으며, 태그의 시작과 끝을 나타내는 문구를 추가하였습니다. 
그 결과 아래와 같은 화면이 보입니다.  

![](/assets/img/2023/01/2023-01-21-vue_slot/styled_slot_use.png)
이처럼 특정 디자인을 여러 군데에서 적용하되, 내부에 들어가는 내용을 바꾸고 싶은 경우, slot을 유용하게 사용할 수 있습니다.  

## Provide / Inject
부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하는 경우 props를 사용할 수 있습니다. ([props 포스팅]())
그런데 만약 컴포넌트의 계층 구조가 복잡해지면 어떨까요?  
props만으로 데이터를 전달하기가 어려워집니다.  

이런 경우 Provide/Inject를 사용해 데이터 전달을 더 쉽게 할 수 있습니다.  
데이터를 전달할 부모 컴포넌트에서는 provide를, 자식 컴포넌트에서는 inject를 이용해 데이터를 주고 받는 것입니다.  

ParentComponent - ChildComponent - GrandChildComponent가 있다고 해봅시다.  
PrarentComponent에서 GrandChildComponent로 props를 이용해 데이터를 전달하려면 중간에 ChildComponent에도 데이터가 전달되어야합니다.  
그러나 provide/inject를 사용하면 한 번에 바로 데이터를 전달할 수 있습니다.  

최상단의 컴포넌트로 `ProvideInjectParent`, 이 컴포넌트에서 사용하는 `ProvideInjectChild`, `ProvideInjectChild`컴포넌트에서 사용하는 `ProvideInjectGrandchild`를 생성해보도록 하겠습니다.  
이 때 최상단 컴포넌트의 데이터는 중간 컴포넌트에서는 사용하지 않고, provide, inject를 이용해 `ProvideInjectGrandchild`에서 사용해보도록 하겠습니다.  

`ProvideInjectParent`의 내용은 아래와 같습니다.   
```vue
<template>
    <ProvideInjectChild />
</template>
<script>
    import ProvideInjectChild from '../components/ProvideInjectChild.vue';
    export default {
        components: {ProvideInjectChild},
        data() {
            return {
                items: ['parent_a', 'parent_b']
            };
        },
        provide() {
            return {
                itemLength: this.items.length,
                items: this.items
            };
        }

    }
</script>
```
`provide()` 내에서 제공할 데이터의 이름과 값을 정의하고 있습니다.  

중간 컴포넌트인 `ProvideInjectChild`코드는 아래와 같습니다.  
```vue
<template>
    <div style="border: 1px solid black">
        <p>In Child</p>
        <ProvideInjectGrandchild/>
    </div>
</template>
<script>
    import ProvideInjectGrandchild from './ProvideInjectGrandchild.vue';
    export default {
        components: {ProvideInjectGrandchild},
    }
</script>
```
여기에서는 `ProvideInjectGrandchild` 컴포넌트를 호출할 뿐 따로 데이터를 넘긴다거나 하고있지 않습니다.  

마지막으로 `ProvideInjectGrandchild` 컴포넌트 코드입니다.  
```vue
<template>
    <div style="border: 1px solid black">
        <p>In Grandchild</p>
        <p>injected data length: {{ this.itemLength }}</p>
        <p>injected data: {{ this.items }}</p>
    </div>
</template>
<script>
    export default {
        inject: ['itemLength', 'items']
    }
</script>
```
`inject: ['itemLength', 'items']` 부분을 통해 사용할 데이터를 받아오고 있습니다.  

위에서 작성한 코드를 통해 아래 화면과 같은 결과를 볼 수 있습니다.  
![](/assets/img/2023/01/2023-01-21-vue_slot/provide_inject_result.png)
부모 컴포넌트에서 정의한 데이터가 잘 전달되었음을 확인할 수 있습니다.  
