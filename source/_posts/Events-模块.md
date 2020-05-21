---
title: Events 模块
date: 2020-05-21 15:34:08
tags: 
	- NodeJs事件
categories:
	- Node.js
---

## EventEmitter类

events提供的对象 events.EventEmitter，封装了事件触发和事件监听器的功能。

#### 监听和触发事件

```js
// index.js
const EventEmitter = require('events').EventEmitter;

// 实例化
let life = new EventEmitter();

let water = who => {
	console.log(`给${who}倒水`);
}

// 监听事件
life.on('comfort', water)

// 触发事件
const hasComfortEventListener = life.emit('comfort', 'will');

// emit()方法返回是否添加了对应的事件监听器
console.log(hasComfortEventListener); // true

```

在 终端运行上述 js 文件：

```bash
$ node index.js
给will倒水
true
```

#### 移除事件

```js
life.removeListener('comfort', water); // 移除单个事件
life.removeAllListeners('comfort'); // 移除一类事件
```

#### 监听事件的个数

```js
const count1 = EventEmitter.listenerCount(life, 'comfort');
const count2 = life.listeners('comfort').length;
```

#### 设置事件监听数的最大值

```js
life.setMaxListeners('comfort');
```




