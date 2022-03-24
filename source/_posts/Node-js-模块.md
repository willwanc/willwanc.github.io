---
title: Node.js模块
date: 2020-05-14 10:20:50
categories: Node.js
tags: 
	- 模块化
	- CommonJs 模块机制
	- ES6 模块与 CommonJS 模块的差异
---

Node.js 使用 CommonJs 模块规范。CommonJs 规范由JavaScript社区发起，在Node.js上应用并得到推广。

## Node.js 模块分类

- 核心模块（在node启动时会预先加载）
- 非核心模块 （包括：文件模块、第三方模块）


## 模块引用

- 通过路径

	```js
  const promise = require('../fileName.js'); 
  ```

- 通过模块名

  ```js
  const promise = require('http');
  ```


## 模块化

### 1. 创建和定义模块

	在 Node.js 中，只需要创建一个js文件就创建了一个模块，通过在 exports 变量上添加属性和方法就可以在向外部暴露想要暴露的数据和方法。

	```js
	// utils.js
	exports.sum = function (a, b) {
		return a + b;
	};
	
	exports.msg = 'hello world';
	```

   

### 2. 使用模块

	我们可以通过 require 方法来引用一个模块，然后就可以使用这个模块暴露出来的数据和方法。

	```js
	// index.js
	const utils = require('./utils.js');
	
	const res = utils.sum(1, 2);
	console.log(res); // 输出 3
	```
	
	

## CommonJs 模块机制

### 1. 模块默认导出对象字面量 {}

	```js
	// utils.js
	console.log(exports); // 输出 {}
	```

	```js
	// index.js
	const utils = require('./utils.js');
	console.log(utils); // 输出 {}
	```
   
   

### 2. 修改模块默认导出数据为其他类型

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
  const utils = require('./utils');
  
  console.log(utils); // 输出：[Function: hello]
  ```

  上述代码说明 module.exports 覆盖了默认导出的 exports 变量，但是原来默认的 exports 变量在模块内部还存在。

### 3. 外部require调用的结果和模块内部的exports对象是同一个引用，所以在模块外部可以修改模块暴露出来的对象

	例如：

	```js
	// index.js
	const utils = require('./utils');
	
	utils.additional = '模块的使用者添加的属性';
	
	utils.sum = '改变模块内部原有的属性';
	
	```
	
	```js
	// utils.js
	exports.sum = function (a, b) {
		return a + b;
	};

	setTimeout(() => {
		console.log(exports); // { sum: '改变模块内部原有的属性', additional: '模块的使用者添加的属性' }
	}, 2000);	
	```
	
	在终端运行 index.js，输出如下：
	
	```bash
	node index.js
	{ sum: '改变模块内部原有的属性', additional: '模块的使用者添加的属性' }。
	```
	
	这就说明，在模块外部，可以修改模块暴露出来的对象。


### 模块导出的是模块内部值的拷贝

```js
// utils.js
let count = 1;
const setCount = (num) =>{ 
  count = num;
  console.log(count);
}

module.exports = {
	setCount: setCount,
	count: count
};


// index.js
const utils = require('./utils');
console.log(utils.count); //输出 1
utils.setCount(2); //输出 2 （内部的count被修改为2）
console.log(utils.count); //输出 1 （暴露出来的count依然是1）
```

上述代码表明，模块一旦导出一个值，模块内部的变化不会再影响这个值。这就说明模块导出的是模块内部值的拷贝。

#### 通过定义get函数可以得到内部值的改变

```js
// utils.js
let count = 1;
const setCount = (num) =>{ 
  count = num;
  console.log(count);
}

module.exports = {
	setCount: setCount,
	get count() {
		return count;
	}
};

// index.js
const utils = require('./utils');
console.log(utils.count); //输出 1
utils.setCount(2); //输出 2 （内部的count被修改为2）
console.log(utils.count); //输出 2
```

### commonjs模块机制和ES模块机制的区别

1. **commonjs模块导出的是模块内部值的拷贝，ES6模块导出的是值的引用**

```js
// utils.js
let count = 1;
const setCount = (num) =>{ 
  count = num;
  console.log(count);
}

export {
	setCount,
	count
};

// index.js
import * as utils from ('./utils.js');
console.log(utils.count); //输出 1
utils.setCount(2); //输出 2 （内部的count被修改为2）
console.log(utils.count); //输出 2

// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script type="module" src="./index.js"></script>
</body>
</html>
```

注意：index.html需要在服务器上测试，如果本地浏览器直接打开会报跨域错误，并且引入 `index.js` 的 script 标签要添加 `type="module"` 属性。

2. commonJS模块是运行时加载模块，ES6模块是编译时编译时输出接口。
  
因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。

3. CommonJS 模块的require()是同步加载模块，ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。

## 参考：

- [ES6 模块与 CommonJS 模块的差异](https://es6.ruanyifeng.com/#docs/module-loader#ES6-%E6%A8%A1%E5%9D%97%E4%B8%8E-CommonJS-%E6%A8%A1%E5%9D%97%E7%9A%84%E5%B7%AE%E5%BC%82)