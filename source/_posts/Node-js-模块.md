---
title: Node.js模块
date: 2020-05-14 10:20:50
categories: Node.js
tags: 
	- 模块化
	- CommonJs
---

Node.js 使用 CommonJs 模块规范。CommonJs 规范由JavaScript社区发起，在Node.js上应用并得到推广。

## Node.js 模块分类

- 核心模块（在node启动时会预先加载）
- 非核心模块 （包括：文件模块、第三方模块）

## 模块引用

- 通过路径

	```js
  var promise = require('../fileName.js'); 
  ```

   

- 通过模块名

  ```js
  var promise = require('http');
  ```



## 模块化

1. 创建和定义模块

	在 Node.js 中，只需要创建一个js文件就创建了一个模块，通过在 exports 变量上添加属性和方法就可以在向外部暴露想要暴露的数据和方法。

	```js
	// utils.js
	exports.sum = function (a, b) {
		return a + b;
	};
	
	exports.msg = 'hello world';
	```

   

2. 使用模块

	我们可以通过 require 方法来引用一个模块，然后就可以使用这个模块暴露出来的数据和方法。

	```js
	// index.js
	var utils = require('./utils.js');
	
	var res = utils.sum(1, 2);
	console.log(res); // 输出 3
	```
	
	

## CommonJs 模块机制

- 模块默认导出对象字面量 {}

	```js
	// utils.js
	console.log(exports); // 输出 {}
	```

	```js
	// index.js
	var utils = require('./utils.js');
	console.log(utils); // 输出 {}
	```
   
   

- 修改模块默认导出数据为其他类型

  通过给 module.exports 赋值可以修改模块导出的数据为字符串、函数等其他任何类型。

  例如：

  ```js
  // utils.js
  exports.sum = function (a, b) {
  	return a + b;
  };
  
  module.exports = function hello() { 
  	console.log('hello');
  }
  
  setTimeout(() => {
  	console.log(exports); // 输出： { sum: [Function] }
  }, 200);
	```

  ```js
  // index.js
  var utils = require('./utils');
  
  console.log(utils); // 输出：[Function: hello]
  ```

  上述代码说明 module.exports 覆盖了默认的 exports 变量，但是原来默认的 exports 变量在模块内部还存在。

- 模块内部的 exports 变量和模块的使用者 require 的对象是同一个引用

	例如：

	```js
	// index.js
	var utils = require('./utils');
	
	utils.additional = '模块的使用者添加的属性';
	
	utils.sum = '改变模块内部原有的属性';
	
	setTimeout(() => {
		console.log(exports); // { sum: '改变模块内部原有的属性', additional: '模块的使用者添加的属性' }
	}, 200);	
	
	```
	
	```js
	// utils.js
	exports.sum = function (a, b) {
		return a + b;
	};
	```
	
	在终端运行 index.js，输出如下：
	
	```bash
	node index.js
	{ sum: '改变模块内部原有的属性', additional: '模块的使用者添加的属性' }。
	```
	
	这就说明，在模块外部，可以修改模块暴露出来的对象。

