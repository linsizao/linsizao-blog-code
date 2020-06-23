---
title: littleVue
subtitle: "Vue 2.x 数据响应式"
date: 2020-03-15 17:18:08
author: "BerniLin21"
header-img: /images/bg1.png
tags:
  - Vue
---

{% blockquote  %}
  Vue / vue 源码
{% endblockquote %}

趁着 Vue 3.0 正式版还没发布来总结一下 Vue 2.x 的核心功能 —— 数据响应式
嘿嘿：

![效果图](/images/littleVue1.gif)

对完整代码有兴趣的话看这里哈 --> https://github.com/linsizao/littleVue


## 响应式原理

在 `new Vue()` 之后。Vue 会调用进行初始化，会初始化生命周期、事件、props、 methods、 data、computed与 watch 等。

vue 会遍历 data 选项的属性，利用 `Object.defineProperty` 为属性添加 `getter` 和 `setter` 对数据的读取进行劫持（`getter` 用来依赖收集，`setter` 用来派发更新），并且在内部追踪依赖，在属性被访问和修改时通知变化。

组件的 watcher 实例，会在组件渲染的过程中记录依赖的所有数据属性（进行依赖收集,还有 computed watcher，user watcher 实例）,之后依赖项被改动时，`setter` 方法会通知依赖与此 data 的 watcher 实例重新计算（派发更新）,从而使它关联的组件重新渲染。


## 实现流程

+ 实现一个数据监听器 Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
+ 实现一个订阅者 Watcher，可以收到属性的变化通知并执行相应的函数，从而更新视图
+ 实现一个指令解析器 Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数

上述流程如图所示：
![效果图](/images/littleVue2.jpg)


## 数据监听器 Observer 

Observer 是一个数据监听器，其实现核心方法就是前文所说的 `Object.defineProperty()` 。如果要对所有属性都进行监听的话，那么可以通过递归方法遍历所有属性值，并对其进行 `Object.defineProperty()` 处理。如下代码，实现了一个 Observer。

```
class Vue {
  ...

  observe(data) {
    if (!data || typeof data !== 'object'){
      return;
    }

    // 遍历该对象
    Object.keys(data).forEach(key => {
      this.defineReactive(data, key, data[key])
    })
  }

  // 数据的相应化
  defineReactive(data, key, val) {

    this.observe(val) // 递归解决数据嵌套

    // 数据劫持
    Object.defineProperty(data, key, {
      get(){
        return val
      },
      set(newVal){
        if(newVal !== val){
          val = newVal
          // console.log(`${key}属性更新为：${val}`)
        }
      }
    })
  }

}
```

监听每个数据的变化后需要通知订阅者，创建一个可以容纳订阅者的消息订阅器 Dep ，订阅器 Dep 主要负责收集订阅者，然后再属性变化的时候执行对应订阅者的更新函数。
所以我们需要一个容器

```
class Vue {
  ...

  observe(data) {
    if (!data || typeof data !== 'object'){
      return;
    }

    // 遍历该对象
    Object.keys(data).forEach(key => {
      this.defineReactive(data, key, data[key])
    })
  }

  // 数据的相应化
  defineReactive(data, key, val) {

    this.observe(val) // 递归解决数据嵌套

    // 初始化 dep
    const dep = new Dep()

    // 数据劫持
    Object.defineProperty(data, key, {
      get(){
        Dep.target && dep.addDep(Dep.target)
        return val
      },
      set(newVal){
        if(newVal !== val){
          val = newVal
          // console.log(`${key}属性更新为：${val}`)
          dep.notify()  // 通知 watcher 去更新
        }
      }
    })
  }
}

class Dep {
  constructor() {
    // 这里存放若干依赖（watcher）
    this.deps = [];
  }

  // 添加 watcher
  addDep(dep) {
    this.deps.push(dep)
  }

  // 通知所有依赖进行更新
  notify() {
    this.deps.forEach(dep => dep.update())
  }
}

```

## 订阅者 Watcher 

Watcher 订阅者作为 Observer 和 Compile 之间通信的桥梁，主要做的事情是:
1、在自身实例化时往属性订阅器 dep里面添加自己
2、自身必须有一个 `update()` 方法
3、待属性变动 `dep.notice()` 通知时，能调用自身的 `update()` 方法，并触发 Compile 中绑定的回调（Compile会提到），则功成身退。

```
class Watcher {
  constructor(vm, key, cb) {
    this.vm = vm
    this.key = key
    this.cb = cb

    // 将当前 watch 实例指定待 Dep 静态属性 target
    Dep.target = this
    this.vm[this.key] // 触发 getter，添加收集依赖
    Dep.target = null //  释放，避免重复添加
  }

  update(){
    // console.log('属性更新了')
    this.cb.call(this.vm, this.vm[this.key])
  }
}


```

可以先模拟一下 Compile 触发 Watcher

```
class Vue {
  constructor (options) {
    this.$options = options

    // 数据响应化
    this.$data = options.data
    this.observe(this.$data)

    // 模拟一下 watch 创建
    // new Watcher();
    // this.$data.test;
    
    ...
```


## 指令解析器 Compile

由于 html 不能识别 vue 的语法，通过 compile 可以解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图。

为了解析模板，首先需要获取到dom元素，然后对含有 dom 元素上含有指令的节点进行处理，因此这个环节需要对dom操作比较频繁，所有可以先建一个 fragment 片段，将需要解析的 dom 节点存入 fragment 片段里再进行处理

```
// 将宿主元素中的代码片段拿出来遍历
node2Fragment(el) {
  const frag = document.createDocumentFragment()
  // 将el中的所有元素搬到frag中
  let child;
  while (child = el.firstChild) {
    frag.appendChild(child)
  }
  return frag
}

```

接下来需要遍历各个节点，对含有相关指定的节点进行特殊处理，以下包含插值文本、双向绑定、指令解析、事件绑定和 html 解析 

```
// compile.js

/**
 * 编译器
 *  @param {*} el 元素
 * @param {*} vm 指定 vue 实例
 */
class Compile {
  constructor(el, vm) {
    // 要遍历的宿主节点
    this.$el = document.querySelector(el);
    this.$vm = vm;

    if (this.$el) {
     // 转换内部内容为片段
    this.$fragment = this.node2Fragment(this.$el)
    //执行编译
    this.compile(this.$fragment)
    // 将编译完的html结果追加至$el
    this.$el.appendChild(this.$fragment)
    }
  }

  // 将宿主元素中的代码片段拿出来遍历
  node2Fragment(el) {
  ...



  // 编译过程
  compile(el) {
    const childNodes = el.childNodes;
    Array.from(childNodes).forEach(node => {
      // 类型判断
      if (this.isElment(node)) { // 元素
        // 查找 “v-”、“@”、“:”
        console.log(node.nodeName, '编译元素' + node.nodeName)
        const nodeAttrs = node.attributes
        Array.from(nodeAttrs).forEach(attr => {
          const attrName = attr.name  // 属性名
          const attrValue = attr.value  // 属性值
          if(this.isDirective(attrName)) {  // 指令v-
            const dir = attrName.substring(2)
            // 执行指令
            this[dir] && this[dir](node, this.$vm, attrValue)
          } else if (this.isEvent(attrName)) {  // 事件@
            const dir = attrName.substring(1)
            this.eventHandle(node, this.$vm, attrValue, dir)
          }
        })
      } else if (this.isInterpolation(node)) { // 插值文本 {{}}
        // console.log(node.nodeName, '编译文本')
        this.compileText(node)
      }
      if ( node.childNodes && node.childNodes.length > 0 ){
        this.compile(node)
      }
    })
  }

  // 编译文本
  compileText(node) {
    //  console.log(RegExp.$1)
    //  node.textContent = this.$vm.$data[RegExp.$1]
    this.update(node, this.$vm, RegExp.$1, 'text')
  }

  /**
  * 更新函数
  * @param {*} node 更新的节点
  * @param {*} vm vue的实例
  * @param {*} exp 表达式
  * @param {*} dir 指令
  */
  update(node, vm, exp, dir) {
    const updaterFun = this[dir + 'Updater']
    // 初始化
    updaterFun && updaterFun(node, vm[exp])
    // 依赖收集
    new Watcher(vm, exp, function (value) {
      updaterFun && updaterFun(node, value)
    })
  }

  // 是否编译元素
  isElment(node) {
    return node.nodeType === 1
  }

  // 插值文本
  isInterpolation(node) {
    return node.nodeType === 3 && /\{\{(.*)\}\}/.test(node.textContent)
  }

  // 指令判断(编译元素判断)
  isDirective(attr) {
    return attr.indexOf('v-') === 0
  }

  // 事件判断(编译元素判断)
  isEvent(attr) {
    return attr.indexOf('@') === 0
  }

  // v-text
  text(node, vm, exp) {
    this.update(node, vm, exp, 'text')
  }

  // v-html
  html(node, vm, exp) {
    this.update(node, vm, exp, 'html')
  }

  // v-model
  model(node, vm, exp) {
    // 指定 input的 value 属性
    this.update(node, vm, exp, 'model')

    // 视图对模型的响应
    node.addEventListener('input', e => {
      vm[exp] = e.target.value
    })
  }

  // v-text 方法
  textUpdater(node, value) {
    node.textContent = value
  }

  // v-html 方法
  htmlUpdater(node, value) {
    node.innerHTML = value
  }

  // v-model 方法
  modelUpdater(node, value) {
    node.value = value
  }

  // 事件处理器
  eventHandle(node, vm, exp, dir) {
    // @click="funName"
    const fun = vm.$options.methods && vm.$options.methods[exp]
    if (dir && fun) { 
      node.addEventListener(dir, fun.bind(vm))
    }
  }
}

```

上面基本完成了 Compile 的功能，然后我们再把 Observer 和 Watcher 整合在一起 ：

```
// vue.js

// new Vue({data: {...}})

class Vue {
  constructor (options) {
    this.$options = options

    // 数据响应化
    this.$data = options.data
    this.observe(this.$data)

    // 模拟一下 watch 创建
    // new Watcher();
    // this.$data.test;
    // new Watcher();
    // this.$data.foo.bar

    new Compile(options.el, this)

    // 执行 created
    if(options.created) {
      options.created.call(this)
    }

  }

  observe(data) {
    if (!data || typeof data !== 'object'){
      return;
    }

    // 遍历该对象
    Object.keys(data).forEach(key => {
      this.defineReactive(data, key, data[key])
      // 代理 data 中的属性到 vue 实例上
      this.proxyData(key)
    })
  }

  // 数据的相应化
  defineReactive(data, key, val) {

    this.observe(val) // 递归解决数据嵌套

    // 初始化 dep
    const dep = new Dep()

    // 数据劫持
    Object.defineProperty(data, key, {
      get(){
        Dep.target && dep.addDep(Dep.target)
        return val
      },
      set(newVal){
        if(newVal !== val){
          val = newVal
          // console.log(`${key}属性更新为：${val}`)
          dep.notify()  // 通知 watcher 去更新
        }
      }
    })
  }

  // 代理data中的属性到vue实例上
  proxyData(key) {
    Object.defineProperty(this, key, {
      get() {
        return this.$data[key]
      },
      set(newVal) {
        this.$data[key] = newVal
      }
    })
  }

}


// Dep：用来管理 Watvher
class Dep {
  constructor() {
    // 这里存放若干依赖（watcher）
    this.deps = [];
  }

  // 添加 watcher
  addDep(dep) {
    this.deps.push(dep)
  }

  // 通知所有依赖进行更新
  notify() {
    this.deps.forEach(dep => dep.update())
  }
}

// Watcher
class Watcher {
  constructor(vm, key, cb) {
    this.vm = vm
    this.key = key
    this.cb = cb

    // 将当前 watch 实例指定待 Dep 静态属性 target
    Dep.target = this
    this.vm[this.key] // 触发 getter，添加收集依赖
    Dep.target = null // 避免重复添加
  }

  update(){
    // console.log('属性更新了')
    this.cb.call(this.vm, this.vm[this.key])
  }
}

```

之后再 html 中 引入 vue.js 和 compile.js 就能实现 vue 几个重要的功能了
嘿嘿 :stuck_out_tongue_winking_eye:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app">

    <!-- 插值绑定 -->
    <p>{{name}}</p>

    <!-- 指令解析 -->
    <p v-text="name"></p>

    <!-- 双向绑定 -->
    <input type="text" v-model="name" />

    <!-- 事件绑定 -->
    <button @click="changeName">修改按钮</button>

    <!-- html 解析 -->
    <div v-html='html'></div>
  </div>


  <script src="compile.js"></script>
  <script src="vue.js"></script>
  <script>
    const app = new Vue({
      el: '#app',
      data: {
        name: 'BerniLin21',
        html: '<a href="#">我是个a标签。。。</a>'
      },

      created() {
        console.log('created')
        setTimeout(() => {
          this.name = '名字变啦'
        }, 1000)
      },

      methods: {
        changeName() {
          this.name = '名字点击变化'
        }
      }
    })
  </script>

</body>
</html>

```