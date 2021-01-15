---
title: ECMAScript 面向对象
date: 2019-03-04 15:41:54
tags: JavaScript
---

## 序言

很多编程语言都有面向对象的编程，ECMAScript同样也是如此。ECMAScript语言的传统方法是通过构造函数定义并生成新对象，自从ES6出来之后，ES6提供了更接近传统语言的写法，引入了Class(类)这个概念作为对象的模板。

<!-- more -->

## ES5的面相对象

### 理解对象

#### 属性类型

ECMAScript中有两种属性：数据属性和访问器属性。

##### 数据属性

数据属性包含一个数据值的位置。在这个位置可以读取和写入值。数据属性有4个描述其行为的特性。

- [[Configurable]]：表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。对象字面量定义这个特性默认值为true
- [[Enumerable]]：表示能否通过for-in循环返回属性。对象字面量定义这个特性默认值为true
- [[Writable]]：表示能否修改属性的值。对象字面量定义这个特性默认值为true
- [[Value]]：包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把这个新值保存在这个位置。这个特性的默认值为undefined

对于对象字面量，[[Configurable]]、[[Enumerable]]、[[Writable]]特性都被设置为true，而[[Value]]特性被设置为指定的值。例如：

```js
var person = {
	name: "sceley"
}
```

要修改属性默认的特性，必须使用ES5的Object.defineProperty()方法。这个方法接收三参数：属性所在的对象、属性的名字和一个描述符对象。其中描述符对象的属性必须是：[[Configurable]]、[[Enumerable]]、[[Writable]]、[[Value]]。设置其中的一个或多个值，可以修改对象的特性值。

##### 访问器属性

访问器属性不包含数据值；它们包含一对儿getter和setter函数(不过，这两个函数都不是必需的)。要读取访问器属性时，会调用getter函数，这个函数负责返回有效的值；在写入访问器属性时，会调用setter函数并传入新值，这个函数负责决定如何处理函数。访问器属性有如下4个属性。

- [[Configurable]]：表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。对象字面量定义这个特性默认值为true
- [[Enumerable]]：表示能否通过for-in循环返回属性。对象字面量定义这个特性默认值为true
- [[Getter]]：在读取属性时调用的函数。默认值为undefined。
- [[setter]]：在写入属性时调用的函数。默认值为undefined。

访问器不能直接定义，必须使用Object.defineProperty()来定义。

```js
var book = {
	__year: 2004,
	edition: 1
}
Object.defineProperty(book, 'year', {
	get: function () {
		return this._year;
	},
	set: function (value) {
		this._year = value;
	}
});
```

### 创建对象

#### 工厂模式

工厂模式是软件工程领域一种广为人知的设计模式，用函数来封装以特定接口创建对象的细节。

```js
function createPerson (name, age, job) {
	var o = new Object();
	o.name = name;
	o.age = age;
	o.job = job;
	o.sayName = function () {
		console.log(this.name);
	};
	return o;
}
```

缺点：工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题。

#### 构造函数模式

ES中的构造函数可以用来创建指定类型的对象。像Object和Array这样的原生构造函数，在运行时会自动出现在执行环境中。此外，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。

```js
function Person (name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = function () {
		console.log(this.name);
	}
}
var person1 = new Person('sceley', 20, "Engineer");
var person2 = new Person('kitty', 20, "Doctor");
```

以这种方式调用构造函数实际上会经历以下4个步骤：

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象(因此this就指向了这个新对象)
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回对象

构造函数的缺点就是每个方法都要在每个实例上重新创建一遍。

#### 原型模式

我们创建的每个函数都有一个prototype(原型)属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。

```js
function Person () {
}
Person.prototype.name = "sceley";
Person.prototype.name = 22;
Person.prototype.sayName = function () {
	console.log(this.name);
}
```

#### 组合使用构造函数模式和原型模式

```js
function Person(name, age, job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.friends = ['sceley', 'kitty'];
}
Person.prototype = {
	constructor: Person,
	sayName: function () {
		console.log(this.name);
	}
};
```

这种构造函数与原型混成的模式，是目前认同度高的一种创建自定义类型的方法。

### 继承

#### 组合继承

```js
function Biology (name, age, category) {
    this.age = age;
    this.name = name;
    this.category = category;
}

Biology.prototype.sayCategory = function () {
    return this.category;
}

Biology.prototype.sayName = function () {
    return this.name;
}

Biology.prototype.sayAge = function () {
    return this.age;
}

function Animal (name, age, feed) {
    this.feed = feed;
    Biology.call(this, name, age, 'animal');
}

Animal.prototype = new Biology();
Animal.prototype.constructor = Animal;
Animal.prototype.doFeed = function () {
    if (this.feed >= 5) {
        this.feed = this.feed - 5;
        return `喂食成功, 饲料剩余${this.feed}斤`;
    } else {
        return '喂食失败, 饲料不足';
    }
}

function Plant (name, age, water) {
    this.water = water;
    Biology.call(this, name, age, 'plant');
}

Plant.prototype = new Biology();
Plant.prototype.constructor = Plant;
Plant.prototype.doWater = function () {
    if (this.water >= 10) {
        this.water = this.water - 10;
        return `浇水成功, 水剩余${this.water}桶`;
    } else {
        return '浇水失败, 水不足';
    }
}
```

缺陷：组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型的时候，另一次是在子类型构造函数内部。子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。

tips:
> 使用原型模式来让所有对象实例共享它所包含的属性和方法。

> ES5中实现子类继承父类是使用了原型链作为主要方法，其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

> 所有引用类型默认都继承了Object，而这个继承也是通过原型链实现的。

> 通过原型来实现继承时，原型实际上会变成另一个类型的实例，解决的方法是借用构造函数技术，即在子类构造函数的内部调用超类型构造函数。

> 原型链的问题是对象实例共享所有继承的属性和方法，因此不适宜单独使用。

> 组合继承，这种模式使用原型链继承共享的属性和方法，而通过借用构造函数继承实例继承属性。


#### 寄生组合继承

所谓的寄生组合继承，即通过借用构造函数来继承属性，通过原型链的混成形势来继承方法。其背后的基本思想是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。

```js
function inheritPrototype(subType, superType) {
	var prototype = Object(superType.prototype);
	prototype.constructor = subType;
	subType.prototype = prototype;
}

function SuperType(name) {
	this.name = name;
	this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function () {
	return this.name;
}

function SubType(name, age) {
	SuperType.call(this, name);
	
	this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototy.sayAge = function () {
	return this.age;
}
```

开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。


## ES6类的面向对象

```js
class Biology {
	constructor (name, age, category) {
		this.name = name;
		this.age = age;
		this.category = category;
	}

	sayName () {
		return this.name;
	}

	sayAge () {
		return this.age;
	}

	sayCategory () {
		return this.category;
	}
};

class Plant extends Biology {
	constructor (name, age, water) {
		super(name, age, 'tree');
		this.water = water;
	}

	doWater () {
	    if (this.water >= 10) {
	        this.water = this.water - 10;
	        return `浇水成功, 水剩余${this.water}桶`;
	    } else {
	        return '浇水失败, 水不足';
	    }
	}	
};

class Animal extends Biology {
	constructor (name, age, feed) {
		super(name, age, 'animal');
		this.feed = feed;
	}

	doFeed () {
	    if (this.feed >= 5) {
	        this.feed = this.feed - 5;
	        return `喂食成功, 饲料剩余${this.feed}斤`;
	    } else {
	        return '喂食失败, 饲料不足';
	    }
	}
}
```