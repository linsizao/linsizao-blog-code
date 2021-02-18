---
title: 微信 JSAPI 支付
date: 2021-02-18 11:26:23
author: "BerniLin21"
header-img: /images/bg1.png
tags: 
  - 微信
---

*微信支付文旦：https://pay.weixin.qq.com/wiki/doc/apiv3/index.shtml*

微信支付在现实使用中越来越广泛，这里提到的 JSAPI 支付适用于微信内 H5 页面。
其余支付方式可见下图：

![img](/images/JSAPIPAY01.png)


## 实现流程
![img](/images/JSAPIPAY02.png)

*下文代码以 jquery 为例，主要实现前端支付流程，个别工具性代码可能不展示*

## 1、用户同意授权，获取 code 和 openId

```

// 微信授权
function getCode() {
  const code = getQueryVariable('code') // 获取 url 里面的 code
  const local = window.location.href
  if (code == null || code === '') {
    window.location.href = `https://open.weixin.qq.com/connect/oauth2/authorize
      ?appid=${appId}'&redirect_uri=${encodeURIComponent(local)}
      &response_type=code&scope=snsapi_userinfo&state=1#wechat_redirect`
  } else {
    getOpenId(code) // 把 code 传给后台获取 openId
  }
}

// 获取 openId
function getOpenId (code) {
  $.ajax({
    type: 'post',
    url, // 后端接口
    data: { code },
    dataType : "json",
    success: function (data) {
      if(data.status === 1) {
        openId = data.data.openId
      } else {
        // ...
      }
    },
  })
}
```

[微信官方文档-授权](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)


## 2、生成预订单

```
// 生成预订单
function createOrderApp () {
  if(!openId) {
    getCode()
  } else {
    const data = {
      ... // 传值
      openId
    }
    $.ajax({
      type: 'post',
      url: ``,
      data,
      dataType: 'json',
      success: function (res) {
        if(res.appid) {
          if (typeof WeixinJSBridge == "undefined") {
            if( document.addEventListener ) {
              document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
            }else if (document.attachEvent) {
              document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
              document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
            }
          }else{
            onBridgeReady(res);
          }
        } else {
          // 提示...
        }
      },
    })
  }
}
```


## 3、调起微信支付

```
// 调起微信支付
function onBridgeReady(params) {
  const { appid, timestamp, noncestr, sign, signType, prepayid } = params
  WeixinJSBridge.invoke( // WeixinJSBridge 只能在微信内打开的网页有效
    'getBrandWCPayRequest', {
      appId: appid, // 公众号名称，由商户传入
      timeStamp, // 时间戳，自1970年以来的秒数
      nonceStr, // 随机串
      package: `prepay_id=${prepayid}`,
      signType, // 微信签名方式
      paySign: sign // 微信签名
    },
    function(res) {
      if(res.err_msg === "get_brand_wcpay_request:ok" ) {
        // 使用以上方式判断前端返回,微信团队郑重提示：
        //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
        // 成功...
      } else if (res.err_msg === 'get_brand_wcpay_request:fail') {
        // 失败...
      }
    }
  )
}
```
