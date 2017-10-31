---
title: JavaScript常用设计模式
date: 2017-10-28 13:24:23
tags: 设计模式
categories: JavaScript
---

## 序言

模式是一种可复用的解决方案，可用于解决软件设计中遇到的常见问题，如在我们编写的JavaScript应用程序的实例中。另一种模式的方式是将解决问题的方法制作成模板，并且这些模板可应用于多种不同的情况。

<!-- more-->

## 常用的设计模式

- 工厂模式
- 单体模式
- 模块模式
- 代理模式
- 职责链模式
- 命令模式
- 模板方法模式
- 策略模式
- 发布-订阅模式
- 中介者模式

### 工厂模式

工厂模式是为了解决多个类似的对象声明的问题，也就是为了解决实例化对象产生的重复问题。

```js
function CreatePerson(name, age, sex) {
    let obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sex = sex;
    obj.sayName = function () {
        return this.name;
    };
    return obj;
};

let person1 = new CreatePerson('xiaowang', '20', 'boy');
let person2 = new CreatePerson('mary', '18', 'girl');
console.log(person1.name);//xiaowang
console.log(person2.name);//may

//返回都是object，无法识别对象的类型，不知道他们是哪个对象的实例。
console.log(typeof person1);//object
console.log(typeof person2);//object
console.log(person1 instanceof Object);//true
```

> 优点: 能解决多个相似的问题
> 缺点: 不能知道对象识别的问题(对象的类型不知道)

### 单体模式

单体模式是一个用来划分命名空间并将一批属性和方法组织在一起的对象。如果它可以被实例化，那么它只能被实例化一次。

对象字面量来创建单体模式

```js
let Singleton = {
    attr1: 1,
    attr2: 2,
    method1: function () {
        return this.attr1;
    },
    method2: function () {
        return this.attr2;
    }
};
```

单体化模式

```js
let Singleton = function (name) {
    this.name = name;
};
Singleton.prototype.getName = function () {
    return this.name;
};
let getInstance = (function () {
    let instance = null;
    return function (name) {
        if(!instance) {
            instance = new Singleton(name);
        }
        return instance;
    }
})();
let a = getInstance('aa');
let b = getInstance('bb');
console.log(a == b);//true
console.log(a.getName());//aa
console.log(b.getName());//aa
```

单体模式的优点:

- 可以来用划分命令空间，减少全局变量。
- 使用单体模式可以使代码组织的更为一致，使代码容易阅读和维护。

### 模块模式

模块模式的思路是为单体模式添加私有变量和私有方法能够减少全局变量的使用。

```js
let Single = (function () {
    //私有变量
    let privateNum = 112;
    //公有变量
    let publicNum = 110;
    //私有函数
    function privateFunc() {
        //业务逻辑代码
    };
    //公有函数
    function publicFunc() {
        //业务逻辑代码
    }；
    //返回一个对象包含公有方法和属性
    return {
        publicNum,
        publicFunc
    };
});
```

### 代理模式

```js
//声明一个妹子
let AGirl = function (name) {
    this.name = name;
};
//声明一个男孩
let ABoy = function (girl) {
    this.girl = girl;
    //送礼物给一个妹子
    this.sendMarriageGift = function (gift) {
        return 'Hi ' + this.girl.name + ', a　boy 送你一个礼物: ' + gift;
    };
};
//代理人
let Proxy = function (girl) {
    this.girl = girl;
    this.sendGift = function (gift) {
        new ABoy(girl).sendMarriageGift(gift);
    };
};
//初始化
let proxy = new Proxy(new AGirl('漂亮妹子'));
proxy.sendGift('结婚戒');//Hi 漂亮妹子, a boy 送你一个礼物：　结婚戒
```