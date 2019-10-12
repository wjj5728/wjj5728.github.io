---
title: 记录一些使用Layabox的技巧
date: 2019-09-30 14:11:11
tags: 小游戏
---

LayaBox 作为新生代的 H5 游戏引擎，号称用到了极致的性能（当然白鹭、cocos 也是这么说的），此文就记录一些工作过程中用到 laya 的一些技巧。

## 构建方式

由于 Laya 的编辑是通过 vscode 改造的，当然很多功能也跟 vscode 进行了结合，编译的 F5 按键，开发的时候可能都已经按到手软。是时候解放双手了。

Laya 在立项的时候可以选择 js、ts、as 作为变成语言，身为 f2er，当然集中于 js 和 ts。

![](/images/laya-skill/layaskill-1.png)

上图为 laya 内置的 ts 编译脚本。gulp+browserify+tsify 将 ts 编译成 js，如果是 js 作为变成语言的话，是 gulp+browserify+babel，都是大同小异。

都啥年代了还用 gulp，这是我不想用它的原因之一，webpack 操作起来。

<details>
<summary>webpack的代码，太长了点开查看</summary>

```
const path = require('path');
const webpack = require('webpack');
const notifier = require('node-notifier');
const HtmlWebpackPlugin = require("html-webpack-plugin");
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');
const merge = require('webpack-merge');
const os = require('os')
let localhost = ''
try {
    var network = os.networkInterfaces()
    localhost = network[Object.keys(network)[0]][1].address
} catch (e) {
    localhost = '0.0.0.0'
}
function resolve(dir) {
    return path.join(__dirname, '..', dir)
}
let output = {}
function createNotifierCallback() {
    return (severity, errors) => {
        if (severity !== 'error') return

        const error = errors[0]
        const filename = error.file && error.file.split('!').pop()

        notifier.notify({
            title: '你又写bug了！！',
            message: severity + ': ' + error.name,
            subtitle: filename || '',
            icon: path.join(__dirname, 'logo.png')
        })
    }

};

let baseConfig = { context: path.resolve(**dirname, './'), entry: path.resolve(**dirname, "./src/Main.js"), output: { publicPath: "./", filename: "./bin/js/bundle.js" }, resolve: { extensions: [".json", ".js"] }, devtool: '#source-map', module: { rules: [ { test: /\.js\$/, loader: 'babel-loader', include: [resolve('./src'), resolve('node_modules/webpack-dev-server/client')] } ] }, plugins: [ ] }; let dev = { output: { publicPath: "./", filename: "./js/bundle.js" }, devServer: { hot: false, contentBase: './bin', port: 7070, host: localhost, publicPath: "/", compress: true, quiet: true, open: true, historyApiFallback: true, disableHostCheck: true, proxy: { '/api': { target: `http://${localhost}:4040` } } }, devtool: 'cheap-module-eval-source-map', plugins: [ new HtmlWebpackPlugin({ hash: false, template: "./bin/index.html" }), new FriendlyErrorsPlugin({ compilationSuccessInfo: { messages: [`Your application is running here: http://${localhost}:7070`], }, onErrors: createNotifierCallback() }) ] }

if (process.env.NODE_ENV == 'prod') { output = baseConfig } else { output = merge(baseConfig, dev) } module.exports = output

```

</details>

这里是 js 的，当我想 ts 加上个 ts-loader 也是没问题的，就时候就出现问题了。~~当我们在 Laya 编辑模式下制作 ui，之后导出的时候。ts 版本下有一个 layaMaxUI.ts（js 版本无）的文件会自动删除然后生成，webpack 会找不到此文件报错，找不到解决办法，只能回退到 gulp 版本。~~ **场景的导出模式用文件模式即可**。

<details>

<summary>gulp的代码，太长了点开查看</summary>

```
gulp.task("watchBin", function() {
  return watch(["./bin/*", "./bin/*/*", "./bin/*/*/*"], () => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      reflesh();
    }, 50);
  });
});
/* 刷新 */
function reflesh() {
  gulp
    .src("bin/index.html") //指定被刷新的html路径
    .pipe(connect.reload());
}
/* 编译 */
function compile() {
  browserify({
    basedir: "./",
    //是否开启调试，开启后会生成jsmap，方便调试ts源码，但会影响编译速度
    debug: false,
    entries: ["src/Main.ts"],
    cache: {},
    packageCache: {}
  })
    //使用tsify插件编译ts
    .plugin(tsify)
    .bundle()
    .on("error", function(err) {
      /* 报错信息 */
      console.log(err.message);
      this.emit("end");
    })
    //使用source把输出文件命名为bundle.js
    .pipe(source("bundle.js"))
    //把bundle.js复制到bin/js目录
    .pipe(gulp.dest("./bin/js"))
    .pipe(connect.reload());
}
/* ui文件是否存在 */
function isExists() {
  /* 先创建一个ui 才会开始编译 */
  fs.exists("./src/ui/layaMaxUI.ts", function(exists) {
    if (exists) {
      compile();
      // _.debounce(compile, 200, true)
    } else {
      clearTimeout(timer2);
      timer2 = setTimeout(() => {
        isExists();
      }, 50);
    }
  });
}
gulp.task("watchSrc", function() {
  return watch(["./src/*", "./src/*/*", "./src/*/*/*"], () => {
    isExists();
  });
});
gulp.task("connect", function() {
  connect.server({
    root: ["./bin"],
    host: localhost,
    port: 7070,
    livereload: true,
    middleware: function(connect, opt) {
      return [
        proxy("/api", {
          target: `http://${localhost}:4040`,
          changeOrigin: true
        }),
        (req, res, next) => {
          next();
        }
      ];
    }
  });
});
gulp.task("default", ["connect", "watchBin"]);
```

只是将 **.laya** 文件夹中的 **compile.js** 复制出来，然后用 gulp 建一个服务器，监视文件刷新。

</details>

## 背景的平铺

css 经常会用到 1px 的背景平铺，Laya 里面也可以用。

在场景中创建一个 **FillTexture** 的组件，按照需求设置一下属性配置。

![](/images/laya-skill/layaskill-2.png)

需要平铺的图片

![](/images/laya-skill/jiku-bg-line.png)

效果图：

![](/images/laya-skill/layaskill-3.png)

**这样高度可以自己调控，还可以减少很多消耗。**

## 字体切片

游戏内经常有一个场景就是 用户所赚取的积分是用特殊字体展示的，如果这些字体能用 ttf 来体现，我们还能用 **Text** 组件，然后设置 font 属性来完成这个需求。但是如果字体是设计师自己话的呢？这里就用到了 Laya 里面的一个字体切片，方法简单，速度快。

例如此效果： ![](/images/laya-skill/layaskill-4.png)

素材： ![](/images/laya-skill/sprites.png)

首先创建一个 **FontClip** 组件。

![](/images/laya-skill/layaskill-5.png)

**skin** 就是我们需要用到的图片切片，而 **sheet** 就是相对于切片所代表的数值，相当于映射。然后只要改变 **value** 就可以显示相对应的数值。

其他的数值可以查看 Laya 对 [FontClip 组件的文档说明](http://layaair.ldc.layabox.com/api2/Chinese/laya/ui/FontClip.html)。

有一个需要注意的点是，每个映射的区块大小要一致，否则会出现切割错误。

部分游戏：

[飞鸡大战](http://huodong.4399.cn/game/minigame/game/AircraftWar/index) <br> [逃出恶魔岛](http://huodong.4399.cn/game/minigame/game/DemonIsland/index)<br> [别碰我](http://huodong.4399.cn/game/minigame/game/doNotHitMe/index)

# End
