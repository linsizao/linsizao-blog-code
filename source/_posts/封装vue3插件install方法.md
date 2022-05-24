---
title: 封装 vue3 插件 install 方法
date: 2022-05-24 11:29:44
tags:
  - Vue
---

插件是一种能为 Vue 添加全局功能的工具代码，这里对插件的 install 方法进行封装

<!-- more -->

插件使用: https://staging-cn.vuejs.org/guide/reusability/plugins.html

#### 定义类型

```js
// typescript.ts
import type { AppContext, Plugin } from "vue";

export type SFCWithInstall<T> = T & Plugin;

export type SFCInstallWithContext<T> = SFCWithInstall<T> & {
  _context: AppContext | null,
};
```

#### 封装方法

```js
// install.ts
import type { App } from 'vue'
import type { SFCWithInstall, SFCInstallWithContext } from './typescript'

// 组件
export const withInstall = <T, E extends Record<string, any>>(
  main: T,
  extra?: E
) => {
  // 定义 install 属性
  ;(main as SFCWithInstall<T>).install = (app): void => {
    for (const comp of [main, ...Object.values(extra ?? {})]) {
      app.component(comp.name, comp)
    }
  }

  if (extra) {
    for (const [key, comp] of Object.entries(extra)) {
      ;(main as any)[key] = comp
    }
  }
  return main as SFCWithInstall<T> & E
}

// 函数
export const withInstallFunction = <T>(fn: T, name: string) => {
  ;(fn as SFCWithInstall<T>).install = (app: App) => {
    ;(fn as SFCInstallWithContext<T>)._context = app._context
    app.config.globalProperties[name] = fn
  }

  return fn as SFCInstallWithContext<T>
}
```

#### 调用

```js
// 使用 withInstall
import { withInstall } from "./install.ts";
import Button from "./src/button.vue";

export const MyButton = withInstall(Button);
export default ElButton;
```

```js
// 使用 withInstallFunction
import { withInstallFunction } from "./install.ts";

import fun from "./src/fun";

export const MyFun = withInstallFunction(fun, "$fun");
export default MyFun;
```
