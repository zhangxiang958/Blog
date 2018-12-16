title: JavaScript 学习笔记——数据类型及应用
date: 2016-02-27 11:25:24
categories: JavaScript
---
这篇博文是个人的学习笔记，并记录下 Javascript 实战中比较有用的数据类型的属性与方法。
<!--more-->
##值类型
*number* 数字类型，说明该值是一个数字值。
*string* 字符串类型，代表单引号或双引号括住的字符集合。和 PHP 不同，Javascript 的字符串使用双引号
或单引号括起的字符串没有区别。字符串一旦创建无法改变值，改变只能销毁原来的值。这句话无法理解看下面。
*boolean* 布尔值，true 或 false。
*undefinded* 未定义的值，为初始化或未定义的值都会自动拥有 undefined 值类型。
*null* 代表空的对象引用，如果声明了一个变量用于存储对象，应该预先初始化为 null，表明这是一个空的对
象指针。
*object* 代表该值是一个对象。

一些有用的方法：
*parseInt()* 或 *parseFloat()*: 将一些字符串转化为数字。
###字符串一旦被创建就无法被改变怎么理解？
```
var a = new String("string");
相当于
var a = "string";
```
内存里面有两个字符串对象，一个是使用 String 构造函数构造的字符串对象，另一个是 a 对这个对象的引用，
无论后续操作是连接字符串还是让变量 a 重新指向另一个对象引用，例如 a = new String("cctv");，都是重新
创建了一个新的字符串对象，填充了 a 存储的对象引用与另外一些字符串，而原来的 a 的字符串和连接的另一部
分字符串如果没有引用，那么垃圾回收机制就会将其回收。

##类型的判断
###typeof
这个是用于判断值类型的操作符。
```
var oobject = null;
var fn;
console.log(typeof fn);    //undefinded
console.log(typeof Fn);    //Fn 由于未定义，同样是 undefinded
console.log(typeof true);  //boolean
console.log(typeof "Jarvis");  //string
console.log(typeof 66);  //number
console.log(typeof oobject);  // null
```
###instanceof
如果判断值类型，typeof 是完美的选择，但是有时候我们不只是需要知道这个变量存储了一个对象，我们还需要
知道是什么类型的对象，instanceof 就是为了鉴别对象是什么类型的对象。
```
var a = new Array();
console.log(a instanceof Array); // true
```
###判断类型的正确姿势
因为如果使用 iframe ，不同的 iframe 有不同的数组定义，所以我们通常使用 Object.prototype.toString.call() 方法来判断。
```
var a = [];
console.log(Object.prototype.toString.call(a));  // "[Object Array]"
```
推荐阅读几篇关于判断对象类型的博文。
[Javascript中判断数组的正确姿势](http://www.tuicool.com/articles/2q6FB3Z)
[JavaScript:Object.prototype.toString方法的原理](http://www.cnblogs.com/ziyunfei/archive/2012/11/05/2754156.html)

##引用类型
JavaScript 中的引用类型，就像是其他面对对象语言里面的类一样，是规定一个集合的属性与方法的数据类型。
引用类型的值就是对象，也就是引用类型的实例。实质上，引用类型之所以被称之为引用类型，是因为它储存在
变量里面的值是一个地址值，浏览器会跟据这个地址，寻找到相应的值，而不像值类型一样，每个变量存储的值
存放在独立的空间。
```
//值类型
var a = 1;
var b = a;
b = 2;
console.log(a);  //1
console.log(b);  //2

//引用类型
var a = {
    name: "Jarvis";
};
var b = a;
b.name = "zhangxiang";

console.log(b.name);  //zhangxiang
console.log(a.name);  //zhangxiang
```
从上面的例子可以看到，如果是引用类型（比如 Object 类型），两个变量都是引用同一个对象的话，修改其中一
个的属性或方法，会使所有实例的属性方法遭到修改。
###深克隆
引用同一个对象会导致一改全改，但是有些时候，我们又必须达到修改一两个属性但又不做到全局修改的目的。重新
声明一个对象（属性方法都一样）可能会达到要求，但是会使代码量增加。这时候就需要用到对象的深克隆。
```
function cloneObject(src) {
    /*
        解释一下 newObj 变量：如果 src 存入的是一个数组对象，那么 newObj 为数组对象，
        否则为Object类型
    */
    var str,newObj = src.constructor == "Array" ? [] : {};
    
    if(typeof src !== 'Object'){  //如果 src 不是对象
        return ;
    } else if(window.JSON) {  //如果 src 是 JSON 对象
        str = JSON.stringify(src);
        newObj = JSON.parse(str);
    } else {  
        for(var i in src){  //深度克隆
            newObj[i] = src[i] === "Object" ? cloneObject(src[i]) : src[i];
        }
    }
    
    return newObj;
}
```
###垃圾回收机制中的引用类型
其实理解引用类型的值存放的是地址值很有必要，对于我们对浏览器运行内存的释放有帮助。
在此之前，先浅略说明垃圾回收机制，它分为两种，一种是标记清除，另一种是引用计数。引用计数就是通过计算
一个值被引用的次数来决定是否被垃圾回收机制回收，但是往往可能会出现相互引用的局面。常用的是标记清除，
标记清除就是通过标记变量是否进入执行作用域，如果进入了作用域，就进行标记不清除，如果离开就会被标记清除，
从而释放内存。
所以为了使页面有更好的性能，需要手动清除变量的引用，设置为 *null*.
```
function createPerson(name) {
    var object = new Object();
    object.name = name;
    return object;
}

var normalPerson = createPerson("jarvis");
//手动清除引用
normalPerson = null;
```
这样的目的是为了让变量脱离执行环境，等待垃圾回收机制下一次回收将其会回收。
###Array 数组类型
这是在 JavaScript 中第二常用的引用类型。与其他语言不同，Javascript 的数组元素类型不必相同，它可以有些
元素是 Number 类型，有些是对象。在编写的时候，为了可维护性，建议尽量用字面量法来定义数组。关于数组的详
细知识在 《Javascript 高级程序设计》中讲述得非常清楚，但本博文只详细讲述我觉得比较重要的知识点。
####数组的 length
数组的 length 特别之处在于它不是只读的，当它发生修改的时候，数组元素会增加或减少。
```
var a = {1,2,3,4,5};

for(var i = 0; i < a.length; i++){
    a.push(i);     //这样会无限循环
}

a.length -- ;   //这样最后一个元素会被删除
```
####强大的 splice
splice 算是数组中最为强大的方法，主要有三个用途：删除，插入，替换。
```
//删除
splice(1,2) 删除第一项位置和要删除 2 项。

//插入
splice(1, 0, "yeloow");  //执行插入动作的时候，第二项参数为 0 ，意思是在第一项位置，插入项 "yellow"

//替换
splice(2, 1, "red", "yellow"); //意思是删除第二项位置起的一项，插入项 "red" , "yellow"。
```
####一些特别的方法
*indexOf()*  从开头处寻找某一项在数组中的位置。lastIndexOf() 就是从末尾处寻找某一项在数组的位置。
*map()* 常用于替代 *for* 循环。*forEach()* 循环数组每一项，对每一项都执行一个回调函数。
*join()* 用特定的符号分割字符串。数组存储，然后用join分割，是序列化长字符串的技巧
###基本包装类型
3 个基本包装类型 是 Number, Boolean, String。有些人会有疑问，这些不是基本类型吗？没错，但是为了我们
操作数据的方便性，javascript 将这些类型与一些方法打包起来，组成一个基本包装类型。基本上，每次读取
一个基本类型值的时候，都会创建一个包装类型的对象实例，调用方法后，再将这个实例销毁。
特别的在于 String 类型字符串中的字符可以通过方括号来访问。
###Function 函数类型
在 Javascript 中函数没有重载。相同名字的函数会被后定义的函数所覆盖。
对于函数，理解的重点在于 函数是对象，是 Function 类型的一个实例，但是函数名是对象指针，指向一个对象。
对于函数内部对象，arguments 与 this 需要特别注意。arguments 是一个类数组对象，主要用于保存参数。
arguments 内部的 callee 属性，可以消除运算与函数名之间的耦合性。caller 属性指向调用函数的函数。
```
function factorial(num) {
    if(num < 1){
        return 0;
    } else {
        return num * arugments.callee(num - 1) ;  //相当于 num * factorial(num - 1);
    }
}
```
this 对象，在网上的更精彩的博文有很多，这里不在累述。更多请点下面传送门。
[this 的使用 ——阮一峰](http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html)
关于函数重要的两个方法： *call()* 和 *apply()* 使用上并没有太大的区别，都是为了拓展函数作用域的。
需要注意的是，*call()* 方法的第二个函数接受的是列举的参数，而 *apply()* 第二个参数接受的是参数数组。
第一个参数就是函数需要执行的作用域。
###RegExp 正则类型
下面这篇博文十分精彩，适合入门，但是需要注意的点是使用字面量创建的正则表达式与使用构造函数创建的
正则表达式是不同的，使用字面量的正则表达式会共享一个对象实例，而使用构造函数构造的正则表达式每一个实
例都是新的实例，也就是说对象实例相同，原来设置的属性不会重置，而新的对象实例的属性会重置。
这个点在循环检验字符串的时候需要注意。
[30分钟入门正则表达式](http://deerchao.net/tutorials/regex/regex.htm)
###对象 Object
为了可维护性，建议尽量采取字面量法创建对象。




