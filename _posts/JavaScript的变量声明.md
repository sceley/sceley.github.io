---
title: JavaScript的变量声明
date: 2019-03-07 12:53:09
tags: JavaScript
---

## 序言

在ES6出世以前，在JavaScript中声明变量的方式只有var，在ES6出世以后，新增了let、const两种变量声明的方式。

<!-- more -->

## ES5的var声明

### 基本用法

ECMAScript的变量是松散类型的，所谓松散类型就是可以用来保存任何类型的数据。换句话说，每个变量仅仅是一个用于保存值的占位符而已。

```js
var message
```

这行代码定义了一个名为message的变量，该变量可以用来保存任何值（像这样未经过初始化的变量，会保存一个特殊的值--undefined）

直接初始化变量，在定义的同时直接给予赋值

```js
var message = 'hi'
```

用var操作符定义的变量将成为定义该变量的作用域中的局部变量。也就是说，如果在函数中使用var定义一个变量，那么这个变量在函数退出后就会被销毁，例如:

```js
function test () {
	var message = 'hi';		//局部变量
}
test();
console.log(message);		//错误
```

给一个未定义声明过的变量赋值时，会创建一个全局变量：

```js
function test () {
	message = "hi";		// 全局变量
}
test();
console.log(message);	// 'hi'
```
给未经声明的变量赋值在严格模式下会导致抛出ReferenceError错误

### 重复声明

JavaScript从来不会告诉你是否多次声明了同一变量；遇到这种情况，它只会对后续的声明视而不见(不过，它会执行后续声明中的变量初始化，如果声明不初始化如果值不变)。

```js
var i = 10;
console.log(i);	//10
var i;
console.log(i);	//10
```

### 变量提升

var声明变量存在变量提升

```js
console.log(foo);	//undefined
var foo = 2;
```

### 全局对象的属性

所有在全局作用域中声明的变量、函数都会变成global(浏览器环境中是window)对象的属性和方法。

```js
var age = 20;
function sayAge () {
	console.log(this.age);
}

console.log(global.age);	//29
sayAge();						//29
window.sayAge();				//29

```

全局变量不能通过delete操作符删除，而直接在global对象上的定义的属性可以。

```js
var age = 20;
global.color = 'red';

delete global.age;					//return false

delete global.color;				//return true

console.log(global.age);			//20
console.log(global.color);			//undefined

```

尝试访问未声明的变量会抛出错误，但是通过查询global对象，可以知道某个可能未声明的变量是否存在。例如：

```js
var newValue = oldValue;				//这里会抛出错误，因为oldValue未定义

var newValue = window.oldValue;		//这里不会抛出错误，因为这是一次属性查询

```

## ES6的let、const声明

### let

#### 基本用法

let声明的变量只在let命令所在的代码块内有效

```js
{
	let a = 10;
	var b = 1;
}

a	// ReferenceError: a is not defined
b	//1
```

for循坏的计数器，就很适合使用let命令。

```js
for(let i = 0; i < arr.length; i++) {};

console.log(i);
//ReferenceError
```
以上代码中的计数器i，只在for循环体内有效。

一下的代码如果使用var，最后将输出10。

```js
var a = [];
for (var i = 0; i < 10; i++) {
	a[i] = function () {
		console.log(i);
	}
}
a[6]();	//10
```

上面的代码中，变量i是var声明的，在全局范围内都有效。所以每一次循环，新的i值都会覆盖旧值，导致最后输出的是最后一轮的i值。

#### 暂时性死区

只要块级作用域内存在let命令，它所声明的变量就“绑定”(binding)这个区域，不再受外部的影响。

```js
var tmp = 123;

if (true) {
	tmp = 'abc';	//ReferenceError
	let tmp;
}
```
上面的代码中存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。

ES6明确规定，如果区块中存在let和const命令，则这个区块对这些命令声明的变量从一开始就形成封闭作用域。只要在声明之前使用这些变量，就会报错。

#### 不允许重复声明

let 不允许在相同作用域内声明同一个变量。

```js
//报错
function () {
	let a = 10;
	var a = 1;
}

//报错
function () {
	let a = 10;
	let a = 1;
}
```

不能在函数内部重新声明参数。

```js
function func(arg) {
	let arg;	//报错
}

function func (arg) {
	{
		let arg;	//不报错
	
	}
}
```

### const

#### 基本用法

const用来声明常量。一旦声明，其值就不能改变。

```js
const PI = 3.1415; 
PI 			//3.1415

PI = 3 	//	TypeError: "PI" is	read-only
```

const声明的常量不得改变值。这意味着，const一旦声明常量，就必须立即初始化，不能留到以后赋值。

#### 全局对象的属性

全局对象是最顶层的对象，在浏览器环境指的是window对象，在Node.js中指的是global对象。在ES5中，全局对象的属性和全局变量是等价的。

```js
window.a = 1;
a //1

a = 2;
window.a //2
```

这种规定被视为JavaScript语言的一大问题，因为很容易不知不觉就创建了全局变量。ES6位了改变这一点，一方面规定，var命令和function命令声明的全局变量依旧是全局对象的属性；另一方面规定，let命令、const命令和class命令声明的全局变量不属于全部对象的属性。

```js
var a = 1;
window.a //1

let b = 1;
window.b //undefined
```


