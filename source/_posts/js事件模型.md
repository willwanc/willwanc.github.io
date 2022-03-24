---
title: js事件模型
tags: []
categories: ['javascript']
date: 2022-03-24 11:17:00
---

## 事件对象

当DOM元素上的发生某个事件时， 会产生一个事件对像 event，这个对象包含着与事件有关的所有信息。事件对象作为参数传给事件监听函数。浏览器原生提供一个Event对象，所有的事件都是这个对象的实例，或者说继承了Event.prototype对象。

### 事件对象的跨浏览器兼容

- 获取事件对象

```js
event ? event : window.event;
```

- 获取事件目标

```js
var event = event || window.event,
  target = event.target || event.srcElement;
```

- 阻止事件默认事件

```js
var event = event || window.event,
if (event.preventDefault){
    event.preventDefault();
} else {
    event.returnValue = false;
}
```

- 阻止事件冒泡

```js
var event = event || window.event,
if (event.stopPropagation){
    event.stopPropagation();
} else {
    event.cancelBubble = true;
}
```

## 事件模型

事件模型有三种：原始事件模型（DOM0 级）、标准事件模型（DOM2 级）、IE 事件模型

### 事件流

DOM 结构是一个树型结构，当一个 HTML 元素发生一个事件时，该事件会在元素结点与根结点之间的路径传播，传播路径经过的结点都会接收到该事件，这个传播过程称为 DOM 事件流。

事件流共包含三个阶段：

- 事件捕获阶段(capture phase)
- 处于目标阶段(target phase)
- 事件冒泡阶段(bubbling phase)

### 原始事件模型（DOM0 级）

绑定事件监听函数：

- HTML 代码中直接绑定

```html
<input type="button" onclick="fun()" />
```

- 通过 JS 代码绑定

```js
var btn = document.getElementById(".btn");
btn.onclick = fun;
```

移除事件监听函数方式：只要将对应事件属性置为 null 即可

```js
btn.onclick = null;
```

#### 特性

- 绑定速度快（但由于绑定速度太快，可能页面还未完全加载出来，以至于事件可能无法正常运行）
- 跨浏览器兼容性好
- 只支持冒泡，不支持捕获
- 同一个类型的事件只能绑定一次

### 标准事件模型（DOM2 级）

在该事件模型中，一次事件共有三个阶段:

- 事件捕获阶段：事件从 document 一直向下传播到目标元素, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行
- 处于目标阶段：事件到达目标元素, 触发目标元素的监听函数
- 事件冒泡阶段：事件从目标元素冒泡到 document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行

绑定事件监听函数

```js
addEventListener(eventType, handler, useCapture);
```

移除事件监听函数

```js
removeEventListener(eventType, handler, useCapture);
```

- eventType: 指定事件类型(不要加 on)
- handler: 事件监听函数
- useCapture: 布尔值，表示监听函数是否在捕获阶段（capture）触发，默认为false（监听函数只在冒泡阶段被触发）。该参数可选。一般设置为 false 与 IE 浏览器保持一致。

#### 特性

- 可以在一个 DOM 元素上绑定多个相同类型的事件监听函数
- 可以指定事件监听函数执行时机
- 兼容差，兼容 IE9+

### IE 事件模型

IE 事件模型共有两个过程：

- 处于目标阶段：事件到达目标元素, 触发目标元素的事件监听函数
- 事件冒泡阶段：事件从目标元素冒泡到 document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行

绑定事件监听函数

```js
attachEvent(eventType, handler);
```

移除事件监听函数

```js
detachEvent(eventType, handler);
```

### 跨浏览器的事件监听程序

添加事件监听程序

```js
function addHandler(element, type, handler) {
    if (element.addEventListener) {
        element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
        element.attachEvent("on" + type, handler);
    } else {
        element["on" + type] = handler;
    }
}
```

移除事件监听程序

```js
function removeHandler(element, type, handler) {
    if (element.removeListener) {
        element.removeListener(type, handler, false);
    } else if (element.detachEvent) {
        element.detachEvent("on" + type, handler);
    } else {    
        element["on" + type] = null;
    }
}
```

**this指向：事件监听函数内部的this指向触发事件的那个元素节点。**

## 事件的代理

由于事件会在冒泡阶段向上传播到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）。

```js
var ul = document.querySelector('ul');

ul.addEventListener('click', function (event) {
  if (event.target.tagName.toLowerCase() === 'li') {
    // some code
  }
});
```

上面代码中，click事件的监听函数定义在`<ul>`节点，但是实际上，它处理的是子节点`<li>`的click事件。

事件代理的好处：**只要定义一个监听函数，就能处理多个子节点的事件。而且以后再添加子节点，监听函数依然有效。**

如果希望事件到某个节点为止，不再传播，可以使用事件对象的stopPropagation方法。

```js
// 事件传播到 p 元素后，就不再向下传播了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, true);

// 事件冒泡到 p 元素后，就不再向上冒泡了
p.addEventListener('click', function (event) {
  event.stopPropagation();
}, false);
```

上面代码中，stopPropagation方法分别在捕获阶段和冒泡阶段，阻止了事件的传播。

**但是，stopPropagation方法只会阻止事件的传播，不会阻止该事件触发<p>节点的其他click事件的监听函数。也就是说，不是彻底取消click事件。**

如果想要彻底阻止这个事件的传播，不再触发后面所有click的监听函数，可以使用stopImmediatePropagation方法。

```js
p.addEventListener('click', function (event) {
  event.stopImmediatePropagation();
  console.log(1);
});

p.addEventListener('click', function(event) {
  // 不会被触发
  console.log(2);
});
```

## 自定义事件（CustomEvent） 接口

CustomEvent 接口用于生成自定义的事件实例。那些浏览器预定义的事件，虽然可以手动生成，但是往往不能在事件上绑定数据。如果需要在触发事件的同时，传入指定的数据，就可以使用 CustomEvent 接口生成的自定义事件对象。

浏览器原生提供CustomEvent()构造函数，用来生成 CustomEvent 事件实例。

```js
new CustomEvent(type, options)
```

- type：事件名称，必传。
- options: 配置对象，可选。配置对象除了接受 Event 事件的配置属性，只有一个自己的属性。

> detail：表示事件的附带数据，默认为null。

```js
var myEvent = new CustomEvent('myevent', {
  detail: {
    foo: 'bar'
  },
  bubbles: true, // 布尔值，可选，默认为false，表示事件对象是否冒泡。
  cancelable: false // 可选，默认为false，表示事件是否可以被取消，即能否用
});

el.addEventListener('myevent', function (event) {
  console.log('Hello ' + event.detail.foo);
});

el.dispatchEvent(myEvent);
```

## 参考

- [阮一峰javascript事件模型](https://javascript.ruanyifeng.com/dom/event.html#toc12)
- [JS每日一题javascript中的事件模型](https://mp.weixin.qq.com/s/avXtM79vyywVq6Gg6ui29A)