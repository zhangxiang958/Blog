title: JavaScript 学习笔记——事件
date: 2016-03-22 16:35:24
categories: JavaScript
---
事件部分原属于 DOM 部分，但是因为事件在网页交互方面有着重要作用，所以特地抽出讲述。
<!--more-->
##事件
其实事件部分是 DOM 2 级的其中一个部分，它是网页与用户交互的重要桥梁。网页通过事件来给网页上的元素在
执行某些动作的时候，采取相应的处理，比如触发某个函数。而具体的实现就是通过软件工程常说的监听者模式。
不过在此之前，我们应该了解事件流的知识。关于 DOM 的知识，请翻阅本人 [DOM 博文](http://zhangxiang958.github.io/2016/03/08/JavaScript%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%E2%80%94%E2%80%94DOM/)。
###事件流
事件流分为 3 个阶段，一、事件捕获，二、处于目标元素节点，三、事件冒泡。
为什么会出现事件流？有一个经典的问题与一个经典的例子，经典的问题是当你点击一个按钮的时候，你究竟点击的
是网页哪个部分，是 < *html* >, 是 < *body* >, 还是 < *button* > ?
一个很经典的例子就是你在纸上画一组同心圆，当你手指指向圆心的同时，你指向的是一组圆,并不是仅仅是一个圆。
当我们点击网页的其中一个按钮的时候，其实也点击了网页的整个部分即 document，而当时 IE 与 Netscape 公司对于事件如何传导分别提出截然相反的概念。
**参考资料：前端必读之[Event Order](http://www.quirksmode.org/js/events_order.html).**
####事件捕获
事件捕获就是从最上层的元素开始，直到最具体的那一个元素。
```
<body>
    <dv>
        <a href="#">Tag</a>
    </div>
</body>


var body = document.getElementsByTagName("body")[0];
var div = document.getElementsByTagName("div")[0];
var tag = div.getElementsByTagName("a")[0];

body.addEventListener("click",function() {
    console.log("body");
},true);  //设置为 true 表示捕获阶段执行

div.addEventListener("click",function() {
    console.log("div");
},true);

tag.addEventListener("click",function() {
    console.log("tag");
},true);

//打印结果为:
//body
//div
//tag
```
####事件冒泡
事件冒泡就是从最具体的那个元素，冒泡至最上层的元素。
```
<body>
    <dv>
        <a href="#">Tag</a>
    </div>
</body>


var body = document.getElementsByTagName("body")[0];
var div = document.getElementsByTagName("div")[0];
var tag = div.getElementsByTagName("a")[0];

body.addEventListener("click",function() {
    console.log("body");
},false); //设置为 false 表示冒泡阶段执行

div.addEventListener("click",function() {
    console.log("div");
},false);

tag.addEventListener("click",function() {
    console.log("tag");
},false);

//打印结果为:
//tag
//div
//body
```
###事件监听器
对于事件监听器，它的作用域是根据指定它的方式来确定的。
####HTML 事件监听器
```
<div onclick="Message()">Click Me</div>

<script>
    function Message() {
        console.log("Clicked");
    }
</script>
```
HTML 事件监听器直接写在标签内部，但是这样不符合内容结构与行为分离的思想，不建议使用，更何况这种形式会出现
时间差的问题，HTML 在页面显示的时候可能还不具备交互能力，因为代码还没有解析到 Javascript 代码。
这种事件监听器可以直接调用元素内变量，this 指向本身。
####DOM 0 级监听器
通过 Javascript 的方法给元素绑定事件，通过取得元素引用，将其一个属性值设置为函数。这种处理程序会在冒泡阶段
被注册。
```
var div = document.getElementsByTagName("div")[0];

div.onclick = function() {
    console.log("Click!");
}
```
这种方法的优点在于它比较简单，而且跨浏览器，但是它不能绑定多个事件处理程序，前一个设置的会被后一个覆盖。
```
div.onclick = function() {
    console.log("Click!");
}
div.onclick = function() {
    console.log("Click Twice!");
}
//打印 Click Twice, 不会打印 Click!
```
接触绑定可以通过设置事件处理程序为 null。
```
div.onclick = null;
```
####DOM 2 级监听器
DOM 2 级的监听器通过 addEventListener 添加事件处理程序，通过 removeEventListener 移除事件处理程序。这种
事件处理程序的好处在于他能够控制事件处理程序在事件流的哪个阶段被执行，并且可以添加多个事件处理程序。
这个方法有三个参数，第一个参数写事件类型，第二个函数写执行函数名或是一个匿名函数，第三个参数是控制事件
处理程序是在哪个阶段被注册，true 为捕获阶段，false 为冒泡阶段。
```
var div = document.getElementsByTagName("div")[0];

div.addEventListener("click",funcition() {
    console.log("first click");
},false);

div.addEventListener("click",funcition() {
    console.log("second click");
},false);

div.removeEventListener("click",funcition() {
    console.log("second click");
},false);

```
####IE 事件监听器
IE 并不支持 DOM 的 addEventListener 和 removeEventListener 接口，但是提供了类似的 attachEvent,detachEvent
值得注意的是，IE 事件监听器接收的事件类型不是 "click" 这种形式的，而是 "onclick" 形式的。
```
var div = document.getElementsByTagName("div")[0];

div.attachEvent("onclick",function(){
    console.log("clicked");
});

div.detachEvent("onclick",function(){
    console.log("clicked");
});
```
并且使用 IE 事件监听器时候，this 关键字指向全局 window，如果需要改变作用域使用 call 方法。这种事件监听器
的执行顺序是最后被定义的程序第一个被执行，一直上升到第一个程序。
###事件对象
在触发一个事件的时候，都会生成一个事件对象。执行完函数随即销毁。此对象包含了所有关于事件的信息。
####DOM 事件对象
DOM 事件模型中会将一个 event 事件对象传入事件处理程序中。
```
var div = document.getElementsByTagName("div")[0];

div.addEventListener("click",function (event) {
    console.log(event.target);  // "click", 事件类型
},false);
```
下面列举比较重要的属性与方法：
1. type 属性：被触发的事件类型
2. target 属性：事件的目标元素
3. currentTarget 属性：当前正在处理函数的元素
4. canceable 属性：只有为 true 才能取消事件默认行为，并且此属性为只读
5. preventDefault() 方法：取消默认行为
6. stopPropagation() 方法：阻止事件的进一步冒泡或捕获。
    

####IE 事件对象
不同于 DOM 事件模型中的对象，IE 的事件对象是以 window 对象的一个属性存在的。
```
var div = document.getElementsByTagName("div")[0];

div.attachEvent("onclick",function() {
    var event = window.event;
    console.log(event.type);  //"click", 事件类型
});
```
IE 事件模型中事件对象比较重要的属性方法：
1. type 属性：事件的类型。
2. srcElement 属性：事件的目标（与 DOM 中的 target 属性相同）。
3. returnValue 属性：设置为 false 就可以取消事件默认行为。
4. cancleBubble 属性：设置为 true 就可以阻止事件的冒泡。

因为 IE 的事件对象是 window 对象的一个属性，那么在事件处理函数中就不能以 this 来指向当前元素，应该使用
srcElement。
```
var div = document.getElementsByTagName("div")[0];

div.attachEvent("onclick",function() {
    var event = window.event;
    console.log(event.srcElement == this);  //false，因为 this 此时指向的是 window 对象。
});
```

###IE 事件模型和 DOM 事件模型之间存在哪些差别
IE 事件模型与 DOM 事件模型都支持 DOM 0 级事件监听器，但是对于 DOM 2 级则采取的不同的方法进行事件的绑定。
关于这个区别请看上面的事件监听器的内容。
此外对于事件对象，IE 的事件对象是作为 window 对象的一个属性存在的，而 DOM 是直接传入一个事件对象。因为
这样的差别，我们在使用事件监听器的时候就要小心 this 值的指向问题。
为了兼容各大浏览器的事件模型，下面将对事件对象进行一个兼容性封装。
###封装兼容性事件绑定对象
为了兼容 IE 与各大浏览器，进行了下面的处理。下面的方法可以绑定事件，解除事件，和获取事件对象。
```
var EventUtil = function() {
    
    //绑定事件
    addHandler: function(element, type, handler){ 
        if(element.addEventListener){
            element.addEventListener(type, handler, false);
        } else if(element.attachEvent){
            element.attachElement("on" + type, handler);
        } else {
            element["on" + type] = handler;
        }
    },
    //移除事件绑定
    removeHandler: function(element, type, handler) {
        if(element.addEventListener){
            element.removeEventListener(type, handler, false);
        } else if(element.detachEvent) {
            element.detachEvent("on" + type, halder);
        } else {
            element["on" + type] = null;
        }
    },
    //获取事件目标
    getTarget: function(event) {
        if(event.target) {
            return event.target;
        } else if(window.event.srcElement) {
            return window.event.srcElement;
        }
    }，
    //获取事件对象
    getEvent: function(event) {
        return event ? event : window.event;
    },
    //阻止事件默认行为
    preventDefault: funciton(event) {
        if(event.preventDefault) {
            event.preventDefault();
        } else {
            event.returnValue = false;
        }
    },
    //阻止事件冒泡
    stopPropagetion: function(event) {
        if(event.stopPropagetion) {
            event.stopPropagetion();
        } else {
            event.cancelBubble = true;
        }
    }
}
```
###事件委托
面对数量较多的元素需要绑定事件的时候，我们不可能每一一遍历然后绑定，这样太耗费性能与效率了。因此我们使用
事件委托。所有按钮事件都适合事件委托。
####事件委托的好处
事件委托主要是用在当页面有很多诸如按钮这样的元素需要绑定事件的时候，通过绑定事件在他们的父元素上，通过
事件冒泡，执行回调函数，只需绑定一个元素。
因为事件的绑定往往是自页面加载完毕后便存在于内存中，如果绑定的事件过多，就会影响性能，因此事件委托的其
中一个好处就是提高性能。同时，在解绑事件的时候，大大地方便了，减少了代码量，因为只绑定了一个事件，因此
解绑也只需解绑一个。
通常来说，我们通过 innerText 或者 innerHTML 来重写某个元素内部的元素的时候，或者更准确地说，当我们要删除
一个元素的时候，为了优化内存，通常需要解绑元素绑定的事件，然后再做删除操作，否则虽然元素从文档消失，但是
事件还是存在于内存中，但是我们使用如果事件委托的话，就没有这方面的顾虑，因为事件都已经绑定在了父元素上。
还有，过多的事件绑定意味着 DOM 结构需要很多个 hook 与 JavaScript 联系，使用事件委托可以帮助代码的解耦。
**参考文献：[Event Delegation](https://www.nczonline.net/blog/2009/06/30/event-delegation-in-javascript/)**
####事件委托怎么用
事件委托是将一些子元素的事件，通过绑定在其父级的更靠近根元素的元素上，通过事件冒泡机制触发事件，从而达到
减少事件处理程序而效果不变的目的。
你绑定在父级上，页面怎么知道你点击的是哪一个？我们可以通过事件对象中的 target 属性来知道用户点击的是哪个
元素。事件委托最形象的例子就是列表绑定事件。
```
<ul>
    <li><a href="javascript:void(0)">Tag1</a></li>
    <li><a href="javascript:void(0)">Tag2</a></li>
    <li><a href="javascript:void(0)">Tag3</a></li>
    <li><a href="javascript:void(0)">Tag4</a></li>
    <li><a href="javascript:void(0)">Tag5</a></li>
</ul>
```
对于上面的 HTML 结构，有下面代码：
```
var list = document.getElementsByTagName("ul")[0];

EventUtil.addHandler(list, "click",function(event){
    var target = EventUtil.getTarget(event);
    switch(target.innerText){
        case "Tag1":
            console.log("one");
            break;
        case "Tag2":
            console.log("two");
            break;
        case "Tag3":
            console.log("three");
            break;
        case "Tag4":
            console.log("four");
            break;
        case "Tag5":
            console.log("five");
            break;    
    }
});
```
通过事件委托，不必再写多个事件处理程序，只需在父级列表上绑定一个事件处理函数，通过事件冒泡机制，触发绑定
在父级的事件，再根据不同的子元素采取不同的行为。
事件委托如何判断子元素是我想要的？有很多种方法可以达到，这个需要根据实际情况采取不同的措施，但是不可能加
上很多没有样式的class，流行的做法是在html标签中加上 data-xxx，通过元素的 dataset 属性访问判别。
[segmentfault上关于事件委托的讨论](https://segmentfault.com/a/1190000000470398)
####事件委托与冒泡机制有什么关联？
个人认为，事件委托是建立在冒泡机制的基础之上的，通过事件冒泡触发在父级的事件处理程序，从而事件委托才能运作
但是选择冒泡而不选择捕获是因为冒泡的兼容性更好。
###彩蛋：原生拖拽实现
个人认为，在原生实现拖拽效果中，很好地体现了事件委托的知识，下面给出实现代码：
```
var drapdrop = function() {
    var dragging = null, diffX = 0, diffY = 0;
    
    function handleEvent (event) {
        var target = event.target;
        switch(target.type) {
            case "mousedown":
                if(target.className.indexOf("dragable") != -1 ){
                    dragging = target;
                    (dragging.className.indexOf("dragged") == -1) && (dragging.className += "dragged");
                    diffX = event.clientX - target.offsetLeft;
                    diffY = event.clientY - target.offsetTop;
                }
                break;
            case "mousemove":
                if(dragging != null) {
                    dragging.style.position = "absolute";
                    dragging.style.left = (event.clientX - diffX) + "px";
                    dragging.style.top = (event.clientY - diffY) + "px";
                }
                break;
            case "mouseup":
                dragging.classList.remove("drgged");
                dragging = null;
                break;
        }
    };
    return {
        enable: function() {
            document.addEventListener("mousedown",handleEvent,false);
            document.addEventListener("mousemove",handleEvent,false);
            document.addEventListener("mouseup",handleEvent,false);
        },
        unable: function() {
            document.removeEventListener("mousedown",handleEvent,false);
            document.removeEventListener("mousemove",handleEvent,false);
            document.removeEventListener("mouseup",handleEvent,false);
        }
    }
}

drapdrop.enable();
```
*这个拖拽只是显示了在 document 对象上的事件委托，并没有实现拖拽的数据传输。*




