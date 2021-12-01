---
title: 微信小程序获取formId
date: 2019-11-08 23:39:33
header-img: /images/bg1.png
author: "BerniLin21"
tags:
  - 微信
---

![img](/images/bg3.png)

小程序实现消息推送

<!-- more -->

小程序要实现消息推送就要先获取用户的 formId。

![效果图](/images/formId.jpg)

## 获取 formId

1、必须通过 form 组件提交才能获取到 formId;

2、给 form 组件设置 report-submit="true"属性;

3、给 form 组件添加 bindsubmit 事件绑定，携带 form 中的数据触发 submit 事件，event.detail = {value : {'name': 'value'} , formId: ''};

4、必须用户手动触发提交表单，不能 JS 模拟提交，所以，页面上必须要有提交按钮;

## 代码实现

```javascript
//  wxml
<form report-submit='true' bindsubmit='userSubmit'>
  <button class='button' bindtap='copy'
    form-type='submit'>获取formId</button>
</form>


//  js
userSubmit: function (e) {
  console.log(e.detail.formId);
},

```

## 注意

开发工具里面是没办法查看到真实的 formId 的，会是这样一句提示"the formId is a mock one"，但是在真机运行时是可以看到具体的内容的，所以呢提交给服务端就可以啦
