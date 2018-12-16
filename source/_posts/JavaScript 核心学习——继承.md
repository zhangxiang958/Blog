title: JavaScript 核心学习——继承
date: 2016-05-18 19:37:24
categories: JavaScript

---
本篇博文讲述如何在 JavaScript 中实现继承，以及原型与原型链的知识，在附录中将会讲述 JavaScript 面向对象
的常见错误。
<!--more-->
##原型与原型链
在 JavaScript 中，使用类将会付出很大的代价，因此我们使用原型。原型不同于类，类代表复制。而 Javascript 使用
的是委托行为。
###原型与原型链
JavaScript 中有一个 [[prototype]] 属性，指向对应的原型对象。在一个对象中查找一个属性，如果本对象中有此
属性，那么就会读取本对象的属性，如果本对象中没有，则会往对象的 prototype 对象去找，这个有点像原型对象版的
作用域链。举个例子：
```
function Parent(){
    this.name = "parent";
}

Parent.prototype.sayParentName = function(){
    console.log(this.name);
}

function Child(){
    this.type = "child";
}

Child.prototype = new Parent();  //继承关联到 Parent.prototype，但不建议使用 new 命令
Child.prototype.isChild = function(){
    if(this.type == "child") {
        console.log(true);
    }
}

var person = new Child();
person.isChild();  //true
person.sayParentName();  //"parent"
```
构建一个名为 person 的对象实例，但是原本并没有在 person 中设置 isChild 方法，这个方法的引用是通过查找对象
的 prototype ，引用了 isChild 方法。而对于对象的 sayParentName 方法，是因为 Child.prototype 继承了 Parent.
prototype 对象，因此在 person 中没有找到 sayParentName, 转而查找 Child.prototype ，发现 Child.prototype 上
也没有，查找 Parent.prototype，查找到了 sayParentName 方法使用。

缺一个说明代码中的关系的原型图
而对于对象的原型链来说，无论它是哪种类型的对象(函数，数组等等)，它们都是对象，既然是对象，那么就会有原型
链，对于所有对象来说，一般它们的原型链尽头是 Object.prototype，即是说定义在 Object 上面的方法，对象都可以
使用。
###属性屏蔽
在对象读取属性与设置属性时都会发生属性屏蔽，如果本对象有该属性，则不会向原型链中查找属性，但是如果是设置，
情况就会有所不同。在设置属性的时候，分为 3 种情况。如果本对象中有这个属性，那么就会直接覆盖，但是如果这个
对象中没有，那么就会向原型链中查找，如果属性名既存在本对象又存在原型链中，那么就会发生屏蔽。属性选择总是
会选择最底层的那个属性。
```
obj.foo = "bar";
```
但是如果属性存在于原型链的上层时，有三种情况：
1. 如果上层的属性为数据属性并且不是只读的，那么就会在这个对象中添加一个屏蔽属性。
```
function Foo(){}

Foo.prototype = {
    constructor: Foo,
    a: 1
}

var obj = new Foo();
obj.a = 2;
console.log(obj.hasOwnProperty("a"));  //true
console.log(obj.a);  // 2
console.log(obj.__proto__.a);  // 1
```
2.如果上层属性存在该属性，但是它为只读，那么就无法修改属性或在对象中添加属性，在严格模式会抛出错误。
```
function Foo(){}

Foo.prototype = {
    constructor: Foo,
}
Object.defineProperty(Foo.prototype, "a", {
    value: 1,
    writable: false
});

var obj = new Foo();
obj.a = 2;
console.log(obj.a);  //1
console.log(obj.hasOwnProperty("a"));  //false
```
3.如果上层存在该属性，并且该属性是访问器属性 [[Set]], 那么就会调用这个 set 函数，属性不会添加到该对象上。
```
function Foo(){}

Object.defineProperty(Foo.prototype, "a", {
    set: function(value){
        return value + 1;
    }
});

var obj = new Foo();
obj.a = 1;
console.log(obj.a);  //undefined
console.log(obj.hasOwnProperty("a")); //false
```

如果想要查看某个实例是否是某个原型的实例，使用 *isPrototype()* 。
##实现继承
继承的本质在于重写原型对象，往往通过重写构造函数的原型对象去实现继承。
###利用原型链
```
function Parent(){}

Parent.prototype = {
    constructor: Parent,
    a: 1,
    b: [1, 2]
}

function Son(){}

Son.prototype = new Parent();  //注意使用这个方法继承后不能采用字面量添加方法，否则会将原型对象重写，失去与其他对象的联系
Son.prototype.getValue = function(){  
    return this.a;
}
var obj1 = new Son();
var obj2 = new Son();
console.log(obj1.a);  //1
console.log(obj1.getValue());  //1
obj2.b.push(3); 
console.log(obj1.b);  //[1, 2, 3]
```
利用原型链确实可以实现继承，但是缺点在于对于象数组这样的属性的时候，每个实例共享属性，如果在一个实例中
修改，那么在所有实例中都会变化，缺乏安全性。所以有了接下来的借用构造函数。
###借用构造函数
```
function Parent(){
    this.b = [1, 2];
}

function Son(){
    Parent.call(this);
}

var obj1 = new Son();
var obj2 = new Son();
obj1.b.push(3);
console.log(obj1.b);
console.log(obj2.b);
```
这样使用 call 方法改变 this 指向，借用构造函数确实可以解决每个实例的属性共用问题，但是有些属性并不需要每
个实例单独拥有，但是每次调用都会为每个实例创建属性，代码复用无从谈起，因此有了组合模式。
###组合继承
```
function Parent(){
    this.b = [1, 2];
}

function Son(value){
    Parent.call(this);
    this.a = value;
}

var obj1 = new Son(0);
var obj2 = new Son(1);
obj1.b.push(3);
console.log(obj1.a);
console.log(obj2.a);
console.log(obj1.b);
console.log(obj2.b);
```
组合继承的核心在于通过 call 方法构造实例独有的属性，并且提高了代码复用率。需要注意的是这里的组合继承与构造
函数的组合模式并不相同，组合继承是其他对象与本对象的联系，而组合模式是构造单个对象时采用的策略。
组合继承是最常用的继承方式。
[对象的构造函数](http://zhangxiang958.github.io/2016/05/13/JavaScript%20%E6%A0%B8%E5%BF%83%E5%AD%A6%E4%B9%A0%E2%80%94%E2%80%94%E5%AF%B9%E8%B1%A1%E4%B8%8E%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/)
###原型继承
```
function object(o) {  //Object.create 其实内部类似于这样
    function Foo(){}
    Foo.prototype = o;
    return new Foo();
}

var obj = {
    name: "obj",
    connect: [1,2]
}

var foo1 = object(obj);
var foo2 = object(obj);
foo1.connect.push(3);
console.log(foo1.connect);  // [1, 2, 3]
console.log(foo2.connect);  // [1, 2, 3]
```
原型继承在内部创建一个临时的构造函数，内部其实是对传入的参数进行了一次浅复制。如果想要让一个对象与另一个
对象保持类似，原型式继承完全可以满足，而不用去创建组合模式。而 ES5 对原型模式规范化后更为简单，提供了一个
Object.create() 方法，该方法接受两个参数，第一个是基对象，第二个是重写属性集合。
```
Object.create(obj, {
    name: {
        value: "jarvis"
    }
});
```
###寄生继承
```
function createObject (original){
    var clone = object.create(original);  //创建（复制）一个对象
    clone.sayHi = function (){    //以某方式增强对象
        console.log("hi");
    };
    return clone;    //返回对象
}
```
这种方式通过封装，在内部增强对象并返回对象。
###寄生组合继承
```
function Parent (){
    this.a = 1;
    this.b = [1,2];
}

Parent.prototype.sayA = function (){
    console.log(this.a);
}

function Son (){
    Parent.call(this);
    this.c = 2;
}

function clonePrototype (sub, sup){
    var prototype = Object.create(sup);
    prototype.constructor = sub;
    sub.prototype = prototype;
}

clonePrototype (Son, Parent);

Son.sayC = function (){
    console.log(this.c);
]
```
对比于组合模式，组合模式在构建 Son 原型对象和 Son 的实例对象的时候，调用了两次 Parent 函数，导致在 Son 
原型中也拥有一份 Parent 的属性，但是在实际中，这个并不需要，因为如果不重写，意味着 Son 的原型与 Parent 的
属性相同，但是这个本来就是要通过继承来解决的，因此寄生组合模式就是解决这个问题的，内部的 Object.create 
只是将 Son 的原型对象链接到 Parent 的原型对象。
##附录：常见的理解错误
1. 实际上，在 JavaScript 中的继承并不叫继承，因为继承代表着复制，但是在 JavaScript 中继承并没有复制操作，而是 **委托** 。JavaScript 中的继承实际上是通过这个对象与另外一个对象进行关联，之间并没有明确的父子类的关系。
2. 在实现继承的时候，推荐使用 Object.create 这个方法，虽然会导致轻微的性能损失（遗弃对象的垃圾回收），但是
相比与 new 命令，免去了创建很多不必要的属性。
3. 检查类的关系，检查对象 a 与对象 b 的关联性的话，推荐使用 a.isPrototypeOf(b) 这个方法。这个的意思是 a 
是否出现在 c 的原型链中，Object.getPrototypeOf() 这个方法可以获取对象的整个原型链。
4. 对象内置的 **.____proto____** 属性其实是一个 getter/setter 函数。
```
Object.defineProperty(object.prototype, "__proto__", {
    get: function(){
        return object.getPrototypeOf(this);
    },
    set: function(o){
        Object.setPrototypeOf(this, o);
        return o;
    }
});
```
5.*Object.create(null)* 会创建 [[prototype]] 链接为空的对象，这样的对象被称为 字典 ，适合用来存储数据。
6.在 ES5 之前的环境中，实现 Object.create() 。
```
if(!Object.create) {
    Object.create = function(o){
        function F(){};
        F.prototype = o;
        return new F();
    }
}
```


