title: JQuery Promise 不执行成功回调
date: 2017-05-27 23:58:24
categories: Web Develop

---
最近和后台小哥进行联调, 因为是一个后台页面, 直接使用了 JQuery 里面的 $.ajax 来调接口, 但是发现异步成功回调
并不执行.
<!--more-->
##问题原因
原因在于对于 dataType 的误解. dataType 的意思的接收的数据按照指定的格式进行解析而非传输的数据格式.
```
$.ajax({
    dataType: 'jsonp',
    url: '/getData'
})
.then(function(){

    console.log('success');
}, function(){
    
    console.log('fail');
});
```
因为这是之前的项目, 所以并不清楚到底接口是一个普通的 post/get 请求还是一个 jsonp 格式请求, 所以在 datatype 
中使用了 jsonp 格式的数据解析, 这样会导致错误, 导致成功回调不会执行, 执行了失败回调, 这是因为接收的数据格式
如果是 json, 但是 dataType 指定使用 jsonp 格式进行解析, 这样会解析出错然后就会执行失败回调.
##解决
去掉 dataType 属性或者与后台人员充分沟通之后使用相对应的 json/jsonp/xml 格式的值.




