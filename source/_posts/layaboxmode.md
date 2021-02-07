---
title: layabox场景的四种导出模式
date: 2019-10-09 14:17:13
categories: 小游戏引擎
tags: Layabox
---

用 layabox 开发了几款游戏，对于场景的导出模式一直没有详细了解，翻了翻资料搞起。

以下代码是在 LayaIde2.0.2 ts 版本下进行。

![](/images/layamode/layamode-1.png)

官网已经很详细的写出了四种模式的特点差异。

## **内嵌模式**

内嵌模式会把编辑器的 UI 内容生成一个场景类代码文件，代码脚本里包含 IDE 创建的 UI 场景的信息，在小游戏和轻游戏还没有问世的时候，不用考虑 js 的大小，正常开发 h5 最常用的选择，而且不涉及异步加载打开页面速度也最快。

![](/images/layamode/layamode-2.png)

<font color=red>注意事项：</font><br>1.因为内嵌没有场景文件，所以无法使用 scene.open，只能在开头 new，然后 addChild 到 stage

```base
import {ui} from './ui/layaMaxUI'
// GameConfig.startScene && Laya.Scene.open(GameConfig.startScene);
let view = new ui.test.TestSceneUI();
Laya.stage.addChild(view)
```

2.不能使用 renderType = “instance” 的单例界面机制，目前这个机制是写在 scene.open 里面。

## **加载模式**

加载模式也会生成场景类，其他的 UI 数据信息会放到一个 bin 文件下的 ui.json 内，使用时需要加载这个 json，同样在没有小游戏的时代不常用，场景信息可以不在 js 中，可以节省 js 包体大小，给小游戏 4m 包节省更多空间。使用时可以作为资源加载。

下面是代码是官方的 2D 实例

由于编辑器加载模式下导出的 layaMaxUI.ts 文件中加载场景用到的 loadScene 是用在加载单独的场景 json，所以需要注释掉，用一个新的场景作为中间件将我们从 ui.json 的场景数据设置一下。首先先加载资源然后将 ui 的配置数据存于[Laya.View.uiMap](http://layaair.ldc.layabox.com/api/laya/ui/View.html#uiMap)，然后在需要的地方取出来。

![](/images/layamode/layamode-4.png)

**Main.ts**

```base
主动加载场景资源
Laya.loader.load([{
    url: 'ui.json',
	type: Laya.Loader.JSON
}], Laya.Handler.create(this, () => {
	let res = Laya.Loader.getRes('ui.json')
	Laya.View.uiMap = Laya.Loader.getRes('ui.json')
	let scene = new mid("test/TestScene")
	Laya.stage.addChild(scene)
}))
```

**midUi.ts**

```base
import GameUI from './GameUI' <!-- 如果没有继承的基类，可以继承于Laya.Scene -->
export default class Midui extends GameUI {
    constructor(name) {
        super()
        this.createView(Laya.View.uiMap[name]);
    }
}
```

<font color=red>注意事项：</font><br>整个感觉不是很好用，还需要一个中间层，如果是场景多的情况下，管理起来会比较麻烦，而且不解耦，万一一个场景编译出错，可能会导致一连串的错误，毕竟 Laya 的编辑器感觉信不过，优点是可以减少一些请求还是不错的，减少包的大小。

## **分离模式**

分离模式是在加载模式基础上，同样也会生成场景类，但他会把每个场景生成单独的场景数据文件，每次单独加载场景文件，区别于加载模式一次把所有场景都加载。在 2.0 以后，开发小游戏或轻游戏，为了减少主包大小和提升加载速度都是常用的模式。

![](/images/layamode/layamode-5.png)

生成了场景类

## **文件模式**

文件模式是 2.0 特有的，为了开发小游戏而创建的，他不生成场景类，也就是能进一步减少 js 包的大小，使用的时候用 Scene.load 方式加载，区别于前三种最大的的不同就是，文件模式不能直接调用场景内的变量，需要 getchild 获取之后进行操作。前三种的场景类里声明了变量，有代码提示直接可以操作内部的变量。

![](/images/layamode/layamode-6.png)

其实两者是差不多的，只是文件模式没有生成场景类，这一点导致二者在使用场景中的 UI 组件时会有一点点小小的差异，分离模式可以直接通过 this 对象来找到组件，例如 this.input，而文件模式只能通过 this.getChildByName('xxx')来获取。个人感觉分离模式会对开发友好一点，但是官方推荐的是文件模式。

文件模式在导出的时候不会对 layaMaxUI.ts 文件进行修改，所以用之前的文章有说到利用 webpack 开发的时候，在运行时因为 layaMaxUI.ts 被重新生成导致的错误就可以解决了，算是一种曲线救国。

参考：

- [https://juejin.im/post/5d86dda35188253f4c3a6047](https://juejin.im/post/5d86dda35188253f4c3a6047)
- [https://ldc2.layabox.com/doc/?nav=zh-ts-3-1-12](https://ldc2.layabox.com/doc/?nav=zh-ts-3-1-12)

# End
