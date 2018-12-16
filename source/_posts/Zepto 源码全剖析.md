title: Zepto 源码全剖析
date: 2017-02-16 22:17:24
categories: JavaScript

---
这是我第一次做源码剖析类的文章,既是对自己的代码素养的提高,也是希望能够为对 Zepto 源码剖析感兴趣的同学有所
启发.
<!--more-->
```
//     Zepto.js
//     (c) 2010-2016 Thomas Fuchs
//     Zepto.js may be freely distributed under the MIT license.

var Zepto = (function() {
  var undefined, 
    key, 
    $, 
    classList, 
    emptyArray = [], 
    concat = emptyArray.concat, 
    filter = emptyArray.filter, 
    slice = emptyArray.slice,

    //save the window.document
    document = window.document,
    elementDisplay = {}, classCache = {},
    cssNumber = { 
      'column-count': 1, 
      'columns': 1, 
      'font-weight': 1, 
      'line-height': 1,
      'opacity': 1, 
      'z-index': 1, 
      'zoom': 1 
    },
    fragmentRE = /^\s*<(\w+|!)[^>]*>/,
    singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/,
    tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig,
    rootNodeRE = /^(?:body|html)$/i,

    //大写字母
    capitalRE = /([A-Z])/g,

    // special attributes that should be get/set via method calls
    methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'],

    //DOM 连接位置
    adjacencyOperators = [ 'after', 'prepend', 'before', 'append' ],
    table = document.createElement('table'),
    tableRow = document.createElement('tr'),
    containers = {
      'tr': document.createElement('tbody'),
      'tbody': table, 
      'thead': table, 
      'tfoot': table,
      'td': tableRow, 
      'th': tableRow,
      '*': document.createElement('div')
    },

    simpleSelectorRE = /^[\w-]*$/,
    
    class2type = {},

    toString = class2type.toString,
    
    zepto = {},
    camelize, uniq,
    //临时的 div,用于 zepto.matches
    tempParent = document.createElement('div'),
    //属性映射
    propMap = {
      'tabindex': 'tabIndex',
      'readonly': 'readOnly',
      'for': 'htmlFor',
      'class': 'className',
      'maxlength': 'maxLength',
      'cellspacing': 'cellSpacing',
      'cellpadding': 'cellPadding',
      'rowspan': 'rowSpan',
      'colspan': 'colSpan',
      'usemap': 'useMap',
      'frameborder': 'frameBorder',
      'contenteditable': 'contentEditable'
    },
    isArray = Array.isArray ||
      function(object){ return object instanceof Array }

  //
  zepto.matches = function(element, selector) {
    //如果没有选择器,或者没有元素,或者节点不是元素类型,返回 false
    if (!selector || !element || element.nodeType !== 1) return false

    //Web API, element.matches, 兼容性写法
    var matchesSelector = element.matches || element.webkitMatchesSelector ||
                          element.mozMatchesSelector || element.oMatchesSelector ||
                          element.matchesSelector
    if (matchesSelector) return matchesSelector.call(element, selector)
    // fall back to performing a selector:
    var match, parent = element.parentNode, temp = !parent
    //如果没有父元素, 就将父元素暂时存到临时变量 tempParent 中
    if (temp) (parent = tempParent).appendChild(element)
    //~ 按位取反 - 1 , 即 1 变成 -2 , 0 变成 -1,当 match 值为 -1 的时候,代表没有找到
    match = ~zepto.qsa(parent, selector).indexOf(element)
    temp && tempParent.removeChild(element)
    return match
  }

  //判断值的类型, 如果是对象的话, 使用 Object.prototype.toString.call(obj); 可以知道对象的类型
  function type(obj) {
    //toString 转型函数
    return obj == null ? String(obj) :
      class2type[toString.call(obj)] || "object"
  }

  //判断是不是函数类型
  function isFunction(value) { 
      return type(value) == "function" 
  }
  //判断是不是 window 对象
  function isWindow(obj)     { 
    return obj != null && obj == obj.window 
  }
  //判断是不是 document 对象
  function isDocument(obj)   { 
      return obj != null && obj.nodeType == obj.DOCUMENT_NODE 
  }
  //判断是不是对象类型
  function isObject(obj)     { return type(obj) == "object" }

  //是不是原生对象 Object
  function isPlainObject(obj) {
    return isObject(obj) && !isWindow(obj) && Object.getPrototypeOf(obj) == Object.prototype
  }

  //判断是否是类数组对象
  function likeArray(obj) {
    var length = !!obj && 'length' in obj && obj.length,
      type = $.type(obj)

    return 'function' != type && !isWindow(obj) && (
      'array' == type || length === 0 ||
        (typeof length == 'number' && length > 0 && (length - 1) in obj)
    )
  }


  function compact(array) { return filter.call(array, function(item){ return item != null }) }
  //扁平化数组
  function flatten(array) { return array.length > 0 ? $.fn.concat.apply([], array) : array }
  
  //将字符串转化为驼峰写法
  camelize = function(str){ return str.replace(/-+(.)?/g, function(match, chr){ return chr ? chr.toUpperCase() : '' }) }
  
  //将字符串转化为连接符
  function dasherize(str) {
    return str.replace(/::/g, '/')
           .replace(/([A-Z]+)([A-Z][a-z])/g, '$1_$2')
           .replace(/([a-z\d])([A-Z])/g, '$1_$2')
           .replace(/_/g, '-')
           .toLowerCase()
  }

  //检查数组中是不是有重复的元素, 将重复的元素去掉,返回一个数组
  uniq = function(array){ return filter.call(array, function(item, idx){ return array.indexOf(item) == idx }) }

  //新建匹配类名的正则表达式
  function classRE(name) {
    return name in classCache ?
      classCache[name] : (classCache[name] = new RegExp('(^|\\s)' + name + '(\\s|$)'))
  }

  //为数值添加 px 后缀单位
  function maybeAddPx(name, value) {
    return (typeof value == "number" && !cssNumber[dasherize(name)]) ? value + "px" : value
  }

  //设置元素默认 display 样式
  function defaultDisplay(nodeName) {
    var element, display
    if (!elementDisplay[nodeName]) {
      element = document.createElement(nodeName)
      document.body.appendChild(element)
      display = getComputedStyle(element, '').getPropertyValue("display")
      element.parentNode.removeChild(element)
      display == "none" && (display = "block")
      elementDisplay[nodeName] = display
    }
    return elementDisplay[nodeName]
  }

  //获取子元素
  function children(element) {
    return 'children' in element ?
      //节点的 children 属性返回节点的元素字节点集合
      slice.call(element.children) :
      //如果没有 children, 那么就用 childNodes, 返回包含的元素节点就可以了(nodeType === 1)
      $.map(element.childNodes, function(node){ if (node.nodeType == 1) return node })
  }

  function Z(dom, selector) {
    //组建 Zepto 对象: 
    //{
    //   dom: []
    //   length: 0
    //   selector: ''
    //}
    var i, len = dom ? dom.length : 0
    for (i = 0; i < len; i++) this[i] = dom[i]
    this.length = len
    this.selector = selector || ''
  }

  // `$.zepto.fragment` takes a html string and an optional tag name
  // to generate DOM nodes from the given html string.
  // The generated DOM nodes are returned as an array.
  // This function can be overridden in plugins for example to make
  // it compatible with browsers that don't support the DOM fully.
  //fragment 函数接收 html 字符串和一个 tag 名, 在 tag 的后代生成 html 标签,
  //返回的结果是以节点数组形式返回
  //核心方法
  zepto.fragment = function(html, name, properties) {
    var dom, nodes, container
    console.log("html is " + html);
    console.log("name is " + name);
    console.log("properties is " + properties);
    // A special case optimization for a single tag
    //RegExp.$[1-99] 是内置正则对象的属性,数字代表正则表达式匹配的第几个字符串
    //这里的正则对象是全局正则对象
    if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1))
    console.log("RegExp.$1 is " + RegExp.$1);
    console.log('dom is ' + dom);
    console.log(dom);
    if (!dom) {
      console.log("no dom");
      if (html.replace) {
       html = html.replace(tagExpanderRE, "<$1></$2>")
       console.log(html); 
      }
      console.log(name);
      if (name === undefined) name = fragmentRE.test(html) && RegExp.$1
      if (!(name in containers)) name = '*'
      console.log(name);
      container = containers[name]
      container.innerHTML = '' + html
      dom = $.each(slice.call(container.childNodes), function(){
        container.removeChild(this)
      })
    }

    if (isPlainObject(properties)) {//是否是原生对象,但是不是 window 对象(document.window)
      console.log("isPlainObject(properties) is " + isPlainObject(properties));
      console.log(dom);
      nodes = $(dom);
      console.log(typeof $(dom));
      console.log(typeof nodes);
      // console.log(nodes);
      // $.each = function(elements, callback){
      //   var i, key
      //   if (likeArray(elements)) { //Object.toString.call(arg) 可以检验真正的对象类型
      //     如果是数组类型
      //     for (i = 0; i < elements.length; i++)
      //       if (callback.call(elements[i], i, elements[i]) === false) return elements
      //     } else {
      //     如果是对象类型
      //     for (key in elements)
      //       if (callback.call(elements[key], key, elements[key]) === false) return elements
      //     }

      //     return elements
      // }
      //问题在于为什么对 nodes 进行操作返回的是 dom? 是不是赋值操作是引用而不是复制?
      //字面量变量才是基本类型, 用 new 操作符都是对象类型,基本类型在栈内部,对象类型在堆内部,所以基本类型才会 play itself,
      //但是对象类型是 itself
      //因为这是引用类型,所以进行操作的时候 nodes 变, dom 也进行变值
      //所以返回的是 dom,为啥多费周章要多一个变量呀?
      $.each(properties, function(key, value) {
        // methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset']
        // console.log(nodes[key]);
        console.log(nodes);
        //如果有上面这些属性的话,就使用 Zepto 对象的方法赋值
        if (methodAttributes.indexOf(key) > -1) nodes[key](value)
        //如果不是就用 attr 方法
        else nodes.attr(key, value)
      })
    }
    console.log(nodes);
    console.log(dom);
    console.log(nodes === $(dom));//是怎么判断两个变量是一样的?
    return dom;
  }

  // `$.zepto.Z` swaps out the prototype of the given `dom` array
  // of nodes with `$.fn` and thus supplying all the Zepto functions
  // to the array. This method can be overridden in plugins.

  zepto.Z = function(dom, selector) {
    //创建一个 Zepto 对象, Object => [dom, length, selector]
    return new Z(dom, selector)
  }

  // `$.zepto.isZ` should return `true` if the given object is a Zepto
  // collection. This method can be overridden in plugins.
  //检验对象是不是 Zepto 对象
  zepto.isZ = function(object) {
    return object instanceof zepto.Z
  }

  // `$.zepto.init` is Zepto's counterpart to jQuery's `$.fn.init` and
  // takes a CSS selector and an optional context (and handles various
  // special cases).
  // This method can be overridden in plugins.
  // $() 这个函数, 这是 Zepto 起手式, 最核心的 API
  zepto.init = function(selector, context) {
    var dom;
    // If nothing given, return an empty Zepto collection
    if (!selector) {
      //如果没有选择器,那么返回一个 zepto 对象
      //zepto.Z = function(dom, selector) {
      //    return new Z(dom, selector)
      //}
      //function Z(dom, selector) {
        //var i, len = dom ? dom.length : 0
        //for (i = 0; i < len; i++) this[i] = dom[i]
        //this.length = len
        //this.selector = selector || ''
      //}
      //返回一个 Zepto 对象可以使用 Zepto 中的工具函数
      return zepto.Z()
    }
    //如果有选择器  
    // Optimize for string selectors
    else if (typeof selector == 'string') {
      // return str.replace(/(^\s* | \s*$)/g, '');
      //去除字符串的前后空格 // str == null ? '' : String.prototype.trim.call(str)
      selector = selector.trim() 
      // If it's a html fragment, create nodes from it
      // Note: In both Chrome 21 and Firefox 15, DOM error 12
      // is thrown if the fragment doesn't begin with <
      
      //如果是 html 标签
      if (selector[0] == '<' && fragmentRE.test(selector))
        //看一下 fragment 函数
        dom = zepto.fragment(selector, RegExp.$1, context), selector = null
      // If there's a context, create a collection on that context first, and select
      // nodes from there
      //如果 context 变量有值,先找到这个 context, 再从这里找值
      // find() 函数?
      else if (context !== undefined) return $(context).find(selector)
      // If it's a CSS selector, use it to select nodes.
      //如果只有选择器,则从 document 对象中查找元素
      else dom = zepto.qsa(document, selector)
    }
    // If a function is given, call it when the DOM is ready
    //selector 不是选择器,而是一个函数
    //如果这个 selector 是一个函数对象, 那么就在 ready 执行这个函数
    //而 ready 状态对应的是 DOMContentLoaded 事件或者 load 事件(document 的 DOMContentLoaded 或者是 window 的 load
    //事件)
    //如果是原生函数对象
    else if (isFunction(selector)) return $(document).ready(selector)
    // If a Zepto collection is given, just return it
    //如果是 Zepto 对象,返回这个 Zepto 对象
    else if (zepto.isZ(selector)) return selector
    else {
      // normalize array if an array of nodes is given
      // 如果传入的参数是 DOM 节点,那么就将这些 DOM 节点规范化,所谓规范化就是将 DOM 对象以数组形式返回
      //如果是数组
      //function compact(array) { return filter.call(array, function(item){ return item != null }) }
      //emptyArray = [],  
      //filter = emptyArray.filter
      //使用原生对象的 filter 方法,
      if (isArray(selector)) {
        dom = compact(selector);
        console.log(dom); 
      }
      // Wrap DOM nodes.
      // 如果是 DOM 原生对象
      else if (isObject(selector))
        // DOM 原生对象转化为数组
        dom = [selector], selector = null
      // If it's a html fragment, create nodes from it
      //如果是 HTML 字符串,则创建 HTML 节点
      else if (fragmentRE.test(selector))
        //使用 fragment API 创建 HTML 节点
        dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
      // If there's a context, create a collection on that context first, and select
      // nodes from there
      // 如果有 context 变量(即 $('#id', $('body')))
      else if (context !== undefined) return $(context).find(selector)
      // And last but no least, if it's a CSS selector, use it to select nodes.
      else dom = zepto.qsa(document, selector)
    }

    // create a new Zepto collection from the nodes found
    //最终是返回一个 Zepto 对象
    return zepto.Z(dom, selector)
  }

  // `$` will be the base `Zepto` object. When calling this
  // function just call `$.zepto.init, which makes the implementation
  // details of selecting nodes and creating Zepto collections
  // patchable in plugins.
  $ = function(selector, context){
    //这就是我们常用的 $ 函数
    return zepto.init(selector, context)
  }

  function extend(target, source, deep) {
    for (key in source)
      if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
        if (isPlainObject(source[key]) && !isPlainObject(target[key]))
          target[key] = {}
        if (isArray(source[key]) && !isArray(target[key]))
          target[key] = []
        extend(target[key], source[key], deep)
      }
      else if (source[key] !== undefined) target[key] = source[key]
  }

  // Copy all but undefined properties from one or more
  // objects to the `target` object.
  $.extend = function(target){
    var deep, args = slice.call(arguments, 1)
    if (typeof target == 'boolean') {
      deep = target
      target = args.shift()
    }
    args.forEach(function(arg){ extend(target, arg, deep) })
    return target
  }

  // `$.zepto.qsa` is Zepto's CSS selector implementation which
  // uses `document.querySelectorAll` and optimizes for some special cases, like `#id`.
  // This method can be overridden in plugins.
  //这个函数是用来选择 CSS 选择器的 querySelectorAll(核心方法)
  zepto.qsa = function(element, selector){
    var found,
        maybeID = selector[0] == '#', //如果有 # 符号就是 id 选择器
        maybeClass = !maybeID && selector[0] == '.',  // 如果没有,有 . 则是类选择器
        //如果只有标签名(没有 id 也没有类) 下面的表达式意思是如果有 id 或 类,则取符号后的名字,如果没有类和名字,则选择出单标签名
        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // Ensure that a 1 char tag name still gets checked
        //看是不是只有一个选择器
        isSimple = simpleSelectorRE.test(nameOnly)

        //第一个判断 element.getElementById 这个方法是否存在, 是不是一个单选择器, 是不是 id 选择器
    return (element.getElementById && isSimple && maybeID) ? // Safari DocumentFragment doesn't have getElementById
      //如果是使用 getElementById 找到元素, 如果能找到,则以数组的形式返回,如果没有则返回空数组, 问题在于为什么要用数组的形式?
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

  function filtered(nodes, selector) {
    return selector == null ? $(nodes) : $(nodes).filter(selector)
  }

  $.contains = document.documentElement.contains ?
    function(parent, node) {
      return parent !== node && parent.contains(node)
    } :
    function(parent, node) {
      while (node && (node = node.parentNode))
        if (node === parent) return true
      return false
    }

  function funcArg(context, arg, idx, payload) {
    return isFunction(arg) ? arg.call(context, idx, payload) : arg
  }

  function setAttribute(node, name, value) {
    value == null ? node.removeAttribute(name) : node.setAttribute(name, value)
  }

  // access className property while respecting SVGAnimatedString
  function className(node, value){
    var klass = node.className || '',
        svg   = klass && klass.baseVal !== undefined
        // console.log(klass.baseVal);
    if (value === undefined) return svg ? klass.baseVal : klass
    svg ? (klass.baseVal = value) : (node.className = value)
  }

  // "true"  => true
  // "false" => false
  // "null"  => null
  // "42"    => 42
  // "42.5"  => 42.5
  // "08"    => "08"
  // JSON    => parse if valid
  // String  => self
  function deserializeValue(value) {
    try {
      return value ?
        value == "true" ||
        ( value == "false" ? false :
          value == "null" ? null :
          +value + "" == value ? +value :
          /^[\[\{]/.test(value) ? $.parseJSON(value) :
          value )
        : value
    } catch(e) {
      return value
    }
  }

  //====================静态方法集==================================
  $.type = type
  $.isFunction = isFunction
  $.isWindow = isWindow
  $.isArray = isArray
  $.isPlainObject = isPlainObject

  $.isEmptyObject = function(obj) {
    var name
    for (name in obj) return false
    return true
  }

  $.isNumeric = function(val) {
    var num = Number(val), type = typeof val
    return val != null && type != 'boolean' &&
      (type != 'string' || val.length) &&
      !isNaN(num) && isFinite(num) || false
  }

  $.inArray = function(elem, array, i){
    return emptyArray.indexOf.call(array, elem, i)
  }

  $.camelCase = camelize
  //去除字符串空格
  $.trim = function(str) {
    return str == null ? "" : String.prototype.trim.call(str)
  }

  // plugin compatibility
  $.uuid = 0
  $.support = { }
  $.expr = { }
  $.noop = function() {}
  //接收一个数组,利用 callback 函数来返回一个符合条件的数组
  $.map = function(elements, callback){
    var value, values = [], i, key
    if (likeArray(elements))
      for (i = 0; i < elements.length; i++) {
        value = callback(elements[i], i)
        if (value != null) values.push(value)
      }
    else
      for (key in elements) {
        value = callback(elements[key], key)
        if (value != null) values.push(value)
      }
    //将数组 values 扁平化,所谓扁平化就是
    return flatten(values)
  }

  $.each = function(elements, callback){
    var i, key
    if (likeArray(elements)) {
      for (i = 0; i < elements.length; i++)
        if (callback.call(elements[i], i, elements[i]) === false) return elements
    } else {
      for (key in elements)
        if (callback.call(elements[key], key, elements[key]) === false) return elements
    }

    return elements
  }

  $.grep = function(elements, callback){
    return filter.call(elements, callback)
  }

  if (window.JSON) $.parseJSON = JSON.parse
  //====================静态方法集 end ==================================


  // Populate the class2type map
  $.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type[ "[object " + name + "]" ] = name.toLowerCase()
  })

  // Define methods that will be available on all
  // Zepto collections
  //工具函数,扩充丰富 Zepto 对象原型链 操作集
  $.fn = {
    constructor: zepto.Z,
    length: 0,

    // Because a collection acts like an array
    // copy over these useful array functions.
    //================数组方法集==================
    forEach: emptyArray.forEach,
    reduce: emptyArray.reduce,
    push: emptyArray.push,
    sort: emptyArray.sort,
    splice: emptyArray.splice,
    indexOf: emptyArray.indexOf,
    //================数组方法集 end =============
    

    //拼接 API, 拼接数组
    concat: function(){
      var i, value, args = []
      for (i = 0; i < arguments.length; i++) {
        value = arguments[i]
        args[i] = zepto.isZ(value) ? value.toArray() : value
      }
      //使用数组原生的 concat 方法,如果是 Zepto 对象,则将对象转化为数组,否则直接拼接
      //将 args 数组与 Zepto 原本的数组进行拼接
      return concat.apply(zepto.isZ(this) ? this.toArray() : this, args)
    },

    // `map` and `slice` in the jQuery API work differently
    // from their array counterparts
    //将原数组以 callback 希望的形式重新组合
    map: function(fn){
      return $($.map(this, function(el, i){ return fn.call(el, i, el) }))
    },
    //对原生的 Array slice 包装(数组选取方法)
    slice: function(){
      return $(slice.apply(this, arguments))
    },
    //ready 函数, 用于在文档加载完毕后执行函数
    ready: function(callback){
      // don't use "interactive" on IE <= 10 (it can fired premature)
      if (document.readyState === "complete" ||
          (document.readyState !== "loading" && !document.documentElement.doScroll))
        setTimeout(function(){ callback($) }, 0)
      else {
        var handler = function() {
          document.removeEventListener("DOMContentLoaded", handler, false)
          window.removeEventListener("load", handler, false)
          callback($)
        }
        document.addEventListener("DOMContentLoaded", handler, false)
        window.addEventListener("load", handler, false)
      }
      return this
    },
    //如果传入的参数 idx(index) 是 undefinded(未传参数),那么就返回这个数组
    //如果有 idx, 先判断 idx 是大于 0,还是小于零
    //如果大于 0, 则返回数组对象中的下标元素,这时是 DOM 元素 this[idx]
    //小于 0,则是从数组后面开始选取元素  this[idx + this.length] 
    //用法: array.get(0);
    get: function(idx){
      return idx === undefined ? slice.call(this) : this[idx >= 0 ? idx : idx + this.length]
    },
    //将对象转化为数组
    toArray: function(){ return this.get() },

    //返回数组长度
    size: function(){
      return this.length
    },
    //从有效的父元素中删除这个子元素
    remove: function(){
      return this.each(function(){
        if (this.parentNode != null)
          this.parentNode.removeChild(this)
      })
    },
    //遍历数组
    //需要弄懂原生的 every,原生 every 接收一个函数,如果返回 false 就停止循环, 理解 every 与 map, some, forEach, filter 区别
    //every 就是每个都, filter 选出符合条件的, some 是只要有, forEach 是每一个都要做某事, map 是每个做完某事后排队 
    each: function(callback){
      emptyArray.every.call(this, function(el, idx){

        return callback.call(el, idx, el) !== false
      })
      return this
    },

    //其实是对数组类型的 filter 函数包装
    filter: function(selector){
      if (isFunction(selector)) return this.not(this.not(selector))
      return $(filter.call(this, function(element){
        return zepto.matches(element, selector)
      }))
    },

    //在某个元素集合中添加元素
    add: function(selector,context){
      return $(uniq(this.concat($(selector,context))))
    },

    //判断数组中的第一个元素是不是符合 CSS 选择器
    is: function(selector){
      //没有找到即不是的时候, zepto.matches 返回 -1(false)
      return this.length > 0 && zepto.matches(this[0], selector)
    },

    //与 filter 相反的功能,将不符合的元素选出来
    not: function(selector){
      var nodes=[]
      if (isFunction(selector) && selector.call !== undefined)
        this.each(function(idx){
          //如果不包含(不符合 selector 条件), 将该元素加入 nodes 数组中
          if (!selector.call(this,idx)) nodes.push(this)
        })
      else {
        //如果是字符串参数(CSS 选择器), 先找到符合条件的元素
        var excludes = typeof selector == 'string' ? this.filter(selector) :
        //如果不是字符串参数,如果是一个类数组对象,或者函数对象,则将这个类数组对象转化为数组,否则新建一个 Zepto 对象(数组形式)
          (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector)

        //如果 excludes 不包含数组中的元素,则将这个不包含在内的元素加入到 nodes 中
        this.forEach(function(el){
          if (excludes.indexOf(el) < 0) nodes.push(el)
        })
      }

      //返回 nodes 
      return $(nodes)
    },

    //是否包含符合条件的元素
    has: function(selector){
      //过滤出符合条件的元素
      return this.filter(function(){

        return isObject(selector) ?
          //如果 selector 是对象
          //
          $.contains(this, selector) :
          //selector 不是对象(是字符串)
          //如果有找到匹配的元素,就会有数组长度,但是没有数组长度,即返回 0 ,则 filter 就会不收纳这个元素
          $(this).find(selector).size()
      })
    },

    //寻找集合中的某个元素
    eq: function(idx){
      console.log(idx + 1);
      console.log(+ idx);
      console.log(+ idx + 1);
      //这里巧妙的在于在 idx 前面添加一个 '+', 将字符串值转化为真正的数值
      //如果直接是 idx + 1, 有可能会是字符串拼接,但是使用 + idx + 1 的话会将 idx 先转化为数值再 + 1
      return idx === -1 ? this.slice(idx) : this.slice(idx, + idx + 1)
    },

    //集合中的第一个元素
    first: function(){
      var el = this[0]
      return el && !isObject(el) ? el : $(el)
    },

    //集合中的最后一个元素
    last: function(){
      var el = this[this.length - 1]
      return el && !isObject(el) ? el : $(el)
    },


    //寻找元素
    find: function(selector){ //传入选择器字符串
      var result, $this = this  //这里的 this 是 Zepto 对象
      console.log('$this is ' + $this);
      console.log($this);
      //如果没有传入 selector, 那么返回一个空 Zepto 对象
      if (!selector) result = $()
      //如果是一个 DOM 对象
      else if (typeof selector == 'object') {
          console.log('object');
          //result 等于过滤后的数组
          result = $(selector).filter(function(){
          var node = this
          //这里的 parent 是当前调用 API 的元素, function(currentVal, index, arr){}
          return emptyArray.some.call($this, function(parent){

            console.log('parent is ' + parent);
            console.log(parent);
            return $.contains(parent, node)
          })
        })        
      }
      //如果是一个 Zepto 对象,且对象 length 长度为 1
      else if (this.length == 1) {
        console.log('string!!!!!!!');
        result = $(zepto.qsa(this[0], selector)); 
      }
      //否则(Zepto 对象, 对象数组长度大于 1 )
      //result 为返回数组
      else result = this.map(function(){ 
        return zepto.qsa(this, selector) 
      });
      return result;
    },

    //获取符合 CSS 选择器的第一个父元素
    closest: function(selector, context){
      //
      var nodes = [], collection = typeof selector == 'object' && $(selector)
      //
      this.each(function(_, node){
        //存在 node, 并且 node 不包含符合选择器的元素, 直到找到包含选择器的 node 
        while (node && !(collection ? collection.indexOf(node) >= 0 : zepto.matches(node, selector)))
          //选择 node 的父元素
          node = node !== context && !isDocument(node) && node.parentNode
        
        if (node && nodes.indexOf(node) < 0) nodes.push(node)
      })
      //
      return $(nodes)
    },

    //==================================== DOM 操作 =============================================
    //返回集合中所有的父元素, 过滤符合条件的父元素
    parents: function(selector){
      var ancestors = [], nodes = this
      while (nodes.length > 0)
        nodes = $.map(nodes, function(node){
          //node 存在父元素, node 不是 document 类型, node 不存在于 ancestors 中(保证 node 在 ancestors 的唯一性)
          if ((node = node.parentNode) && !isDocument(node) && ancestors.indexOf(node) < 0) {
            ancestors.push(node)
            return node
          }
        })
      return filtered(ancestors, selector)
    },

    //直接获取父元素
    parent: function(selector){
      //使用 pluck 返回 parentNode, 然后将返回来的对象过滤
      return filtered(uniq(this.pluck('parentNode')), selector)
    },

    //返回直接后代
    children: function(selector){
      //返回后代元素之后再根据 selector 过滤
      return filtered(this.map(function(){ return children(this) }), selector)
    },

    //内容
    contents: function() {
      //返回元素包含的内容, 使用 DOM 的 contentDocument 属性或者 childNodes 返回内容
      return this.map(function() { return this.contentDocument || slice.call(this.childNodes) })
    },

    //兄弟元素
    siblings: function(selector){
      
      return filtered(this.map(function(i, el){
        //使用 el.parentNode, 过滤出 el.parentNode 的子元素, 但是不包括 this 中的元素
        return filter.call(children(el.parentNode), function(child){ return child!==el })
      }), selector)
    },

    //清空后代
    empty: function(){
      //使用 innerHTML 为 '', 高效有效地清空子元素
      return this.each(function(){ this.innerHTML = '' })
    },
    // `pluck` is borrowed from Prototype.js
    //使用对象方括号访问 DOM 对象的 parentNode 等属性
    pluck: function(property){
      return $.map(this, function(el){ return el[property] })
    },
    
    //动效 API , 显示
    show: function(){

      return this.each(function(){

        this.style.display == "none" && (this.style.display = '')
        //使用 web api-- getComputedStyle 获取 CSS 样式(元素, 样式)
        //这里明显是获取本元素全部样式属性
        if (getComputedStyle(this, '').getPropertyValue("display") == "none")
          this.style.display = defaultDisplay(this.nodeName)
      })
    },

    //替换内容
    replaceWith: function(newContent){
      console.log(this.before);
      return this.before(newContent).remove()
    },
    

    //=============================== 包裹元素 ================================
    wrap: function(structure){
      var func = isFunction(structure)
      if (this[0] && !func)
        var dom   = $(structure).get(0),
            clone = dom.parentNode || this.length > 1

      return this.each(function(index){
        $(this).wrapAll(
          func ? structure.call(this, index) :
            clone ? dom.cloneNode(true) : dom
        )
      })
    },
    wrapAll: function(structure){
      if (this[0]) {
        $(this[0]).before(structure = $(structure))
        var children
        // drill down to the inmost element
        while ((children = structure.children()).length) structure = children.first()
        $(structure).append(this)
      }
      return this
    },
    wrapInner: function(structure){
      var func = isFunction(structure)
      return this.each(function(index){
        var self = $(this), contents = self.contents(),
            dom  = func ? structure.call(this, index) : structure
        contents.length ? contents.wrapAll(dom) : self.append(dom)
      })
    },
    unwrap: function(){
      this.parent().each(function(){
        $(this).replaceWith($(this).children())
      })
      return this
    },
    //=============================== 包裹元素 end ================================

    clone: function(){
      return this.map(function(){ return this.cloneNode(true) })
    },
    hide: function(){
      return this.css("display", "none");
    },
    toggle: function(setting){
      return this.each(function(){
        var el = $(this)
        //这些前面加 ';' 是为了避免在多文件压缩的时候出现缺少 ; 的错误 
        ;(setting === undefined ? el.css("display") == "none" : setting) ? el.show() : el.hide()
      })
    },

    //遍历
    prev: function(selector){ return $(this.pluck('previousElementSibling')).filter(selector || '*') },
    next: function(selector){ return $(this.pluck('nextElementSibling')).filter(selector || '*') },


    html: function(html){
      return 0 in arguments ?
        this.each(function(idx){
          var originHtml = this.innerHTML
          $(this).empty().append( funcArg(this, html, idx, originHtml) )
        }) :
        (0 in this ? this[0].innerHTML : null)
    },
    text: function(text){
      return 0 in arguments ?
        this.each(function(idx){
          var newText = funcArg(this, text, idx, this.textContent)
          this.textContent = newText == null ? '' : ''+newText
        }) :
        (0 in this ? this.pluck('textContent').join("") : null)
    },
    //===============================DOM 操作 END ==================================

    //============================================属性操作===============================================
    //设置属性值或获取属性值
    attr: function(name, value){
      var result
      return (typeof name == 'string' && !(1 in arguments)) ?
        //getAttirbute 是 DOM 原生的属性操作方法
        (0 in this && this[0].nodeType == 1 && (result = this[0].getAttribute(name)) != null ? result : undefined) :
        this.each(function(idx){
          if (this.nodeType !== 1) return
          if (isObject(name)) for (key in name) setAttribute(this, key, name[key])
            //在 setAtteribute 中利用的就是 DOM 自带的 setAttribute 与 removeAttribute
          else setAttribute(this, name, funcArg(this, value, idx, this.getAttribute(name)))
        })
    },
    //移除属性值
    removeAttr: function(name){
      return this.each(function(){ this.nodeType === 1 && name.split(' ').forEach(function(attribute){
        //在 setAtteribute 中利用的就是 DOM 自带的 setAttribute 与 removeAttribute
        setAttribute(this, attribute)
      }, this)})
    },
    //读取属性
    prop: function(name, value){
      name = propMap[name] || name
      return (1 in arguments) ?
        this.each(function(idx){
          this[name] = funcArg(this, value, idx, this[name])
        }) :
        (this[0] && this[0][name])
    },
    //移除属性
    removeProp: function(name){
      name = propMap[name] || name
      //值得注意的是这里使用 delete 操作符来完成移除操作, 但是像 className 这些属性浏览器禁止移除
      return this.each(function(){ delete this[name] })
    },
    //读取或写入 data 属性(自定义属性)
    data: function(name, value){
      var attrName = 'data-' + name.replace(capitalRE, '-$1').toLowerCase()

      var data = (1 in arguments) ?
        this.attr(attrName, value) :
        this.attr(attrName)

      return data !== null ? deserializeValue(data) : undefined
    },
    //设置或获取元素的值
    val: function(value){
      if (0 in arguments) {
        if (value == null) value = ""
        return this.each(function(idx){
          this.value = funcArg(this, value, idx, this.value)
        })
      } else {
        return this[0] && (this[0].multiple ?
           $(this[0]).find('option').filter(function(){ return this.selected }).pluck('value') :
           this[0].value)
      }
    },
    //是否有这个类名
    hasClass: function(name){
      if (!name) return false
        console.log(this);
      //需要知道 some 方法的使用详解, some(callback[, thisObject]);
      //如果在 callback 函数后面加上一些变量或者值, this 值就变成传入的那个值
      //Array.prototype.some.call([ele], function(ele){
      //    console.log(this);  这里的 this 是后面传入的值 /\s/ 或 true(包装类型) 
      //}, /\s/或 true);
      return emptyArray.some.call(this, function(el){
        console.log(this);
        console.log(el);
        console.log(className(el));
        console.log(classRE(name));
        return this.test(className(el))
      }, classRE(name))
    },
    //添加某个类名
    addClass: function(name){
      if (!name) return this
      return this.each(function(idx){
        if (!('className' in this)) return
        // console.log(this.className);

        classList = []
        var cls = className(this), newName = funcArg(this, name, idx, cls)
        // console.log(this);
        //cls 是元素现有的类名, newName 是新增类名的字符串
        newName.split(/\s+/g).forEach(function(klass){
          //forEach 函数后面指定 this 为元素
          console.log(this);
          //如果元素中还没有这个类名,就将在这个类名加入 classlist 数组中
          if (!$(this).hasClass(klass)) classList.push(klass)
        }, this)
        //如果有新类名, 就将类名加入元素的 className 中
        classList.length && className(this, cls + (cls ? " " : "") + classList.join(" "))
      })
    },
    //移除某个类名
    removeClass: function(name){
      return this.each(function(idx){
        if (!('className' in this)) return
        if (name === undefined) return className(this, '')
        //classlist 是类名数组
        classList = className(this)
        funcArg(this, name, idx, classList).split(/\s+/g).forEach(function(klass){
          //将要移除的类名替换成空格
          classList = classList.replace(classRE(klass), " ")

        })
        //将元素的类名 className 替换成去除空格后的 classList
        className(this, classList.trim())
      })
    },
    //如果存在类名则移除,如果不存在则添加
    toggleClass: function(name, when){
      if (!name) return this
      return this.each(function(idx){
        var $this = $(this), names = funcArg(this, name, idx, className(this))
        names.split(/\s+/g).forEach(function(klass){
          (when === undefined ? !$this.hasClass(klass) : when) ?
            $this.addClass(klass) : $this.removeClass(klass)
        })
      })
    },
    //=======================================属性操作 end =============================================

    //获取数组的第几个元素
    index: function(element){
      return element ? this.indexOf($(element)[0]) : this.parent().children().indexOf(this[0])
    },
    //样式操作
    //有几种调用形式:
    //1.css('style'); 返回值
    //2.css('style', 'value'); 设置值
    //3.css('style', ''); 清空值
    //4.css({ style: 'value'});  这样 length == 1 ,设置值
    //5.css([prop]);  读取值
    css: function(property, value){
      console.log(arguments);
      //(arguments 是类数组对象)
      if (arguments.length < 2) {
        //是第一,第五种调用形式
        var element = this[0]
        if (typeof property == 'string') {
          //第一种调用形式
          //如果没有 element,直接返回
          if (!element) return
          //如果有 element, 那么使用 style 对象访问[驼峰写法],或者使用 getComputedStyle(ele, null).getPropertyValue()
          return element.style[camelize(property)] || getComputedStyle(element, '').getPropertyValue(property)
        } else if (isArray(property)) {
          //第五种形式
          if (!element) return
          var props = {}
          var computedStyle = getComputedStyle(element, '')
          $.each(property, function(_, prop){
            props[prop] = (element.style[camelize(prop)] || computedStyle.getPropertyValue(prop))
          })
          return props
        }
      }

      //第二,三,四种调用方式
      //css 样式字符串
      var css = ''
      //如果属性是字符串形式
      if (type(property) == 'string') {
        if (!value && value !== 0)
          //dasherize 是将驼峰写法转化为连接符写法,如果没有赋值或值为 undefined/null 的话, 那么就移除元素上的属性
          this.each(function(){ this.style.removeProperty(dasherize(property)) })
        else
          //拼接字符串
          css = dasherize(property) + ":" + maybeAddPx(property, value)
      } else {
        //如果属性是对象形式
        for (key in property)

          if (!property[key] && property[key] !== 0)
            this.each(function(){ this.style.removeProperty(dasherize(key)) })
          else
            //拼接字符串
            css += dasherize(key) + ':' + maybeAddPx(key, property[key]) + ';'
      }
      
      //使用 cssText 与 css 字符串变量一次性写入修改操作,高效便捷
      return this.each(function(){ this.style.cssText += ';' + css })
    },
    //获取元素的相对 document 高度
    offset: function(coordinates){
      if (coordinates) return this.each(function(index){
        var $this = $(this),
            coords = funcArg(this, coordinates, index, $this.offset()),
            parentOffset = $this.offsetParent().offset(),
            props = {
              top:  coords.top  - parentOffset.top,
              left: coords.left - parentOffset.left
            }

        if ($this.css('position') == 'static') props['position'] = 'relative'
        $this.css(props)
      })
      if (!this.length) return null
      if (document.documentElement !== this[0] && !$.contains(document.documentElement, this[0]))
        return {top: 0, left: 0}

      //getBoundingClientRect 可以获取元素的位置, 元素距离网页上沿的距离与距离网页左边的距离
      //计算 left, top 的值的时候为了兼容性问题,使用了 window.pageXoffset 或者 window.pageYOffset ,而不是 window.scrollTop, window.scrollLeft
      var obj = this[0].getBoundingClientRect()
      return {
        left: obj.left + window.pageXOffset,
        top: obj.top + window.pageYOffset,
        width: Math.round(obj.width),
        height: Math.round(obj.height)
      }
    },
    //垂直滚动条高度
    scrollTop: function(value){
      if (!this.length) return
      var hasScrollTop = 'scrollTop' in this[0]
      if (value === undefined) return hasScrollTop ? this[0].scrollTop : this[0].pageYOffset
      return this.each(hasScrollTop ?
        function(){ this.scrollTop = value } :
        function(){ this.scrollTo(this.scrollX, value) })
    },
    //水平滚动条长度
    scrollLeft: function(value){
      if (!this.length) return
      var hasScrollLeft = 'scrollLeft' in this[0]
      if (value === undefined) return hasScrollLeft ? this[0].scrollLeft : this[0].pageXOffset
      return this.each(hasScrollLeft ?
        function(){ this.scrollLeft = value } :
        function(){ this.scrollTo(value, this.scrollY) })
    },
    //获取元素位置
    position: function() {
      if (!this.length) return

      var elem = this[0],
        // Get *real* offsetParent
        offsetParent = this.offsetParent(),
        // Get correct offsets
        offset       = this.offset(),
        parentOffset = rootNodeRE.test(offsetParent[0].nodeName) ? { top: 0, left: 0 } : offsetParent.offset()

      // Subtract element margins
      // note: when an element has margin: auto the offsetLeft and marginLeft
      // are the same in Safari causing offset.left to incorrectly be 0
      offset.top  -= parseFloat( $(elem).css('margin-top') ) || 0
      offset.left -= parseFloat( $(elem).css('margin-left') ) || 0

      // Add offsetParent borders
      parentOffset.top  += parseFloat( $(offsetParent[0]).css('border-top-width') ) || 0
      parentOffset.left += parseFloat( $(offsetParent[0]).css('border-left-width') ) || 0

      // Subtract the two offsets
      return {
        top:  offset.top  - parentOffset.top,
        left: offset.left - parentOffset.left
      }
    },
    //获取元素相对已经定位过的父元素位置
    offsetParent: function() {
      return this.map(function(){
        var parent = this.offsetParent || document.body
        while (parent && !rootNodeRE.test(parent.nodeName) && $(parent).css("position") == "static")
          parent = parent.offsetParent
        return parent
      })
    }
  }

  // for now
  $.fn.detach = $.fn.remove

  // Generate the `width` and `height` functions
  //获取元素的 width, height
  ;['width', 'height'].forEach(function(dimension){
    var dimensionProperty =
      dimension.replace(/./, function(m){ return m[0].toUpperCase() })

    $.fn[dimension] = function(value){
      var offset, el = this[0]
      if (value === undefined) return isWindow(el) ? el['inner' + dimensionProperty] :
        isDocument(el) ? el.documentElement['scroll' + dimensionProperty] :
        (offset = this.offset()) && offset[dimension]
      else return this.each(function(idx){
        el = $(this)
        el.css(dimension, funcArg(this, value, idx, el[dimension]()))
      })
    }
  })

  //使用递归加回调访问节点
  function traverseNode(node, fun) {
    fun(node)
    for (var i = 0, len = node.childNodes.length; i < len; i++)
      traverseNode(node.childNodes[i], fun)
  }

  // Generate the `after`, `prepend`, `before`, `append`,
  // `insertAfter`, `insertBefore`, `appendTo`, and `prependTo` methods.
  adjacencyOperators.forEach(function(operator, operatorIndex) {
    var inside = operatorIndex % 2 //=> prepend, append

    //before 函数
    $.fn[operator] = function(){
      // arguments can be nodes, arrays of nodes, Zepto objects and HTML strings
      //arguments 参数可以是 DOM 节点,节点数组, Zepto 对象, html 字符串
      var argType, nodes = $.map(arguments, function(arg) {
            
            var arr = []
            argType = type(arg)
            //如果是数组
            if (argType == "array") {
              arg.forEach(function(el) {
                if (el.nodeType !== undefined) return arr.push(el)
                else if ($.zepto.isZ(el)) return arr = arr.concat(el.get())
                arr = arr.concat(zepto.fragment(el))
              })
              return arr
            }
            return argType == "object" || arg == null ?
              arg : zepto.fragment(arg)
          }),
          parent, copyByClone = this.length > 1
      if (nodes.length < 1) return this

      return this.each(function(_, target){
        parent = inside ? target : target.parentNode

        // convert all methods to a "before" operation
        target = operatorIndex == 0 ? target.nextSibling :
                 operatorIndex == 1 ? target.firstChild :
                 operatorIndex == 2 ? target :
                 null

        var parentInDocument = $.contains(document.documentElement, parent)

        nodes.forEach(function(node){
          if (copyByClone) node = node.cloneNode(true)
          else if (!parent) return $(node).remove()

          parent.insertBefore(node, target)
          if (parentInDocument) traverseNode(node, function(el){
            if (el.nodeName != null && el.nodeName.toUpperCase() === 'SCRIPT' &&
               (!el.type || el.type === 'text/javascript') && !el.src){
              var target = el.ownerDocument ? el.ownerDocument.defaultView : window
              target['eval'].call(target, el.innerHTML)
            }
          })
        })
      })
    }

    // after    => insertAfter
    // prepend  => prependTo
    // before   => insertBefore
    // append   => appendTo
    $.fn[inside ? operator+'To' : 'insert'+(operatorIndex ? 'Before' : 'After')] = function(html){
      $(html)[operator](this)
      return this
    }
  })

  zepto.Z.prototype = Z.prototype = $.fn

  // Export internal API functions in the `$.zepto` namespace
  zepto.uniq = uniq
  zepto.deserializeValue = deserializeValue
  $.zepto = zepto

  return $
})()

// If `$` is not yet defined, point it to `Zepto`
window.Zepto = Zepto
// console.log(typeof Zepto);
window.$ === undefined && (window.$ = Zepto)
// console.log($);

```




