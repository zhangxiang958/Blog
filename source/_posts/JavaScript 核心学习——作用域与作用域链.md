title:  JavaScript 核心学习——作用域与作用域链
date: 2016-03-30 18:17:24
categories: JavaScript
---
作用域与作用域链是 JavaScript 的核心，想要透彻理解 JavaScript 必须学习这份方面的知识。
作用域对于理解 JavaScript 有很大帮助，从哪里正确地访问变量到 this 的正确赋值，不管是从功能还是性能，
理解作用域都会有很大的进步。
本篇博文是尽本人目前最大努力的深究，是作为我的学习笔记，日后还会继续深究，不断更新博文。
<!--more-->
##编译原理
JavaScript 其实是一门编译语言，它在执行代码之前会做好执行代码的准备，也就是说 JavaScript 在执行代码之前
会对代码进行编译。举个浅显的例子：var a = 2; ，这句代码，Javascript 引擎首先会读到 var a ，在作用域中寻
找 a 变量，如果没有，则会创建一个变量并命名为 a ，最终的结果是 a 变量被找到（创建）然后执行赋值操作。

在编译器的方面来说，在查找变量的时候，其术语为 LHS，L 是 left 的意思，在获取变量值的时候，其术语为 RHS，
R 是 right 的意思。LHS 意思是在赋值操作符的左边的操作，RHS 是赋值操作符右边的操作，更加浅显地说，就是
如果是对变量赋值操作，则会有 LHS，如果是获取变量的值，则会有 RHS，在 var a = b 这个例子中，首先会有 LHS
查找 a 变量，然后有 RHS 查找 b 变量以获取 b 变量的值，然后将值赋于 a 。
这部分知识对后面作用域的理解很有帮助，希望读者可以理解。
##作用域
作用域，用浅显的话语来讲，就是可以起作用的范围。其实《JavaScript 权威指南》中对作用域的解释十分到位：函数
运行在被定义的作用域中，而不是被执行的作用域中。
作用域其实是一套根据变量名称查找变量值的规则。
###如何理解作用域
```
function foo(para) {
    var name = para;
    console.log(name);
}

foo("zhangxiang"); // "zhangxiang"
```
作用域可以理解为 Javascript 引擎与编译器与作用域之间的对话：
引擎: 首先我想对 *foo* 进行一个 *LHS* 。
作用域：*foo* 在全局作用域下，编译器将它定义为一个函数了。
引擎：我还想对 *para* 变量进行 *LHS* 。
作用域：*para* 在 *foo* 函数内部，它是 *foo* 函数的形参。
引擎：我还想对 *name* 变量进行一个 *LHS* 和对 *para* 进行 *RHS* 。
作用域：找到了，*name* 位于 *foo* 作用域下。*para* 是函数形参,它的值为 *"zhangxiang"* 。
*name* 变量被赋值了，现在将 *name* 的值传进 *log* 中，打印出结果。
###执行上下文
执行上下文，其实就相当于 C 语言中的栈帧，当一个函数被执行的时候，会将该函数环境压入栈中，等函数执行完，再
将函数环境出栈，将当前执行环境归还给上一个函数环境或全局环境。
###作用域链
作用域链其实就是作用域的嵌套，理解它就通过一个类比，类比于一栋大厦，第一层是当前作用域，如果当前作用域没有所需的变量，则会不停地往上找，像坐电梯一样不断往上，直到最上层的全局作用域为止。
但是不要错误地认为函数在什么地方执行，那么它的作用域链就是被外层函数，外外层函数等逐层嵌套，这是不一定的。
因为函数定义在函数内部，那么它的上一层作用域就是外层函数，但是如果函数定义在全局作用域中，但是只是被运行
在某个函数内部，那么它的上一层作用域应该是全局作用域！更多的解释请看下面的部分。
##词法作用域及理解误区
词法作用域，用简明的话语来说就是代码作用在它所写的地方。看下面两个例子应该会帮助理解：
第一个例子：
```
var a = 2;

function foo2() {
    console.log(a);
}

function foo() {
    var a = 1;
    foo2();
}

foo(); //2
```
在这个例子中，也许会有初学者认为打印出 "1" , 但是其实并不是，因为 *foo2* 函数是在全局作用域下定义的，虽然
*foo2* 函数是在 *foo* 函数中被执行的，但是 *foo2* 函数的作用域链中根本没有 *foo* 函数，因为其实函数的作用
域在定义的时候就已经被创建。针对这个例子，说得简明一点，就是 foo2 与 foo 函数都是在全局作用域下定义的函数
并且函数是拥有块级作用域的，所以 foo2 也就没有权限访问 foo 内部的变量了。
第二个例子：
```
var a = 2;

function foo() {
    var a = 1;
    function foo2() {
        console.log(a);
    }
    foo2();
}

foo(); //1
```
在这个例子中，结果为 "1" ，与上面的例子不同的是，*foo2* 函数定义在 *foo* 函数内部，所以拥有对 *foo* 函数
内部变量访问的权限，并且在作用域链中，有一个遮蔽效应，所谓遮蔽效应，就是作用域如果在这一层中找到命名相同
的变量，就不会继续往下寻找了，就好比坐电梯在第二层找到了想要的东西，也就不会再到下一层寻找想要的东西了。
如果我们想要使用下一层或者全局的变量的时候，可以在前面加上相应的对象，使用对象属性访问规则,比如这里想要
打印出 "2" 的话，那么就使用 console.log(window.a);
###词法欺骗
所谓词法欺骗就是使用 eval 或者 with 语句，欺骗词法作用域，仿佛代码原本就是写在那里的，在本质上，这两个函数
在作用域链的前端添加了一个块级作用域，在这里不展开阐述，只需知道不推荐使用这两个方法，因为使用这两个方法的
话，代码执行的性能就会下降，其原因在于 Javascript 引擎一旦遇到有 eval 或 with 的代码块时，不再对代码进行优
化。
##变量提升
变量提升，是将变量提升到所在作用域的顶端。
```
console.log(a);

var a = 2;
```
很多人在这里会误解，认为这里脚本会出错，但是其实真实的是 console 出 *undefinded* 。为什么呢？其实引擎在
翻译 Javascript 的时候，会将变量的声明提升了，然后到了对变量执行赋值操作的那一行代码的时候才进行赋值操作。
至于为什么打印出 *undefinded* , 这是因为变量在只有声明而没有赋值的时候，自动拥有 *undefinded* 值。所以，
上面的代码应该被理解为：
```
var a;

console.log(a);

a = 2;
```
###函数优先
```
foo(); // "1"

var foo;

function foo() {
    console.log("1");
}

foo = function () {
    console.log("2");
}
```
在声明一个变量的时候，如果一个变量与一个函数的命名冲突，那么该变量会被优先定义为一个函数。而对于函数表达式
与函数声明，在第五行的代码是一个函数声明，而在第九行是一个函数表达式，他们的区别在于函数声明会提升，而函数
表达式是在代码执行到那一行才会被赋值为函数。区分函数声明与函数表达式的方法就是看代码的第一个字符是不是
function ，如果是，那么就是一个函数声明，如果不是，那么就是一个函数表达式。对于函数块级作用域的加以利用，
就出现一个模式叫做 IIFE 立即执行函数表达式。通过这个避免全局变量的污染，从而提高代码复用性。如下面例子：
```
var a = 1;

(function (){
    var a = 2;
})();

console.log(a); // "1"
```
不可靠的行为：
```
foo();  // "2"

var a = true;

if(a) {
    function foo() {
        console.log("1");
    }
} else {
    function foo() {
        console.log("2");
    }
}
```
上面的结果为 2 ，是因为如果有两个同名的函数声明，后面的声明将会覆盖前面的声明。而在 Javascript 中，if 语句
和 for 语句等等，都没有块级作用域，因此函数不会根据 a 变量的值去选择声明哪个函数。
##ES6 的块级作用域
在 ES 6 中，有了块级作用域命令的出现——let，与 const。这里不在阐述，只需知道 let 用于声明块级作用域的变量，
而 const 是声明一个块级作用域常量。更多关于 ES6 特性请看阮一峰老师的 ES6 入门。
[传送门](http://es6.ruanyifeng.com/)
##作用域有什么用
上面讲述了作用域的知识，如果没有实际作用就未免有点学院派了，其实在 《高性能 JavaScript》中已经提到作用域
对于功能以及性能方面有非常有帮助。功能就是上面所说的，找到合适的变量访问权限。而性能就是下面所说的代码优化
###代码优化
我们知道，作用域是一级一级地往下寻找变量的，那么引擎就需要不停地往下寻找，但是我们知道，如果作用域在当前
作用域就能找到变量的话，就不必花费额外的消耗寻找了。看下面的例子：
```
var btn1 = document.getElementById("btn1");
var btn2 = document.getElementById("btn2"); 
var btn3 = document.getElementById("btn3"); 
var btn4 = document.getElementById("btn4"); 
```
像上面，在性能方面就非常损耗，进过改进，得到如下:
```
var doc = window.document;
var btn1 = doc.getElementById("btn1");
var btn2 = doc.getElementById("btn2"); 
var btn3 = doc.getElementById("btn3"); 
var btn4 = doc.getElementById("btn4"); 
```
参考文献：
[作用域链 Scope Chain](https://github.com/goddyZhao/Translation/blob/master/JavaScript/%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%93%BE%EF%BC%88Scope%20Chain%EF%BC%89.md)
[执行上下文 Execution Context](https://github.com/goddyZhao/Translation/blob/master/JavaScript/%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%EF%BC%88Execution%20Context%EF%BC%89.md)
[变量对象 Variable object](https://github.com/goddyZhao/Translation/blob/master/JavaScript/%E5%8F%98%E9%87%8F%E5%AF%B9%E8%B1%A1%EF%BC%88Variable%20object%EF%BC%89.md)
《JavaScript 高级程序设计》
《你所不知道的 JavaScript 上卷》

