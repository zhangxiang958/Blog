title: Zepto 源码简单剖析
date: 2017-02-16 22:06:24
categories: JavaScript
---

我希望能对一些工具库能够有多一些深入了解, Zepto 在做移动项目运用很多,希望借此能够对 Javascript 可以有更多
的深入与了解.
加上在目前的技术趋势,原生 Javascript 越来越强大, 各浏览器之间的兼容问题也相比以前较少, 在单页面框架内使用原生 Javascript 代替 JQuery 这样的工具库的声音越来越大.
<!--more-->
##一些常用的工具函数
###选择元素 *zepto.qsa*
*zepto.qsa* 选择元素这个 *API* 在 *zepto.init* (即 *$* 函数)中扮演着非常重要的角色,  
```javascript
    // `$.zepto.qsa` is Zepto's CSS selector implementation which
    // uses `document.querySelectorAll` and optimizes for some special cases, like `#id`.
    // This method can be overridden in plugins.
    //这个函数是用来选择 CSS 选择器的 querySelectorAll
  zepto.qsa = function(element, selector){
    var found,
        maybeID = selector[0] == '#', //如果有 '#' 符号就是 id 选择器
        maybeClass = !maybeID && selector[0] == '.',  // 如果没有,有 '.' 则是类选择器
        //如果只有标签名(没有 id 也没有类) 下面的表达式意思是如果有 id 或 类,则取符号后的名字,如果没有类和名字,则选择出单标签名
        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // Ensure that a 1 char tag name still gets checked
        //看是不是只有一个选择器
        isSimple = simpleSelectorRE.test(nameOnly)

        //第一个判断 element.getElementById 这个方法是否存在, 是不是一个单选择器, 是不是 id 选择器
    return (element.getElementById && isSimple && maybeID) ? // Safari DocumentFragment doesn't have getElementById
      //如果是使用 getElementById 找到元素, 如果能找到,则以数组的形式返回,如果没有则返回空数组, 问题在于为什么要用数组的形式? 个人认为是为了统一 Zepto 对象选择元素 API 的统一性(无论数组大小都是数组形式返回).
      ( (found = element.getElementById(nameOnly)) ? [found] : [] ) :
      //如果不存在 getElementById 这个方法, 或者多重选择(后代), 或者不是 id 选择器
      //则判断 如果不是元素节点(!== 1) 或不是 document (根节点 !== 9) 或者不是 文档片段类型documentFragment(!== 11)
      //则返回空数组,因为其他节点找不到元素
      (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :
      //如果是元素节点或者 document 或 documentFragment 节点
      // slice: function(){
      //return $(slice.apply(this, arguments))
      //},
      slice.call(
        //判断是否单选择, 不是 id 选择器, 有 getElementByClassName 方法
        isSimple && !maybeID && element.getElementsByClassName ? // DocumentFragment doesn't have getElementsByClassName/TagName
          //第一层判断的前选择, 如果不是单选择 id 选择器
          //第二层判断
          //如果是 class 则使用 getElementByClassName()
          maybeClass ? element.getElementsByClassName(nameOnly) : // If it's simple, it could be a class
          //不是则是标签, getElementByTagName() 
          element.getElementsByTagName(selector) : // Or a tag
          //这个是第一层判断的后选择
          //如果不是简单选择,则要将所有的元素选择出来使用 querySelectorAll() 试一下用这个 API
          element.querySelectorAll(selector) // Or it's not simple, and we need to query all
      )
  }
```
在这个 *API* 中, 都使用了三段式判断来判断返回的内容,有个疑问在于与 *if...else* 语句相比, 三段式判断的好处
在于比 *if...else* 语句要简洁,对于这种工具库源码,使用三段式判断来避免大范围的 *if...else* 与花括号的相互
嵌套的繁琐,实在是有利于源码的阅读.

###生成标签 *zepto.fragment*

```
  // `$.zepto.fragment` takes a html string and an optional tag name
  // to generate DOM nodes from the given html string.
  // The generated DOM nodes are returned as an array.
  // This function can be overridden in plugins for example to make
  // it compatible with browsers that don't support the DOM fully.
  //fragment 函数接收 html 字符串和一个 tag 名, 在 tag 的后代生成 html 标签,
  //返回的结果是以节点数组形式返回
  zepto.fragment = function(html, name, properties) {
    var dom, nodes, container

    // A special case optimization for a single tag
    //RegExp.$[1-99] 是内置正则对象的属性,数字代表正则表达式匹配的第几个字符串
    //这里的正则对象是全局正则对象
    if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1))

    if (!dom) {

      if (html.replace) {
       html = html.replace(tagExpanderRE, "<$1></$2>")

      }

      if (name === undefined) name = fragmentRE.test(html) && RegExp.$1
      if (!(name in containers)) name = '*'

      container = containers[name]
      container.innerHTML = '' + html
      dom = $.each(slice.call(container.childNodes), function(){
        container.removeChild(this)
      })
    }

    if (isPlainObject(properties)) {//是否是原生对象,但是不是 window 对象(document.window)

      nodes = $(dom);

      //问题在于为什么对 nodes 进行操作,函数返回的是 dom ? 是不是赋值操作是引用而不是复制 ?
      //答案是
      //字面量变量才是基本类型, 用 new 操作符都是对象类型,基本类型在栈内部,对象类型在堆内部,所以基本类型才会 play itself,
      //但是对象类型是 itself
      //因为这是引用类型,所以进行操作的时候 nodes 变, dom 也进行变值
      //所以返回的是 dom
      $.each(properties, function(key, value) {
        // methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset']

        //如果有上面这些属性的话,就使用 Zepto 对象的方法赋值
        if (methodAttributes.indexOf(key) > -1) nodes[key](value)
        //如果不是就用 attr 方法
        else nodes.attr(key, value)
      })
    }

    return dom;
  }
```

###Ready
```
function ready(callback){
    if(document.readySatte === 'complete' || (document.readyState !== 'loading' && document.documentElement.doScroll)) {
        setTimeout(function(){
            callback();
        }, 0);
    } else {
        
        var handler = function(){
            document.removeEventListener('DOMContentLoaded', handler, false);
            window.removeEventListener('load', handler, false);
        }
        
        document.addEventListener('DOMContentLoaded', handler, false);
        window.addEventListener('load', handler, false);
    }
}
```
###Trim
```
function trim(str){
    return str == null ? '' : String.prototype.trim.call(str);
}
```

###Type
type 函数是用来判断值的类型的,我们知道在 Javascript 中值有 5 个基本类型: undefinded, null, String, number,
boolean.而对象 Object 中又有 Array, String, Number 等包装类型,虽然 typeof 可以判断基本值类型,但是对于 Object 类型却很难判断是哪种引用类型.我们知道 toString 函数是可以将值转化为对象的字符串表示.所以:
```
function type(val){
    return type == null ? String(val) : Object.prototype.toString.call(val);
}
```


###children
返回直接后代, 在简单实现中发现元素对象中的 *childNodes* 与 *children*, 两者很像,但是需要加以区分.
childNodes 返回的是 NodeList 集合,是该元素的字节点集合, 而 children 返回的是 HTMLCollection 集合,是该元素
的子元素集合.
```
//这里的 children API 的需求是返回元素的子元素
function children(el){
    return 'children' in el ? 
            Array.prototype.slice.call(el.children) :
            Array.prototype.map.call(el.childNodes, function(son){
                if(son.nodeType == 1) return son;
            });
}
```

###Pluck
DOM 遍历的基本 API.
```
function pluck(el, prop){
    return el[prop];
}
```

###Matches
matches 的兼容性写法.
```
function matches(ele, selector){
    
    var matchSelector = ele.matches || ele.wekitMatchesSelector || ele.mozMatchesSelector || ele.oMatchesSelector || ele.matchesSelector;
    if(matchesSelector) return matchesSelector.call(ele, selector);
    var matches, parentNode = ele.parentNode, temp = !parentNode;
    if(temp) parentNode = document.createElement('div').appendChild(ele);
    match = ~zepto.qsa(ele, selector).indexOf(ele);
    temp && parentNode.removeChild(ele);
    return matches;
}
```
###Contains
用于检查某个 DOM 元素是否是某个元素的后代.
```
function contains(parent, node) {
    if(document.documentElement.contains) {
        return function(parent, node){
            return parent.contains(node);
        }
    } else {
        return function(parent, node){
            while(node && (node = node.parentNode)) {
                if(node === parent) {
                    return true;
                }
            }
            return false;
        }
    }
}
```

###Clone
主要使用的是 DOM 自带的 API-- cloneNode, 克隆节点, 如果 cloneNode 传入参数为 true, 那么就进行后代节点的全
克隆,如果输入 false, 就是只复制这个节点.
```
function clone(el){
    return el.cloneNode(true);
}
```

###Attr
其实在 DOM API 中,但是运用了 DOM 本身的 API-- getAttribute, setAttribute, removeAttribute 来实现属性的修改
添加与移除.

###CSS
元素中的 style 是 style 元素 CSSStyleDeclaration 类, style 对象有 removeProperty() 方法移除样式属性.
在简单实现的时候发现 Zepto 这样拼接字符串使用 csstext 一次性写入貌似高效,但是好像没有考虑去重的问题.
```
//先确定 CSS 函数的作用

 1. 只输入样式属性名, 则获取该样式属性的值
 2. 输入样式属性名与相对应的值,设置样式属性值

//难点也在于 style 集合中的属性名是以驼峰写法的, 但是内联样式的属性却是连接符形式
function CSS(el, prop, val){
    if(!el || !prop) return
    //优先选择内联样式, getComputedStyle 是计算元素的最终样式的
    else if(!val && val == undefined) return el.style[prop] || getComputedStyle(el, null).getPropertyValue(prop);
    
    var css = '';
    css += prop + ':' + maybeAddPX(val) + ';';
    el.style.cssText += ';' + css;
}

//为数值添加 'px' 后缀单位
function maybeAddPX(prop, val) {
    /*
        cssnumber = {
            'column-count': 1,
            'columns': 1,
            'font-weight': 1,
            'line-height': 1,
            'opacity': 1,
            'z-index': 1,
            'zoom': 1
        }
    */
    return ((typeof prop == 'number' && !(cssNumber[prop])) ? val + 'px' : val);
}
```

***使用 getComputedStyle 获取伪类信息***
getComputedStyle(ele, '伪类') 与 getPropertyValue() 与 element.style.name 相比,可以获取 style 标签或者 link 中的 CSS 样式.并且重要的一点在于 getComputedStyle 可以获取伪类信息
```
//大部分情况下我们只需要调用 window.getComputedStyle 这个 API 就可以了, 只有在 firefox 中获取子框架(iframe) 才需要用到 document.defaultView
//兼容性代码如下:
//getComputedStyle = document.defaultView && document.defaultview.getComputedStyle;
<style>
    p:after {
        content: 'the class'
    }
<style>

<script>
    var p = document.getElementsByTagName('p')[0];
    
    console.log(getcomputedStyle(p, ':after').content);   //'the class'
</script>
```
备注: [getComputedStyle On MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle)
###Class

 1. *hasClass*
 ```
 function hasClass(el, name){
        if(name == undefined) return 
        else if(! 'className' in el) return
        
        return Array.prototype.some.call([el], function(el){
            return this.test(el.className);
        }, new RegExp('(^|\\s)' + name + '(\\s|$)'));
 }
 
 hasClass(document.getelementByTagName[0], 'test');
 ```
 2. *addClass*
 ```
 function addClass(el, name){
        if(!el || !name) return 
        else if(! 'className' in el) return
        
        var cls = el.className, nameList = name.split(','), newNameList = [];
        Array.prototype.every.call(nameList, function(newName){
            
            if(!hasClass(el, newName)) newNameList.push(newName);
            return true;
        });
        
        newNameList.length && (el.className = cls + (cls ? ' ' : '') + newNameList.join(' ')); 
 }
 
 addClass(document.getElementByTagName('p')[0], 'test,p');
 ```
 3. *removeClass*
 ```
 function removeClass(el, name){
        if(!el || !name) return
        else if(! 'className' in el) return
    
        var cls = el.className, nameList = name.split(','), nowClass = cls.split(' '), newClass = '';
        Array.prototype.every.call(nameList, function(theName){
            
            if(hasClass(el, theName)) newClass += cls.replace(new RegExp('(^|\\s)' + theName + '(\\s|$)'), '');
            
            return true;
        });
        el.className = newClass.trim();
 }
 ```
 4. *toggleClass*
 ```
 function toggleClass(el, name){
 
        if(hasClass(el, name)) removeClass(el, name);
        else addClass(el, name);
 }
 ```
###offset
想要理解 offset 一系列的 API, 我觉得关键在于弄清楚浏览器对于页面的距离位置的几个变量的关系.

```
//在简单实现的时候发现一个 API, getBoundingClientRect(), 可以方便地获取元素的位置大小.
var p = document.getelementsByTagName('p')[0];

var left = p.getBoundingClientRect().left + window.pageXoffset;
var top = p.getBoundingClientRect().top + window.pageYOffset;
var width = p.getBoundingClientRect().width;
var height: p.getBoundingClientRect().height;
```
##附录
###在 *eq* API 中发现的小技巧:
```
function eq(index){
    return index === -1 ? Array.prototype.slice.call(this, idx) : Array.prototype.slice.call(this, idx, + idx + 1);
}
```
*+ idx + 1* 这个表达式可以将字符串形式的数字转化为真正的数值之后才进行计算,防止了 *NaN* 的出现.
###正则表达式的运用
以前在做项目的时候,对于正则表达式总是不以为然,只是在验证手机号,邮箱地址等地方运用,但是其实它在驼峰与连接符
写法之间的来回切换是非常有用与高效的.
