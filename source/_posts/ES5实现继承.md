---
title: ES5实现继承
tags: [继承]
categories: [javascript]
date: 2021-12-17 16:39:31
---

## 原型链继承

把 A 类的实例赋值给 B 类的原型对象，就通过原型链实现了 B 继承了 A 类。
通过原型链继承，B 类可以继承 A 类的原型对象上的属性和方法，以及 A 类的实例的属性和方法。
例如：

```js
// 声明父类
function SuperClass() {
  this.superValue = true;
}

// 为父类添加共有方法
SuperClass.prototype.getSuperValue = function () {
  return this.superValue;
};

// 声明子类
function SubClass() {
  this.subValue = false;
}

// 继承父类
SubClass.prototype = new SuperClass();

// 为子类添加共有方法
SubClass.prototype.getSubValue = function () {
  return this.subValue;
};

var sub_1 = new SubClass();

// 父类实例的属性
console.log(sub_1.superValue); // true

// 父类原型对象的方法
console.log(sub_1.getSuperValue()); // true
```

### 原型链继承的原理

首先，父类构造函数的实例拥有父类的实例的属性和方法。此外，这个实例的原型属性 `_proto_` 指向父类的原型对象，这个实例就能通过原型属性 _proto_ 访问到原型对象上的属性和方法。 然后，把这个实例赋值给子类的原型对象，而子类的原型对象是子类的所有实例所共享的，这样子类就继承了：父类的原型对象上的属性和方法，以及父类实例的属性和方法。这就是原型链继承的原理。

### 原型链继承的问题

注意：由于子类的原型对象被赋值为父类的实例，父类的实例的原型属性指向父类的原型对象，而这个原型对象的 constructor 指向父类的构造函数，所以子**类实例的 constructor 指向了父类的构造函数 SuperClas**s。

```js
// 子类实例的 constructor
console.log(sub_1.constructor); // 指向父类的构造函数 SuperClass 继承原理
```

### 判断原型和实例的关系

通过 instanceof 操作符可以检测某个对象是否是某个构造函数的实例，具体来说**instanceof 是通过检测对象的原型链上是否有某个构造函数的原型对象**。

例如：

```js
console.log(sub_1 instanceof SubClass); // true
console.log(sub_1 instanceof SuperClass); // true
console.log(sub_1 instanceof Object); // true
console.log(SubClass instanceof SuperClass); // false
console.log(SubClass.prototype instanceof SuperClass); // true
```

### 原型链继承的缺点

1.引用类型属性共享问题

我们知道原型链继承是通过把父类的实例赋值给子类是的原型对象实现的，而子类的原型对象中的属性和方法是子类的所有实例所共享的，所以原型对象中的属性是引用类型数据被子类实例所共用。如果子类的某个实例修改个这些引用类型数据，子类的其他实例也会受到影响。

例如：

```js
function SuperClass() {
  this.books = ["html", "css", "js"];
}

function SubClass() {}

SubClass.prototype = new SuperClass();

var sub_1 = new SubClass(),
  sub_2 = new SubClass();

console.log(sub_2.books); // ["html", "css", "js"]
sub_1.books.push("设计模式");
console.log(sub_2.books); // ["html", "css", "js", "设计模式"]
```

2.无法在实例化子类时向父类传递参数

由于上述的两个问题，实践中很少单独使用原型链继承。

## 借用构造函数实现继承

为了解决原型链继承的问题，出现了借用构造函数继承的技术。在新创建的构造函数内部，利用 call()或 apply()方法调用父类的构造函数，就实现了构造函数式继承。

例如：

```js
function SuperClass(id) {
  this.books = ["html", "css", "js"];
  this.id = id;
}

function SubClass(id) {
  // 调用父类的构造函数
  SuperClass.call(this, id);
}

var sub_1 = new SubClass(10),
  sub_2 = new SubClass(20);

console.log(sub_2.books); // ["html", "css", "js"]
sub_1.books.push("设计模式");
console.log(sub_2.books); // ["html", "css", "js"]

console.log(sub_1.id); // 10
console.log(sub_2.id); // 20 借用构造函数继承的问题
```

### 借用构造函数继承的问题

**只能继承父类的实例的属性和方法，不能继承父类原型对象中定义的属性和方法**，所以借用构造函数继承也很少单独使用。

## 组合继承

组合继承是指把原型链继承和借用构造函数继承结合在一起的继承模式。实现思路是：利用原型链实现对原型属性和方法的继承，通过借用构造函数实现对实例属性的继承。

例如：

```js
function SuperClass(name) {
  this.name = name;
  this.colors = ["red", "green", "blue"];
}

SuperClass.prototype.sayName = function () {
  console.log(this.name);
};

function SubClass(name, age) {
  // 继承实例属性
  SuperClass.call(this, name); // 第二次调用父类构造函数

  this.age = age;
}

// 继承原型方法
SubClass.prototype = new SuperClass(); // 第一次调用父类构造函数
SubClass.prototype.constructor = SubClass; // 修正子类的 constructor

// 添加子类原型方法
SubClass.prototype.sayAge = function () {
  console.log(this.age);
};

var sub_1 = new SubClass("wan", 26),
  sub_2 = new SubClass("li", 28);

sub_1.colors.push("black");
console.log(sub_1.colors); // ["red", "green", "blue", "black"]
console.log(sub_2.colors); // ["red", "green", "blue"]

sub_1.sayName(); // wan
sub_1.sayAge(); // 26
sub_2.sayName(); // li
sub_2.sayAge(); // 28
```

组合继承避免了原型链和借用构造函数继承的缺陷，融合了他们的优点，是 JavaScript 中常用的继承模式。

### 组合继承的问题：调用两次父类构造函数

上面例子中，第一次调用父类构造函数，会在子类的原型上添加 name 和 colors 属性；第二次调用父类构造函数，又会在子类的实例对象上添加 name 和 colors 属性，这显然是多余的。

## 原型式继承

原型式继承是道格拉斯·克罗克福德于 2006 年提出的一种继承模式。它的思想是：不必创建自定义类的情况下，借助原型基于已有的对象创建对象。文字不太好理解，看一下它的函数实现：

```js
function inheritObject(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

### 继承原理

原型式继承的实现过程是：创建一个临时的构造函数，然后把要继承的对象赋值给构造函数的原型，最后返回构造函数的一个实例对象。因为构造函数的原型是要继承的对象，返回的构造函数的实例自然继承了它要继承的对象。原型式继承本质是生成一个以继承对象为原型的对象。

应用示例：

```js
var wan = {
  name: "wan",
  friends: ["zhang", "wang"],
};

var li = inheritObject(wan);
li.name = "li";
li.friends.push("zhao");

var ban = inheritObject(wan);
ban.name = "ban";
ban.friends.push("lu");

console.log(wan.friends); // ["zhang", "wang", "zhao", "lu"]
```

原型式继就是以一个对象作为基础，通过继承函数 inheritObject 创建被继承函数的副本，然后根据具体需求对得到的副本对象加以修改。

### Object.create 方法

ECMAScript5 通过新增 Object.create()方法规范了原型式继承。

> Object.create(proto, propertiesObject) 创建一个新对象，使用现有的对象来作为新创建的对象的**proto**。

- proto 新创建对象的原型对象。
- propertiesObject 可选。要添加到新创建对象的属性。

例如：

```js
var wan = {
  name: "wan",
  friends: ["zhang", "wang"],
};

var li = Object.create(wan, {
  name: {
    value: "li",
  },
});

console.log(li.name); // li
```

注意：Object.create(proto, propertiesObject)兼容 IE9+。

### 原型式继承的问题

原型式继承跟原型链继承一样，是通过构造函数的原型实现的继承，所以原型式继承也有引用类型属性共享问题。

## 寄生式继承

寄生式继承的思路是封装一个仅用于实现继承过程的函数，在该函数的内部来增强对象，最后返回该对象。之所以称作寄生式继承，是因为实现继承过程的函数内部是借助其他函数实现的继承。

寄生式继承示例：

```js
var wan = {
  name: "wan",
  friends: ["zhang", "wang"],
};

function createPerson(original) {
  var person = inheritObject(original);
  person.sayHi = function () {
    alert("Hi");
  };

  return person;
}

var personA = createPerson(wan);
personA.sayHi();
```

注意：示例中借用的继承函数 inheritObject 不是固定的，任何能够返回新对象的函数都可以用于寄生式继承模式。

## 寄生组合式继承

**所谓寄生组合式继承，就是通过借用构造函数继承父类的实例属性，通过寄生式继承来继承父类的原型。**

寄生式继承父类原型的实现如下：

```js
function inheritPrototype(subType, superType) {
  var prototype = Object.create(superType.prototype); //创建对象
  prototype.constructor = subType;
  subType.prototype = prototype; // 指定对象
}
```

接下来，用 inheritPrototype()函数替换组合继承中为子类原型赋值的语句就实现了寄生组合式继承。代码如下：

```js
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "green", "blue"];
}

SuperType.prototype.sayName = function () {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name, age);

  this.age = age;
}

inheritPrototype(SubType, SuperType); // 继承父类原型

SubType.prototype.sayAge = function () {
  console.log(this.age);
};
```

## 总结

寄生组合式继承是借用构造函数和寄生式继承结合实现继承的方式，是最理想的继承实现方式。
