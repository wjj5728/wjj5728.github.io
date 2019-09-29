---
title: Promise的顺序执行
date: 2019-09-18 19:03:59
tags: promise
---

# 应用场景

项目中，通常要判断各种条件才能执行最终的业务代码，例如判断是否在 app 环境，是否登录，是否需要升级等等。在每个事件进行 if 判断是一个方法，但是代码堆积太多。使用 async/await 来阻止代码的执行时另外一个选择。

```base
function checkapp() {
    return new Promise((resolve,reject) => {
        if ('在app环境') {
            resolve();
        } else {
            reject();
        }
    })
}
function checklogin() {
    return new Promise((resolve,reject) => {
        if ('是否登录') {
            resolve();
        } else {
            reject();
        }
    })
}

let el = document.querySelector('.test')
el.addEventListener('click', async () => {
    await checkapp();
    await checklogin();
    <!-- 业务代码 -->
    ....
})
```

如果需要检测的模块很多，一堆的 await 开起来特别扎眼。Promise 并没有提供方法实现多个 Promise 的顺序执行，需要自己模拟一个。

```base
...

function dd(fn) {
  var sequence = Promise.resolve();
  fn.forEach((item, index) => {
    sequence = sequence.then(item)
  })
  return sequence
}

el.addEventListener('click', async () => {
  await dd([checkapp,checklogin])
  alert('业务')
})
```

将所有的 Promise 改造成一条 Promise 链，这样上一个执行完如果没有 reject 或者跑出错误就会进入下一个 then，一直顺序执行下去，但是现在的实现还有点不完整，可能 Promise 需要传参的，需要将它改造一下。

```
...
function dd(fn,arg) {
  var sequence = Promise.resolve();
  fn.forEach((item, index) => {
    if (arg[index]) {
      if (Array.isArray(arg[index])) {
          sequence = sequence
                  .then(() => {
                    return item.apply(null, arg[index]);
                  })
       } else {
         sequence = sequence.then(item.bind(null, arg[index]))
       }
    } else {
      sequence = sequence.then(item)
    }


  })
  return sequence
}
let el = document.querySelector('.test')
el.addEventListener('click', async () => {
  await dd([a,b],[false,true])
  alert('业务')
})
```

这里踩了一个低级的坑， 在用 apply 以及 call 传参数的时候，忘记 apply、call 是立即执行函数，导致全部 Primise 都执行了，一度怀疑人生。

[完整 demo](https://codepen.io/wjj5728/pen/qBWJaOx)

# 参考：

[https://aotu.io/notes/2016/09/02/Different-Binding/index.html](https://aotu.io/notes/2016/09/02/Different-Binding/index.html) [https://blog.boltdoggy.com/articles/2018/06/07/1528305762.html](https://blog.boltdoggy.com/articles/2018/06/07/1528305762.html)

# End
