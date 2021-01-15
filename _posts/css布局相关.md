---
title: css布局相关
date: 2019-04-04 11:16:10
tags: CSS
---

## 序言

学习前端，学习CSS还是有必要的，CSS在布局方面非常重要，而如果我们能熟悉掌握CSS，那么在布局的时候就能如鱼得水。
<!--more-->

## margin

### margin与可视尺寸

- 适用于没有设定width/height的普通block的水平元素(float元素 absolut/fixed元素 inline水平, table-cell)
- 只适用于水平尺寸
- 实现一侧定宽的自适应布局

### margin与占据尺寸

- block/inline-block水平元素均适用
- 与有没有设定width/height值无关
- 适用于水平方向和垂直方向
- 实现滚动容器内上下留白(适用margin)

### margin的百分比

- 水平方向百分比/垂直方向百分比
- 普通元素百分比/绝对定位元素的百分比
- 普通元素的百分比margin都是相对于容器的宽度计算的
- 绝对定位的元素的百分比margin是相对于第一个定位祖先元素(absolute/relative/fixed)的宽度计算的

### margin重叠通常特性

- block水平元素(不包括float和absolute元素)
- 不考虑writing-mode, 只发生在垂直方向(margin-top/margin-bottom)

### margin重叠的3种情境

- 相邻的兄弟元素
- 父级和第一个/最后一个子元素
- 空的block元素

### margin-top重叠

- 父元素非块状格式化上下文
- 父元素没有border-top设置
- 父元素没有padding-top值
- 父元素和第一个子元素之间没有inline元素分隔

### margin-bottom重叠

- 父元素非块状格式化上下文
- 父元素没有border-top设置
- 父元素没有padding-top值
- 父元素和第一个子元素之间没有inline元素分隔
- 父元素没有height, min-height, max-height限制

### 空block元素margin重叠其他条件

- 元素没有border设置
- 元素没有padding值
- 里面没有inline元素
- 没有height, 或者min-height

### 重叠取值

- 正正取大值
- 负负最负值
- 正负值相加

### 理解margin auto

元素有时候, 就算没有设置width或height, 也会自动填充；原本应该填充的尺寸被width/height强制变更, 而margin: auto就是为了填充这个变更的尺寸设计的；如果一侧定值, 一侧auto, auto为剩余空间大小; 如果两侧军均是auto, 则平分剩余空间的；图片为何不居中呢？因为此图片是inline水平, 就算没有width,其也不会占据整个容器；

absolute与margin居中

```css
.father{height: 200px; position: relative}
.son{position: absolute; top: 0px; right: 0px; bottom: 0px; left: 0px;}

.father{height: 200px; position: relative}
.son{position: absolute; top: 0px; right: 0px; bottom: 0px; left: 0px; margin: auto}
```

### margin”无效”

- inline水平元素的垂直margin无效
- margin重叠
- position: absolute与margin(绝对定位元素的非定位方位值”无效”)
- 绝对定位的margin值一直有效, 只是不像普通元素那样, 可以和兄弟元素插科打诨


## position

### absolute

#### absolute特性

- 包裹性
- 破坏性
- 去浮动
- 位置跟随性
- 超越overflow
- 脱离文档流

#### absolute的强大之处

没有宽度和高度声明实现的全屏自适应效果

```css
position: absolute; 
left: 0; top: 0; right: 0; bottom: 0;//实现布满整个容器
```

### relative

#### relative特性

- 相对自身
- 无浸入(不影响其他盒子模型的定位)(原本的位置还存在)

#### relative的作用:限制absolute

- 限制left/top/right/bottom定位
- 限制z-index层级(可以对fixed的限制有效)
- 限制在overflow下的嚣张气焰
- 尽量少用relative

## float

### float的诞出的初始目的

- 文字环绕效果

### float的特性

- 包裹性
- 破坏性
- 脱离文档流
- float浮动去空格
- 浮动与display:inline-block化(破坏性造成紧密排列)

### 清除浮动(应该应用在其父级元素上)

```css
.clearfix:after{
	content:’’; 
	display: block; 
	height:0; 
	clear:both
}
```

## line-height

### line-height的定义

- 行高, 两行文字基线之间的距离
- 一行文字也是有行高的(两行的定义已经决定了一行的表现)

### line-height与行内框盒子模型

- 所有内联元素的样式表现都与行内框盒子模型有关!例如浮动的图文环绕效果
- “内容区域”，是一种环绕文字看不见的盒子。 “内容区域”的大小与font-size大小相关
- “内联盒子”，”内联盒子”不会让内容成块显示，而是排成一行。如果外部含inline水平标签(span, a, em等),则属于”内联盒子”。如果是个光秃秃的文字，则属于”匿名内联盒子”
- “行框盒子”,每一行就是一个”行框盒子”，每个”行框盒子”又是由一个个”内联盒子组成”
- p标签所在的”包含盒子”, 此盒子由一行行的”行框盒子”组成
- 没有定高的盒模型并不是由文字撑开, 是由行高决定
- 行高由于其继承性，影响无处不在，即使单行文本也比例外
- 行高只是幕后黑手，高度的表现不是行高，而是内容区域和行间距
- 内容区域高度 + 行间距 = 行高
- 内容区域高度至于字号以及字体有关，与行高没有任何关系
- 行高决定内联盒子高度；行间距墙头草，可大可小(甚至负值),保证高度正好等同于行高
- 多行文本的高度就是单行文本高度累加
- line-height: normal(默认属性值)
- 行高不会影响图片的占据的高度
- 隐匿的文本节点

### 消除图片底部间隙的方法

- 图片块状化-无基线对齐
- 图片底线对齐img{vertical-align: bottom}
- 行高足够小-基线位置上移.box{line-height: 0;}

### line-height的实际应用

大小不固定的图片,多行文字垂直居中

```css
.box{
	line-height: 300px; 
	text-align: center;
}
.box > img{
	vertical-align: middle;
}
```

多行文本水平居中

```css
.box{
	line-height: 250px; 
	text-align: center;
}
.box > .text{
	display: inline-block; 
	line-height: normal; 
	text-align: left; 
	vertical-align: middle;
}
```

## padding

### padding的特性

- padding不支持任何形式的负值
- padding百分比均是相对于宽度计算的

### padding对于block水平元素的影响

- padding值暴走，一定影响尺寸
- width非auto, padding影响尺寸
- width为auto或box-sizng为border-box, 同时padding值没有暴走，不影响尺寸

### padding对inline水平元素的影响

- 水平padding影响尺寸, 垂直padding不影响尺寸，但是会影响背景色(占据空间)
- 可以用来实现高度可控的分隔符

### inline水平元素的padding百分比值

- 相对于宽度计算
- padding会断行
- 默认的高度宽度细节有差异
- inline元素的垂直padding会让”幽灵空白节点”显现(造成padding宽高不等)

## border

### border的特性

- border-width不支持百分比(类似的outline, text-shadow, border-shadow)
- border-style特性(solid(实线), dashed(虚线), dotted(点线, double(双线)))
- 不指定border-color时默认就是color(类似的text-shadow, border-shadow)

## overflow

### overflow的特性

- 属性(visible, hidden, scroll, auto, inherit)
- 如果overflow-y=overflow-x值相同, 则等同于overflow;
- 如果overflow-y=overflow-x值不同, 且其中一个属性的值被置为visible, 而另一个被赋予hidden, - scroll, auto,那么这个visible被重置为auto;
- 无论什么浏览器, 默认滚动条均默认来自html而不是body标签
- 浏览器默认IE8+html(overflow: auto)
- overflow的padding-bottom缺失(除chrome其他都缺失)
- overflow与BFC(块状化格式上下文)——(清除浮动, 自适应布局等)

### 触发BFC(overflow: auto, hidden, scroll)———(页面结界, 内部元素再怎么也不影响外部)

- 应用(清除浮动影响, 避免margin穿透问题, 两栏自适应布局)
- overflow对绝对定位元素失效(避免失效:verflow元素自身为包含块(relative), overflow子元素为包含块)
- resize属性,resize:both(水平垂直),resize:horizontal(只有水平拉), resize:vertical(只有垂直拉),(此声明要想起作用,overflow属性值不能是visible)
- overflow与锚链接

## vertical-align

### vertical-align的特性

- 线类 baseline, top, middle, bottom
- 文本类 text-top, text-bottom
- 上标下标类 sub, super
- 数值百分比类(在baseline对齐基础上上下偏移对应数值大小)
- 20px,2em,20%(百分比是相对于Line-height计算的)
- 应用于Inline水平以及”table-cell”元素

inline水平

- inline: img, span
- inline-block: input, button

table-cell

- table-cell: td(对自身起作用)
- inline-block的基线是正常流中最后一个line box的基线，除非这个line box里面既没有Line boxes或者本身overflow属性的计算值而不是visible, 这种情况下基线是margin底边缘

### 线性类属性值

bottom

- inline/inline-block元素： 元素底部和整行的底部对齐
- table-cell元素: 单元格底padding边缘和表格行的底部对齐

top

- inline/inline-block元素： 元素顶部和整行的顶部对齐
- table-cell元素: 单元顶部padding边缘和表格行的顶部对齐

middle

- inline/inline-block元素： 元素的垂直中心点和父元素的基线上1/2x_height处对齐对齐
- table-cell元素: 单元格填充盒子相对于外面的表格行居中对齐

文本类属性

- vertical-align: text-top;(盒子的顶部和父级content area的顶部对齐)
- vertical-align: text-top;(盒子的底部和父级content area的底部对齐)

上标, 下标

- <sup> => vertical-align: sup;
- <sub> => vertical-align: sub;
- vertical-align: sup(提高盒子的基线到父级合适的上标基线位置)
- vertical-align: sub(降低盒子的基线到父级合适的下标基线位置)

## CSS3


## BFC(块格式化上下文)

块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视化CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

下列方式会创建块格式化上下文：

- 根元素或包含根元素的元素
- 浮动元素（元素的 float 不是 none）
- 绝对定位元素（元素的 position 为 absolute 或 fixed）
- 行内块元素（元素的 display 为 inline-block）
- 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
- 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
- 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
- overflow 值不为 visible 的块元素
- display 值为 flow-root 的元素
- contain 值为 layout、content或 strict 的元素
- 弹性元素（display为 flex 或 inline-flex元素的直接子元素）
- 网格元素（display为 grid 或 inline-grid 元素的直接子元素）
- 多列容器（元素的 column-count 或 column-width 不为 auto，包括 column-count 为 1）
- column-span 为 all 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中

块格式化上下文包含创建它的元素内部的所有内容.

块格式化上下文对浮动定位（参见 float）与清除浮动（参见 clear）都很重要。浮动定位和清除浮动时只会应用于同一个BFC内的元素。浮动不会影响其它BFC中元素的布局，而清除浮动只能清除同一BFC中在它前面的元素的浮动。外边距折叠（Margin collapsing）也只会发生在属于同一BFC的块级元素之间。
