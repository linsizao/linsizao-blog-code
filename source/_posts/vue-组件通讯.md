---
title: vue 组件通讯
date: 2019-12-15 15:59:06
header-img: /images/bg1.png
tags:
  - Vue
---


{% blockquote  %}
  Vue / vue 组件通讯
{% endblockquote %}


组件式开发作为 Vue 框架的核心思想，那么组件之间的通讯就显得尤为重要了


## 1、`props`

`props` 是最常见的父组件向子组件传值的属性。当父组件该属性变换时，子组件也会自动变换。`props` 可以是数组或者对象，对象允许配置高级选项。使用 kebab-case (短横线分隔命名) 命名

```
// 简单语法
props: ['size', 'myMessage']

// 对象语法
props: {
  property: {
    type: Number, // 传参限定类型 String/Number/Boolean/Array/Object/Date/Function/Symbol
    default: any, // 默认值,对象或数组默认值必须从一个工厂函数获取如 default:()=>[]
    required: Boolean, // 定义是否必填项
    validator: Function, // 自定义验证函数
  }
}

```

## 2、`$emit`

`$emit` 是最常见的子传父的方法，子组件触发父组件给自己绑定的事件，同时附加参数传给监听器回调。

```
// 父组件
<home @call-back="callBackFun">

// 子组件
this.$emit('call-back', data)

```

## 3、`v-model`

`v-model` 用于模板中输入框value值的数据双向绑定，但在组件中 `v-model` 则会等价于:

```
<base-input
  v-bind:value="value"
  v-on:input="value = $event"
/>
```

将其 `value` 特性绑定到一个名叫 `value` 的 `prop` 上 在其 `input` 事件被触发时，将新的值通过自定义的 `input` 事件抛出

```
// 子组件
Vue.component('base-input', {
  props: {
    value: {
      type: [String, Number],
      default: ''
    }
  },
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})

父组件
<base-input v-model="value"></custom-input>

```

## 4、`$refs`

通过给子组件添加 `ref` 属性，然后通过该属性访问子组件实例

```
// 父组件
<base-input ref="input"/>

mounted(){
  console.log(this.$refs.input) // 即可拿到子组件的实例，就可以直接操作 data 和 methods
}

```

## 5、`.sync`

子组件可以修改父组件中的值，举个弹窗的栗子
```
// 子组件
props: {
  show: {
    type: Boolean,
    default: () => false
  },
computed: {
  showDialog: {
    get() {
      return this.showDialog
    },
    set(val) {
      // 通过 this.$emit('update: 父组件传入属性名', 设置的值) 改变父组件指定属性的值
      this.$emit('update:showDialog', val)
    }
  }
}
```


## 6、`$root`

获取当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。

```
mounted(){
  console.log(this.$root) // 获取根实例，最后所有组件都是挂载到根实例上
  console.log(this.$root.$children[0]) // 获取根实例的一级子组件，一般不建议这样使用
}
```

## 7、`v-slot`

子组件内数据可以被父页面拿到，用来代替 `.slot`，`slot-cope` 和 `scope`

```
// 父组件
<todo-list>
 <template v-slot:todo="slotProps" >
   {{slotProps.user.firstName}}
 </template> 
</todo-list> 
//slotProps 可以随意命名
//slotProps 接取的是子组件标签slot上属性数据的集合所有v-bind:user="user"

// 子组件
<slot name="todo" :user="user" :test="test">
    {{ user.lastName }}
 </slot> 
data() {
  return {
    user:{
      lastName:"Zhang",
      firstName:"yue"
    },
    test:[1,2,3,4]
  }
},
// {{ user.lastName }}是默认数据  v-slot:todo 当父页面没有(="slotProps")

```

## 8、`$parent` 和 `$children`

`$parent` 属性指向的父实例

`$children` 属性指向当前实例的直接子组件数组

```
// 父组件
mounted(){
  console.log(this.$children) 
  // 可以拿到 一级子组件的属性和方法
  // 所以就可以直接改变 data,或者调用 methods 方法
}

// 子组件
mounted(){
  console.log(this.$parent) // 可以拿到 parent 的属性和方法
}

```

## 9、`$attrs` 和 `$listeners`

`$attrs` 获取子传父中未在 `props` 定义的值。当一个组件没有声明任何 `prop` 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件

`$listeners` 包含了父作用域中的 (不含 .native 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件

```
// 组件A
Vue.component('A', {
  template: `
    <div>
      <p>this is parent component!</p>
      <B :messagec="messagec" :message="message" v-on:getCData="getCData" v-on:getChildData="getChildData(message)"></B>
    </div>
  `,
  data() {
    return {
      message: 'hello',
      messagec: 'hello c' //传递给c组件的数据
    }
  },
  methods: {
    // 执行B子组件触发的事件
    getChildData(val) {
      console.log(`这是来自B组件的数据：${val}`);
    },
    
    // 执行C子组件触发的事件
    getCData(val) {
      console.log(`这是来自C组件的数据：${val}`);
    }
  }
});

// 组件B
Vue.component('B', {
  template: `
    <div>
      <input type="text" v-model="mymessage" @input="passData(mymessage)"> 
      <!-- C组件中能直接触发 getCData 的原因在于：B组件调用 C组件时，使用 v-on 绑定了 $listeners 属性 -->
      <!-- 通过v-bind 绑定 $attrs 属性，C组件可以直接获取到 A组件中传递下来的 props（除了 B组件中 props声明的） -->
      <C v-bind="$attrs" v-on="$listeners"></C>
    </div>
  `,

   props: {
    message: {
       type: String,
       default: ''
     }
   }
  data(){
    return {
      mymessage: this.message
    }
  },
  methods: {
    passData(val){
      //触发父组件中的事件
      this.$emit('getChildData', val)
    }
  }
});

// 组件C
Vue.component('C', {
  template: `
    <div>
      <input type="text" v-model="$attrs.messagec" @input="passCData($attrs.messagec)">
    </div>
  `,
  methods: {
    passCData(val) {
      // 触发父组件A中的事件
      this.$emit('getCData',val)
    }
  }
});
    
var app=new Vue({
  el:'#app',
  template: `
    <div>
      <A />
    </div>
  `
});

```

## 10、`provide` 和 `inject`

在父组件中通过 `provider` 来提供属性，然后在后代组件中通过 `inject` 来注入变量。不论子组件有多深，只要调用了 `inject` 那么就可以注入在 `provider` 中提供的数据。

```
// 父组件:
provide: { // provide 是一个对象,提供一个属性或方法
  foo: '这是 foo',
  fooMethod:()=>{
    console.log('父组件 fooMethod 被调用')
  }
},

// 后代组件
inject: ['foo','fooMethod'], // 数组或者对象，注入到组件
mounted() {
  this.fooMethod()
  console.log(this.foo)
}
// 在父组件下面所有的后代组件都可以利用 inject

```


## 11、`vue-router` 路由传参

当需要实现跨路由组件的传参时，可以查看[官方](https://router.vuejs.org/zh/guide/essentials/passing-props.html)提供的解决方案，或者是看我之前写的 [Vue Router](https://linsizao.gitee.io/Vue-%E8%B7%AF%E7%94%B1/#%E7%BC%96%E7%A8%8B%E5%BC%8F) 的文章


## 12、`EventBus`

当需要实现兄弟组件间通讯，并且项目规模不大的情况下，我们可以通过使用中央事件总线 `EventBus` 的方式实现
EventBus 通过新建一个 Vue 事件 `bus` 对象，然后通过 `bus.$emit` 触发事件，`bus.$on` 监听触发的事件
如果项目规模是大中型的，那建议使用后面即将介绍的 [Vuex](https://vuex.vuejs.org/) 状态管理

```
// 在 main.js
Vue.prototype.$eventBus=new Vue()

// 传值组件
this.$eventBus.$emit('eventTarget','这是eventTarget传过来的值')

// 接收组件
this.$eventBus.$on("eventTarget",v=>{
  console.log('eventTarget',v);//这是eventTarget传过来的值
})

```

## 13、`Vue.observable`

让一个对象可响应。
Vue 内部会用它来处理 `data` 函数返回的对象。返回的对象可以直接用于渲染函数和计算属性内，并且会在发生改变时触发相应的更新。
可作为最小化的跨组件状态存储器，用于简单的场景

```
// /store/store.js
import Vue from 'vue'

export const store = Vue.observable({ count: 0 })
export const mutations = {
  setCount (count) {
    store.count = count
  }
}

//使用
<template>
  <div>
    <label for="bookNum">数 量</label>
    <button @click="setCount(count+1)">+</button>
    <span>{{count}}</span>
    <button @click="setCount(count-1)">-</button>
  </div>
</template>

<script>
import { store, mutations } from '@/store/store'

export default {
  name: 'Add',
  computed: {
    count () {
      return store.count
    }
  },
  methods: {
    setCount: mutations.setCount
  }
}
</script>

```

## 14、`Vuex` 状态管理

`Vuex` 是状态管理工具，实现了项目状态的集中式管理。详细的关于 `Vuex` 的介绍，请查看[官网文档](https://vuex.vuejs.org/zh/)