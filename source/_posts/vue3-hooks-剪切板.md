---
title: vue3 hooks 剪切板
date: 2022-05-19 10:04:15
header-img: /images/bg1.png
author: "BerniLin21"
tags:
  - Vue
---

<br>

![img](/images/bg1.png)

我之前常用的复制方法是创建个 `input` 获取并选中后使用 `document.execCommand('copy')` 来实现复制，
实现这种情况下需要操作 deom 存在性能隐患等问题，而且该方法将被弃用

![img](/images/useClipboard1.jpeg)

所以建议使用 [ Clipboard API ](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA) 来替代

![img](/images/useClipboard2.jpeg)

以下是使用 vue3 hooks 写法实现是剪切板功能

```js
// useClipboard.ts 剪切板

import { ref, unref, Ref } from 'vue'
import { isNull } from '@/utils/validate/base-type-validate'

type RefString = string | Ref<string>

type TuseClipboard = {
  setClipboard: (v: RefString) => Promise<void> // 复制到剪切板
  cleanListener: () => void // 清理监听
  text: Ref, // 文本
}

type Toptions = {
  canRead?: boolean // 是否支持读取
}

export default (options: Toptions = {}): TuseClipboard => {
  const { canRead = false } = options

  const isSupport = Boolean(navigator && 'clipboard' in navigator) // 是否支持获取剪切板

  const text = ref<string>('') // 文本

  // 复制到剪切板
  async function setClipboard(v: RefString): Promise<void> {
    try {
      if (isNull(v)) {
        throw new Error('无效内容')
      }

      const str = unref(v)
      await navigator?.clipboard.writeText(str)
      text.value = str
    } catch (error) {
      console.log(error)
    }
  }

  // 获取剪切板
  async function getClipboard(): Promise<void> {
    if (!isSupport) return

    try {
      text.value = await navigator.clipboard?.readText()
    } catch (error) {
      console.log(error)
    }
  }

  // 添加监听
  const events = ['copy', 'cut']
  if (canRead && isSupport) {
    for (const e of events) {
      window.addEventListener(e, getClipboard)
    }
  }

  // 清理监听
  function cleanListener(): void {
    for (const e of events) {
      window.removeEventListener(e, getClipboard)
    }
  }

  return {
    setClipboard,
    cleanListener,
    text,
  }
}

```

使用示例：

```html
<template>
  <div>
    <input :value="text" />
    <button @click="setClipboard('123')">复制</button>
  </div>
</template>

<script setup lang="ts">
  import useClipboard from "../hooks/useClipboard";

  const { setClipboard, cleanListener, text } = useClipboard();
</script>
```
