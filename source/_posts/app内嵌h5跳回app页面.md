---
title: app内嵌h5跳回app页面
date: 2020-05-24 21:07:56
header-img: /images/bg1.png
author: "BerniLin 21"
tags:
  - 移动web
---

## 判断终端

在 H5 页面中，可以直接利用如下的方法来进行判断是安卓还是 ios

```
// android 终端或者 uc 浏览器
const isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1

// ios 终端
const isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)

```

## 实现跳转

通过调用 app 方法实现跳转

###  android

```
// 通过地址实现
window.location.href = 'ttg://com.hxb.coupon/help_list?id=' + param

// or

// 通过调用方法实现
const json = JSON.stringify({id: param})
window.android[method](json) //  'android' 是定义的变量，'method' 是定义的方法名
// 安卓只能传基本类型
```

### ios

```
// 向 ios 发送消息
const iosParam = {id: param}
window.webkit.messageHandlers[method].postMessage({ body: iosParam })
```


## 完整代码

```
const params = {
  method: '', // 方法名
  iosParam: {}, // ios传值
  androidParam: {} // android传值
}


// 跳转app
jumpApp(obj) {
  const u = navigator.userAgent
  const isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1 // android终端或者uc浏览器
  const isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) // ios终端
  const { method, iosParam, androidParam } = obj
  if (isAndroid) {
    const str = JSON.stringify(androidParam)
    console.log(str)
    window.android[method](str)
  } else if (isiOS) {
    window.webkit.messageHandlers[method].postMessage({ body: iosParam })
  }
}
```

