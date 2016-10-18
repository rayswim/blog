# JavaScript 原型解析

## null
物理学告诉我们宇宙最初始什么都没有，佛学告诉我们四大皆空，三国杀告诉我们可以无中生有，世间种种都阐述了一个道理，世界万物是从无到有。那么在javascript中也有一个空，那就是null。我们可以按照世间万物的发展规律，把javascript这个空间的起点约定为null。

## 起点
既然我们定了null位JavaScript中万物的起点，那么我们便来找一下从无到有这个过程中第一个产生的事物是什么。那么在JavaScript与null发生直接联系的是什么呢？
```javascript
Object.prototype.__proto__ === null //true
```
在JavaScript中`Object.prototype.__proto__`的值为null，那么`Object.prototype.__proto__`究竟是个什么鬼？这之前我们来了解下`prototype`和`__proto__`

## \__proto__

在JavaScript中，每个对象（除了null）都有一个隐藏属性[[prototype]]，目前主流浏览器将该属性通过\__proto\__暴露出来。对象中的\__proto\__指向构造该对象的构造函数的原型，也就是指向该对象的原型对象。这句话听着很绕口，用大白话讲就是\__proto\__是对象中的DNA，把每个对象看成人，那么你是由你爸的原型对象构造出来的，你的DNA指向了你爸，也可以说你的\__proto\__指向了你爸的原型对象。同理，你爸指向了你爷爷，你爷爷指向了....，也就是说通过\__proto\__可以访问到对象继承的所有的原型对象。

## prototype
JavaScript中，一切皆对象。那么函数就是对象，同时函数也是函数。这句话听着有点绕口，其实把对象比作女人，那么函数就是男人。男人和女人都有x染色体，男人还有y染色体。从生物学上讲男人的xy染色体是女人xx染色进化到一定阶段的产物，那么也可以说男人是女人，同时男人也是男人。那么对于每个男人，也就是每个函数实例，都有独有的y染色体，在JavaScript就是prototype，原型对象。  
函数的原型对象为`Function.prototype`,函数实例的原型对象为该函数的`prototype属性`。
```javascript
var Foo = function(){};
Foo.__proto__ === Function.prototype; // true
var a = new Foo();
a.__proto === Foo.prototype; //true
```
每个函数对象都有prototype这个属性，它是这个函数对象的原型对象。顾名思义，prototype指向的是个对象。在原型对象中有一个属性为`constructor`，指向的是构造函数的本身。
```javascript
Foo.prototype.constructor === Foo //true
```
既然prototype为对象，那么它也有隐藏的\__proto\__属性，对象的原型对象就是`Object.prototype`。
```javascript
Foo.prototype.__proto__ === Object.prototype; //true
```

## 鸡生蛋，蛋生鸡
到这里我们就明白了最开始的`Object.prototype.__proto__`的含义为Object.prototype的原型对象。该值为null，那么我们可以暂且认定JavaScript中第一个非空物为`Object.prototype`。又因为  
```javascript
Function.prototype.__proto__ === Object.prototype; //true
```
我们可以看到函数的原型对象的构造函数的原型是Object.prototype，我们之前说只有函数才有prototype，那么Object的原型对象又是Function.prototype 。  
```javascript
Object.__proto__ === Function.prototype; //true
```
这俨然又成了一个鸡生蛋，蛋生鸡的问题。其实我们可以这样理解：
```javascript
var Object = function() {...};
var Function = function() {...};
```
世上先有了Object.prototype这个东西，Object是个函数，其中它的prototype属性被上帝手动指向了Object.prototype。是时候祭出下图了：  
![https://pic2.zhimg.com/e83bca5f1d1e6bf359d1f75727968c11_b.jpg](https://pic2.zhimg.com/e83bca5f1d1e6bf359d1f75727968c11_b.jpg)  
我们可以看到：Object、Function、Foo都可以看做为构造函数，其\__proto\__指向了Function.prototype，所有对象的\__proto\__最终都指向了Object.prototype，Object.prototype的constructor指向了function Object()，Object.prototype.\__proto\__指向了null。

## 原型链
由上图我们可以看到每个构造函数都有一个原型对象`prototype`，每个由构造函数生成的对象都有一个`__proto__`指向构造函数的原型对象，这样每个对象通过`__proto__`找到其原型对象`prototype`，这样不断的层层递进最终找到了null。JavaScript在寻找某个对象的属性的时候，如果在该对象内没有找到，变会去该对象的原型对象中找，如果仍没找到，就去该对象的原型的原型去找，直到找到或者到null为止，这样都就成了以`__proto__`为桥接的原型链。

## 原型继承

```javascript
var human = function(arg) {
	this.type = '人';
	this.test = arg;
}

human.prototype.getType = function() {
	console.log(this.type);
}

var male = function() {
	this.jj = true;
}

male.prototype = new human();

male.prototype.constructor = male;

new male().getType();
```

根据原型链我们能够实现原型继承，上述代码为原型继承的一个最基础的例子。里面最为核心的一句为`
male.prototype = new human()`。将male的原型对象手动设置为human对象，这样male的实例的`__proto__`便指向了human的实例，同时human的实例的`__proto__`指向了`human.protoype`，这样male的实例就继承了human的属性type和getType方法。
