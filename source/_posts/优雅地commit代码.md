---
title: 优雅地 commit 你的代码
date: 2019-10-04 20:13:16
author: "BerniLin21"
header-img: /images/bg1.png
tags:
  - git
---

![img](/images/bg1.png)

养成良好的 commit 规范，有助于团队开发

<!-- more -->

## 常用的修改项

- feat: 新特性
- fix: 修改问题
- refactor: 代码重构
- docs: 文档修改
- style: 代码格式修改, 注意不是 css 修改
- test: 测试用例修改
- chore: 其他修改, 比如构建流程, 依赖管理.
- scope: commit 影响的范围, 比如: route, component, utils, build...
- subject: commit 的概述
- body: commit 具体修改内容, 可以分为多行
- footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.
- perf 性能优化
- test 测试更新
- build 构建系统或者包依赖更新
- ci CI 配置，脚本文件等更新

## 常用的 emoji

使用完 emoji 就是这亚子啦：
![效果图](/images/git.png)

<table><tr><th>emoji</th><th>emoji 代码</th><th>说明</th></tr><tr><td>art</td><td>```:art:```</td><td>提高代码的结构/格式</td></tr><tr><td>zap</td><td>```:zap:```</td><td>改善性能</td></tr><tr><td>fire</td><td>```:fire:```</td><td>删除代码或文件</td></tr><tr><td>bug</td><td>```:bug:```</td><td>修复一个bug</td></tr><tr><td>sparkles</td><td>```:sparkles:```</td><td>引入新特性</td></tr><tr><td>pencil</td><td>```:pencil:```</td><td>写文档</td></tr></table>

more: https://gitmoji.carloscuesta.me/
