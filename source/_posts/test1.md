---
title: 记一次canvas不断绘制在安卓低版本下的bug
date: 2019-06-19 16:00:00
---

近期在一个用到 canvas 的项目中，利用 ctx.clearRect()清空画板，但是在安卓低版本（4.2.2）会无法清除，导致重影。

### html

```base
<body>
  <canvas id="canvas"></canvas>
</body>
```

### js

```base
let canvas = document.getElementById('canvas')
let ctx = canvas.getContext("2d");
let x = 0;
let step = 10
animationInterval = setInterval(func, 300);

function func () {
 ctx.clearRect(0, 0, canvas.width, canvas.height);
 ctx.fillRect(x, 0, 10, 150);
 x += step;
}
```
