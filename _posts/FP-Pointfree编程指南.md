---
title: FP-Pointfree编程指南
date: 2017-10-25 13:55:32
tags: JavaScript
---

## 序言

<!--more-->

## Pointfree

### Pointfree概念

pointfree 模式指的是，永远不必说出你的数据

不使用所要处理的值，只合成运算过程。

example

```js
let compose = function (f, g) {
    return function (x) {
        return f(g(x));
    };
};
let addOne = x => x + 1;
let square = x => x * x;
let addOneSquare = compose(square, addOne);
addOneSquare(2);
//=>9
```

上面一个栗子，把两个函数组合，然后求值。

addOneThenSquare是一个合成函数。定义它的时候，根本不需要提到要处理的值，这就是 Pointfree。

### Pointfree的本质

Pointfree 的本质就是使用一些通用的函数，组合出各种复杂运算。上层运算不要直接操作数据，而是通过底层函数去处理。这就要求，将一些常用的操作封装成函数。

example: 读取对象的role属性，不要直接写成obj.role, 而是要把这个操作封装成函数。

```js
let prop = (p, obj) => obj[p];
let propRole = curry(prop)('role');
```