title: JavaScript DOM 基础
date: 2015-09-22 18:10:24
categories: JavaScript
---
    本博文是学习 JavaScript DOM 编程艺术 的读书笔记。主要讲 DOM 的一些基本知识和操作方法。
<!--more-->
##基本知识
###样式，结构，行为的分离。
    在编程的时候，应该将 JavaScript 的代码用文件打包起来，在 HTML 中用 <script> 标签链接到 HTML 文档中，
    并且将 <script> 标签放在 <body> 的闭合标签之前，这样运行效率会比较快。
    样式与结构的分离也非常重要。建议不要将样式直接写在 HTML 的标签中。比如：
```
<p style = "color: red;">Jarvis DOM</p>
```
这样不便于代码的维护，通常的做法是将样式集中到一个文件中 CSS 文件中，方便管理。另外，在做其他比较大的项目的时候，我们还可以将项目中的颜色样式集中到一个 *color.css* 文件里面，把布局集中到 *layout.css* 文件中，最后用一个 *basic.css* 文件将其他的样式表文件包括在内。
```
@impotr url(style/color.css);
@import url(style/layout.css);
```
这也是最浅层的模块化编程的思想。
#### CSS 编程习惯
    关于 CSS 在编程过程中的一些良好的习惯，我们应该在日常的时候就养成，这样方便日后对自己代码的维护以及增加代码的可读性与优雅性。
    这块内容牵扯到近日所做的项目——一个关于摄影的网站开发。在开发过程中，发现自己的 CSS 命名的习惯很不好，遇到了很多的坑。在这里会讲述相关知识，防止日后在陷入同样的坑。
    
##JavaScript DOM 操作方法
###DOM 检索（抓取）元素
常用的检索方法：1. getElementsByTagName(); 2. getElementById(); 3. getElementsByClassName(); 
另外，还有 getAttrbute("property","value"); 和 setAttrbute(("property","value"); , 不过这两个方法不属于 Document 对象，不能够通过 document 对象来调用，需要和常用的检索方法相互配合。
getElementsByTagName(); 和 getElementsByClassName(); 都是获取到很多子元素然后存储在一个数组里面。

###DOM 添加元素
常用的添加方法：1.createElement(); 2.createTextNode(); 3.appendChild(); 4.insertBefore(new,target);
第一个方法是生成一个元素节点，但是没有添加到 HTML 文件中。第二个方法是生成一个文本节点。第三个方法是将一个节点插入到目 标节点的最后。第四个方法是将一个节点插入到目标节点的前面。
###获取样式或是修改样式
使用 Element.style.property 来修改样式的值。Element.style.property 是没法获取通过链接在 HTML 文件上的 CSS 文件的 style 属性值
的，只能获取内联样式，但是它能够写入，因此这样可以改变样式。其实实际上，最好的办法是设置多组 Class ，通过修改类名来达到改变样式的目的。
###节点关系
注意，这里是节点而不是元素，节点包括文本节点，属性节点，元素节点。
childNodes 所有的子节点
nodeType 节点类型(返回 1 表示元素节点，返回 2 表示属性节点，返回 3 表示文本节点，共12种取值)
nodeValue 节点值
firstChild 第一个子节点
lastChild 最后一个子节点
nextSibling 下一个兄弟节点
previousSibling 上一个兄弟元素
##DOM 的一些思想与一些有用的封装函数
###平稳退化与渐进增强的思想
所谓平稳退化，就是当浏览器无法加载 Javascript 文件时，不妨碍用户访问到网页的核心内容。这里就需要多做一些兼容性检测和少做假设。比如给 <a> 标签的 href 属性加上确定的值。
所谓渐进增强，就是用信息层将原始数据包裹起来。
###有用的封装函数
包括DOM 本身没有提供的 insertAfter()函数，制造加载队列的 addLoadEvent()，添加类名函数 addClassName()
```
function addLoadEvent(func) {
    var load = window.onload;
    if(typeof window.onload != "function") {
        window.onload = func;//如果 window.onload 没有被赋值为函数
    } else {
        window.onload = function() {
            //如果 window.onload 已经赋值为函数，则形成一个执行函数队列
            load();
            func();
        }
    }
    
}
```

```
function innerAfter(newElement, targetElement) {
    var parent = targetElement.parentNode;
    //检查目标元素是不是目标元素的父元素的最后一个节点
    if(targetElement == parent.lastChild) { 
        parent.appendChild(newElement);
    } else { //不是则在目标元素的下一个兄弟元素之前插入新元素 
        parent.insertBefore(newElement, targetElement.nextSibling);
    }
}
```
```
function addClassName(element,value) {
    if(!element.className) {//如果 element 没有类名
        element.className = value;
    } else { //如果 element 原本有类名
        newClassName = element.className;
        newClassName += " "; //类名之间有空格
        newClassName += value;
        eleement.className = newClassName;
    }
}
```
###简单初略地介绍 AJAX
什么是 AJAX ？ 就是网页通过异步请求，做到网页的局部更新与更换内容。它的核心就是 XMLHttpRequest 对象，此对象充当客户端与服务器之间的中间角色，可以通过 Javascript 自行向服务器端发请求，简单来说，就是不必 URL 跳转也可以看到更新内容。
由于 AJAX 在不同浏览器之间所使用的 XMLHttpRequest 对象的版本不同，所以为了兼容不同浏览器，就有了 getHTTPObject 函数。
```
function getHTTPObject() {
    if(typeof XMLHttpRequest == "undefind")
        XMLHttpRequest = function() {
            try { return new ActiveXobject("Msxml2.XMLHTTP.6.0"); }
                catch (e) {}
            try { return new ActiveXObject("Msxml2.XMLHTTP.3.0"); }
                catch (e) {}
            try { return new ActiveXObject("Msxml2.XMLHTTP"); }
                catch (e) {}
            return false;           
        }
        console.log(new XMLHttpRequest());
        return new XMLHttpRequest();    
}
```
另外，提交请求有 5 种状态，分别用数字 0-4 来表示。
```
0 表示未初始化
1 表示正在加载
2 表示加载完毕
3 表示正在交互
4 表示完成
```
所以在 AJAX 中有一个 onreadystatechange 事件是专门用来响应当状态码改变的时候，执行一个回调函数的。
request.onreadstatechange = dosomthing; 
```
request.onreadystatechange = function () {
        if(request.readyState == 4) {
            if(request.status == 200 || request.status == 0) {
                ***********
            } else {
               ************
            }
        }
    };
request.send(null); 
```
目前来说，前端许多框架都对 AJAX 兼容很好，例如 JQuery，在写 AJAX 请求时不像以上的原生代码一样量多，所以善用工具可以简化我们的工作量。
作者 张翔
2015 年 09 月 22 日
[详解CSS选择器][1]


  [1]: http://developer.51cto.com/art/201009/226852.htm



