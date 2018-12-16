title: JavaScript 学习笔记——BOM
date: 2016-03-13 13:23:24
categories: JavaScript
---
这部分是学习原生 JavaScript 的学习笔记， BOM 的相关知识，BOM 方面会注重讲述深入定时器。
<!--more-->
##BOM
###location 对象
对于 location 对象，我们最常用的就是使用它来获取 url 的参数名值对。使用的是 location 的 search 属性。
```
function getQueryStringArgs(){

    //取得查询字符串并去掉开头的问号,search 属性
    var qs = (location.search.length > 0 ? location.search.sustring(1) : "");
    
    //保存数据的对象
    var args = {};
    
    //取得每一项名值对
    var items = qs.length ? qs.split("&") : [];
    var item = null, name = null, value = null;
    
    for(var i = 0, len = items.length; i < len; i++){
        item = items[i].split("=");
        name = decodeURLComponent(item[0]);
        value = decodeURLComponent(item[1]);
        
        if(name.length) {
            args[name] = value;
        }
    }
    return args;
}
```
还有对 URL 的位置操作，如果需要对 URL 的位置进行操作的话，常用的就是 location.assign() 方法。它接受的
是一个 URL ，用于改变当前访问的 URL。
而如果要禁用浏览器后退按钮，就要使用 location.replace() 方法，将 url 重定向至指定的 url 中。无法使用后退
按钮。
###客户端检测
对于客户端检测，如果简单地需要知道用户使用的浏览器是什么，可以使用 navigator 对象。常用的就是使用它来检测
浏览器安装了什么插件，navigator.plugins 属性保存了浏览器安装了的信息数组。
但是如果需要代码以较强的兼容性运行在各个浏览器，那么就不能只是简单地检测浏览器的名称或版本。
####能力检测
一般来说，我们是通过检测浏览器有没有某种能力，而不是检测浏览器是哪个浏览器。进行能力检测的时候，先检测能
达到目的的最常用的特性。
```
//能力检测
if(obj.hasSomeAbility){
    //doAbility
}
```
我们进行检测的时候，不是仅仅检测某个对象有该属性，而是要检测这个对象的这个属性是不是我们想要的达到目标的
属性。比如假设我们需要使用一个 obj 对象 的 sort 函数。
```
var obj = {
    sort: "sort"
};

if(obj.sort){
    return !!obj.sort;
}
```
上面的例子是错误的，因为 sort 并不是一个函数，它只是一个属性，因此更可靠的能力检测是通过 typeof 检测我们
需要的那个属性是否是合适的类型。
```
if(obj.sort){
    return typeof obj.sort == "function";
}
```
####怪癖检测
怪癖检测的目标是检测浏览器的特殊行为。因为浏览器的某些 BUG 可能被修复也可能没有，因此我们需要检测这个 BUG
是否存在，从而判断下面的代码如何编写。建议在脚本开始仅仅检测那些有影响的怪癖。
####用户代理检测
用户代理是客户端检测不到万不得已不会使用的一个方法。用户代理头部通常会加载 HTTP 响应头部上，我们可以通过
navigator.userAgent 属性访问，但是这个方法不可靠在于用户代理头部存在电子欺骗。所谓电子欺骗就是浏览器之战
遗留的问题，IE 为了浏览器份额修改了自己的头部伪装成 Nesscape 的浏览器。其中用户代理检测中最常用的就是检测
呈现引擎，通常这样就已经可以编写正确的代码了。
用户代理检测的原理就是通过获取 navigator.userAgent 字符串，通过正则匹配，检测是否是想要的浏览器。
其中需要注意的点是，因为用户代理字符串不一致，因此要按照正确的顺序来检测。第一个检测的是 opera，第二步需
要检测的是 Webkit，第三步需要检测的是 KHTML，最后一步就是 IE。
###定时器
setTimeout(fn,time)，隔一段时间后执行一个回调函数或者一段代码，每次调用定时，返回一个特定 ID。
setInterval(fn,time)，每隔一段时间执行一个回调函数或者一段代码，每次间歇定时，返回一个特定 ID。
clearTimeout(id)，clearInterval(id) 用于清除定时器。
setTimeout 看起来与 setInterval 没有多大区别，其实不然，setTimeout 的回调函数只执行一次，而 setInterval
是无限次调用，直到定时器被清除或浏览器关闭。
```
function fn(){
    console.log(new Date());
    setTimeout(fn, 1000);
}
setInterval(function() {
    console.log(new Date());
},1000);
```
上面这两段代码看起来没有什么不同，其实不是，如果代码段的执行时间大于 1 秒，setTimeout 会等待代码执行完，
而 setInterval 是强制某段时间后执行代码段而不管代码有没有执行完，因此我们常说 setInterval 不精准就是这个
原因。
很多人会对两个定时器后面的时间误解，认为是过了某段时间，回调函数一定执行，实际上不一定，其实那个时间是
将回调函数加入到回调队列里面的时间，当 JavaScript 的执行栈为空时，才会执行回调队列中的函数。这也就是
[事件处理机制](http://zhangxiang958.github.io/2015/07/15/%E8%AF%95%E8%B0%88JavaScript%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86%E6%9C%BA%E5%88%B6/)的知识。
```
var start = 0;
function move(){
    var a = 0;
    console.log(a);
    a ++;
    start = setTimeout(move, 1000);   //模拟 setInterval
}
```
setInterval 会导致事件回调的函数堆叠，建议不要使用。我们谈谈这个模式为什么好？
因为 setInterval 是间歇调用函数，每隔一段时间就会向队列添加定时器代码，但是由于浏览器在运行 setInterval 定时器代码的时候，如果队列中如果已经有定时器代码的话，就不会再为队列添加定时器代码了，导致某些定时器代码
会被跳过，造成一些不可预测的时间间隔。
在为某些函数添加 setInterval定时器的时候，如果函数本身执行时间大于setInterval的定时器代码添加到队列的时间的话，在原来代码执行完毕的时候，就会立刻执行定时器代码，这并不是我们想要的，我们想要的是当函数代码执行后
过了某段时间，才重新被执行。
![setInterval](http://7xns9g.com1.z0.glb.clouddn.com/setInterval.png)
如果是使用上面的 setTimeout 代码就可以避免这个问题，因为定时器是加在代码最后，当前面的代码，即我们的函数
代码执行完，才会又一次将定时器加入到队列中去，保证了时间间隔。
对于定时器的使用，我在初学时遇到一个坑，就是定时器的第一个参数不一定是函数，它可以是一个字符串，请看下面的例子。
```
function fn(){
    //something
}
function bar(){
    function fn(){
        //do
    }
    setTimeout(fn(),1000);
}
bar();
```
如果是像上面的代码的话，就会执行第一个 fn 函数，因为定时器第一个参数如果传入的是一个字符串的话，那么
就会隐式调用一个 eval 函数，这样 eval() 函数会在全局对象下执行。并且如果是像上面这样使用定时器调用函数
的话，感觉是对 JavaScript 的函数理解不够，因为函数名是一个指针，我们要调用的是函数名所指向的函数段，如
过写成 fn() 的话，就会是传入 fn 函数的返回值，而不是执行函数，因此如果需要调用一个需要传参的函数的话，
应该使用一个匿名函数来调用函数。
####清除计时器
另外尽量使用变量存放 setTimeout 或 setInterval，以便于后来可以手动清除定时器。因为定时器返回的 ID 值是
递增的，所以我们可以通过记录最大的那个定时器的 id 值，来达到清除全部定时器的功能。
```
for(var i = 0; i < number; i++){
    clearTimeout(i);  //number 为定时器 id
}
```
####轮播图的实现
我们动手写一个轮播图插件相信会对定时器的使用有更深的了解。下面只给出轮播图插件的 Javascript 代码。
```
var doc = window.document;
var box = doc.getElementsByTagName("div")[0];
var imgList = doc.getElementsByTagName("img");  //5 张图片
var num = doc.getElementsByTagName("ul")[0];
var listNum = num.getElementsByTagName("li");  //5 个下标
var index = play = 0;

function autoPlay(){
    index ++;
    index == 5 && index = 0;
    for(var i = 0, len = listNum.length; i < len; i++){
        img[i].className = "";
        listNum[i].calssName = "";
    }
    img[index].className = "current";
    listNum[index].className = "current";
    play = setTimeout(autoPlay, 1000);
}
autoPlay();

//鼠标移到图片停止动作
box.addEventListener("mouseover",function(){
    clearTimeout(play);
},false);

//鼠标移开图片恢复动作
box.addEventListener("mouseout",function(){
    play = setTimeout(autoPlay, 1000);
},false);

```
demo 地址：[GITHUB 前端 UI 轮子库](https://github.com/zhangxiang958/FrontEnd-UI-Wheel/tree/master/%E7%BB%84%E4%BB%B6%E8%BD%AE%E5%AD%90/%E8%BD%AE%E6%92%AD%E5%9B%BE)
博文分享: [定时器原理](https://segmentfault.com/a/1190000002633108)
###cookie
首先 cookie 是什么？cookie 是本地存储信息的只有 4k 到 10k 大小的存储名值对的文件。cookie 对于我们做登陆框
的友好度是非常有作用的。但是建议不要使用 cookie 来存储个人保密信息，因为 cookie 是所有文件都可以使用的。
cookie 的设置，删除等都是对应着域名的，因此本地存储 cookie 可能会出现错误，可以使用 xampp 来搭建本地服务器
来调试 cookie。因为 cookie 是要被加进响应头部的，如果 cookie 过大，就会出现性能问题，因此不要在 cookie 存
储过多信息。
```
var cookieUtil = function() {
    get: function(name) {
        var cookieName = encodeURLComponent(name) + "=";//对 name 进行编码
        var cookieStart = document.cookie.indexOf(cookieName); //找到 cookieName
        var cookieValue = null;
        
        if(cookieStart > -1){  //如果有这个 cookieName
            var cookieEnd = document.cookie.indexOf(";", cookieStart);
            if(cookieEnd == -1){  //如果只有这个名值对
                cookieEnd = document.cookie.length;
            }
            cookieValue = decodeURLCompontent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));  //取值
        }
        return cookieValue;
    }
    
    set: function(name, value, expires, path, domain, secure) {
        var cookieText = encodeURLComponent(name) + "=" + encodeURLComponent(value);
        
        if(expires instanceof Date) {
            cookieText += "; expires=" + expires.toGMTString();
        }
        
        if(path) {
            cookieText += "; path=" + path;
        }
        
        if(domain) {
            cookieText += "; domain=" + domain;
        }
        
        if(secure) {
            cookieText += "; secure";
        }
        
        document.cookie = cookieText;
    }
    
    unset: function(name, path, domain, secure) {
        this.set(name, "", new Date(0), path, domain, secure); //删除通过设置 cookie 过期时间来删除
    }
}
```





