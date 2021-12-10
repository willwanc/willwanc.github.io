---
layout: w
title: position 定位属性
date: 2021-12-10 13:59:10
tags: [position，定位属性]
categories: [css]
---

## position 定位属性

定位允许您从正常的文档流布局中取出元素，并使它们具有不同的行为，例如放在另一个元素的上面，或者始终保持在浏览器视窗内的同一位置。

### 属性值

- static：默认值，无特殊定位，遵循正常的文档流
- absolute：绝对定位，将对象从文本流脱离出来，相对于定位上下文(static 定位以外的第一个父元素) 进行定位；如果不存在这样的父元素，则依据body元素定位
- relative：相对定位，相对于其正常位置进行定位
- fixed：固定定位，相对于浏览器窗口进行定位
- sticky：粘性定位，该定位基于用户滚动的位置定位。页面未滚动时，它的行为就像 position:relative; 而当页面至粘性定位元素滚动超出目标区域时，它的表现就像 position:fixed;，它会固定在目标位置。

**注意: **Internet Explorer, Edge 15 及更早 IE 版本不支持 sticky 定位。 Safari 需要使用 -webkit- prefix (查看以下实例)。

### 绝对定位和相对定位的区别：

a.	参照物不同：绝对定位的参照物是包含块，相对定位的参照物是元素本身；
b.	绝对定位将对象从文档流中脱离出来因此不占据空间，相对定位不破坏正常的文档流顺序；无论是否进行移动，元素仍然占据原来的空间；


### Z-index层叠属性：检索或设置对象的层叠顺序

属性值：

- Auto：默认值，遵从其父对象；
- Number：无单位的整数值，可为负数；

**说明：** number值大对象会覆盖在number值较小的对象之上。如果两个决定定位对象的属性具有相同的number值，那么他们将依据他们在HTML文档中的声明顺序层叠。
此属性仅作用于position属性值为relative，fixed和absolute的对象；该属性不会作用于窗口控件上，如：select元素。

## 参考

- [MDN——定位](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout/Positioning)
- [菜鸟教程——CSS position 属性](https://www.runoob.com/cssref/pr-class-position.html)