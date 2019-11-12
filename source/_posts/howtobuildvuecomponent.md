---
title: 利用Vue-cli3构建一个Vue组件库(一)
date: 2019-11-07 13:57:17
tags: Vue
---

市面上有很多优秀的组件库，例如[iView](https://www.iviewui.com/),[ElementUi](https://element.eleme.io)等等，很多大公司因为业务上的重复性。都开始搭建自己的组件库。而搭建组件库的前提是需要搭建一个组件库架子。

主要是利用 vue-cli3 中的 [lib 模式](https://cli.vuejs.org/zh/guide/build-targets.html#%E5%BA%93)。

## 创建一个 Vue 项目

```
vue create demo-ui
```

这样会生成一个默认的目录，这里需要添加一个文件夹存放组件，命名为 packages，同时在添加 index.js 的入口文件

**index.js**

```
// 导入组件
import Com1 from "./component1";
const components = [Com1];
const install = function(Vue) {
  // 遍历注册全局组件
  components.map(component => Vue.component(component.name, component));
};
// 判断是否是直接引入文件
if (typeof window !== "undefined" && window.Vue) {
  install(window.Vue);
}

export default {
  install
};
export { install, Com1 };

```

index 文件为所有组件的入口文件，也是 lib 模式的入口文件，接下来没增加一个组件就在 packages 下添加一个文件夹，

![](/images/howtobuildvuecomponent/2.png)

图中组件文件下的 index 为单一组件的入口文件，src/main.vue 为组件主文件。

**Component1/index.js**

```
import Component1 from "./src/main";

/* istanbul ignore next */
Component1.install = function(Vue) {
  Vue.component(Component1.name, Component1);
};

export default Component1;

```

这就是添加一个组件的流程

![](/images/howtobuildvuecomponent/1.png)

## 打包

代码组织完成后，就可以打包了，在 package.json 中添加运行脚本，利用库模式打包, --target 为打包的模式， --name 为库的名称， --dest 为入口文件。

```
"lib": "vue-cli-service build --target lib --name demo-ui --dest lib package/index.js"
```

## 发布

这一步很简单，先注册一个 npm 账号

```
npm login
然后
npm publish
```

我们可以限制一下发布的文件，在 package.json 中添加 files 字段

```
"files": [
  "lib",
  "package"
],
```

## 如何使用

```
npm i -D demo-ui
```

在入口文件中

```
import DemoUi from "demo-ui";
Vue.use(DemoUi);
```

有点要注意的是，每次 publish， package.json 中的 version 都要比之前大。否则无法发布。

参考：

- [element](https://github.com/ElemeFE/element)

# End
