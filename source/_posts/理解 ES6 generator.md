title: 理解 ES6 generator
date: 2018-02-25 18:06:24
categories: JavaScript

---

工作中经常使用 generator, 让我们来全面了解一下它.
<!-- more -->

## 生成器的类型判断
### 如何判断生成器对象
```
function isGenerator(obj) {
    return obj && typeof obj.next === 'function' && typeof obj.throw === 'function';
}
```
与 promise 对象类似，这里运用鸭子模型进行判断，如果对象中有 next 与 throw 两个方法，那么就认为这个对象是一个生成器对象。
### 如何判断生成器函数
```
function isGeneratorFunction(obj){
    var constructor = obj.constructor;
    if(!constructor) return false;
    if('GeneratorFunction' === constructor.name || 'GeneratorFunction' === constructor.displayName) return true;
    return isGenerator(constructor.prototype);
}
```
利用函数的 constructor 构造器的名字来判断，为了兼容性使用 name 与 displayName 两个属性来进行判断. 这里递归调用 isGenerator 判断 constructor 的原型是因为有
自定义迭代器的存在.
## yield 与 next 传值问题
这个问题的答案需要清楚， 因为单独的 generator 是没有办法做异步任务流程化的。
```
function *gen() {
    var re = yield 1;
    return re + 2;
}

gen = gen();
var g = gen.next();
console.log('1: ', g.value);
g = gen.next();
console.log('2: ', g.value);
g = gen.next();
console.log('3: ', g.value);
```
猜一下以上代码的 value 值分别为多少? 答案是 1, NaN, undefined. 为什么会出现这样的结果呢? gen 生成器函数本意是想做一个 1 + 2 简单的加法运算, 但是最后得到的结果是 NaN. 其实 generator 函数内部的 yield 是需要我们一个一个
使用 next 函数去调用一步一步得到的. 而 next 函数调用之后, 返回一个对象, 对象中有两个属性值:
```
{
    value: val,  // 任意值
    done: true // 或者 false
}
```
而其中的 value 的值就是 yield 语句后面的 "值", 注意这里的值不一定是 javascript 中的基本数据类型，yield 后面可以是函数，表达式(运算结果)，对象等等的值。可以查看下面的例子:
```
function *gen(){
    yield 1;
    yield { who: 'me' };
    yield function(){
        console.log('hello');
    };
    yield 1 + 2;
}

gen = gen();
var g1 = gen.next();
console.log(g1.value);  // 1
var g2 = gen.next();
console.log(g2.value);  // { who: 'me' }
var g3 = gen.next();
console.log(g3.value);  // function(){ console.log('hello'); }
var g4 = gen.next();
console.log(g4.value); // 3
```
那么问题出现了，既然 yield 后面的值可以通过 next 方法得到，那 yield 语句本身有没有返回值或者 yield 语句返回值如何得到呢？这个问题是 generator 异步流程控制的第一个问题， 上面说到 yield 后面是可以接函数或者表达式或基本值等等，但是
如果像 var tmp = yield 1; 这样的语句， tmp 变量并没有取到 1 的话，流程控制就无从谈起，甚至 yield 与生成器函数本身的作用也不太大， 所以这里就需要一个消息数据双向通道。
所谓消息双向通道就是我们可以从 next 函数中拿到 yield 语句后面的值，然后可以通过 next 函数传值把传进去的值变为 yield 语句的返回值。所以这也就解释了第一段代码为什么得到值是 NaN 了， 因为在处理的时候，我们没有给 next 函数传值，导致
yield 语句返回值为 undefined， undefined + 2 得到的当然就是 NaN.
这里还要解释一下第一段函数最后没有 yield 语句，但是 value 还有有值， 而且得到最后的结果 NaN，因为 value 的值是如果存在 yield 那么, 它的值就是 yield 后面接着的值, 如果没有 yield 那么就取 return 后面的值, 若都没有则返回 undefined(其实是相当于有个默认的 return undefined).
所以为了实现第一段代码的功能，我们需要：
```
function *gen() {
    var re = yield 1;
    return re + 2;
}

gen = gen();
var g = gen.next();
console.log('1: ', g.value);  // 1
g = gen.next(g.value);
console.log('2: ', g.value);  // 3
g = gen.next();
console.log('3: ', g.value);  // undefined
```
最后一个是 undefined 是因为 yield 语句数量只有一个，在调用两个 next 函数之后已经结束( done === true )了， 所以 value 为 undefined.
## next 函数与 yield 个数不对等?
先看一个例子:
```
function *gen(){
    var ret = yield 1 + 2;
    var num = yield 43;
    ret = yield ret + num;
    console.log(ret);
}
```
我们怎么实现 1 + 2 + 4 这个功能呢？来看以下代码:
```
gen = gen(); // 转换为 generator 对象
var g = gen.next(); // 第一步启动
var firstStep = g.value; // 取得第一个 yield 后面的表达式，即 1 + 2 为 3 
g = gen.next(firstStep); // 从 next 函数传入第一步的值，则 ret 值为 3
var secondStep = g.value; // 取得第二步 yield 的值，即 4
g = gen.next(secondStep); // 从 next 函数传入第二步的值， 即 num 为 4
// 由于下面没有了 yield 语句，所以直接执行完函数，ret 为 { value: 7, done: false }
g = gen.next(g.value);
console.log(g.done); // true
```
可以看到， 其实第一个 next 函数是为了启动 generator，因为在还没有启动的时候，前面还没有 yield 语句，
所以即使你往第一个 next 函数中传值也没有用，它不会替代任何一个 yield 语句的值，所以我们会倾向于不向
第一个 next 函数中传值(undefined)。
然后接下来你可以看到，第一个 next 函数之后每个 next 函数对应着一个 yield 语句, 其实 next 函数通俗来讲就是
运行上一个 yield 语句与当前 yield 语句之间的代码, 而 next 函数传进去的值会变成上一个 yield 语句的返回值.
由此可知, next 函数调用次数与 yield 语句个数总是不对等, next 函数调用次数总是比 yield 语句多 1, 因为需要
第一个进行启动.
## 异步流程控制
单独的生成器作用并不大, 特别是在异步流程控制中, 即使 yield 后面可以添加异步任务, 但是我们仍然需要一个一个
地调用 next 函数, 如果需要流程化控制, 就需要自动执行 next 函数.而对于 yield + promise 结合的异步流程控制,
核心思想就是通过 next 函数取得 yield 后面的值, 然后将这个值转化为 Promise, 通过 Promise 来控制异步任务, 
异步任务完成后递归重新调用 next 函数重复上面的操作.这里我推荐阅读 co 类库的源码, 它的思想与上面一致, 是
yield + promise 的精妙实现.具体可以查看我的另一篇博文-- [co 源码剖析](http://zhangxiang958.github.io/2018/02/25/co%20源码剖析/).
这里我们试试写一个简单的 promise + yield 控制器来理解一下:
```
function run(gen){
    var args = [].slice.call(arguments, 1);
    if(typeof gen === 'function') gen = gen.apply(this, args);
    
    return new Promise(function (resolve, reject) {
        onFulfilled();
        
        function onFulfilled(res){
            var it = gen.next(res);
            next(it);
        }
        
        function next(res) {
            if(res.done) return resolve(res.value);
            return Promise.resolve(res.value)
                    .then(onFulfilled);
        }
    });
}
```
注意此时 yield 后面的东西需要是一个 Promise 对象.

## yield 与 yield * 的区别
在实际场景中, 异步任务可能并没有像 demo 中那么简单的关系, 可能会有 A-B-C 的异步任务, 而 B 中又包含 B-1, 
B-2, B-3 这样的异步任务, 按照逻辑是需要完成 A 后再逐个完成 B-i 再执行 C.
问题在于不可能在一个函数内完成全部逻辑, 我们会需要在多个函数中编写逻辑, 这个时候 yield 后面可能需要加上
一个包裹型的函数:
```
function *doTask() {
    var a = yield A();
    var b = yield run(B);
    var c = yeld C();
}

function *B() {
    var b1 = yield B1();
    var b2 = yield B2();
    var b3 = yield B3();
}

run(doTask);
```
我们可以使用上面编写的 run 函数来做管理, 但是存在更好的方式就是使用 yield *, 注意这里和常规的 yield 多了
一个 * 符号.其实 yield * 的"学名"叫做 yield 委托, 看下面的例子:
```
function *foo() {
    console.log('start *foo');
    yield 3;
    yield 4;
    console.log('end *foo');
}

function *bar(){
    yield 1;
    yield 2;
    yield *foo();
    yield 5;
}

 var g = bar();
 console.log(g.next().value); // 1
 console.log(g.next().value); // 2
 // start *foo
 console.log(g.next().value); // 3
 console.log(g.next().value); // 4
 // end *foo
 console.log(g.next().value); // 5
```
我们可以看到, 使用 yield * 可以让函数先进入另外一个生成器函数内部执行完内部的步骤之后, 再返回上一层继续
执行上一层剩下的步骤.
简单地来说, yield * 提供了调用生成器函数的方法, 由于生成器方法的特殊, 所以 generator 提供了一个特殊的方式
调用生成器函数.好处在于你可以简单地执行嵌套的 yield, 而无需自己编写像 run 函数这样的工具.
yield * 除了后面加我们自己编写的生成器函数, 还可以加非一般的生成器函数, 比如数组的迭代器:
```
yield *[1,2,3];

it.next().value; // 1
it.next().value; // 2
it.next().value; // 3
```
但是注意这里这样的 yield * 语句是始终没有返回值的, 或者说是 undefined.

## ES5 中的生成器
生成器是 ES6 中的语法, 但是 ES6 的语法都是可以通过工具来转化为 ES5 的语法的, 为了更全面地认识生成器, 我们
来自己转化一下, 先假定有一个生成器函数:
```
function bar(){
    return new Promise(function(resolve, reject){
        request('www.test.com/api', function(res){
            resolve(res);
        });
    });
}

function *foo() {
    try {
        console.log('start yield');
        var tmp = yield bar();
        console.log(tmp);
    } catch(err) {
        console.log(err);
        return false;
    }
}

var it = foo();
```
既然是转化为 ES5 的语法, 那么变量 it, 也就是 foo 函数的返回值就应该是:
```
function foo() {
    ....
    return {
        next: function(){
            ...
        },
        throw: function(){
            ...
        }
    }
}
```
我们知道 yield 可以"暂停"函数运行, 那么换言之, 在内部就应该有一个对应的状态变量来标记函数运行到哪一步, 而
对于控制来说, switch 语句很合适, 根据变量采取对应的操作: 看一下完整代码
```
function foo() {
    var state;  // 全局状态, 一开始值并不是 1, 因为需要一个 next 函数来启动
    var val; // val 代表的是 yield 语句的返回值
    function progress(v) {
        swtich(state){
            case 1:
                console.log('start yield');
                return bar();
            case 2:
                val = v;
                console.log(v);
                // yield 之后没有 return, 默认提供一个 return;
                return;
            case 3:
                // catch 中的逻辑
                var err = v;
                console.log(err);
                return false;
        }
    }
    // 生成器返回值
    return {
        next: function(v){
            if(!state) {
                state = 1;
                return {
                    done: false,
                    value: progress()
                }
            } else if(state == 1){
                state = 2;
                return {
                    done: true,
                    value: progress(v)
                }
            } else {
                return {
                    done: true,
                    value: undefined
                }
            }
        },
        throw: function(e){
            if(state == 1) {
                state = 3;
                return {
                    done: true,
                    value: progress(e)
                }
            } else {
                throw e;
            }
        }
    }
}
```
也就是说在转化为 ES5 的语法的时候, 生成器函数中的代码逻辑被分段, 上一个 yield 与下一个 yield 之间的代码被
放到 switch 语句的中一个 case 中去, 然后根据一个闭包中的状态变量 state 执行一个个 case.然后 switch 包含在
一个类似 progress 可供重复执行的函数中. progress 可以传入值, 值记录在一个闭包变量 val 中, 会使用在需要 
yield 语句返回值的地方( var v = yield fn(), v 的值就是闭包变量 val 的值), progress 函数是一个私有方法.
然后原生成器函数变为一个函数, 包含以上的逻辑, 返回值是一个对象, 其中包含 next 函数与 throw 函数的实现.
next 函数通过判断 state 的值, 不断将 state 值改变, 比如当 state 为 1 时, 执行 progress case 为 1 的逻辑,
然后修改 state 为 2(下一个状态), 当然 next 函数也可以直接传值, 传入值直接传到 progress 中, 然后执行刚说的
val 的逻辑.
throw 函数就是传入一个错误, 将初始状态(1)转为错误状态, 执行错误时的逻辑( progress ), 其他的状态直接抛出错
误 throw.




