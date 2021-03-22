---
title: Vue Router
date: 2019-04-27 15:39:06
header-img: /images/bg1.png
author: "BerniLin21"
# cdn: 'header-on'
tags:
  - Vue
---


{% blockquote  %}
  Vue / vue Router
{% endblockquote %}


使用 Vue 创建单页应用势必会用到 Vue Router，
此前关于 Vue Router 的知识比较零散，现在得空便来整理 Vue Router。。。

## 简单路由

路由跳转可使用申明式跳转和编程式跳转，

### 申明式

```javascript
  <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
  <router-link to="/foo">Go to Foo</router-link>

  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>

```

### 编程式

```javascript
// 命名的路由
router.push({ name: 'user', params: { userId: '123' }}) // 页面获取：this.$route.params.id


// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }}) // 页面获取：this.$route.query.id


```
`params` 和 `query` 都是路由传值方式，但是如果提供了 `path`，`params` 将会被忽略

ps：其他的编程式跳转
  1)、this.$router.replace()
    不同就是，它不会向 history 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 history 记录。
  2)、this.$router.go()
    这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 window.history.go(n)。


### 动态路由

```javascript
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User },
    ...

```

## 路由嵌套以及路由懒加载的实现

路由嵌套可简单理解为组件下再放组件，并实现路由切换

``` code

/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+

```

异步组件定义为返回一个 `Promise` 的工厂函数 ：

```javascript
const Foo = () => Promise.resolve({ /* 组件定义对象 */ })
```
然后我们可以使用动态 `import` 语法来定义代码分块点：

```javascript
import('./Foo.vue') // 返回 Promise
```

结合这两者，这就是如何定义一个能够被 `Webpack` 自动代码分割的异步组件。
```javascript
const Foo = () => import('./Foo.vue')
```


最后代码如下

```javascript

  {
    path: '/redirect',
    component: Layout,
    hidden: true,
    children: [     //子组件
      {
        path: '/redirect/:path*',
        component: () => import('@/views/redirect/index')   //懒加载
      }
    ]
  }

```

## 命名路由及命名视图

由命名路由组成命名视图

### 命名路由

```javascript
//路由设置
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',             //命名
      component: User
    }
  ]
})

//router-link 跳转
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

//router.push()跳转
this.$router.push({ name: 'user', params: { userId: 123 }})

```

### 嵌套命名视图

业务逻辑：

``` code 
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
```

代码实现：

``` javascript
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>

//路由配置

{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}

```

## 重定向和别名

### 重定向
当用户访问 `/a` 时，`URL` 将会被替换成 `/b`，然后匹配路由为 `/b`

```javascript
  routes: [
    { path: '/a', redirect: '/b' }
  ]
```

### 别名
`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，`URL` 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。

```javascript
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
```


## 路由守卫

有的时候，我们需要通过路由来进行一些操作，比如最常见的登录权限验证，当用户满足条件时，才让其进入导航，否则就取消跳转，并跳到登录页面让其登录。


### 全局守卫

`vue-router` 全局有三个守卫：

1、`router.beforeEach` 全局前置守卫 进入路由之前
2、`router.beforeResolve` 全局解析守卫(2.5.0+) 在 `beforeRouteEnter` 调用之后调用
3、`router.afterEach` 全局后置钩子 进入路由之后


```javascript
    // main.js 入口文件
    import router from './router'; // 引入路由
    router.beforeEach((to, from, next) => {
      next();
    });
    router.beforeResolve((to, from, next) => {
      next();
    });
    router.afterEach((to, from) => {
      console.log('afterEach 全局后置钩子');
    });

```
每个守卫方法接收三个参数：
`to`: 即将要进入的目标 路由对象
`from`: 当前导航正要离开的路由
`next`: 这个参数是个函数，且必须调用，否则不能进入路由(页面空白)。

#### 全局后置钩子的应用


```javascript
    import router from './router'; // 引入路由
    router.afterEach((to, from) => {
      if (未登录 && to.name !== 'login') {
        router.push({ name: 'login' }); // 跳转login
      }
    });
```



```javascript
  next() 进入该路由。
  next(false): 取消进入路由，url地址重置为from路由地址(也就是将要离开的路由地址)。
  next 跳转新路由，当前的导航被中断，重新开始一个新的导航。

```


### 路由独享守卫

路由配置上直接定义 `beforeEnter` 守卫：

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

### 路由组件内的守卫：

1、`beforeRouteEnter` 进入路由前
2、`beforeRouteUpdate` (2.2) 路由复用同一个组件时
3、`beforeRouteLeave` 离开当前路由时

```javascript
  beforeRouteEnter (to, from, next) {
    // 在路由独享守卫后调用 不！能！获取组件实例 `this`，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用 可以访问组件实例 `this`
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用，可以访问组件实例 `this`
  }
```

### 完整的导航解析流程

1、触发进入其他路由。
2、调用要离开路由的组件守卫：`beforeRouteLeave`
3、调用局前置守卫：`beforeEach`
4、在重用的组件里调用：`beforeRouteUpdate`
5、调用路由独享守卫：`beforeEnter`
6、解析异步路由组件
7、在将要进入的路由组件中调用：`beforeRouteEnter`
8、调用全局解析守卫：`beforeResolve`
9、导航被确认
10、调用全局后置钩子的 `afterEach` 钩子。
11、触发DOM更新(`mounted`)。
12、用创建好的实例 `beforeRouteEnter` 守卫中传给 next 的回调函数


## 参考文档
* 官方：  https://router.vuejs.org