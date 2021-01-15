---
title: 浅谈JavaScript的闭包
date: 2019-03-08 14:12:04
tags: JavaScript
---

## 序言

在JS中闭包是一个很重要的概念，很多地方都运用到了闭包，所以有必要深入理解一下闭包。

<!-- more -->

## 基本概念

闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。

```js
function createCompareFn (propertyName) {
	return function (object1, object2) {
		var value1 = object1[propertyName];
		var value2 = object2[propertyName];
		
		if (value1 < value2) {
			return -1;
		} else if (value1 > value2) {
			return 1;
		} else {
			return 0;
		}
	}
}
```

内部函数访问了外部函数的变量propertyName，之所以还能够访问这个变量，是因为内部函数的作用域链中包含外部函数的作用域链。