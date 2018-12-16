title: JavaScript 核心学习—— this 关键字
date: 2016-05-01 13:16:24
categories: JavaScript
---
this 关键字是 JavaScript 面向对象编程重要一环，它对于函数式编程极其有用。
<!--more-->
##this 指向
在我前面的博文已经说过了，JavaScript 中的作用域是词法作用域，即代码写在哪里作用域就在哪里。但是这一情况到了
this 就有所改变了。this 机制类似于动态作用域，它的值不是在编写的时候决定的，而是在调用的时候决定的，因此会有初学者弄不清 this 的值而弃用。
###为什么要使用 this
```
function foo() {
    console.log(this.name);
}

var foo1 = {
    name: "Jarvis"
};

var foo2 = {
    name: "zhngxiang"
};

foo.call(foo1);  //"Jarvis"
foo.call(foo2);  //"zhangxiang"
```
这样使用 this ，可以不需要为每一个对象编写一个函数， foo 函数可以复用，当然也可以使用传输参数，但是随着
编写的代码越来越复杂，管理参数将会耗费巨大的精力。
###确切理解
this 不是指向本函数的作用域的（即使有时候正确），也不是指向本对象的，它是根据调用栈去判断自己的值的。
那么怎么判断 this 值呢？
##如何判断 this
this 是根据它的调用栈来定自己的值的，它的值等于调用位置，而调用位置就是调用栈的第二个位置。
```
var name = "global";
function fn () {
    var name = "me";
    debugger;
    console.log(this.name);  //global
}
fn();
```
在需要知道的 this 值前面，加上一个 debugger 语句，然后在控制台中可以查看到该函数的调用位置，第二个位置
就是 this 的真实调用位置，即它的值。
###this 的 四个规则
this 值的判断光是有上面所说的调试技巧是不够的，我们需要深入理解，以至于随心所欲地使用 this。关于 this 值
的判断有四个规则，根据这四个规则，可以准确地判断 this 值。
####默认绑定
象上面的那段代码一样，就是默认绑定。所谓默认绑定，就是绑定到默认的 window 对象。我们知道，全局变量与全局
对象的属性是同一个东西，只是写法不同而已。当一个函数调用，没有通过对象调用或其他方式，像上面的 fn 函数一
样，就会使用默认绑定，绑定到全局 window 对象上，因此打印出 "global"，对于这种情况，我们可以采用上面说的
调试技巧来确认 this 值。
####隐式绑定
隐式绑定，就是通过对象调用来隐式地绑定了 this 。举个例子：
```
var name = "global";
function fn () {
    console.log(this.name);  //global
}

fn(); //global

var obj = {
    name: "obj",
    foo: fn
}

obj.foo(); //obj
```
并且，隐式绑定如果有一长串的对象嵌套调用，this 值是绑定到最后一个对象的。
值得注意的是隐式绑定容易发生隐式丢失，隐式丢失常常发生在赋值操作中。举个栗子：
```
var name = "global";
var obj = {
    name: "me",
    fn: function() {
        console.log(this.name);
    }
};

var foo = obj.fn;
foo();  //global
```
为什么呢？ 因为对于 JavaScript 来说，对象内的方法不算是严格意义上的属于对象，只是对于函数的一个引用，像
上面那样将对象中的方法赋值给一个变量，那么 this 与对象之间的连接就会被切断了，而变成一个普通函数，运用
默认绑定，this 指向全局对象。这种情况常发生在赋值或将函数通过参数传递的时候。
####显式绑定
显式绑定，就是为了解决上面那种 this 值飘忽不定的情况，将 this 值绑定不变。通过 call，apply，bind 方法实现
显式绑定。
```
var name = "global";
function fn() {
    console.log(this.name);
}

var foo = {
    name: "me"
}

fn.call(foo); // “me”
```
通过 call ， apply 方法显式绑定函数的 this 的值，这里的栗子就是绑定 this 指向 foo 对象。但是还是没有解决
隐式丢失的问题，因为如果要赋值，显式绑定无法解决。所以我们可以使用硬绑定。
```
function fn() {
    console.log(this.name);
}

var obj = {
    name: "me"
}

var bar = function(){
    fn.call(obj);
}

bar(); // "me"
bar.call(window); //"me"
```
通过了硬绑定，即使想要通过 call 改变也无法改变，其实现方法就是通过一个函数包裹。因为硬绑定很常用，所以
Javascript 内置了一个 bind 方法来硬绑定，它返回一个硬绑定的函数。
####new 绑定
在说 new 绑定之前，要先知道 new 命令做了哪些工作。
new 命令在使用时进行了 4 个操作：
1.创建一个新的对象
2.为这个新建对象进行 [[prototype]] 连接。
3.将 this 值绑定到这个新建对象上。
4.如果函数没有返回其他对象就返回这个对象。
```
function Foo()  {
    this.a = 1;
}
var b = new Foo();
console.log(b.a);  //1
```
所以如果函数是通过 new 命令来调用的，那么 this 值就是指向新建的对象。
对于这四个规则，其优先级是：new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定。
通俗的话就是如果有 new ，那么 this 就绑定到新对象，如果没有 new，而有 call ，apply，bind 方法的话，就是
绑定到指定的对象。如果函数是通过对象方法调用的，那么就是绑定到那个调用的对象。如果以上都没有，就采取默认
绑定，绑定到全局对象。
##如何使用 this
对于如何使用 this，其实就是上面所说的判断 this 方法。使用 this 就是通过上面那四种方法去调用。
通过 this 调用全局对象属性值，通过 this 调用对象属性值，通过 this 访问指定的对象值，通过 this 访问 new
对象。

参考文献：
[this 用法](http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html)



