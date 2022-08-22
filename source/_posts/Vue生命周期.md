---
title: Vue 生命周期
date: 2019-04-20 18:30:37
header-img: /images/bg1.png
author: "BerniLin21"
tags:
  - Vue
---

![img](/images/bg1.png)

### vue2

```javascript

<template>
  <div class="hello">
    <h1>{{msg}}</h1>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Vue组件生命周期'
    }
  },
  methods:{

  },
  beforeCreate(){
    console.log('创建前状态,el和data未初始化,在这里加个loding事件')
  },
  created(){
    console.log('创建完毕状态，完成了data初始化,el还未初始化，结束loading，做初始化操作')
  },
  beforeMount(){
    console.log('挂载前状态，完成了el和data的初始化,发起后端请求，拿到数据，配合路由钩子做一些事情')
  },
  mounted(){
    console.log('挂载结束状态,完成了挂载')
  },
  beforeUpdate(){
    console.log('更新前状态')
  },
  updated(){
    console.log('更新完成状态,data里面的值被修改后，将会触发update的操作')
  },
  beforeDestroy(){
    console.log('beforeDestro组件销毁前状态,页面组件推出前执行一些操作')
  },
  destroyed(){
    console.log('组件销毁完成状态')
  }
}
</script>

```

### v2 to v3

| v2            | v3(Options)     | v3(Composition)   | des                                              |
| ------------- | --------------- | ----------------- | ------------------------------------------------ |
| beforeCreate  | beforeCreate    |
| created       | created         |
| beforeMount   | befortMount     | onBeforeMount     |
| mounted       | mounted         | onMounted         |
| beforeUpdate  | beforeUpdate    | onBeforeUpdate    |
| updated       | updated         | onUpdated         |
| beforeDestroy | beforeDestroy   | onBeforeUnmount   |
| destroyed     | destroyed       | onUnMounted       |
| activated     | activated       | onActivated       |
| deactivated   | deactivated     | onDeactivated     |
| errorCaptured | errorCaptured   | onErrorCaptured   | 在捕获了后代组件传递的错误时调用                 |
|               | renderTracked   | onRenderTracked   | dev only，当组件渲染过程中追踪到响应式依赖时调用 |
|               | renderTriggered | onRenderTriggered | dev only，当响应式依赖的变更触发了组件渲染时调用 |
|               | serverPrefetch  | onServerPrefetch  | ssr only，在组件实例在服务器上被渲染之前调用     |

### 生命周期图示

<img src="/images/lifecycle.png" width="688px" height="1088px" style="max-height:none">

<br/>
<br/>

<em> ps: Options 模式下的 `beforeCreate` 和 `created` 的内容可在 Composition 模式下的 `setup` 内调用 <em>
