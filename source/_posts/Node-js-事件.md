---
title: Node.js 事件
date: 2021-06-07 10:31:03
categories: Node.js
tags: 
	- 事件
---

## Node.js 事件的基本用法

```js
const EventEmitter = require('events').EventEmitter;

// 实例化
let life = new EventEmitter();

let water = who => {
	console.log(`给${who}倒水`);
};

// 监听事件
life.on('comfort', water);

life.on('comfort', who => {
	console.log(`给${who}按摩`);
})

// 触发事件
const hasComfortEventListener = life.emit('comfort', 'will');
console.log(hasComfortEventListener);

// 移除事件
life.removeListener('comfort', water); // 移除单个事件
life.removeAllListeners('comfort'); // 移除一类事件

// 监听事件的个数
console.log(life.listeners('comfort').length);

// 设置事件监听数的最大值
life.setMaxListeners(10);
```
