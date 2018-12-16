title: JavaScript核心知识学习笔记
date: 2015-07-27 21:15:24
categories: JavaScript
---

# JavaScript核心知识学习笔记

------------

JavaScript的核心知识如下：
> * 对象
> * 原型与原型链
> * 作用域与作用域链
> * 闭包
> * this关键字

<!--more-->
## JavaScript的对象
JavaScript与Java等语言有所不同，是一门基于对象的语言，在JavaScript中一切均为对象（基本类型除外如number/string/null/boolean/undefined）。JavaScript的对象与Java中的类相对，JavaScript的对象实例与Java的对象相对。那么，对象是什么？
我的理解是，**对象**，是一个无序元素的集合。举个栗子，人包含了有鼻子，眼睛等相同的基本**属性**，也有如走，吃等基本的行为（这在JavaScript中称为**方法**，可以理解成值为函数的属性）。对象里面是有多对 名/值对 组成，如下面的栗子。
```
var person{
    eyes : 2,
    nose : "true",
    walk : function(){
      console.log('walk');
    }
}
```
最简单的对象构造方法：原始法
```
var dog1 = {};//构造一个新对象
 dog1.name = '哈士奇';
 dog1.color = '白色';
 dog1.run = function(){
    console.log("dog1.name + 'run'");
 }
 var dog2 = {};//构造一个新对象
 dog2.name = '柯基';
 dog2.color = '黄色';
 dog2.run = function(){
    console.log("dog2.name + 'run'");
 }
 //或者这样写
 var dog1 = {//这样称为 对象字面量法
    name = '哈士奇'，
    color = '白色'，
    run = function(){
    console.log("dog1.name + 'run'");
    }
}
```
**这两种方法都是简单，并且容易理解，但是由于它们的属性和方法是后绑定的，当需要构造大量的对象时，代码的复用性不高，效率较低，并且，对象实例之间没有联系。**


*关于如何构造对象的几种方法，这里不详细展开。但是会在文章中穿插有所涉及。*

------
#原型与原型链
## 原型
原型也是对象，我对于它的理解是，模板（父类）。由原型通过构造器——**构造函数**构造出来的对象实例会拥有原型的属性和方法。构造函数有一个指针——**protoytpe**（实际上这个是构造函数的一个隐式属性），去指向原型，而原型会有一个指针——**constructor**指向原型，而每个对象实例都有一个指针（隐式属性）——**___proto___**指向原型。
```seq 
Title: about the proto 
Proto原型->function构造函数: constructor属性 
function构造函数-->Proto原型:prototype属性
function构造函数-->object对象实例:构造实例  
object对象实例-->>Proto原型: __proto__ 属性
```
可以用 IsProtoytpeOf() 方法去检测一个实例是不是这个原型所构造出来的。
*可以通过原型来构造对象*：
```
function dog = {};//先构造一个空的函数
dog.prototype = {//将属性和方法定义在对象的原型上
    name = '哈士奇',
    color = '白色',
    run = function(){
        console.log("name + 'run'");
    }
};
var dog1 = new dog();
```
但是代码这样对吗？我先定义一个新的函数作为构造函数，在这个构造函数的prototype即原型上，定义了实例中所需的属性和方法，先不讨论这种方法的优缺点，这段代码看似正确，但是忽略了上面所讲的知识点，*dog.prototype*这个对象应该有**constructor**这个属性去指向构造函数*dog*。因此，应该向*dog.prototype*的字面量中加入下面这段代码：
```
constructor : dog,
```
但是通过这样的方法并不是很好的方法，因为一个原型通过一个构造函数会制造出很多个对象实例（如果只是制造一个没必要用这样的方法），所以，原型中的属性是共用的，会导致数据不安全，会出现一改全改的现象。


##原型链
上面就是最简单的原型链。原型链，顾名思义，就是一个个原型层层堆叠，串联一起，构成链状结构（注意，原型可以是另一个原型所购造出来的实例）。用以前的栗子来理解的话，就像物种链一样，human<--mammals<--animals，从人类到哺乳动物再到动物。
```flow 
st=>start: 原型A 
io=>inputoutput:  原型B
op=>operation: 原型C 
e=>end

st->io->op 
```
最顶层的原型就是object.prototype。
其中继承中有一种就是通过原型链来实现继承的，称为原型继承。这里不展开讲述原型继承，只做浅谈，但是其中原理就是通过向原型加入共用的方法或是属性，使下面的实例能够调用，达到提高代码复用率的目的。
```
function animals(){
    //构造一个新构造函数
}
animals.prototype = {
    constructor : animals,
    eat : function(){
        console.log(eat);
    }
}
function createHuman(name,age){
    name = name,
    age  = age,
    speak : function(){
       console.log("this.name + 'speak'"); 
    }
}
var human = new animals();
human.constructor = createHuman();//这里的意思是让human原型指向createHnman这个构造函数，建立起原型与构造函数之间的关系。
var human1 = new createHuman('zhangxiang',20);
var human2 = new createHuman('Jarvis',21);
//此时，human1 与 human2 这两个对象实例既有了 animals 的 eat 方法，也有了本身的 name，age，speak等属性，实现了继承。
```

---------
#作用域与作用域链
##作用域
有C语言基础的童鞋在这个方面应该不难理解作用域。作用域，就是变量，元素等在代码中的有效范围（或者说执行范围）。作用域在函数被创建之时即会生成，分为两种，一种是全局作用域，另一种是局部作用域。
###全局作用域
```
var name = 'Jarvis';
function person(){
    var age = 20;
    var walk = function(){
    } ;
}
alert(name);//Jarvis
alert(age);//undefinded
```
在函数外部和在全局声明的变量，拥有全局作用域。而在函数内部声明的只能在函数内部使用，即拥有局部作用域。
###局部作用域
```
var name = 'Jarvis';
function person(){
    var age = 20;
    height = 178；
    var walk = function(){
    } ;
}
alert(name);//Jarvis
alert(height);//178
alert(age);//undefinded
```
在函数内部，不声明（即不用var关键字进行声明）直接赋值的，视为全局变量，同样具有全局作用域。
##作用域链
作用域链，即是一个个变量的作用域层层堆叠的链状结构，实际上，作用域链是一个栈式的结构。
首先要说明的是，在函数中，能够使用全局变量，像上面的代码，*person()*这个函数完全可以使用*name*变量，但是像上面看到的结果，全局下不可以使用函数内部声明的变量。实际上，函数内部的实现机制如下图
```seq 
Title: 作用域链开始 
函数内部对象->global对象（全局对象） :  

```
*此图虽然是横的，但是实际上是栈式结构，优先在当前作用域中查找*
当使用变量时，内存会先在内部对象中匹配相对应的标识符，取到相对应的值，如果函数内部对象中没有匹配到对应的标识符，会继续向上一层（上一个父函数或者直接就是global对象中）查找，像上面的栗子就是在global对象中查找。
如果代码中有*with*语句或是*try/catch*语句，那么他们的优先级最高，会优先遍历这两个代码块中的变量对象。但是*with*语句会造成性能损耗，不建议使用。
了解了作用域链的知识，就可以对代码进行优化了。既然代码在执行过程中会层层深入地去查找所需的变量，那么我们可以把需要常用到的变量先存储在当前作用域吗？答案是肯定的。举个栗子，如下代码：
```
function person(){
 var name = document.getElementById('Jarvis');
 var height = document.getElementById('zhangxiang');
 var color = document.getElementById('xiaoha');
}
```
上面的代码，每次声明的变量赋值时，都需要遍历一次DOM树，不是最优的方法。但是如果在他们之前加上
```
var doc = document;
```
先把document放在当前作用域中，这样就可以较快地查找到所需的元素。（这里还会涉及到二维查找的知识，但因为学习理解原因，以后再做补充）
特别地，类似原型链，在作用域链中，被创建的函数都有一个隐式属性——[[scope]],这个属性可以查看到包含所有上层对象的变量对象，同时也有一个scope属性，但是这个属性是属于上下文的。（关于执行上下文的知识以后再做补充）

-------
#闭包
有了上面的作用域的知识，就会有疑惑：既然父级无法调用子级的变量，但是在实际运用中我们需要用到的话，这样怎样解决？没错，答案就是使用闭包，这里的闭包，可不是离散数学里面的闭包，那是不同的两个概念。闭包，我的理解是，抽象地看是函数内部与外部的联系（桥梁），运用中可以看做是函数内部的函数。举个栗子：
```
function person(){
    var name = 'Jarvis';
    function getThePoint(){
        console.log(name);
    }
    return getThePoint;
}
person()();//相当于getThePoint()，结果是Jarvis
```
通过上面的代码，我们就可以使用到*person*内部的*name*变量了，而getThePoint函数就是我们所说的闭包，通过闭包使用函数内部的变量。这是闭包的其中一个用途：访问内部变量。
```
function mainFunction(){
　　　　var name = 'Jarvis';
　　　　changeName = function(){
　　　　    name += ' zhangxiang';
　　　　}
　　　　function innerFunction() {
　　　　　　console.log(name);
　　　　}
　　　　return innerFunction;
　　}
　　var result = mainFunction();
　　result(); // 打印 Javis
　　changeName();
　　result(); // 打印 Javis zhangxiang
```
而上面这段代码，展现的是闭包的另外一个用途：将变量存放在内存中，通过全局函数直接可以调用变量。

-----------

#This关键字
如何理解this关键字呢？它并不是对象的一个变量，它简单地理解，就是某段代码执行的环境。
##在全局中的this
```JavaScript
a = 10;//全局变量
alert(this.a);//相当于window对象调用 a 变量
```
##在局部中的this
```JavaScript
function person(name){
    this.name = name;
}
var Jarvis = new person('zhangxiang');
var zhangxiang = new person（'xiaoha'）;
console.log(Jarvis.name);
console.log(zhangxiang.name);
```
this 关键字，简单地来说，就是谁调用它，谁来提供这个运行环境，它就是代表谁。有个方法来判断它，看调用括号的左边，如果是引用类型，那么 this 关键字就是引用类型的 base 对象，例如：person.walk();/dog.speak();等等。
由于 this 不是一个变量，因此不可能通过赋值来改变它的值。我们通过call（）或apply（）方法来手动改变 this 的值。
```
person.walk.call(dog);//意思是用dog这个对象代替person对象来执行walk()方法
```
*call*()和*apply*()都可以类似以上改变 this 的值，他们都接受两个参数，第一个参数都是所要调用的对象名，不同的是，call（）接受的第二个参数可以为任意值，而apply（）第二个值必须是数组的形式。

以上。
这些是我一周以来的学习笔记以及个人的理解，部分知识还未完全理解透，所以有些知识点写得不多以及简陋，望体谅。




作者   张翔  
2015 年 07月 27日     



