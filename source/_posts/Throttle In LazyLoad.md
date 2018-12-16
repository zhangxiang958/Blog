title: Throttle In LazyLoad
date: 2017-01-09 21:59:24
categories: Web Develop 
---

在做许愿墙项目的时候,发现懒加载绑定在 scroll 事件上, lazyload 函数会执行非常多次,极大地影响了页面性能,
因此采用节流函数来限制 loayload 函数的执行次数.
<!--more-->
##throttle 节流函数
有两种方式, 一种是利用定时器延迟执行函数,另一种是利用定时器延迟执行并同时在一定时间间隔执行函数.
```
function throttle(context, method, args, delay){
    
    clearTimeout(method.tid);  //清除定时器
    //使用函数变量来储存定时器 id 
    method.tid = setTimeout(function(){
        
        method.apply(context, args);  //因为采用 apply, 所以 args 应为数组形式
    }, delay);
}
```

```
function throttle(context, method, args, delay, duration) {
    
    var start = new Date();  //函数开始执行时间
    var timer = null;  //储存定时器 id
    return function(){
    
        var now = new Date();  //目前时间
        clearTimeout(timer);  //清除定时器
        
        if(now - start >= duration) {  //如果函数执行时间间隔大于规定时间
            
            method.apply(context, args);    //立刻执行
            start = now;
        } else {
            timer = setTimeout(function(){
                
                method.apply(context, args);  //因为采用 apply, 所以 args 应为数组形式
            }, delay);
        }
        
    }
}
```
这里只需采用第一种节流函数即可.
##说明原理
###第一种方法
因为函数是对象,因此将定时器 id 存储在函数对象的属性中,在执行 throttle 函数之前,先清除之前设定的定时器.
个人比较喜欢这种方法,因为简单高效,但是有个明显的缺点就是只有在设置的 500 ms 之内没有触发这个函数才能执行,
比如绑定在 input 框的 input 事件上只会在结束输入才会执行函数.  
###第二种方法
第二种方法采用了闭包的方法(返回一个函数), 使用一个变量存储定时器 id ,只有闭包才能有权访问这个 timer 变量,
从而修改或清除定时器.同样的, 在执行函数之前先存储一个时间戳,在一定的时间间隔之后,立刻执行,否则不在时间
间隔内设定定时器延迟执行,优化性能.注意在达到时间间隔直接执行函数之后,要修改 start 时间戳为 now.
另外,
```
ele.onevent = throttle(context, method, args, delay, duration); //直接调用 throttle 函数
```
应该像这样去调用函数,因为 throttle 返回一个函数, throttle 会立即执行,然后每次触发事件只会执行 throttle 
返回的函数,而不是 throttle 函数.