---
title: layabox场景的四种导出模式
date: 2019-10-09 14:17:13
tags: 小游戏
---

用 layabox 开发了几款游戏，对于场景的导出模式一直没有详细了解。

以下代码是在 LayaIde2.0.2 ts 版本下进行。

![](/images/layamode/layamode-1.png)

官网已经很详细的写出了四种模式的特点差异。

**内嵌模式**：内嵌模式会把编辑器的 UI 内容生成一个场景类代码文件，代码脚本里包含 IDE 创建的 UI 场景的信息，在小游戏和轻游戏还没有问世的时候，不用考虑 js 的大小，正常开发 h5 最常用的选择，而且不涉及异步加载打开页面速度也最快。

![](/images/layamode/layamode-2.png)

<font color=red>注意事项：</font><br>1.因为内嵌没有场景文件，所以无法使用 scene.open，只能在开头 new，然后 addChild 到 stage

```base
import {ui} from './ui/layaMaxUI'
// GameConfig.startScene && Laya.Scene.open(GameConfig.startScene);
let view = new ui.test.TestSceneUI();
Laya.stage.addChild(view)
```

2.不能使用 renderType = “instance” 的单例界面机制，目前这个机制是写在 scene.open 里面。

**加载模式**: 加载模式也会生成场景类，其他的 UI 数据信息会放到一个 bin 文件下的 ui.json 内，使用时需要加载这个 json，同样在没有小游戏的时代不常用，场景信息可以不在 js 中，可以节省 js 包体大小，给小游戏 4m 包节省更多空间。使用时可以作为资源加载。

参考： <br> [https://juejin.im/post/5d86dda35188253f4c3a6047](https://juejin.im/post/5d86dda35188253f4c3a6047) <br> [https://ldc2.layabox.com/doc/?nav=zh-ts-3-1-12](https://ldc2.layabox.com/doc/?nav=zh-ts-3-1-12) <br> (https://ask.layabox.com/question/39263)[https://ask.layabox.com/question/39263]

# End
