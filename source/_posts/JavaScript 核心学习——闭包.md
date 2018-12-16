title: JavaScript 核心学习——闭包
date: 2016-04-26 18:17:24
categories: JavaScript

---
闭包是 JavaScript 中的核心概念，它的用途广泛，因此这里特地深入学习。
<!--more-->
##什么是闭包
闭包其实是 JavaScript 中极为有用的知识。但是它的真实面貌是什么呢？
闭包，使用通俗的话来说，就是一个函数内部的函数，是外界与函数内部之间的桥梁，但是准确地来说，这个解释不对。
闭包其实真实含义是保持对函数作用域的引用，这个引用叫做闭包。
我们知道外部作用域是不可以访问内部作用域的，详情请看本人的作用域博文，请点击下面的链接。至于为什么闭包
拥有访问函数内部的权利，是因为函数在执行的时候，创建了一个活动对象，并将这个活动对象推到作用域链的前端，
而函数内部的函数，因为定义在函数内部，拥有了包含了它的函数的作用域链，因此拥有访问权利，如果没有进行销毁
处理的话，作用域中的变量将会一直存放在内存中。
```
//一个简单的闭包使用例子
function foo () {
    var a = 1;
    return function () {
        console.log(a);
        return a;
    }
}

var value = foo()();
console.log(value);
```
但是不必局限于闭包的形式，其实如果从外部使用到了函数内部的变量，就是使用了闭包。其实我们写过很多闭包函数，
只是当时还不知道它叫做闭包而已，象事件处理函数，ajax，setTimeout/setInterval等等都使用了闭包。
下面来看一个常见的错误：
```
for(var i = 0; i < 3; i++) {
    setTimeout( function() {
        console.log(i);
    }, 1000);
}
```
上面的这个例子，原本是祈祷打印出 0,1,2 的，但是真实结果是打印出 3 个 3，为什么呢？因为 JavaScript 除了函数
体以外没有块级作用域，因此定时器引用的变量 i 都是同一个变量，i 变量在循环结束的时候，已经变为了 3，因此打
印出 3 个 3。所以改进方法就是使用闭包。
```、
for (var i = 0; i < 3; i++) {
    setTimeout((function (num){
        console.log(num);
    })(i), 1000);
}

//或者如下
for(var i = 0; i < 3; i++) {
    (function() { //IIFE
        var j = i;
        setTimeout(function() {
            console.log(j);
        },1000);
    })();
}
```

 [JavaScript 作用域学习笔记](http://zhangxiang958.github.io/2016/03/30/JavaScript%20%E6%A0%B8%E5%BF%83%E5%AD%A6%E4%B9%A0%E2%80%94%E2%80%94%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE/)
##闭包有什么用
闭包是 JavaScript 中很强大的机制，如果我们能够善用它，那么将会使我们功力大涨。闭包是对函数封闭的作用域的
引用，如果闭包能与块级作用域结合，那么非常强大。这里介绍一种 JavaScript 中创建块级作用域的方法：IIFE。
什么是 IIFE，它的意思是立即执行函数表达式，如何区分函数声明与函数表达式呢？如果第一个字符不是 function,
那么它就是一个函数表达式。看下面的例子：
```
(function(){
    console.log("IIFE");
})();

//Boostrap 插件里面的 IIFE
+function(){
    console.log("anothre IIFE");
}();
```
###模仿块级作用域
模仿块级作用域是通过 IIFE 与闭包共同完成的。
```
for(var i = 0; i < 3; i++) {
    (function() { //IIFE
        var j = i;
        setTimeout(function() {
            console.log(j);
        },1000);
    })();
}
```
如果象下面这样去使用闭包的话，不用担心内存问题，因为执行完就能立即销毁作用域链了，因为没有对匿名函数的引用
```
(function(){
    //块级作用域
})();
```
###私有变量
```
(function (){
    var age = 20;
    function sayAge() {
        console.log(age);
    }
    
    MyObj = function(){};
    
    MyObj.protoytpe.publicAPI = function() {
        age ++;
        return sayAge();
    };
})();

console.log(age);  //报错，age 未定义
var person1 = new MyObj(); 
console.log(person1.publicAPI()); //21
var person2 = new MyObj();
console.log(person2.publicAPI()); //22
console.log(person1.publicAPI()); //23
```
上面的 age 变量被称为静态私有变量，外部无法访问，只能通过公共 API 来访问。
```
function fun(n,o) {
  		console.log(o);
  		return {
   		fun: function(m){
      				return fun(m,n);
    			}
  		};
}
var a = fun(0);  a.fun(1);  a.fun(2);  a.fun(3);
//undefinded   0  0  0  
var b = fun(0).fun(1).fun(2).fun(3);
//undefinded   0  1  2  
var c = fun(0).fun(1);  c.fun(2);  c.fun(3);
//undefinded   0  1  1
```
解释上面这个例子：
对于返回的对象中的 fun 方法，返回了 fun 函数。
对于 a ，第一次调用的时候，o 变量是未定义的，打印出 undefinded，使用 a.fun(1) 的时候，使用的是对象返回的
方法，由于 1 是赋值给 m 变量的，而对于 n 变量，根据闭包的作用域，找到 n ，发现第一次的时候被赋值为 0 ，因
此打印出 0 。后面的数次调用都是同样的道理。
对于 b，第一次调用 o 变量未定义，打印 undefinded，变化发生在后面，由于这里是链式调用，第一次 fun(0) 返回的
对象是不同于 fun(1) 返回的对象的，同理， fun(2) 返回的对象也不同于 fun(1) ,总之返回的对象各不相同。在 
fun(1) 的时候，m 被赋值为 1，而 o 被赋值为 n ，而在作用域链中，n 被赋值为 0 （因为fun(0)）,所以打印出 0.
这个是因为 fun(1) 的对象闭包了 fun(0) ，所以后面的 fun(2) 也同样是闭包了 fun(1), 而 fun(1) 里面的 n 已经
被赋值为 1 了。所以 fun(2) 打印出来的是 1，所以后面的原理类似。
对于 c，前面的 undefinded，0 与 b 例子原理是相同，而后面的 1，1 因为调用的都是第二层对象，因此都是 1。
###模块机制
闭包最强大的功能：模块机制。
```
var single = function(id) {
    
    var name = "me";
    function sayName() {
        console.log(name);
    }
    
    function sayID() {
        console.log(id);
    }
    
    return {
        name: name,
        sayID: sayID,
        sayName: sayName
    };
};

var foo =single("foo");
foo.sayName(); //"me"
foo.sayID(); // "foo"
```
两个必要条件：
1.必须要有外部函数，至少被调用一次。
2.至少返回一个内部函数。
适用于创建一些单例对象，如果需要创建一个需要初始化数据，并可以公开一些访问私有变量的对象的话，就适合使用
模块模式。

增强的模块机制：
```
var single = function() {

    var name = "Jarvis";
    function sayName() {
        console.log(name);
    }
    
    var obj = {
        name: name,
        age: 18,
        sex: "man",
        sayName: sayName
    };
    
    return obj;
}

var anotherSingle = function() {

    var name = "Jarvis";
    function sayName() {
        console.log(name);
    }
    
    var obj = new Person();
    
    obj.name = name;
    obj.sayName = sayName;
    
    return obj;
}
```
增强的模块模式是适用于那些单例但是有需要添加一些别的特别的属性或是别的类型的对象的情况。

现代模块机制：
```
//本段代码摘自 You Dont Konw JS
var module = (function Manager() {
    var modules = {};
    
    function define(name, deps, impl) {
        for(var i = 0; i < deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }
    
    function get(name) {
        return modules[name];
    }
    
    return {
        define: define,
        get: get
    };
})();

module.define("bar", [], function() {
    function hello(who) {
        console.log(who);
    }
    
    return {
        hello: hello
    };
});

module.define("foo", ["bar"], function() {
    var hungry = "hippo";
    function awesome() {
        console.log(bar.hello(hungry).toUpperCase() );
    }
    
    return {
        awesome: awesome
    };
});

var bar = module.get("bar");
var foo = module.get("foo");

console.log(bar.hello("hippo"));
foo.awesome();
```






