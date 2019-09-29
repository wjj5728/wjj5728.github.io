---
title: 记一次canvas不断绘制在安卓低版本下的bug
date: 2019-06-19 16:00:00
tags: canvas
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

[线上 demo](https://codepen.io/wjj5728/pen/WNeZRQg)

### 解决方法

这是一个在安卓 4.1-4.2 的 bug，采用以下方法进行 canvas 的重新绘制

```base
ctx.clearRect(0, 0, canvas.width, canvas.height);
canvas.style.display = 'none';// Detach from DOM
canvas.offsetHeight; // Force the detach
canvas.style.display = 'inherit'; // Reattach to DOM
```

---

参考:[https://stackoverflow.com/questions/53832712/android-4-2-stock-browser-canvas-clearrect-sometimes-doesnt-work](https://stackoverflow.com/questions/53832712/android-4-2-stock-browser-canvas-clearrect-sometimes-doesnt-work)

# End
