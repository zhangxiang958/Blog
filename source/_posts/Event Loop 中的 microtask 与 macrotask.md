title: Event Loop 中的 microtask 与 macrotask
date: 2018-02-03 15:24:24
categories: JavaScript

---
Javascript 的事件循环会被常常提及, 而且在实际开发中, 经常需要使用事件相关的知识, 所以特地深入了解一下.
<!--more-->
## 深入 event loop
事件循环是用来做异步任务处理的, 与之相同的做异步任务处理的还有多线程, 但是由于 javascript 的单线程特性, 最终使用 event loop 的方式.
或许你可以从下面的简略的伪代码看出 event loop 是什么:
```
eventQueue = [];
event;
while(1){
    if(eventQueue.length > 0) {
        event = eventQueue.shift();

        try {
            event();
        } catch(err) {
            reportError(err);
        }
    }
}
```
event loop 有两种, 一种是在浏览器上下文, 一种是在 worker 上下文的.浏览器上下文一般会至少有一个 event loop, 像一个 iframe, 浏览器窗口
都会有一个 event loop, 而对于 worker 上下文的则比较简单, worker 进程管理着一个 event loop.这里着重将浏览器上下文的 event loop.
### event loop 运行流程
根据规范 [event loop](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops) 所讲的, 它的流程如下:
```
当一个 event loop 存在, 它会按照以下的步骤运行:
1. 在最老的任务队列中取出最老的任务, 如果没有任务, 那么就会跳到 microtask 队列的执行.这里的最老我个人理解是像任务调度算法中的等待时间最长的意思.
2. 将 event loop 当前任务设置为最老的那个任务.
3. 执行当前任务队列中最老的那个任务
4. 将 event loop 当前运行任务设置为 null
5. 将刚刚执行的那个最老的任务从它的队列中移除
6. 执行检查 microtask 队列算法(microtask checkpoint 这个稍后详谈), 这里先粗略理解为执行 microtask 队列
7. 更新渲染(update the rendering)
8. 如果是一个 worker event loop, 但是没有任务, 并且 WorkerGlobalScope 对象的 closing flag 值为 true 的, 就销毁 event loop 并中止这些步骤,
并进行 web worker 中的 run worker 算法.
9. 返回到第一步, 此为一个事件循环.
```
由上面的步骤我们可以知道, 在一个循环当中, 每执行一个任务, event loop 都会尝试去清空 microtask 队列, 也就是对应的第六步.同时我们可以看到, 在做完
上面的操作之后, 才会进行渲染操作, 防止过多的操作重复渲染造成性能问题.
### microtask checkpoint
每一个 event loop 都有一个 microtask 的队列.我们从规范看到 [event loop](https://html.spec.whatwg.org/multipage/webappapis.html#performing-a-microtask-checkpoint), microtask 队列的流程如下:
```
如果用户代理的 checkout point flag 值为 false 的时候, 就会按照下面的步骤进行执行:
1. 设置 performing a microtask checkpoint flag 值为 true.
2. 当 microtask 队列不为空时:
    2.1 选择队列中最老的任务队列
    2.2 设置当前运行任务为选择的最老任务
    2.3 执行这个最老的任务
    2.4 设置当前运行任务为 null
    2.5 将刚刚运行的任务从它的任务队列中移除.
    2.6 回到 2
3. 每一个 environment settings object 他们的 responsible event loop 就是当前 event loop, 会给 environment settings object 发出一个 rejected promise 的通知.
4. 清理 indexed db 事务
5. 将 performing a microtask checkpoint flag 设置为 false
```
### microtask 与 macrotask 的区别
这个应该是 event loop 中比较核心的问题, 究竟 timer 一类设定的 macrotask 与 promise 一类设定的 microtask 有什么区别?
从上面对规范的解读可以看出, microtask 与 macrotask 在执行上有区别, 一次 event loop 会取一个 macrotask 执行, 但是会将一个 microtask 队列
清空, 也就是说, 如果一个 microtask 队列过长, 确实会阻塞下一个 macrotask 的开始执行时间.可以看出, 在异步中, js 虽然是异步非阻塞, 但是却是使用
**同步**的方式来执行 microtask 的.
另外从字面上来说, macrotask 属于 task, 也就是大型任务, microtask 属于 job, 也就是小型任务, 而对于如何更详细的区分, 规范并没有说, 而是从产生类型
上将两类分开:
```
macroTask: setTimeout, setInterval, setImmediate, I/O, rendering
microTask: promise, process.nextTick, Object.observe, MutationObserver
```
我提供一个巧记的方式, 越靠近定时器一类的就是 macrotask, 越靠近 promise 一类的是 microtask.
至于什么时候需要使用 microtask 呢? 我觉得这个问题很好地指出两者(macrotask, microtask)的不同, 在你觉得需要将这个异步任务同步化的时候, 就使用
microtask , 否则就使用 macrotask.换种说法, 也就是这个任务你需要尽可能快地执行, 就使用 microtask.
举个例子:
```
console.log('script start');

setTimeout(() => {
    console.log('setTimeout');
}, 0);

Promise.resolve().then(() => {
    console.log('promise1');
}).then(() => {
    console.log('promise2');
});

console.log('script end');
```
上面的例子在 chrome 中的顺序是 script start, script end, promise1, promise2, setTimeout.
### event loop 的产生源
event loop 的 macrotask 的产生源有很多个, 其中包括:
1. DOM 元素操作, 比如以非阻塞的方式插入一个元素
2. 用户交互
3. 网络
4. history 操作源, 比如 history.back() 等等.

task 的任务源很多, 像常见的 ajax, setTimeout, DOM click 事件都可以产生任务.当然, 不同的任务源会被加到不同的任务队列中去.比如 ajax 操作的异步非阻塞任务就会被加到 ajax 源的
队列中, DOM 事件产生的任务就会被添加到 DOM 事件的任务队列中去.

### 总结
下面, 我使用一个图来总结一下:
![](http://img.ijarvis.cn/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20180205123559.jpg)

参考资料:
1. https://html.spec.whatwg.org/multipage/webappapis.html
2. https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/