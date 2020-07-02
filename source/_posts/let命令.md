---
title: let命令
date: 2020-07-02 14:08:46
tags: ['']
categories: ['ES6']
---

## 基本用法

ES6 新增，用于声明变量。用法类似于 var，只在 let 所在的代码块内有效。

## 声明 for 循环的计数器时适合使用 let 命令

1.  可以避免污染 for 循环外部作用域

```js
for (let i = 0; i < 10; i++) {
	// ...
}

console.log(i);
// ReferenceError: i is not defined
```

上面代码中，计数器 i 只在 for 循环体内有效，在循环体外引用就会报错。

2.  计数器 i 只在本轮循环有效

```js
var a = [];
for (let i = 0; i < 10; i++) {
	a[i] = function () {
		console.log(i);
	};
}
a[6](); // 6
```

上面代码中，变量 i 只在本轮循环有效，所以每一次循环的 i 其实都是一个新的变量，所以最后输出的是 6。JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 i 时，就在上一轮循环的基础上进行计算。

3.  另外，for 循环设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域

```js
for (let i = 0; i < 3; i++) {
	let i = 'abc';
	console.log(i);
}
// abc
// abc
// abc
```

上面代码正确运行，输出了 3 次 abc。这表明函数内部的变量 i 与循环变量 i 不在同一个作用域，有各自单独的作用域。

## 不存在变量提升

var 命令会发生“变量提升”现象，即变量可以在声明之前使用，值为 undefined。

为了纠正这种现象，let 命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

```js
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

## 暂时性死区

只要块级作用域内存在 let 命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```js
var tmp = 123;

if (true) {
	tmp = 'abc'; // ReferenceError
	let tmp;
}
```

在代码块内，使用 let 命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

总之，暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

## 不允许重复声明

let 不允许在相同作用域内，重复声明同一个变量。

```js
// 报错
function func() {
	let a = 10;
	var a = 1;
}

// 报错
function func() {
	let a = 10;
	let a = 1;
}
```

因此，不能在函数内部重新声明参数。

```js
function func(arg) {
	let arg;
}
func() // 报错

function func(arg) {
	{
		let arg;
	}
}
func() // 不报错
