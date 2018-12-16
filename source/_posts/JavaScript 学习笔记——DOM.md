title: JavaScript 学习笔记——DOM
date: 2016-03-08 15:15:24
categories: JavaScript
---
这是本人学习 Javascript 的学习笔记，主要讲述 DOM 及 DOM 2 , 3 级的拓展。
<!--more-->
## DOM 是什么？
DOM 是 JavaScript 的三大组成部分之一，它是针对 XML 的一个 API，用于操作网页的组件。
##DOM 中的节点操作及其应用
DOM 中的节点类型比较多，在这里挑选了比较重要的几个类型来详细讲述。
###Node 类型
Node 类型获取的都是节点，包括文本节点，元素节点，属性节点等。其中我们经常用到的是 NodeType 和
NodeValue 这两个属性。在判断节点类型的时候，应该使用它的数值判断而不是字符串类型判断，因为数值
判断是兼容所有浏览器的。
在对 Node 类型的操作上，我们很多属性来移动节点，parentNode，childNodes，nextSibling，previousSibling
这些属性可以判断两个节点是否有同一个父节点，比如 parentNode，而对于兄弟节点，可以使用 nextSibling 和
previousSibling，而 childNodes 获取的是一个类数组对象，包含了该节点的所有子节点。
在操作方法上，我们有 appendChild() 方法， insertBefore() 方法，看到方法名就知道，其实这两个方法都是
用于插入节点的，appendChild 会在父节点的最后一个节点之后插入节点，而 insertBefore 会在该节点之前插
入节点，但是很多人会有这样的需求，就是 insertAfter ，但是 Node 并没有这样的方法，所以需要我们自己去
实现，下面给出实现代码：
```
function insertAfter(target, new) {
    if(new == null) return ;
    if(target == target.parentNode.lastChild){
        target.parent.appendChild(new);
    } else {
        target.nextSibling.insertBefore(new);
    }
}
```
###Document 类型
我们如果要对 DOM 进行操作，那么一定少不了 document.getElementsByTagName() 和 document.getElementById().
getElementsByTagName() 获取的是一个 NodeList 对象，这是一个类数组对象，它是动态的，所以在使用递归的时候
应该建立一个快照。还有一个常用的方法就是 getElementsByName() ，通过 name 值获取一批元素，但是值得注意的
是浏览器可能会返回一些错误的信息，因为有些浏览器会误将表单的 name 值与一些元素的 id 值混淆，所以为了避
免这样的错误，就应当避免表单的 name 值与元素的 id 重合。
###Element 类型
Element 类型恐怕是我们见得最多并且是操作的最多的元素了。通常我们需要操作 Element 类型的样式，取得它的
类名，id 名，取得特性 getAttribute()，设置属性 setAttribute()，移除属性 removeAttribute() 。

###Text 类型
创建一个文本节点，我们会使用 createTextNode() 方法，而使用这样的方法创建节点，浏览器在解析的时候不会
创建相邻的文本节点，因此如果需要相邻的文本节点，可以在父元素上使用 noemalize() 方法。
需要分割文本节点的时候使用 solitText() 方法。

###Attr 类型
我们基本上很少直接操作 attr 类型节点。一般会使用 getAttribute(), setAttribute(), removeAttribute() 
这三个方法去获取元素的属性。而这三个方法通常是用于取自定义的属性，而不是一些公认的属性比如 className等。
###DocumentFragment 类型
这是一个文档碎片类型，它是一个轻量级的节点，不属于文档树，没有相对应的标签。但是这个类型很大的一个
作用就是它作为一个 DOM 片段仓库，可以包含与控制节点，将一些繁琐的 DOM 操作先集合到它身上，然后将它通过 appendChild()方法插入到文档中，但是它并不会插入到文档节点中，而是将它的子孙后代插入到文档中。
```
var frag = document.createDocumentFragment();
var div = document.createElement('div');
var text = document.createTextNode("Jarvis");
var target = document.getElementById("test");
div.appendChild(text);
frag.appendChild(div);
target.appendChild(frag);
```
这样的话可以减少重绘和重排，减轻浏览器性能消耗。
## DOM 拓展的 API
###querySelector 和 querySelectorAll
这部分是主要的拓展 API，用于选择元素，querySelector 返回的是第一个元素，而 querySelectorAll 是返回
一个 NodeList 对象。这里不得不说的就是关于常用的类库 JQuery 的选择器部分，它的大致思想就是获取一个
选择符，从而快速选择元素，抛弃的 getElementById 和 getElementsByTagName 这样繁琐的方法。
下面给出我自己实现的一个小的仿 JQuery 选择器：
```
 function query(selector, root) {
    var element = [];  //存放元素列表
    var allChildren = null;
    root = root || document;
    switch(selector[0]){
        case "#":  //id 选择器
            element.push(root.getElementById(selector.substring(1)));
            break;
        case ".": //类选择器
            if(root.getElmentsByClassName){  //如果支持 getElementsByClassName 方法
                element = root.getElementsByClassName(selector.subtring(1));
            } else {
                var classMode = new RegExp("\\b" + selector.substring(1) + "\\b");
                allChildren = root.getElementsByTagName("*");
                for(var i = 0, len = allChildren.length; i < len; i++){
                    if(classMode.test(allChildren[i].className)){
                        element.push(allChildren[i]);
                    }
                }
            }
            break;
        case "[":  //属性选择器
            if(selector.indexOf(" ") === -1){  //只有属性名
                allChildren = root.getElementsByTagName("*");
                for(var i = 0 , len = allChildren.length; i < len; i++) {
                    if(allChildren[i].getAttribute(selector.splice(1,-1)) !== null) {
                        element.push(allChildren[i]);
                    }
                }
            } else {  //既有属性名也有属性值
                var index = selector.indexOf("=");
                allChildren = root.getElementsByTagName("*");
                for(var i = 0 , len = allChildren.length; i < len; i++) {
                    if(allChildren[i].getAttribute(selector.splice(1,index)) === selector(index+1, -1)) {
                        element.push(allChildren[i]);
                    }
                }
            }
            break;
        default: 
            element = root.getElementsByTagName(seclector);
    }
    return element;
 }
 
 function $(selector){
    if(selector == document) {
        return document;
    }
    selector.trim();  //去除选择符前后的空格
    if(selector.indexOf(" ") !== -1 ){  //如果有后代
        var selectorAll = selector.split(" ");
        return query(selectorAll[1], query(sectorAll[0])[0])[0];
    } else {
        return query(selector,document)[0];
    }
 }
 
```
##DOM 2 和 3 级
DOM 1 ： dom core 和 dom html  映射文档结构
DOM 2 ： dom views 和 dom events 和 dom style 和 dom 遍历和范围
DOM 3 ：dom load and save 和 dom validation
元素大小这一部分的知识非常引人注意。
1. 偏移量，是指用户可见的元素的大小，包括 offsetWidth 和 offsetHeight.
2. 客户区，是指内边距和内容区的大小。
3. 滚动大小，scrollTop 是指可滚动元素上方被隐藏的像素，scrollWidth是指可滚动元素的原宽度。
4. offsetTop 和 offsetLeft 是指元素到最近的包含块的距离

##理解 DOM 的性能问题
理解 DOM ，最重要的就是理解它的性能问题。因为 NodeList 对象， HTMLCollection 对象等是动态的，不断更新的。
如果我们经常操作他们，浏览器每次都要重新更新一次，会造成性能问题。因此减少对上面两个对象的访问是对性能
优化的一步，最常用的操作就是将要访问的 NodeList 对象， HTMLCollection 对象用变量存储起来。
我们经常说 DOM 很慢，倒不是说操作这些对象很慢，而是操作这些对象会导致浏览器的重绘或重排行为，会对渲染，
性能造成很大损耗。为了减少这些行为，例如我们可以将一些 DOM 操作集合起来，最后再插入到 DOM 树里面，就是
通过上面所说的 DocumentFragment 对象来作为 DOM 操作的 “ 仓库 ”，减少实际操作 DOM 的行为。
###DOM 优化
下面举出一些编写代码时候的建议，可以尽可能地优化性能。
1. 尽量将修改样式的代码批量操作，例如使用 cssText 属性，批量修改。
2. 将大量的 DOM 节点插入工作放在一个离线的 DOM 元素，比如 DocumentFragment 对象。
3. 对于访问的需要计算的值，应该存进一个快照里面，例如将元素的 offsetHeight,offsetLeft 等的这些属性。

### DOM 的未来
即使我们上面采取了一系列的措施，DOM 的性能问题还是很庞大，其原因在于每一个 DOM 元素都有一个庞大的
原型链，在创建元素的时候，原型链会为它添加了大量的属性，因此 DOM 的性能问题是非常头疼的。因此未来的
方向应该会是使用例如 AngularJS 这样的 MVVM 框架来编写网页。


