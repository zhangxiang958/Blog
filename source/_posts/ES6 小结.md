title: ES6 小结
date: 2017-06-13 21:21:24
categories: JavaScript

---


最近在编写代码的时候已经基本使用 ES6 的语法了, 所以想抽空将 ES6 的语法中需要注意的东西总结一个.
<!--more-->
##Let && const
*let, const* 命令为 *javascript* 语法增添了块级作用域的便捷性, 对比 *var*, *let* 与 *const* 不能**重复声明**
并且在声明之前不能使用, 也就是存在暂时性死区, var 声明的变量相当于全局对象的属性, 但是 let, const 声明的变量
变更不会变成全局对象的属性.
注意 const 声明的是常量, 需要在声明的时候作初始化, 并且它保持不变的是引用变量的地址, 而非值, 也就是说, 对于
对象这种值, 可以修改 const 声明的对象的属性.
```
/*let 命令编译前:*/
for(let i = 0; i < 3; i++ ) {
    console.log(i);
}
console.log(typeof i);

/*let 命令编译后*/
for(let _i = 0; _i < 3; _i++){
    console.log(_i);
}

console.log(typeof i);

/*let 命令编译前 */
var li = $('li');
for(let i = 0, length = li.length; i < length; i ++) {
    li.eq(i).on('click', function(){
        console.log("let:" + i);
    });
}

for(var j = 0, length = li.length; j < length; j++) {
    li.eq(j).on('click', function(){
        console.log("var:" + j);
    });
}

/*let 命令编译后*/
var li = $('li');

var _loop = function _loop(i){
    li.eq(i).on('click', function(){
        console.log(i);
    });
}

for(var _i = 0, _length = li.length; _i < _length; _i ++) {
    _loop(_i);
}

for(var j = 0, length = li.length; j < length; j++) {
    li.eq(j).on('click', function(){
        console.log("var:" + j);
    });
}
```
建议使用 ES6 语法后使用 let 与  const 取代 var 的大部分场景.

##解构赋值
只要符合一定的模式, 就可以按照某种模式进行赋值就是解构赋值, 解构赋值在我们提取 JSON 格式数据, 数组元素,
当函数返回多个值或函数参数设置默认值时非常有用.
需要记住的是在对对象类型的值进行解构赋值的时候, 需要先知道对象的类型, 不能使用 [] 对对象进行赋值.并且在解
构赋值中, 是先找到模式, 再对变量进行赋值:
```
let { foo: baz } = { foo: 'shawn' };

console.log(baz);  //'shawn'
console.log(foo);  // Error

//foo 是模式, 在等号右边找到对象相同的属性名后, 将属性对应的值赋给后面的变量, 我们常写的 let { foo } = {foo: 'shawn'} 是简写.
```

解构赋值不能重复声明(let, const 命令):

```
let foo;
let { foo } = {foo: 123};
//以上代码报错

let foo;
({foo} = {foo: 123});
//正确
```
只有在模式对应的值是严格等于 undefined 的时候, 才会使用默认值
```
let { foo = 456, baz = 'shawn', bar = 'zhang' } = { foo: 'test' , baz: null };

foo;  // 'test'
baz;  // null
bar;  // 'zhang'
```
解构赋值减轻了我们在函数参数设置上的代码量, 我们的函数参数往往会有默认值, 如果没有解构赋值, 那么往往需要编
写类似 x = arguments[0] || 'zhang' 这样的代码, 如果使用解构赋值, 那么可以 foo(x = 'zhang') 这
样清晰简略的代码.
另外需要注意的是字符串数字布尔值这样基本类型在解构赋值的时候会自动变为包装类型: let {toString: s} = 123.
解构赋值在声明时不能带有括号, 在函数参数时不能带有括号, 不能将括号带入模式中, 否则将会报错.

##字符串
在 ES5 中, 字符串监测是否含有另一个字符串只能通过 indexOf 返回的数字是否是 -1 来判断, 但是在 ES6 中, 新增
了几个有用的 API, 添加了灵活性, 我们在 ES6 中可以使用 includes 来判断字符串是否含有另一个字符串, 返回的是
布尔值, 更加便捷语义更鲜明, 而 startsWith 可以判断源字符串开头是否是另一个字符串, endsWith 可以判断字符串
结尾是否是另一个字符串.
另外, ES6 中也有几个个人觉得挺好用的 API, repeat 可以重复某个字符串若干次并返回新的字符串, 如果输入 NaN,
会返回空字符串. 而自动补全 API: padStart(num, 'xx') 和 padEnd(num, 'xx') 可以分别在开头和结尾补全字符串.
对于省略第二个参数则会补空.
ES6 字符串最有用的则是模板字符串, 使用反引号, 中间可以使用 ${} 这样将表达式括起来. 

```

let a = 'world';

`hello ${ a }`

`hello ${a + b}`

`hello ${ a() }`
```

模板字符串最有用的是编译模板， 函数后面跟模板字符串可以用于防范 XSS
```

var template = `
    <ul>
        <% for(var i = 0; i < data.supplies.length; i++) { %>
            <li><%= data.supplies[i]  %></li>
        <% } %>
    </ul>
`;

function complie(template) {
    var evalExpr = /<%=(.+?)%>/g;
    var evpr = /<%([\s\S]+?)%>/g;
    
    template = template
                .replace(evalExpr, '`); \n echo($1); \n echo(`')
                .replace(expr, '`); \n $1 \n echo(`');
    
    template = 'echo(`' + template + '`);';
    
    var script = 
    `(function parse(data){
        var output = '';
        
        function echo(html) {
            output += html;
        }
        
        ${ template }
        
        return output;
    })`;
    
    return script;
}

```

## 函数

可以使用解构赋值来进一步简化函数代码, ES5 中如果使用 x || 1 这种表达式也有缺点, 如果 x 值本身赋予的
就是 false 这样的布尔值, 那么就会被赋值为后面的值, 所以为了保证值的安全和准确往往会使用判断 if x == undefined 这样的式子, 但是这样代码就会冗余, 在 ES6 使用解构赋值就可以简化代码了.

```

function foo(x = 1, y = 2){
    console.log(x, y);  //免除了 x = 'test' || 1
}

```
默认赋值的作用域与 ES5 中一样, 根据函数的变量对象来查找
```

function foo(x = y, y = 2){
    console.log(x, y);  //throw error, y is not defined
}

```
###rest 参数 && 拓展运算符
rest 参数和拓展运算符就像逆运算符, rest 参数是将参数集合到一个数组中, 而拓展运算符是将数组中的拆分出来.

```

function foo(a, ...arg) {
    var arr = [...arg];   //快速将 arguments 等类数组对象变为数组对象
    var length = [...arg].length;  
}

```
如果对于 MySQL 或者 MongoDB 等数据库查询返回的一列对象, 在返回的时候, 使用拓展运算符可以便捷赋值.
```

var queryObj = doQuery();
res.send(...queryObj);

```
在合并数组的时候不再像 ES5 中需要借助数组方法的 concat.
```

var arr = [2, 3, 4, 5];
var newArr = [1, ...arr];  //[1, 2, 3, 4, 5]

```
###name 属性
提供获取本函数的名称, 彻底废弃 arguments.callee 这种调用
```

var func = function(){
    
    
}
//ES5 
func.name  //''

//ES6
func.name  //'func'

```
###箭头函数

```

//如果函数不需要参数传参, 那么使用一个空括号
var func1 = () => return 5;
//简化函数代码
var func2 = a => a;
//如果函数代码块需要的语句超过两句的时候需要使用花括号括起来
var func3 = (a) => { a ++; return a + 5; } 
//因为花括号会被认为代码块, 所以如果需要返回对象, 对象需要括号括起来
var func4 = (name) => { return ({ name: name }); }

```
箭头函数与 ES5 的函数最大的不同点在于, 箭头函数中的 this 值是不变的, 这是因为箭头函数中根本没有自己的 this 对象, 它的定义的时候就使用了父级的作用域的 this 对象.正因为箭头函数没有 this 对象, 所以它的 this 是固定的,也是不可变的.
arguments, super, new.target 在箭头函数中也是不存在的. 

## 面对对象
对于面向对象来讲, Javascript 里面依然没有类, 依旧是原型继承, ES6 中的 class 只是为了简化原型继承的语法糖.
### Class
class 只是 Javascript 中原型继承的语法糖，与 ES5 中的继承一致.
```
class MyClass {
    constructor(x, y){
        this.x = x;
        this.y = y;
    },
    method(){

    }
}

相当于
MyClass.prototype = {
    constructor(){

    },
    method() {

    }
}


```

在 class 声明的方法不能被枚举, 并且都是在原型上的。class 没有变量提升，所以在声明之前使用会报错。
实例对象可以通过 persion1.____proto____ 访问到原型。person1.____proto____ === person2.____proto____ === Person.
name 属性会返回 class 名
可以使用下面这样的形式：

```
//创建了一个实例
const person = class Me{
    constructor(name){
        this.name = name;
    }
}('shawn');

const Person = class Me{

}

这里如果使用 Person.name 会打印出 Person， Me 只能是内部使用，指代当前类

```
类只能使用 new 命令来调用, 而且 class 命名不能提升, 也就是存在暂时性死区, 而且内部代码会自动运行在严格模式下, 类中所有方法都是不可枚举的, 而且类中不能修改类名:
```
let Klass = (function(){
    'use strict';

    const Klass = (function(name){

        if(typeof new.target === 'undefined') {
            throw new Error('');
        }

        this.name = name;
    }());

    Object.defineProperty(Klass.prototype, 'syaName', {
        value: function(){

        },
        enumerable: false,
        writable: true,
        configurable: true
    });
}());
```
从上面也明白, 为什么在内部不可修改类名(const), 而且每个定义的方法都会使用 Object.definePerproty 来定义
### extends

```
class Son extends Parent {
    constructor(...args){
        super(...args);
    }
}
```

在子类中继承父类，constructor 中一定要先使用 super 关键字，因为子类是没有自己的 this 对象的，需要先调用父类的 constructor 方法构造出 this 对象，然后在对 this 对象进行加工，
这个和 ES5 继承不同，ES5 是先构造出 this 对象，在使用 Parent.call(this); 这样，所以如果子类 constructor 没有 super 会报错。

```
class A {}

class B extends A {

}

B.__proto__ === A
B.prototype.__proto__ === A.protoytpe

AB 的继承其实是按照
extends 做了什么？
其实就是将
setPrototypeof(B.protoytpe, A.prototype); //表示实例对象的继承
setPrototypeof(B,A);  //表示构造函数的继承

setPrototypeof(obj, proto) {
    obj.__proto__ = prto;
}

```

class 是构造函数的另一种写法，class.prototype.constructor === class , 和 ES5 一样.

```
|-------|
| class |-------|constructor 
|-------|       |
   |            | 
   |protoyepe   | 
   |            |
|-----------------|
| class.prototype | 
|-----------------| 
```

如果前面有 static 命令，说明这是静态（私有）方法，不会被实例继承，只能通过类来调用, 但是可以被子类继承
```
class Parent {
    static method(){

    }
}

var parent1 = new Parent();
parent1.method();  //error
Parent.method();  //right

class Son extends Parent {
    constructor(){
        super.method();
    }
}

Son.method();  //right
```
class 只有静态方法没有静态属性，也就是说，如果属性写在 class 花括号里面都是无效的, 静态属性也就是设置在类上，而不是设置在实例上的属性。
```
class Parent {
    prop: 2,  //无效
    static prop: 2  //无效
}

Parent.prop = 2;  //有效
```

new.target 可以用于限定构造器只能通过 new 命令来调用。
如果继承类没有 constructor, 那么在 new 调用的时候会自动使用 super, 而对于有 constructor 的类 super() 必须要在 this 前调用.
extends 可以继承内置对象, 可以解决 ES5 不能完美继承数组对象的 bug.
## Module
ES6 中模块机制为我们带来了极大的便利, 真正地为 Javascript 带来了模块化编程, 并且它与 commonJS 这样的模块
规范不同, ES6 Module 设计出来是为了引入时能够进行编译优化, commonJS 引入的模块是运行时载入的, 而 ES6 是
编译时静态载入的, 可以在编译的时候就进行优化, 不必引入整个模块, 举个例子:
```
//commonJS
var { foo, bar, baz } = require('test');

//相当于
var test = require('test');
var foo = test.foo, bar = test.bar, baz = test.baz;
//这样相当于载入整个模块, 将模块中的某个部分赋予变量

//ES6 module
let { foo, bar, baz } from 'test';
//ES6 module 只是载入了 foo, bar, baz 三个方法, 模块的其他部分不载入, 虽然导致了模块本身不可用, 但是优化了编译时的速度与效率
```
对于 import, 不能使用在条件语句或者在函数块中, 因为这些都有条件判断的意味在, 正正和 ES6 Module 设计的初衷
违背, 并且 import 进来的模块具有提升作用, 即是可以在引入前使用, 相当于变量提升, 但是为了语义性, 还是建议
在模块的开头进入那些需要的模块.(对于 export 也一样)
```
//暴露命令, 将模块以一种形式暴露出去被其他模块引入
export
export {
    foo as Foo
}
export default //暴露模块默认值
//相当于 暴露出一个名叫 default 的变量
// export { xxx as defualt }
// import { default as Module } form './module';

//引入命令
import 
import * form './module';  //将模块全部方法属性引入
import { foo, bar, baz } from './module';  //引入 foo, bar, baz 三个方法
import { * as module } from './module'; // 给引入的模块加一个别名
import { foo as FooModule } from './module'; // 给引入的方法加一个别名
```
module 加载的实质就是 Linux 的符号链接, 是引用而不是像 commonJS 的复制, commonJS 对引入变量改变不会影响
原本模块的值, 而 module 的加载是在载入时不引入, 只是生成一个符号链接, 当运行的时候再去调取模块的值.
当然, 因为这个特性, 模块引入的值地址不变, 也就是说对于引用类型(对象等)属性可变, 但是对象本身 ```obj = {}``` 这样的修改是不可以的, 其他基本类型不能变.
相比于 CommonJS , CommonJS 是单对象输出单对象加载的格式, 和 ES6 的多对像输出多对象加载不同. 
### Module 加载原理
Module 加载的本质其实就是将引入模块链接到另一个文件中, 通过类似软链接的形式, 当代码运行需要模块中的变量或
方法的时候再去调取, 不同于 CommonJS 的调取, CommonJS 的调取是同步的, 并且是将另一模块的值复制后赋到文件中
的.
所以 Module 的加载取得的值有可能会随着时间的变化而不同，而 CommonJS 的加载是复制的，所以除非清除系统缓存， 否则第一次加载和第 n 次加载的值是一样的。
目前浏览器端都不支持 ES6 的模块机制, 需要使用 babel 这样的工具, 转为 CommonJS 规范再用 webpack 这样的工具
打包在一起.

 - 语法解析:检查语法
 - 加载: 没有被标准化,需要最终实现
 - 连接: 将模块的变量连接到一个新的作用域
 - 如果导入的变量在模块中没有,则报错
 - 

### 循环引用
循环引用就是两个文件相互依赖的情况。而 CommonJS 与 ES6 处理循环引用的方式也是不同的。
CommonJS 遇到循环引用，由于是同步引用，所以会将依赖的文件先执行的部分输出，未执行的部分不输出，那么
也就意味着 CommonJS 循环依赖必须有值返回，否则会出错。
但是 ES6 的模块引用是动态的，只是符号链接，所以只要代码里面有声明就可以运行。

以上就是 ES6 中最最常用的部分了.


#### updated in 2017/08/27