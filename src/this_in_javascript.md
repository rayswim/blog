# JavaScript关键字this

> 本文中JavaScript的运行环境均为浏览器

在JavaScript中，每当进入可执行代码时，变会创造一个可执行上下文（Execution Context）。可执行上下文也可以理解为当前代码的执行环境。在可执行上下文创建的过程中会分别生成变量对象，建立作用域链，确定this指向。因此我们能够得到一个结论，**this的指向是在函数执行时确定的**。所以在JavaScript中一个函数中的this指向是非常灵活的。this指向一旦确定后便不能再进行更改了。

this的指向是动态确定的，一般分为如下几种情况：

## 全局上下文
全局上下文，即全局环境中的this是一种特殊情况。无论是否处于严格模式，全局上下文的this总是指向全局对象。

```javascript
var a = 10;
console.log(this); // window
console.log(this.a); //10
```

## 函数上下文
可以这样说，处于函数中的this指向取决于函数的调用方式，也可以说函数中的this指向调用该函数的对象。

### 独立调用
在严格模式下，如果一个函数独立调用，那么函数中的this指向undefined；在非严格模式下指向window。
```javascript
function foo() {
	console.log(this === window)
}

foo(); // true
```

```javascript
'use strict'
function foo() {
	console.log(this === undefined);
}

foo(); // true
```

### 对象方法
如果函数是作为对象方法调用，那么函数中的this指向调用函数的对象。在这种情况下，无论函数是在什么地方定义均不影响函数中this的指向。
```javascript
var a = 1;
function foo() {
	console.log(this.a);
}

var obj = {
	a: 2,
	foo: foo
}

foo(); //1
obj.foo(); //2
```
如果函数是原型链上的方法，那么该函数中的this指向也指向调用该函数的对象。

```javascript
function Foo() {
	this.a = 1;
}

Foo.prototype.func = function() {
	console.log(this.a)
}

var foo = new Foo();
foo.func(); //1
```

### 构造函数
在JavaScript中通过关键字new将函数做为构造函数使用来创建对象时，如果该方法不返回任何值或者`返回值不是一个对象`，那么函数中的this指向新创建的对象实例，若显示返回一个对象，那么函数的this没有任何意义。
```javascript
function Foo1() {
	this.a = 1;
}

function Foo2() {
	this.a = 1;
	return 1;
}

function Foo3() {
	this.a = 1;
	return {b:1}
}

var foo1 = new Foo1();
var foo2 = new Foo2();
var foo3 = new Foo3();

console.log(foo1.a); //1
console.log(foo2.a); //1
console.log(foo3.a); //undefined
```
foo3.a之所以为undefined是因为foo3指向的是{b:1}这个对象。

### DOM事件回调函数的
如果一个函数是DOM事件处理函数时，函数中的this指向该DOM元素。
```javascript
//html
<button id="btn">

//javaScript
var btn = document.getElementById('btn');

function foo() {
	console.log(this === btn);
}
btn.addEventListener('click',foo);

btn.click(); //true
```

### 箭头函数
ES6中新增了箭头函数这个概念。箭头函数没有自己的this指向，其中的this是包含该箭头函数的函数的this。
```javascript
//ES6
function foo() {
	this.a = 1;
	return () => this.a
}
```

```javascript
//转码为ES5
function foo() {
	this.a = 1;
	var _this = this;
	return function() {
		return _this.a;
	}
}
```

## 显示指定this
在JavaScript中可以通过Function.prototype.apply Function.prototype.call显示指定函数的this指向。ES5里还引入了 Function.prototype.bind来对某方法绑定指定对象。其中不同的是bind方法返回绑定指定对象后的方法，而call和apply则是绑定指定对象后立即运行。
```javascript
var obj = {
	a: 1
}

function foo(){
	console.log(this.a)
}

var foo1 = foo.bind(a);

foo1(); //1
foo.call(obj); //1
foo.apply(obj); //1
```
如果一个函数做了bind操作后，无论再进行bind或者call还是apply都不会改变this指向，函数中的this只会指向第一次bind的对象。为了解释这个原因，我们可以写一个_bind来模拟原生的bind。

```
Function.prototye._bind = function(obj){
	var self = this;
	return function() {
		self.apply(obj,arguments)
	}
}
```
那么
```javascript
var obj1 = {a:1};
var obj2 = {a:2};

function foo(){
	console.log(this.a)
}

var foo1 = foo._bind(obj1)._bind(obj2)
```
等价于

```javascript
var foo1 = function() {
	foo.apply(obj1,arguments)
}._bind(obj2)
```
所以foo1后面不管bind多少次，都只会执行foo.apply(obj1,arguments)这个操作。

```javascript
foo1(); // 1
```

## 结束语
可以这样说，只有深刻理解JavaScript中的关键字this，才真正算是入门了。借住理解this，我们可以深入去了解JavaScript函数的执行环境，而这是理解闭包等其他概念的基础。
