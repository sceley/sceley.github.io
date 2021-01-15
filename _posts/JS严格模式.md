---
title: JS严格模式
date: 2019-04-12 11:57:55
tags: JavaScript
---

## 序言

除了正常运行模式，ECMAscript 5添加了第二种运行模式："严格模式"（strict mode）。顾名思义，这种模式使得Javascript在更严格的条件下运行。

<!-- more -->

## 初衷

- 消除Javascript语法的一些不合理、不严谨之处，减少一些怪异行
- 消除代码运行的一些不安全之处，保证代码运行的安全；
- 提高编译器效率，增加运行速度；
- 为未来新版本的Javascript做好铺垫。

## 进入标志

进入"严格模式"的标志，是下面这行语句:

```js
"use strict";
```

## 如何调用

"严格模式"有两种调用方法，适用于不同的场合。

### 针对整个脚本文件

将"use strict"放在脚本文件的第一行，则整个脚本都将以"严格模式"运行。如果这行语句不在第一行，则无效，整个脚本以"正常模式"运行。

```js
<script>
　　"use strict";
	console.log("这是严格模式。");
</script>
```

### 针对单个函数

将"use strict"放在函数体的第一行，则整个函数以"严格模式"运行。

```js
function strict(){
　　"use strict";
	return "这是严格模式。";
}
```

## ES6

ES6的模块自动采用严格模式，不管有没有在模块头部加上"use strict"。

## 严格模式下的限制

- 变量必须声明后再使用。
- 函数的参数不能有同名属性，否则报错。
- 不能使用with语句。
- 不能对只读属性赋值，否则报错。
- 不能使用前缀0表示八进制，否则报错。
- 不能删除不可能删除的属性，否则报错。
- 不能删除变量(delete prop)，会报错，只能删除属性(delete global[prop])。
- eval不会在其外层作用域引入变量。
- eval和arguments不能被重新赋值。
- arguments不会自动反映函数参数的变化。
- 不能使用arguments.callee。
- 不能使用arguments.caller。
- 禁止this指向全局对象。
- 不能使用fn.caller和fn.arguments获取函数调用的堆栈。
- 增加了保留字(比如protected、static和interface)。

