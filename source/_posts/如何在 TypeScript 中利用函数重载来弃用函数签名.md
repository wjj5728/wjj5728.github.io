---
title: 如何在 TypeScript 中利用函数重载来弃用函数签名
date: 2021-02-13 09:57:03
categories: 转载
tags: typescript
---

在本文中，我们将看到如何利用**[TypeScript 的函数重载](https://www.typescriptlang.org/docs/handbook/functions.html#overloads)**来弃用函数签名。

通常，当您需要弃用某个函数时，可以在该函数上添加@deprecated 来通知后续分开发者，然后创建一个名称可能相似或不同的新函数，并指示开发人员在 JSDoc 消息中使用该新函数和/或显示控制台警告。并最终在该库的将来版本中删除不推荐使用的功能。

如果你不想重新命名这个函数或者不想重新定义一个新的函数，相反，你只需要少量的更改以及更新函数签名以支持其他选项，同时启用旧的 API 用法

为了证明我们如何实现这个想法，我们会使用之前文章[Replacing If Statements With Array.find](https://altrim.io/posts/replacing-if-statements-with-array-find)中的<font color="ForestGreen">\`getColorForStockAmount()\`</font>这个函数，同时尝试添加一些新的变化。最后我们还将看到代码编辑器如何通过可视化的形式来告诉开发者该函数已启用

```ts
// 颜色常量 作为函数的返回类型
enum AlertColor {
  Red = 'red',
  Yellow = 'yellow',
  Green = 'green',
}

// items 的定义类型
type Amount = {
  amount: number;
  color: AlertColor;
};

// 函数现在接受一个参数 并且参数类型为number
const getColorForStockAmount = (stock: number = 0): AlertColor => {
  const items: Amount[] = [
    { amount: 100, color: AlertColor.Red },
    { amount: 500, color: AlertColor.Yellow },
    { amount: Number.MAX_SAFE_INTEGER, color: AlertColor.Green },
  ];

  const item = items.find(item => stock <= item.amount);

  return item?.color;
};
```

从上面的代码我们可以看到，我们有**(stock: number): AlertColor**作为函数的签名

现在让我们作出一些改变，我们想将函数函数的参数由 number 变成一个带有参数的 object，同时我们想向这个 object 加入一些自定义属性。例如：<font color="ForestGreen">\`updatedAt\`</font>

通过上面的变化，我们得到下面这段代码

```ts
enum AlertColor {
  Red = 'red',
  Yellow = 'yellow',
  Green = 'green',
}
type Amount = {
  amount: number;
  color: AlertColor;
  updatedAt: Date; // 新添加updatedAt属性
};

// 定义新选项对象的类型
type Options = {
  amount: number;
  updatedAt?: Date;
};

// 使用重载列表 定义函数类型的接口
interface StockOptions {
  // The new signature
  (options: Options): AlertColor | undefined;

  /** @deprecated
   * Use { amount: number; updatedAt?: string; } object instead
   * */
  (stock: number): AlertColor | undefined;
}

const getColorForStockAmount: StockOptions = (options: number | Options) => {
  const { amount, updatedAt } =
    typeof options === 'number'
      ? { amount: options, updatedAt: undefined }
      : options;

  const items: Amount[] = [
    { amount: 100, color: AlertColor.Red, updatedAt: new Date('2021-01-21') },
    {
      amount: 500,
      color: AlertColor.Yellow,
      updatedAt: new Date('2021-01-21'),
    },
    { amount: 750, color: AlertColor.Green, updatedAt: new Date('2021-01-21') },
  ];

  const item = items.find(item => {
    if (updatedAt) {
      return amount <= item.amount && updatedAt < item.updatedAt;
    }
    return amount <= item.amount;
  });

  return item?.color;
};
```

让我们来分析一下上面的代码，首先我们来看看最重要的部分- <font color="ForestGreen">\`StockOptions\`</font> 接口

```ts
interface StockOptions {
  (options: Options): AlertColor | undefined;

  /** @deprecated
   * Use { amount: number; updatedAt?: string; } object instead
   * */
  (stock: number): AlertColor | undefined;
}
```

我们定义一个拥有两个函数签名的接口，来作为函数重载列表

- 第一个签名 <font color="ForestGreen">\`(options: Options): AlertColor | undefined\`</font> 接受一个 <font color="ForestGreen">\`Options\`</font> 对象并且返回 <font color="ForestGreen">\`AlertColor | undefined\`</font>.
- 第二个签名 <font color="ForestGreen">\`(stock: number): AlertColor | undefined\`</font> 是最初始的签名并接受一个 <font color="ForestGreen">\`number\`</font> 并且同样返回类型 <font color="ForestGreen">\`AlertColor | undefined\`</font>.

请记住，函数重载也可以返回不同的类型。另外，从上面的例子中可以看出，我们将第二个签名标记为<font color="ForestGreen">\`@deprecated\`</font>，以警告开发者并提醒他们可以使用 option 作为代替。在编辑器中使用 JSDoc 中的关键词<font color="ForestGreen">\`@deprecated\`</font>，编辑器会以可视化的形式显示该函数已经弃用，这个等下我们会看到效果。

一旦我们定义了接口，我们就可以像下面这样定义函数类型

```ts
const getColorForStockAmount: StockOptions = (options: number | Options) => {};
```

现在我们已经有了正确的函数类型定义，剩下要做的就是处理这两种情况，这样我们就不会破坏前面的实现。为此，我们首先检查参数类型，然后根据类型定义变量

```ts
const { amount, updatedAt } =
  // 如果参数的类型是number类型 （最初始的签名）
  // 我们将其分配给 amount，并将 updatedAt 保持为未定义
  // 否则我们使用新签名中的对象
  typeof options === 'number'
    ? { amount: options, updatedAt: undefined }
    : options;
```

最后在<font color="ForestGreen">\`items.find\`</font>，我们会检测如果<font color="ForestGreen">\`updatedAt\`</font>已经定义了，我们在条件检查中也使用了这个日期，否则退回到以前的实现（第一个签名），用于向下兼容。

```ts
const item = items.find(item => {
  if (updatedAt) {
    return amount <= item.amount && updatedAt < item.updatedAt;
  }
  return amount <= item.amount;
});
```

最后，在调用具有两个签名的函数时，如果一切就绪，您应该会在编辑器中看到如下内容(来自 VSCode 的屏幕截图)

![](/images/deprecated/deprecated-function-signature.png)

正如你可以从上面的截图中看到的，只有当我们用初始签名调用函数时，我们才会得到一个删除线警告，否则当我们使用选项对象的新签名时，看起来没有问题。如果我们将鼠标悬停在编辑器中已弃用的函数调用上，就会看到弃用消息，就像我们在 docblock 中定义的那样

![](/images/deprecated/deprecated-function-signature-popup.png)

原文：

- [https://altrim.io/posts/deprecating-function-signature-in-typescript](https://altrim.io/posts/deprecating-function-signature-in-typescript)

# End
