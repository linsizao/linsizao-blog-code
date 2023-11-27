---
title: TypeScript npm 包
date: 2023-11-27 23:24:06
tags:
  - TypeScript
  - 工程化
---

![img](/images/bg5.png)


## 创建 TypeScript 项目

首先创建基本 Node.js 项目并添加 TypeScript 作为开发依赖项

```bash
npm init -y
npm install typescript --save-dev
```

创建 `src` 文件夹以及 `src` 内的 `index.ts`，作为入口

```bash
mkdir src
touch src/index.ts
```

由于Node.js 和浏览器不理解 TypeScript，因此需要设置 `tsc`（TypeScript 编译器）将 TypeScript 代码编译为 JavaScript。

可以通过运行以下命令将 `tsconfig.json` 文件添加到的项目中

```bash
npx tsc --init
```

在 `tsconfig.json` 配置 根目录和输出目录

```bash
{
	"compilerOptions": {
        // ... Other options
		"rootDir": "./src",
        "outDir": "./dist",
}
```

在 `package.json` 添加打包命令

```json
{
    "scripts": {
        "build": "tsc"
    }
}
```

这时候运行 `npm run build`，将出现一个新的 **`dist`** 文件夹，其中包含已编译的 JavaScript

## 设置 `tsc` 提升开发者体验

`tsconfig.json` 默认的 `target` 配置为`“es2016”`，而现代浏览器仅支持 `“es2015”`

ts 项目使用这个包时无法正确的得到类型提示，因此我们需要把 ts 的申明也打包出来，`declaration` 设置为 `true`，这将与我们的 文件一起生成声明文件 (**`.d.ts`**)

我们可以进一步添加 \*\*[declarationMap](https://www.typescriptlang.org/tsconfig#declarationMap)。\*\*这样，将生成源映射 (.d.ts.map)，以将我们的声明文件 (.d.ts) 映射到我们的原始 TypeScript 源代码 (`.ts` ）。

这意味着代码编辑器可以在使用“转到定义”时转到原始 TypeScript 代码，而不是编译后的 JavaScript 文件。

此时，**[sourceMap](https://www.typescriptlang.org/tsconfig#sourceMap)** 将添加源映射文件 (**`.js.map`**)，允许调试器和其他工具在实际处理发出的 JavaScript 文件时显示原始 TypeScript 源代码。

最终 `tsconfig.json` 文件：

```json
{
    "compilerOptions": {
        "target": "es2015",
        "module": "commonjs",
        "strict": true,
        "esModuleInterop": true,
        "rootDir": "./src",
        "outDir": "./dist",
        "sourceMap": true,
        "declaration": true,
        "declarationMap": true,
    }
}
```

## package.json

这里的事情要简单得多。当用户导入包时，我们需要指定包的入口点。因此，让我们将 `main` 设置为 `dist/index.js`。

除了入口点之外，我们还需要指定主要类型声明文件。在本例中，即为 `dist/index.d.ts`。

`package.json` 包含所有内容：

```json
{
  "name": "ts-package-to-test",
  "version": "1.0.1",
  "description": "",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "BerniLin21",
  "license": "ISC",
  "files": [
    "dist",
    "src"
  ],
  "devDependencies": {
    "typescript": "^5.3.2"
  }
}

```

## 发布到 NPM

1.  确保您的 `package.json` 设置正确
2.  构建项目（如果您遵循指南，则使用 `npm run build`）
3.  如果您还没有这样做，请使用 `npm login` 对 npm 进行身份验证（需要一个 **[npm 帐户](https://www.npmjs.com/signup)）**
4.  运行 `npm publish`

*更新版本前需增加 `package.json` 的 `version` 选项*

*另外可以使用 `release-it` 做自动化版本发布*
