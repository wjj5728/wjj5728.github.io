---
title: 如何使用rollup构建一个JS库
date: 2021-02-01 11:13:56
categories: 前端构建
tags: rollup
---

首先抛开 rollup，先说一下一个标准的 js 库所要具备的特性

1. 完善的代码智能提示
2. 同时生成 AMD、CMD、UMD、ES Module、commonjs 等模块化方案
3. 单元测试
4. 还有一些其他编辑器规范、协议等等

目前市面上的构建工具有 webpack、rollup 等，还有一些最近大火的：microbundle、vite（这两款都是基于 rollup）

项目结构
|-- dist
|-- example
|-- src
|-- types
|-- .browserslistrc
|-- .editorconfig
|-- .eslintignore
|-- .eslintrc.js
|-- .gitignore
|-- config.js
|-- package.json
|-- rollup.config.json
|-- tsconfig.json

## rollup.config.js

**input** 就是入口文件路径
**output** 是导出文件的路径，可以是一个对象，也可以是一个数组，因为我们要考虑到打包后的文件的多个模块形式。所以可以根据配置配置多个。
**external** 告诉 rollup 填写的包不进行打包，而是作为外部依赖
**plugins** 是 rollup 使用的一些插件，常用的如下

- @rollup/plugin-node-resolve rollup 无法识别 node_modules 中的包，使用这个插件解决
- @rollup/plugin-node-resolve node_modules 中的包大部分都是 commonjs 格式的，要在 rollup 中使用必须先转为 ES6 语法
  注：之前的插件是 rollup-plugin-node-resolve，目前 rollup 已经将一些插件放到@rollup 命名下面。

```js
import nodeResolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
var config = require('./config.js');
const env = process.env.NODE_ENV == 'production';
module.exports = {
  input: 'src/index.ts',
  output: [
    {
      file: 'dist/index.js',
      format: 'umd',
      name: config.name,
      banner: config.banner,
      /**如果需要去除Object.definePropert 这个ie8不支持 */
      esModule: false,
      plugins: [],
    },
    {
      file: 'dist/index.es.js',
      format: 'es',
      banner: config.banner,
      plugins: [],
    },
  ],
  external: [],
  plugins: [
    nodeResolve({
      main: true,
      extensions: ['.ts', '.js'],
    }),
    commonjs({
      include: 'node_modules/**',
    }),
  ],
};
```

## Typescript 支持

Typescript 作为 Javascript 的超集，在 github 的受欢迎程度今年飙升。
![](/images/rollup/16072619203588.png)

首先我们可通过

```javascript
tsc --init
```

然后选择一些自己配置需要的

```json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "ESNext",
    "noImplicitAny": true,
    "declaration": true, // 生成.d.js的声明文件  用于代码提示
    "declarationDir": "./types" // 声明文件的存放文件夹
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
```

然后安装 ts 插件

```js
import typescript from 'rollup-plugin-typescript2';

plugin: [
  ...,
  typescript({
    useTsconfigDeclarationDir: true, // 生成声明文件的配置使用的是tsconfig中的
  }),
];
```

## 代码压缩

**rollup-plugin-terser** VS **rollup-plugin-uglify**

> Note: uglify-js is able to transpile only es5 syntax. If you want to transpile es6+ syntax use terser instead

**rollup-plugin-uglify**无法更好转义 es6 以上的语法，所以现在基本都是使用的**rollup-plugin-terser**

```js
import { terser } from 'rollup-plugin-terser';

plugins: [
    ...,
    terser()
]
```

## 代码检测 eslint

```js
import { eslint } from 'rollup-plugin-eslint';
plugins: [
    ...,
    eslint({
      throwOnError: true,
      throwOnWarning: true,
      include: ['src/**/*.ts'],
      exclude: ['node_modules/**'],
    }),
]
```

## 注意

如果是要打包静态资源，比如 css，图片等，虽然 rollup 是有插件支持，但是术业有专攻，还是使用 webpack。

# End
