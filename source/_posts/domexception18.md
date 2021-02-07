---
title: Dom Exception 18
date: 2019-12-08 10:01:16
categories: 遇到的问题
tags: canvas
---

在制作分享图，一般都是通过 canvas 绘制，然后导出 base64 生成分享图的。在导出 base64 的过程因为同源策略通常会遇到图片跨域的问题，会报**Uncaught Error: SECURITY_ERR: DOM Exception 18**的错误，这个问题，也是常见的，有挺多解决方法的。

1. 图片地址采用 base64

```
img.src= "data:image/gif;base64,R0lGODlhCwALAIAAAAAA3pn/ZiH5BAEAAAEALAAAAAALAAsAAAIUhA+hkcuO4lmNVindo7qyrIXiGBYAOw==";
```

如果图片较多，内嵌 base64 会导致文件体积变大，通常利用接口转成 base64，但是是用时间换空间。

2. 给 img 标签设置 **crossorigin** 属性

```
img.crossOrigin = "anonymous";
```

这个需要服务端设置 **Access-Control-Allow-Origin** 响应头，允许浏览器发起跨域请求。

crossOrigin 在 pc 端上有兼容性问题，例如 IE10，而在移动端，我在安卓 4.2.2 以及 4.2.3 遇到设置了仍然报错。

3. 通过 ajax 请求

```
var xhr = new XMLHttpRequest();
xhr.onload = function() {
  if (xhr.status == 200) {
    var uInt8Array = new Uint8Array(xhr.response);
    var i = uInt8Array.length;
    var binaryString = new Array(i);
    while (i--) {
      binaryString[i] = String.fromCharCode(uInt8Array[i]);
    }
    var data = binaryString.join("");
    var base64 = window.btoa(data);
    var dataUrl =
      "data:image/png;base64," + base64;
    ...
  }
};
xhr.open("GET", url, true);  // url为线上图片地址
xhr.responseType = "arraybuffer";
xhr.send();
```

简单的一个请求即可得到相对应的 base64，也可以封装一个 canvas 的资源加载器，利用该方法即可，一些游戏引擎，比如 layabox 就是通过 ajax 来加载资源的。

# End

参考

- [https://www.jianshu.com/p/c3aa975923de](https://www.jianshu.com/p/c3aa975923de)
- [https://www.zhangxinxu.com/wordpress/2018/02/crossorigin-canvas-getimagedata-cors/](https://www.zhangxinxu.com/wordpress/2018/02/crossorigin-canvas-getimagedata-cors/)
