---
title: 浅谈MVX
date: 2017-10-21 13:06:30
tags: MVX
categories: Architecture

---

## 序言

MVC框架最早出现在Java领域，然后慢慢的在前端开发中也被提到，后来又出现了MVP，以及现在最成熟的MVVM

<!--more-->

## MVX

### MVC

MVC（Model-View-Controller）是应用最广泛的软件架构之一，一般MVC分为: Model(模型)、Controller(控制器)和View(视图)。这主要是基于分层的目的，让彼此的职责分开。

控制器（Controller），一组行为的集合。(业务逻辑)

模型（Model），数据相关的操作和封装。(数据保存)

视图（View），视图的渲染。(用户界面)

View一般通过Controler来和Model进行联系的。Controller是Model和View的协调者，View和Model不直接联系。基本联系都是单项的。

![](http://image.beekka.com/blog/2015/bg2015020105.png)

1. View 传送指令到Controler
2. Controller 完成业务逻辑，要求 Model 改变状态
3. Model 将新的数据发送到　View，用户得到反馈

接受用户指令时，MVC 可以分成两种方式。一种是通过 View 接受指令，传递给 Controller。

![](http://image.beekka.com/blog/2015/bg2015020106.png)

另一种是直接通过controller接受指令。

![](http://image.beekka.com/blog/2015/bg2015020107.png)

### MVP

MVP模式将 Controller改名为Persenter,　同时改变了通信方向。

![](http://image.beekka.com/blog/2015/bg2015020109.png)

1. 各部分之间的通信，都是双向的。
2. View 与 Model不发生联系。都通过Persenter传递。
3. View 非常薄, 不部署任何业务逻辑，称为"被动视图"，即没有任何主动性，而Persenter非常厚，所有逻辑都部署在那里。

### MVVM

MVVM模式将Persenter改名为ViewModel，基本上与MVP完全一致

![](http://image.beekka.com/blog/2015/bg2015020110.png)

唯一的区别是，它采用双向绑定(data-binding): View的变动，自动反映在ViewModel，反之亦然。Angular和Vue采用这种模式。