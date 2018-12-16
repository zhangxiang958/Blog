title: Event In Zepto
date: 2017-02-20 8:57:24
categories: JavaScript
---

本篇博文是对 Zepto 的事件模块进行解读.
<!--more-->
##On
###add
add 函数是 on API 中最重要的函数, 如果想要理解 On, 需要先看懂 add 函数.
```
function add(element, events, fn, data, selector, delegator, capture){
    //事件 ID, handlers 是事件处理函数缓存池,方便移除(包括匿名函数)
    //一个元素一个 id, handlers 对应元素 id 存放元素的所有事件监听器
    var id = zid(element), set = (handlers[id] || (handlers[id] = []))

    events.split(/\s/).forEach(function(event){
      if (event == 'ready') return $(document).ready(fn)
      var handler   = parse(event)
      console.log(handler)
      handler.fn    = fn
      handler.sel   = selector
      // emulate mouseenter, mouseleave
      //模拟 mouseenter, mouseleave

      //如果事件是 mouseenter, mouseleave 模拟成 mouseover, mouseout 处理
      if (handler.e in hover) fn = function(e){

        //relatedTarget 返回 mouseover 事件刚刚离开的那个节点或者 mouseout 事件刚刚进入的那个节点
        var related = e.relatedTarget
        //!relateed 不存在即不是 mouseover 或 mouseout 事件
        //(related !== this && !$.contains(this, related)) 
        //this 是目标元素也就是调用 API 的那个元素
        //这里的评定标准是 related 不是 this, 并且 related 不是 this 的子元素
        //也就是说从元素外部移入目标元素
        if (!related || (related !== this && !$.contains(this, related)))
          
          //立刻执行函数
          return handler.fn.apply(this, arguments)
      }

      //delegator 是委托,代理
      handler.del   = delegator
      var callback  = delegator || fn
      //代理函数
      handler.proxy = function(e){
        e = compatible(e)
        //如果 e.stopImmediatePropagation() 执行过, 那么同类型的事件全部停止响应
        if (e.isImmediatePropagationStopped()) return

        //将 data 传入 event.data
        e.data = data
        //执行回调函数
        var result = callback.apply(element, e._args == undefined ? [e] : [e].concat(e._args))
        //如果回调函数返回 false, 也就是 event.preventDafult 的效果
        //那么就调用 preventDefault, 否则就阻止事件冒泡
        if (result === false) e.preventDefault(), e.stopPropagation()
        return result
      }

      //添加本次事件监听器的下标
      handler.i = set.length
      //本次添加的事件监听器添加到监听器队列的末尾
      set.push(handler)
      //如果有 DOM2级 事件监听器
      if ('addEventListener' in element)
        //realEvent 就是将 mouseenter/mouseleave 变为 mouseover/mouseout
        //将 focus/blur 变成 focusin/focusout
        element.addEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))
    })
  }
```
###on 核心
```
//增加事件监听器
  $.fn.on = function(event, selector, data, callback, one){
    var autoRemove, delegator, $this = this  //this 是 Zepto 对象
    //========,但是 event 参数不是一个单个字符串(即由空格隔开的字符串[多个事件]) ===================
    //如果输入了 event 事件,并且 event 参数是字符串
    if (event && !isString(event)) {
      console.log("!!!!!!!!!!!!!!!!!");
      //使用 Zepto 核心的 each 方法(其实就是分开对象与数组的方式进行循环)
      $.each(event, function(type, fn){
        console.log('===============================')
        console.log('event is ' + event);
        //递归调用本函数
        $this.on(type, selector, data, fn, one)
      })
      return $this
    }
    console.log('event is' + event);
    console.log('selector is' + selector);
    console.log('data is' + data);
    console.log('callback is' + callback);
    console.log('one is' + one);
    console.log(callback !== false);
    
    //下面的代码逻辑是逐步检查 API 调用的参数的正确性或者说意图
    //如果 selector 不是字符串, 并且 callback 不是函数, 并且 callback 不等于 false
    if (!isString(selector) && !isFunction(callback) && callback !== false)
      //将 data 的值赋给 callback, selector 的值赋给 data, selector 赋为 undefined
      //也就是将参数的顺序后移
      //如果符合就是下面这种调用方式, data 参数是用于 event.data 中的数据的. 
      //$(el).on(event, undefined, data, callback, one) 这个调用方式意图 
      callback = data, data = selector, selector = undefined
    if (callback === undefined || data === false)
      //如果 callback 是 undefined, 也就是 $(el).on(event, undefined, undefined, callback) 这种调用方式意图
      //再次将参数后移
      callback = data, data = undefined

    if (callback === false) callback = returnFalse

    //
    return $this.each(function(_, element){
      console.log(element)

      //如果有 one 参数,表示这个事件只运行一次
      if (one) autoRemove = function(e){
        //移除事件监听器
        remove(element, e.type, callback)
        //执行回调函数
        return callback.apply(this, arguments)
      }

      //如果有选择器, 表示只有在匹配这个选择器的情况才会执行函数, 也就是在 $() 中的元素一般是父级元素像 document 这样的
      //启用事件代理(委托)
      if (selector) delegator = function(e){

        var evt, match = $(e.target).closest(selector, element).get(0)
        //在 element 中查找最先匹配的 slector 元素的第一个元素
        if (match && match !== element) {
          //如果匹配并匹配的元素不是 $() 中的元素即非代理元素
          evt = $.extend(createProxy(e), {currentTarget: match, liveFired: element})
          return (autoRemove || callback).apply(match, [evt].concat(slice.call(arguments, 1)))
        }
      }

      //element 是调用 on API 的元素, event 是事件的字符串, callback 是回调函数, data 是 event.data
      //add(element, events, fn, data, selector, delegator, capture)
      add(element, event, callback, data, selector, delegator || autoRemove)
    })
  }
```
##Off
###remove
remove 函数是 off API 中最重要的函数, 如果想要理解 Off, 需要先看懂 remove 函数.
```
//移除事件监听器
  function remove(element, events, fn, selector, capture){
    //找到元素对应在事件监听器缓存池里面的 id
    var id = zid(element)
    //如果有指定 events 就选择 events, 否则使用 ''
    ;(events || '').split(/\s/).forEach(function(event){

      findHandlers(element, event, fn, selector).forEach(function(handler){
        //使用 delete 操作符来删除
        //handlers 存放着很多元素的事件监听器, handlers[id] 找到我们需要的元素拥有的所有事件监听器
        //每一个事件监听器在添加的时候会被添加一个 i 属性当作在 handler 的下标, 利用 handler.i 可以找到特定的某个事件监听器
        //从缓冲池中删去
        delete handlers[id][handler.i]

      //如果支持 removeEventListener DOM 2级 
      if ('removeEventListener' in element)
        element.removeEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))
      })
    })
  }
```
###off 核心
```
//移除事件
  $.fn.off = function(event, selector, callback){
    //this 是调用函数的元素
    var $this = this
    //由于 event 可以是对象类型
    if (event && !isString(event)) {
      //多个移除
      $.each(event, function(type, fn){
        $this.off(type, selector, fn)
      })
      return $this
    }


    //参数后移
    if (!isString(selector) && !isFunction(callback) && callback !== false)
      callback = selector, selector = undefined


    if (callback === false) callback = returnFalse

    //使用 remove 移除事件监听器  
    return $this.each(function(){
      remove(this, event, callback, selector)
    })
  }
```
##小结
在解读的过程中,发现 Zepto 源码使用了事件监听器缓冲池, 通过为每个元素生成一个不重复的数字作为元素 id, 这里
的不重复的数字是通过一个全局变量依次递增来达到目的的, 然后在 handlers 对象中, 依照元素的 id ,找到元素对应
的事件监听器队列, 队列是以数组的形式存在的, 在事件监听器 push 进队列之前, 都会为这个事件监听器添加一个 i
属性来记录下面,以便找到与删除.这个方法的好处在于可以集中管理事件监听器,并去掉了 DOM 中的 addEventListener
与 removeEventListener 不能去除匿名回调函数监听器的缺点.
在 Zepto 中甚至还使用了 document 的自定义事件函数来自定义函数:
```

var event = document.createEvent('Events'), bubbles = true;
//type 是自定义的事件名, bubbles 是定义事件是否冒泡, 最后的 true 代表是否可以阻止默认事件
event.initevent(type, bubbles, true);

//触发事件, 这个 dispatchEvent 只有 DOM 节点才有的 API
el.dispatchEvent(type);
```


##附录:
###在 Event 中的 *preventDefault, stopPropagation, stopImmediatePropagation*

```
var methods = {
    'preventDefault': 'isPreventDefault',  //阻止元素的默认行为
    'stopImmediatePropagation': 'isStopImmediatePropagation', //阻止元素的冒泡并且同类型的事件监听器全部停止
    'stopPropagation': 'isStopPagation'  //阻止元素事件的冒泡
}


```
###取消元素的默认行为
在剖析源码的时候,发现 *on API* 有一个调用方式是 *$(document).on('click', 'nav a', false)*, 这个调用的意图
是取消 *nav* 中的所有 *a* 标签的默认点击行为,调用仅仅输入了 *false*, 引起了我的兴趣.
我发现, 下面的两种方式都可以取消元素默认行为:
```
ele.addEventListener('click', function(event){
    event.preventDefault();
}, false);

ele.addEventListener('click', function(){
    return false;
}, false);
```
对比, 第二种显得简洁高效,而且不用在 callback 添加一个 event argument 再调用 event 对象的 preventDefault,
在一些对参数数量要求苛刻或情况复杂的时候显得有效.

###判断两个对象相同
之前不知道怎么判断两个对象(函数, 数组等)是相同的, 借鉴了 Zepto 中的 zid 函数之后豁然开朗,因为对象是引用类
型,是依照堆内存存放的,我们可以在对象中添加一个 id 属性,如果这两个对象的 id 属性相同, 就是相同的对象,前提是
要保证 id 属性的值不同的对象不会重复.在 Zepto 中是用于判断两个函数对象是否相同(主要是为了匿名函数):
```
var _id = 1;

function zid(element){
    return element._id || element._id = _id ++;
}

if(zid(obj1) == zid(obj2)) {
    return true;
}
```


