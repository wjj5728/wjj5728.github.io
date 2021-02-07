---
title: never的一些使用场景
date: 2021-02-07 14:07:14
categories: 代码
tags: typescript
---

因为在使用 ts 的过程中，经常遇到 never 一直没有深入的去了解，这一次深入地去学习了一下，并找出了几个使用场景。

收集了各个地方对于 never 的解释

- 从字面上理解，never 代表着永远不可能
- 收窄类型
- never 是一个空集合，任何值都不能冠以类型 never

```
T | never => T
```

1. 一个来自于@尤雨溪收窄类型类型的例子

```typescript
interface Foo {
  type: 'foo';
}

interface Bar {
  type: 'bar';
}

type All = Foo | Bar;
```

在 switch 当中判断 type，TS 是可以收窄类型(discriminated union)的 ：

```javascript
function handleValue(val: All) {
  switch (val.type) {
    case 'foo':
      // 这里 val 被收窄为 Foo
      break;
    case 'bar':
      // val 在这里是 Bar
      break;
    default:
      // val 在这里是 never
      const exhaustiveCheck: never = val;
      break;
  }
}
```

注意在 default 里面我们把被收窄为 never 的 val 赋值给一个显式声明为 never 的变量。如果一切逻辑正确，那么这里应该能够编译通过。但是假如后来有一天你的同事改了 All 的类型：

```typescript
type All = Foo | Bar | Baz;
```

然而他忘记了在 handleValue 里面加上针对 Baz 的处理逻辑，这个时候在 default branch 里面 val 会被收窄为 Baz，导致无法赋值给 never，产生一个编译错误。所以通过这个办法，你可以确保 handleValue 总是穷尽 (exhaust) 了所有 All 的可能类型。

2. 收窄类型

```typescript
type get<T> = T extends (arg: infer R) => any ? R : never;

const ddd: get<'asdasdasd'> = 3; /** 不能将类型“number”分配给类型“never **/
const ddd: get<(a: number) => number> = 3; /** 🆗 **/
```

3. 抽取公共的部分

```typescript
type A = {
  a: number;
  b: 'b';
  c: never;
};
type B = keyof A; // type B = "a" | "b" | "c"
type C = A[B]; // number | "b", 这里 never 去除了
```

当有一个 interface，想找出某些匹配类型的 key 值

```typescript
interface Person {
  id: number;
  name: string;
  lastName: string;
  load: () => Promise<Person>;
}
```

就需要遍历接口的每一个属性，然后判断类型返回

```typescript
type FilterName<Base, Name> = {
  [P in keyof Base]: Base[P] extends Name ? P : never;
}[keyof Base];
type Result = FilterName<Person, string | number>; // type Result = "id" | "name" | "lastName"
```

# 参考：

- [https://www.zhihu.com/question/354601204](https://www.zhihu.com/question/354601204)

# End
