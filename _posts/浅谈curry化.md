---
title: 浅谈curry化
date: 2017-10-25 20:12:02
tags: JavaScript
---

## 序言

在函数式编程中我了解到了柯里化，并且觉得它很有意思，所以对它的实现原理进行了探索，其中难点还真不少。

<!--more-->

## curry化的概念

curry 的概念很简单：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

你可以一次性地调用 curry 函数，也可以每次只传一个参数分多次调用

## curry化的实现代码

```js
let sub_curry = function(fn) {
    let args = [].slice.call(arguments, 1);
    return function() {
        return fn.apply(this, args.concat(Array.from(arguments)));
    };
};
let curry = function(fn, len) {
    len = len || fn.length;
    return function () {
        if(arguments.length < len) {
            let combined = [fn].concat(Array.from(arguments));
            return curry(sub_curry.apply(this, combined), len - arguments.length);
        }
        else {
            return fn.apply(this, arguments);
        }
    };
};
```

## curry化使用的哪些知识点

- 闭包
- 递归

## curry难点解析

难点是下面这句话的调用

```js
sub_curry.apply(this, combined)
```

这个语句返回一个闭包，并且这个闭包中的fn是前面一个函数。

附上一张图理解。

![](/img/closure.png)

这个最终的目的就是把所有的参数传递给fn调用

附上演示的栗子:

```js
let multiply = function (a, b, c) {
    return a * b * c;
};
let multiplyCurry = curry(multiply);
console.log(multiplyCurry(3)(2)(4));
//=>24
```

> 另一种实现

```js
// 另一种简单实现，参数只能从右到左传递
function createCurry(func, args) {
    var arity = func.length;
    var args = args || [];
    return function() {
        var _args = [].slice.call(arguments);
        [].push.apply(_args, args);
        // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
        if (_args.length < arity) {
            return createCurry.call(this, func, _args);
        }
        // 参数收集完毕，则执行func
        return func.apply(this, _args);
    };
};
```