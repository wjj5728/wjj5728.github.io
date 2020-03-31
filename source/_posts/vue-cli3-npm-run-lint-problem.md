---
title: vue-cli3中npm run lint遇到的问题
date: 2020-03-31 10:26:30
tags: vue eslint
---

# 背景

想给利用**[husky](https://github.com/typicode/husky)**给项目加一个 pre-commit 钩子，保证上传到 git 的代码格式都是统一的，因为每个人 clone 代码下来后，
可能因为自身的习惯问题，缩进等的都大不相同。

在使用 **husky** 之前，先试用了一下 npm run lint

因为在 vue-config.js 使用了 var utils = require('./build/utils.js')

commonjs 的模块引入方式 在**@typescript-eslint/eslint-plugin**中不推荐被使用，检测不被通过

```
error: Require statement not part of import statement (@typescript-eslint/no-var-requires) at vue.config.js:1:15:
> 1 | const utils = require('./build/utils');
```

# 方案

1. 在 rule 中关闭 @typescript-eslint/no-var-requires

```
rules: {
    ...
    '@typescript-eslint/no-var-requires': 0,
}
```

但是这个有一个弊端，就是在 tsx 以及 ts 文件中也不检测该项，既然使用了 typescript，当然还是推荐使用 es6 的模块化 Module-import 和 export

2. 使用 eslint 的覆盖功能来禁用 js 文件上的不兼容规则

在 github 找到了一个更加完善的[方案](https://github.com/typescript-eslint/typescript-eslint/issues/1724)

```
{
  "rules": {...},
  "overrides": [
    {
      "files": ["*-test.js","*.spec.js"],
      "rules": {
        "@typescript-eslint/no-var-requires": "off"
      }
    }
  ]
}
```

在一些指定的 js 文件夹，选择性的关闭这项规则，又能保证对 ts 文件的检测。

果然，鱼和熊掌有时候是可以兼得。

# 参考：

- [https://github.com/typescript-eslint/typescript-eslint/issues/1724](https://github.com/typescript-eslint/typescript-eslint/issues/1724)
- [https://eslint.org/docs/user-guide/configuring#disabling-rules-only-for-a-group-of-files](https://eslint.org/docs/user-guide/configuring#disabling-rules-only-for-a-group-of-files)

# End
