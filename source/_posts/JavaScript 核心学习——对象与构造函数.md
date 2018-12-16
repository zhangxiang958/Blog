title: JavaScript 核心学习——对象与构造函数
date: 2016-05-13 15:26:24
categories: JavaScript

---
作为一门面向对象的语言，对象是核心中的核心，而又因为 JavaScript 是一门基于原型的面向对象的语言，所以
其中的机制不同于其他面向对象语言。本篇博文篇幅会有点长，讲述对象，构造函数的内容。
<!--more-->
##对象
在 JavaScript 中，对象是键值对的集合，它的属性值可以为数字，字符串，甚至是函数。在访问它们的属性的时候，
除了常规的 "." 操作符，还可以通过 "[]" 来访问属性，[] 里面可以使用变量。
###属性特性
这里所说的属性特性是可配置性 *Configurable*, 可读性 *Writable*, 可枚举性 *Enumberable* 。
```
Object.defineProperty(obj, prop, {
    value: "value",
    configurable: true,
    writable: false,
    enumberable: true
});
```
这里配置了一个可配置的，不可写，可枚举的数据属性，其值为 value 。如果想要删除某个对象属性，就应当使用
delete 命令，但是如果 configurable 特性设置为 false 的话，就不能被删除。并且如果设置 configurable 属性为
false, 那么就不能再配置该对象属性了。
###访问器属性
访问器属性是一对函数，[[Get]] 和 [[Set]]。
```
var obj = {
    a: 2
};
Object.defineProperty(obj, "b", {
    get: function(){
        return this.a;
    },
    set: function(val){
        this._a_ = val + this.a;
    }
});

console.log(obj.b);  //2
obj.b = 10;
console.log(obj.b);  //2
console.log(obj._b_);  //12
```
###可访问性与遍历
对于可访问性，如果我们设置一个属性的值为 undefinded，但是如果没有设置这个对象属性，返回值也是 undefinded，
如何区分两者呢？因此有了 hasOwnProperty() 检查本对象是否有这个属性。
```
var obj = {
    a: 2
};

obj.hasOwnProperty("a");  //true
obj.hasOwnProperty("b");  //false
Object.prototype.hasOwnProperty.call(obj, "a");  //true
```
*hasOwnProperty* 对比于 *in* 操作符，*hasOwnProperty* 只会检查本对象有没有，而 *in* 操作符则会检查包括本对象
的原型链中有没有这属性，又因为有一些对象没有进行 *propertype* 连接操作，因此可能会没有 *hasOwnProperty* 方法，
所以才有了第三个方法使用 *call* 方法调用。

对于遍历，遍历对象一般使用 in 操作符，使用 for..in 循环。in 操作符会将原型链中的所有可枚举属性全部遍历。
```
//遍历
for (var prop in obj) {
    console.log(prop);  //属性名
    console.log(obj[prop]);  //属性值
}
```
##构造函数
上面所说的创建一个对象，一般是使用字面量法，但是在实际中，如果每个对象都需要这样编写，代码量将会很庞大，
而且各个对象之间也没有什么联系，因此我们会使用构造函数来创建对象。
对于构造函数的命名，我们会将函数名的首字母大写，以区分构造函数与一般函数。
###工厂模式
```
function Person(name, age){
    var o = new Object();
    o.name = name;
    o.age = age;
    return o;
}
```
这种模式虽然创建了很多相似的对象，但是对象之间没有联系。
###构造函数模式
```
function Person(name, age) {
    this.name = name;
    this.age = age;
}

var person1 = new Person("Jarvis", 20);
var person2 = new Person("zhangxiang", 20);
```
这里构造函数 Person 通过 new 命令创建了两个对象,与工厂模式不同在于构造函数模式没有返回一个对象，将属性
赋给了 this 对象。对象之间的联系在于它们的 constructor 属性都指向了 Person，但是在深入理解上并不是这两个
对象的 constructor 属性，这里可以暂时先理解为这样。constructor 属性是构造器属性，可以浅显地理解为这样，
两个对象 person1 和 person2 都是被 Person 这个函数构造出来的。
构造函数模式是比较好，但是也存在缺点，比如浪费内存，一些共有的属性方法在创建对象的时候都要再创建一次。
虽然可以将一些方法写到全局中，让对象去引用，但是这样又污染了全局变量。所以出现了下面的原型模式。
关于 new 命令请阅读本人博文 [JavaScript 核心学习—— this 关键字](http://zhangxiang958.github.io/categories/JavaScript/)。
###原型模式
构造函数在创建的时候都已创建一个 *prototype* 属性，构造函数的 prototype 属性指向一个对象
```
function Person(){}
Person.prototype.name = "Jarvis";
Person.prototype.age = 20;

var person1 = new Person();
var person2 = new Person();
console.log(person1.name);  //"Jravis"
console.log(person2.name);  //"Jarvis"
```
关系图如下：图为 《JavaScript 高级程序设计》中的原图
![](http://7xns9g.com1.z0.glb.clouddn.com/how-prototypes-work.png)
实例对象中的 ____proto____ 属性指向构造函数的 prototype 。但是并不是所有浏览器都支持____proto____ 属性，
所以不推荐使用这个属性。
对于原型模式，如果在一个对象中没有找到这个属性，那么引擎就会往对象的原型 prototype 上去寻找，这个有点像
对象版的作用域链，即原型链，但是由于原型上的属性方法都是共用的，如果是一般属性还好，但是如果是像数组这样
的话，如果在一个实例中改动值，那么改动将会体现在全部的对象中，因此原型模式很少会单独使用。
###组合模式
组合模式，就是将构造函数与原型模式结合起来。将实例特有的属性方法放在构造函数中，将共有属性方法放到原型中。
```
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
}

Persn.prototype = {
    constructor: Person,  //为什么要设置 constructor 属性？因为采用字面量创建对象意味着重写，之前的属性都会被清理掉，因此要在字面量里面重新设置
    sayName: function(){
        console.log(this.name);
    }
}

var person1 = new Person("jarvis", 20, "student");
person1.sayName(); // "jarvis"
```
###寄生模式
如果想要创建一些特别的对象，比如一些特别的数组对象，但是直接在 Array.prtotype 上设置是不推荐的，因为那样
会造成维护困难，因此需要采用寄生模式。
```
function SpecialArray(){
    var array = new Array();
    array.push.apply(array, arguments);
    array.toPied = function(){
        return this.join(",");
    }
    return array;
}

var values = new SpecialArray();
values.toPied();
```
这样的好处是可以拓展对象的方法而不影响原生对象，但是这样不好之处在于这样创建出来的对象与构造函数之间没有
联系，与构造函数的 prototype 也没有关系，这样就和工厂模式的缺点是一样的。
###动态原型模式
动态原型模式的出现是为了解决构造函数与原型独立开来的问题的。
```
function Person(name, age){
    this.name = name;
    this.age = age;
    
    if(typeof this.sayName != "function") {
        Person.prototype.sayName = function(){
            console.log(this.name);
            return this.name;
        }
    }
}
```
这样的话，if 语句只会执行一次，并且直接在构造函数中完成对原型对象的增强。
###稳妥模式
```
function Person(name, age){
    var o = new Object();
    o.sayName = function(){
        console.log(name);
    };
    return o;
}
var person1 = Person("jarvis", 20);
person1.sayName();  //"jarvis"
```
这样的话，除了 sayName 这个方法之外，没有其他办法可以访问到构造函数中的数据，其实是用了闭包的知识。





