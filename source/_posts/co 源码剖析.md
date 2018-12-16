title: co 源码剖析
date: 2018-02-25 18:06:24
categories: JavaScript

---

对于如何将生成器与 Promise 结合作为异步任务流程控制，这里我推荐阅读 co 类库的代码，co 类库是一个很优秀的异步流程控制的工具库，下面我们就阅读它的代码，领会它的思想:
<!-- more -->
co 有两种调用形式，一种是 co(fn), 一种是 co.wrap(fn), 两者原理是一样的，只是 co 会将传入的函数立刻执行，而 co.wrap 会返回一个函数，你可以理解为 co.wrap 返回一个函数表达式。
而 co 的思想在于不管 yield 后面的是什么，先全部转化为 Promise 对象，注意这里的不管是指是数组，对象，生成器对象，生成器，函数这几种类型，如果不是这几种类型就会报错的。
另外我们知道生成器对象需要一个一个调用 next 函数来逐个执行 yield 语句，但是在流程化控制里面我们不可能一个一个去执行，所以需要自动化地执行 next 函数。那么所谓自动化执行，在 co 里面就是将上面所说的转化为 Promise，然后为 Promise
添加 then 处理逻辑，成功后自动调用下一个 yield 也就是 next 函数。
上面就是 co 类库实现的核心思想，下面来看看代码：
```
function co(gen){
    // 记录当前作用域
    var ctx = this;
    // 记录 gen 函数传参，gen 可能需要传递参数
    var args = Array.prototype.slice.call(arguments, 1);

    // co 本身返回一个 Promise
    return new Promise((resolve, reject) => {
        if(typeof gen === 'function') gen = gen.apply(ctx, args);
        if(!gen || typeof gen.next !== 'function') return resolve(gen);
        
        // 先调用一次 generator, 启动 generator
        onFulfilled();
        // fulfilled 态处理函数
        function onFulfilled(res) {
            var ret;
            try {
                // 将 res 也就是异步任务的结果传入 next 函数, 那么对应 yield 语句的值就是 res
                ret = gen.next(res);  // ret 这里是 { value: val, done: false } 这样的对象
            } catch(err) {
                reject(err);
            }
            // 递归调用 next 函数
            next(ret);
            return null;
        }
        
        // reject 态的处理函数
        function onRejected(res) {
            var ret;
            try {
                // 利用 throw 函数抛出错误
                ret = gen.throw(res);
            } catch(err) {
                reject(err);
            }
            //递归调用 next
            next(ret);
        }
        
        function next(ret) {
            // 如果 done 为 true, 结束流程
            if(ret.done) return resolve(ret.value);
            // 将 value 转化为 promise 对象
            var value = toPromise(ret.value);
            // 这里需要检测 value 是不是一个 promise 对象, value 并不一定是一个 promise 对象
            // 如果是则为该 promise 添加 resolve 与 reject 处理函数
            if(value && isPromise(value)) return value.then(onFulfilled, onRejected);
            // 如果不是一个 promise 对象, 说明 toPromise 函数转化不成功,抛出错误
            return onRejected(new TypeError('you only can yield array, object, function, generator'));
        }
    });
}
```
我们可以看到实际上就是将 yield 后面的东西包装成一个 Promise 对象, 然后等待 Promise 完成之后, 将结果传入
next 函数中实现 yield 语句的替换(var ret = yield xPromise), 然后通过递归调用 next, 也就是递归调用 gen 的
next 函数实现流程自动化.
然后接下来就是最重要的 toPromise 函数的实现, 这个函数支持对象, 数组, 偏函数, generator 的 primise 转化:
```
function toPromise(obj) {
    if(!obj) return obj;
    if(isPromise(obj)) return obj;
    // 如果是生成器相关直接调用 co 函数
    if(isGenerator(obj) || isGeneratorFunction(obj)) return co.call(this, obj);
    // 对偏函数进行转化
    if(typeof obj === 'function') return thunkToPromise.call(this, obj);
    // 对数组进行转化
    if(Array.isArray(obj)) return arrayToPromise.call(this, obj);
    // 对对象进行转化
    if(isObject(obj)) return objectToPromise.call(this, obj);
    // 不属于上面的类型直接返回
    return obj;
}
```
对于生成器的 promise 转化无需多说, 也就是 co 函数的逻辑. 对于偏函数的 promise 转化实际也很简单, 也就是包裹
一层 promise.所谓偏函数就是使用了函数柯里化固定了某若干个函数参数的函数, 举个例子:
```
function readFile(path) {
    return function(callback){
        fs.readFile(path, callback);
    }
}
// 如果使用 co 类库，这里的 callback 是 co 自动填入的，详细请看下面源码
var file = yield readFile('a.js');
```
所以对应的转化函数为:
```
function thunkToPromise(fn) {
    var ctx = this;
    return new Promise(function(resolve, reject){
        // 调用函数，自动传入 callback 函数，callback 的作用是将返回结果传入 promise 中
        // 然后通过 promise(onFulfilled 函数) 中 generator 函数的 next 函数进行 yield 语句返回值替换
        fn.call(ctx, function(err, res){
            if(err) return reject(err);
            // 这里的 arguments 不是指 thunkToPromise 的， 而是指当前这个 function回调的
            // 这里这个 if 判断是为了支持回调传入多个参数的，如果返回值是以 callback(null, 1, 2, 3);
            // 这样的方式进行回调，那么 res 就是 [1, 2, 3].
            if(arguments.length > 2) res = Array.prototype.slice.call(arguments, 1);
            resolve(res);
        });
    });
}
```
对于数组的转化, 需要注意的就是数组内部递归的转化, 数组里面的值可能是复杂类型的:
```
function arrayToPromise(array) {
    // 使用 map 函数进行递归转化
    return Promise.all(array.map(toPromise, this));
}
```
对于对象的转化, 需要注意的是如果对象属性值为基本类型, 那么直接将对应值附到结果集对象中，如果是一个异步任务(promise, function 等)，就需要转化为 Promise，当结果返回再将值
附到结果集上：对于属性值是否是基本类型可以通过 toPromise 函数本身来做判断
```
function objectToPromise(obj) {
    var results = new obj.constructor();
    var keys = Object.keys(obj);
    var promises = [];
    for(var i = 0; i < keys.length; i++) {
        var key = keys[i];
        var value = toPromise(obj[key]);
        if(value && isPromise(value))  defer(key, value);
        else results[key] = value;
    }

    return Promise.all(promises).then(function() {
        return results;
    });

    function defer(key, promise) {
        results[key] = undefined;
        promises.push(promise.then(function(res) {
            results[key] = res;
        }));
    }
}
```
说到这里，co 的核心代码已经细读过一遍了。
## 一些类型判断
上面的就是 co 类库中的核心代码，接下来就简单说说 co 源码中用到的一些数据类型判断函数：
### 1. 判断 Promise 对象：使用鸭子模型进行判断
```
function isPromise(obj) {
    return obj && typeof obj.then === 'function';
}
```
### 2. 判断 generator 对象：使用鸭子模型进行判断
```
function isGenerator(obj) {
    return obj && typeof obj.next === 'function' && typeof obj.throw === 'function';
}
```
### 3. 判断 generator 函数：
```
function isGeneratorFunction(obj) {
    var constructor = obj.constructor;
    if(!constructor) return false;
    if('GeneratorFunction' === constructor.name || 'GeneratorFunction' === constructor.displayName) return true;
    return isGenerator(constructor.prototype);
}
```
思考一下这里为什么需要递归使用 isGenerator 函数进行判断？
这个是因为可能存在自定义迭代器， 用户可能有特殊需要进行自定义迭代器:
```
function makeIterator(array) {
    var index = 0;
    return {
        next: function(){
            return index < array.length ? { value: array[index], done: false } : { done: true };
        }
    }
}
```
上面的 makeIterator 也是一个生成器函数，只不过是自定义的，上面的代码是简单的说明，如果是实际使用，
还应该加上 throw 函数的实现。

### 4. 判断数值是否是对象：
```
function isObject(obj) {
    return Object === obj.constructor;
}
```
## 一些关于源码的思考与收获
### 为什么要将 onFulfilled，onRejected 等函数包裹在一个 Promise 里面呢？
原本在旧版本的 co 类库中， 会创建一个 Promise 链，并且这个链会因为一直保持引用而不会被 v8 回收:
```
// 简单模仿旧代码
function co() {
    ....
    return Promise.resolve(onFulfilled());

    function onFulfilled() {
        ...
        return next();
    }
    
    function next(res) {
        if(res.done) return Promise.resolve(res.value);
        ....
    }   
}
```
像上面的代码会创建一个 Promise 链，而 next 中使用 Promise.resolve 是会创建一个新的 Promise 的，而且这个 promise 因为保持引用所以无法被 v8 回收。
所以在 co@4.x 修复了这个 bug，将 onFulfilled，onRejected， next 函数包裹在一个 Promise 中，并且不直接 return Promise.resolve 而是直接使用 resolve。
### throw 语句与 throw() 的区别
说说 co 中为什么调用 generator 的 throw 而不是直接 throw, throw 语句是肯定不能用的, 因为这里包裹了很多层函数
直接 throw 没有任何作用, 异步函数外层的 try...catch 无法捕获内部的错误。
而使用 generator 的 throw 函数, 可以抛出一个错误, 也就是说可以通过以下方式捕获:
```
try {
    var ret = yield doFuture();
} catch(err) {
    console.log(err);
}
```
可以使用 try...catch 捕捉错误而不影响后续的函数执行.





