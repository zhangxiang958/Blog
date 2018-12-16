title: Zepto 点击穿透问题深究与解决
date: 2017-03-04 23:04:24
categories: Web Develop 
---

##问题现象
在项目中,遇到的问题是有一个弹出层, 弹出层有一个按钮点击之后表示操作完成并且隐藏遮罩与弹出框,但是点击按钮之后
弹出层下面的元素却触发了 click 事件,导致 bug 的出现.
<!--more-->
##问题根源
首先, 在原生是没有 tap 事件的, 工具库中的 tap 事件是通过 touch 事件封装出来的, 基本上是下面这种顺序执行事件
```
touchstart -> touchmove -> touchend -> tap
```
通俗来讲, 可以理解为 tap 只是一个自定义事件, 当我们的手指点击到屏幕时, 触发了 touch 事件, 然后根据 Zepto 的
源码显示, 在 touchend 事件之后没有操作,也就是没有再次点击的话,就会使用定时器在 250 ms 后触发 tap 事件,从而
使 tap 的回调函数触发, 一般来说我们回调会隐藏函数等等.
第二点, 在移动端, 我们仍然需要 click 事件的支持, 因为响应式网站的存在, 因此通常是由 touch 事件带动触发 mouse
事件的, 触发事件顺序如下:
```
touchstart -> touchmove -> touchend -> click
```
其中, 在 touchend 事件结束之后, 浏览器会等待 300ms 观察用户是否会再次触发 touch 事件也就是是否有双击行为, 如
果没有的话, 那么就会执行 click 事件
###Zepto 为什么要封装 tap? 
这是为了解决 click 在移动端设备有 300ms 的延迟, 因此封装了一个 tap 事件, 在 touchend 后 250ms 触发, 这也就是
为什么 tap 事件不能加到 320ms 或者 350ms 以及更多时间后响应,因为它本身的出现就是为了解决移动端 click 的 300
ms 的延迟出现,加快事件响应速度的, 如果是 300ms 以后根本没有意义.所以说 tap 在 300ms 之前执行不是在搞事情
那现在比较清晰了, tap 事件先于 click 事件执行, 因此当遮罩消失后, 也就是 tap 事件生效之后, click 事件会执行
但是先前点击的遮罩的位置遮罩已经消失了,此时如果有一个元素刚好绑定了 click 事件或者是 input 框,那么就会触发
click 事件或者 input 有焦点.

**移动端 click 300ms 延迟不是为了搞事情而是浏览器需要知道用户到底是想要点击还是双击放大**.
###能不能阻止浏览器的默认行为?
理论上可以通过阻止 touchend 事件的默认行为,但是在 Zepto 中,由于 tag 事件是封装出来的, 而 tag 的触发是通过
setTimeout 定时器在设置 250ms 后触发的(详细可以看 Zepto 的 touch 模块), 换言之在回调中调用 event 对象的
event.preventDefault 根本无效, 因为 tag 事件根本没有事件对象, 更重要的是定时器是异步队列,也就是说 touchend
事件完成之后我们再在回调中阻止事件根本一点作用都没有.
而且这个方法遗憾的是并不是所有浏览器都支持, 安卓与 IOS 移动浏览器的不同.
##解决方法
###在遮罩之后加一个透明的 div 在 350ms 后消失
但是会增加一个累赘的 div 元素.
###遮罩使用动画在 350ms 后消失
基本是不用添加任何东西的情况下的最好选择
###使用 CSS3 pointer-events
在 tap 事件触发的时候, 将下层元素添加一个属性 poniter-events 为 none, 然后下层元素就不会响应 click 事件了,
也就是说设置了 poniter-events 为 none 不会成为 click 事件的目标, 然后设置一个定时器在 tap 事件响应后的
400ms 后将 pointer-events 设置为 auto 恢复正常.有人说如果下层元素并不是一个简单的层呢, 而是很多很多元素
那该怎么办? 这种情况下这个属性仍然适用, 因为它具有继承性, 我们只要在下层父级层添加就好了.
###使用 fastclick
使用了 fastclick 之后,所有点击事件就可以都用 click 事件了,因为已经解决了 click 事件原本的 300ms 延迟问题.
而对于 fastclick 的原理, 就是取消 touchend 延迟 300ms 后的 click 事件, 基本是通过 event.preventDefault()
与某些浏览器的兼容(比如需要取消 mousemove 的默认行为, 因为它有时会比 touchstart 还快而且触发 click), 然后
在 touchend 事件发生之后, 获取了点击元素, 并且马上触发了 click 事件,这样响应速度上来了,也不会点击穿透,需要
注意的是用了 fastclick 之后就使用 click 处理点击吧, 因为响应速度已经上来了,并且 tap 有点击穿透的现象(因
为 tap 事件的 setTimeout 原因).



归根结底, 无论是 Zepto 的 tap 事件还是 fastClick 都是在解决移动端 click 300ms 延迟问题.个人觉得 fastclick
做得更好.

##好文推荐:

 - [深入移动端事件](http://www.cnblogs.com/yexiaochai/p/3462657.html)