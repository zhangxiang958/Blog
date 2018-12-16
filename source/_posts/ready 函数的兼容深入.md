title: ready 函数的兼容深入
date: 2017-03-15 08:49:24
categories: JavaScript


---
原本以为高程里面说的模拟 DOMContentLoaded 方法是比较好的, 但是书也有自己本身的限制.
<!--more-->
##load 与 DOMContentLoaded
###load 
load 事件是什么时候执行的 ? load 事件是等待页面所有资源加载好了之后才触发的一个事件, 一般来说比较慢,因为如果
页面有大量的图片资源, load 事件会等待图片资源全部加载好之后触发, 这样的话我们知道浏览器渲染是渐进式的, 会先显示一部分已经加载好的网页, 如果我们把事件绑定都绑定在 load 事件里面, 页面在完全加载好而又显示一部分的时候会陷入假死状态, 可见但不可交互, 大大降低了用户体验, 因此我们很少会使用到 load 事件用于绑定事件.

*load 事件需要注册在 window 对象.*
###DOMContentloaded
DOMContentLoaded 事件是什么时候执行的 ? DOMContentLoaded 事件到底在等待什么? 
DOMContentLoaded 事件是会在完整的 DOM 树构建完之后触发, 不用等待图片等资源的加载, 网上有很多说法, 基本上是
DOMContentLoaded 事件是在 html, style, script 加载好了之后才触发, 但是在高程里面却是另外一种说法, 按照高程的说法, DOMContentLoaded 事件是不必等待图片, CSS 文件, JS 文件的加载, 在 DOM 树构建完成后就会触发, 这让我困惑,到底哪一种说法是正确的?
DOMContentLoaded 不等待使用 document.createelement('script'); 这样添加的 script, DOMContentLoaded 会等待
那些添加了 defer 属性的, 和 inline script, 使用 script 标签的 src 属性添加的脚本. DOMContentLoaded 会等待
这些脚本都执行完在触发. DOMContentLoaded 会确保 BODY 解析完与 HTML 文档解析完.
test demo:
```
    <div id="event1"></div>

	<script type="text/javascript">
        setTimeout(function(){
            var div1 = document.getElementById('event1');
            console.log(div1 + 'setTimeout');
        },0);
        document.addEventListener('DOMContentLoaded',function(){
            var div1 = document.getElementById('event1');
            console.log(div1);
        },false);
        window.addEventListener('load', function(){
            console.log('loaded');
        }, false);
        </script>
        <script type="text/javascript">
    	    for(var x=0;x<5e8;x++){}//模拟占用大大大量时间js代码
	    </script>
	    <script type="text/javascript" src="./src.js"></script>
	    <script type="text/javascript" src='./src2.js' defer></script>
	    <!-- <script type="text/javascript">
		    var script = document.createElement('script');
		    script.src = './src2.js';
		    script.setAttribute('defer', 'defer');
		    document.body.appendChild(script);
	<   /script> -->
```

####怎么兼容 IE 9 以下版本的浏览器? 
IE 9 以下版本的浏览器并不支持 DOMContentLoaded 事件, 我们需要 hack.
对于 IE 来说, 总体可以分为有 frame 与没有 frame 这两种, 而对于没有 frame 的文档来说, 可以使用那个著名的
hack -- document.documentElement.doScroll('left'); 来判断 DOM 树是否已经渲染好了
```

if(document.documentElement.doScroll) {
    function tryScroll(){
        try {
            document.documentElement.doScroll('left');
            ready();
        } catch(e) {
            setTimeout(tryScroll, 10);
        }
    }
    tryScroll();
}
```
如果是有 frame 的话, 上述的方法不成功, 因此使用 onreadystatechange 来代替 DOMContentLoaded 事件.
但是对于 readystatechange 事件来说, 它有 5 个状态, 分别是未初始化, 加载中, 加载完成, 交互, 全部完成. 但是
并不是所有元素加载过程都会有这几个状态, 而且交互状态也并不是每次都在全部完成状态之前, 而且也不能保证交互
状态一定会发生在 load 事件前面, 所以我们在使用 onreadystatechange 事件的时候倾向于监听两个状态交互与全部
完成.
```
document.addEventListener('readystatechange', function(){
    if(document.readyState === 'interavtive' || document.readyState === 'ready') {
        ready();
    }
}, false);
```

####高程里面所说的代码段是什么意思? 为什么可以模拟 DOMContentLoaded ?
高程里面的代码段是:
```
setTimeout(function(){
    //事件绑定
}, 0);
```
有人可能会有疑惑, 我看到 segmentfault 上有人提出这个问题, 首先这个问题先理清 DOMContentLoaded 的发生时刻,
目前建立的前提是 HTML 页面中的 script 标签都不是动态插入的, 也就是说 DOMContentLoaded 等待这些脚本同步执行
完成之后再马上触发, 那么如果浏览器不支持 DOMContentLoaded 事件, 我通过代码模拟当这些脚本都执行完了之后马上
触发一个函数不也可以达到类似的效果吗? 答案是通过 setTimeout 定时器在 0ms 时将事件插入到异步队列中, 那么根
距 JavaScript 函数的执行机制, 当 callback stack 回调栈中已经执行完了函数(即那些 script 标签脚本执行完了),
就会调用异步队列中的函数, 而里面就会有我们刚刚通过 setTimeout 添加的函数被执行了, 这也就是为什么定时器设
置 0 时刻可以模拟 DOMContentLoaded 事件了.
当然像高程里面也说了, 这个方法不能百分百地模拟 DOMContentLoaded 事件, 需要根据你的代码来定, 比如你的异步队
列中已经有很多待执行函数了, 而此时再通过 setTimeout 添加函数, 那么和 DOMContentLoaded 事件的执行时刻就会相
差地比较远,因为 setTimeout 添加的函数的时间不是代表过了多久函数执行的意思, 而是添加到队列的意思.所以高程里
面建议使用这种方法需要设置这个定时器为第一个定时器.但是这样也并不能保证定时器中的函数会先于 load 事件发生
之前.(可以看上面的 demo) 结果如下:
![](http://img.ijarvis.cn/domcontentloaded.png)

总的跨浏览器兼容代码这里不再阐述, 这里只是讲解了为什么要这么做, 详细的兼容代码可以看下面的链接.


##link

 - [demo addr](https://testdrive-archive.azurewebsites.net/HTML5/DOMContentLoaded/Default.html)
 - [onload and DOMContentLoaded](http://javascript.info/tutorial/onload-ondomcontentloaded)
 - [主流框架 DOMContentLoaded 事件实现](http://www.cnblogs.com/pigtail/archive/2012/06/18/2553556.html)


