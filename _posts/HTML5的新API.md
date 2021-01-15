---
title: HTML5的新API
date: 2019-04-18 10:38:28
tags: HTML5
---

## 序言

HTML5出现了很多新的API，而在这里我总结了两个可能会使用到的api，requestAnimationFrame、MutationObserver。

<!-- more -->

## requestAnimationFrame

### 基本概念

requestAnimationFrame方法用于以一种更好的性能来实现动画。

通过window.requestAnimationFrame方法，浏览器可以具有将各种并发性动画结合入一个单一的页面进行创建及渲染的能力，这种能力将使得动画的实现具有更好的性能。当用户将浏览器标签切换到其他标签窗口时，当前页面中的动画将被暂停运行，以减少CPU、GPU与内存的消耗。

window.requestAnimationFrame方法告诉浏览器希望执行动画并请求浏览器调用指定的函数在下一次重绘之前更新动画。它采用系统时间间隔，保持最佳的绘制效率，不会因为时间间隔过短，造成过度绘制，增加开销；也不会因为间隔太长，使动画卡顿不流畅

### 使用示例

```html
<div class="box"></div>
<script type="text/javascript">
    let height = 2;
    function change () {
        const box = document.querySelector('.box');
        height += 2;
        box.style.height = height + 'px';
        if (height <= 600) {
            loop();
        }
    }
    function loop() {
        requestAnimationFrame(change);
    }
    loop();
</script>
```

### 优势

- 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率。
- 在隐藏或不可见的元素中，将不会进行重绘或回流
- 页面不激活状态下，进入睡眠状态，暂停动画运行。

## Mutation Observer

MutationObserver api的出现是为了检测页面变化。

我们首先需要创建MutationObserver对象，创建方法如下所示：

```js
function onchange(mutationRecords, mutationObserver) {
	console.log('检测到动画变化');
}

var mo = new MutationObserver(onchange);
```

MutationOberver对象的构造函数使用一个参数，参数值为当检测到页面变化时所需执行的函数，该函数又使用两个参数，第一个参数值为包含页面变化时所产生的一系列信息的数组，第二个参数值为被调用的MutationObserver对象实例。

需要调用MutationObserver实例的observer方法进行检测。

使用示例:

```html
<div id="mutation">mutation</div>
<button id="btn">change</button>
<script>
    const mutation = document.querySelector('#mutation');
    const btn = document.querySelector('#btn');
    btn.onclick = function () {
        if (mutation.style.color == 'red') {
            mutation.style.color = 'inherit';
        } else {
            mutation.style.color = 'red';
        }
        
    }
    const options = {
        attributes: true
    };
    const mo = new MutationObserver((mutationRecords) => {
        console.log(mutationRecords);
    });
    mo.observe(mutation, options);
</script>
```

MutationObserver对象的observer方法使用两个参数，其中一个参数值为被观察的目标节点，第二参数值为观察时所使用的各种选项。

options:

- attributes 是否观察目标节点的所有属性
- characterData 是否观察目标节点的子文字节点
- childList 是否观察目标节点的子节点



