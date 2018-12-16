title: Ajax
date: 2017-01-31 21:24:24
categories: JavaScript

---

对于一名 Web 开发工程师, 了解 Ajax 内部原理是非常必要的,不能因为目前工具的完备性而放弃理解了解其内部原理.
这样有利于更好地使用工具或发明工具.
<!--more-->
##createXHR 函数
这个函数是为了创建一个跨浏览器兼容的 XHR 对象.对于 IE 浏览器,所使用的是 ActiveXObject 函数来创建 XHR 对象.
而对于其他浏览器则直接使用 XMLHttpRequest API 来创建 XHR 对象.
```
function createXHR(){
    if(typeof XMLHttpRequest != 'undefinded') {
    
        return new XMLHttpRequest();
    } else if(typeof ActiveXObject != 'undefined') {
    
        if(typeof argument.callee.activeXString != 'string') { //函数形参
            
            var versions = ['MSXML2.XMLHttp', 'MSXML2.XMLHttp.2.0', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp.4.0', 'MSXML2.XMLHttp.5.0', 'MSXML2.XMLHttp.6.0'];
            
            for(var i = 0; i < versions.length; i++) {
                
                try {
                    new ActiveXObject(versions[i]);
                    argument.callee.activeXString = versions[i]; //用于告知与创建 ActiveXOPbject 对象版本
                    break;
                } catch(ex){
                    //跳过
                }
            }
            
        }
        
        return new ActiveXObject(argument.callee.activeXString);
    } else {
        
        alert('Your bowers is not support XHR!');
    }
}
```
###使用 AJAX 请求数据
```
var xhr = createXHR();
//在 open 之前设置 onreadystatechange 是为了浏览器兼容性问题, 这是因为 XHR 的浏览器规范实现的不同
xhr.onreadystatechange = function(){
    
    if(xhr.readyState == 4) {
        if(xhr.status == 200) {
            var data = xhr.responseText;// xhr.responseXML
        }
    }
}

xhr.open('get', 'www.example.com/test', true);  //只是预处理连接,并没有实际建立连接或发送请求
//xhr.setRequestHeader('Content-type', 'text/plain');  //设置请求头部
xhr.send(null);
```
附 readystatechange reason:
![](http://img.ijarvis.cn/onreadystatechangereason.png)
一个较完整的 sendData 函数:
```
function sendData(method, url, data, callback, async){
    
    var xhr = createXHR();
    
    xhr.onreadystatechange = function(){
        
        if(xhr.readyState == 4) {
            
            if(xhr.state == 200) {
                var data = xhr.responseText;
                callback(data);
            }
        }
    }
    if(method == 'POST') {
        xhr.setRequestHeader('Content-Type', 'x-www-form-urlencoded');
    }
    xhr.open(method, url, async);
    xhr.send(data);
}
```
处理 GET 请求:
```
xhr.get = function(url, data, callback, async){
    var queue = [];
    for(var key in data) {
        
         queue.push(encodeURLComponent(key) + '=' + encodeURLComponent(data[key]));
    }
    //注意 GET 请求中的数据放在 URI 中(查询字符串, '?' 后面的数据)   
     xhr.send('GET', url + (queue.length ? '?' + queue.join('&') : '') , null, callback, true);
}
```
处理 POST 请求:
```
xhr.post = function(url, data, callback, async){
    var queue = [];
    for(var key in data) {
        
        queue.push(encodeURLComponent(key) + '=' + encodeURLComponent(data[key]));
    }
    //注意与 GET 请求的区别
    xhr.send('GET', url, queue.join('&'), callback, true);
}
```

*注: encodeURIComponent 与 encodeURL 的区别在于 encodeURIComponent 是用来编码 url 组件的,像查询字符串, params 等, 而 encodeURI 是用于编码整个 URI 的, 它对于 ASCII 编码的字符是不会进行编码的, 像上面对数据进行
编码应该使用 encodeURIComponent ,这是为了避免数据中含有 ACSII 编码的字母字符或者数字而进行彻底进行转义.*

##跨域问题 CORS
跨域问题一直是 Web 开发的一个难题,涉及了 Web 安全,浏览器内核等方面的知识,如果只是一直使用像 Jquery 这样的
工具库而不去了解它的内部原理的话,那么遇到浏览器的报错
```
XMLHttpRequest cannot load http://api.ijarvis.com. Origin http://test.ijarvis.com is not allowed by Access-Control-Allow-Origin.
```
就会一头雾水,显然浏览器不会报出详细错误,但是如果我们了解其中原理则可以会很快得到解决方案.
###什么是跨域？
跨域问题是因为浏览器出于安全考虑采取同源策略,所谓同源策略就是 ***同协议,同域名,同端口***, 如果请求的资源的
URI 不符合这三个规定,请求的资源就无法得到,这就称之为跨域问题(即跨域资源共享问题).
在浏览器已经提供了解决这个问题的方案了, 就是在请求头部加上 *Origin* 字段说明发起请求的域名,而服务端的响应
头部加上 *Access-Control-Allow-Origin* 字段说明允许请求资源的域名,域名不在值中的无法正确请求资源.
```
Origin: url
Access-Control-Allow-Origin: url
```
如果需要正确使用 CORS 的话,需要知道下面的几点,因为 *HTTP* 请求头部中 *Origin*, *Javascript* 并没有权利
控制, 而是由浏览器这个用户代理来添加的,想想也知道, 如果 *Javascript* 有这个权利的话, 跨域问题也就不存在了,
浏览器安全也无从谈起.
###怎么解决跨域问题
其实对于跨域资源共享, XHR 对象已经有实现, 在 xhr 对象的 open 方法中的 url 参数填入绝对路径就可以实现跨域资源共享(本地使用相对路径可以加以区分).
而对于 IE 浏览器,可以使用 XDR 对象, 即 XDomainRequest 对象来实现跨域.
它们有共同点:不能访问响应头部,不能发送 cookie 与接收 cookie,而 XDR 只能用 get, post 方法.
```
function createCORSObject(method, url){
    var xhr = new XMLHttpRequest();
   if('withCredentials' in xhr){ // true or false,  值为 true 可以使用 cookies
        
        xhr.open(method, url, true);
   } else if(typeof XdomainRequest != 'undefinded'){
        
        xhr = new XDomainRequest();
        xhr.open(method, url); //XDomainRequest 对象的 open 方法只有两个参数,只能是异步请求
   } else {
        xhr = null;
   }
   
   return xhr;
}

var request = creeateCORSObject('GET', 'www.ijarvis.cn/test');
if(request) {
    request.onload = function(){
        var data = request.responseText;
    }
    //建议加上 error 事件的监听,不然出错没有任何报错
    request.onerror = function(){
        alert('request error');  //alert 出来就可以了, error 事件也没有事件对象属性可以打印出错误信息
    }
    request.send();
} else {
    alert('Your bower is not support CORS!');
}


```
####Preflight Request
![](http://img.ijarvis.cn/optionsDelete.png)
关于上面 DELETE 请求前面出现 OPTIONS 请求,在 Javascript 高级程序设计中被称作 Preflighted Request 的透明
验证机制, 如果想使用 GET, POST, HEAD 之外的方法或者其他三个数据格式, 则浏览器需要先发送一个 OPTIONS 请求,
向浏览器验证此域名是否有相对应的权限(称作 Preflight 请求), 然后再进行真正的请求.
```
Access-Control-Request-Method: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Request-Headers: Content-type
Access-Control-Max-Age: 19000

//Express 中设置头部
app.all("*", function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "XOrigin, X-Requested-With, Content-Type, Accept");
  res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
  res.header("X-Powered-By", ' 3.2.1')
  res.header("Content-Type", "application/json;charset=utf-8");
  next();
});
```
这个消耗是在 preflight 请求的缓存时间过期之前, 在第一次请求之前多一次 HTTP 请求. 在简单请求一般不会使用
这个,但是我们一般会使用 RESTful API, 这意味着我们会使用像 PUT, DELETE 这样的 HTTP 方法, application/json
这样的数据格式.

####*图像 Ping*
所谓 *图像 Ping* ,就是利用 img 标签能够跨域请求资源的特性来请求资源,但是一般都是请求图片或者文本这样的数据
```
var img = new Image();
img.src = 'expmale.ijarvis.cn/test';
img.onload = function(){
    alert('request done');
}

```
####*JSONP*
*JSONP* 就是 *JSON with Padding*, 即填充 *JSON*, *JSONP* 实际上是依赖于动态加载脚本,因为 *script* 标签可以
跨域请求资源,那就意味着我们可以利用这个发送请求.
```
//在 HTML 页面中直接使用
<script>
		function json_callback(data){
			console.log(data);
		}

		var script = document.createElement('script');
			
			script.setAttribute('type', 'text/javascript');
			script.src = 'http://localhost:18081/jsonp?callback=json_callback';

			document.body.appendChild(script);
</script>
//如果想在 AngularJS, VueJS 等框架使用 JSONP 请求,可以使用框架的 Ajax 函数的 jsonp API.
```
在 Node.JS 中使用 JSONP 实现跨域:
```
//在 Express 框架下
router.get('/jsonp', function(req, res, next){

	console.log(req.query.callback); // 打印出 'jsonp_callback'

	var data = {
		test: 123
	};
	
	console.log(JSON.stringify(data));

	res.jsonp(JSON.stringify(data));

});
```
![](http://img.ijarvis.cn/jsonpHeader.png)
可以看到请求的 URI 中的 responseHeader 并没有 Access-Control-Allow-Origin 这样的头部信息,这说明服务端并没
有开启跨域请求允许,但是请求成功了,说明 *JSONP* 跨域请求成功了.
有些同学可能不理解运行过程,我觉得看一下 response 的字符串就知道了.
```
// 请求返回的 js 文档内容
typeof jsonp_callback === 'function' && jsonp_callback({'test': 123}); // jsonp_callback 是上面发送请求的时候说明指定的函数名, jsonp_callback 需要是全局函数.
```
总结流程就是:

 1. 在 HTML 中动态生成一个 script 元素节点,利用 src 属性的特性, 插入到 document.body 中跨域请求资源.
 2. 发送请求的时候将回调的函数名指定并发送给服务端
 3. 服务端接收到 jsonp 回调函数名, 将生成的 json 格式的数据通过参数的形式传入生成的一个函数,名为客户端
 传过来的函数名,最后生成一个 javascript 语法的文档(内容如上).
 4. 客户端接收到文档, 解析数据, 执行文档的内容, 根据文档的语法逻辑最后执行回调函数将数据打印出来.

##XHR 2
XHR 2 级是对 XHR 对象的增强,增加了一些有用的 API,但是大部分的浏览器都只实现了部分的 XHR2 特性.
```
//Dataform API 用于创建表单格式的数据
var data = new Dataform();
data.append('test', '123'); //创建一条数据
```
```
//overrideMimeType 用于重写 mime, 是为了重写 mime 格式, 不让 XML 格式的数据被当作 text 格式来渲染
xhr.overrideMimeType('text/xml');
```
```
//设定 1 秒后就会超时并关闭 XHR 连接
xhr.timeout = 1000;
xhr.ontimeout = function(){
    
}
```
###进度条
我们在页面上为了提高用户体验通常会添加一个进度条,这样可以使用 XHR 2 级的进度事件-- *load* 与 *progress*.
常见的进度条都是使用 load 事件的, 我不知道为啥,我比较倾向于使用 *progress* 事件, 通过 *event* 对象的 
*position* 与 *totalSize* 可以知道已经接收的大小与总大小, 个人认为可能是因为 *progress* 事件过于细腻,比较
适合使用在单文件,而 *load* 比较适合多文件加载的进度条.


附录:
form-data 与 x-www-form-urlencoded 的区别在于 form-data 可以传文件也可以传值,最终转化为一条信息
x-www-form-urlencoded 会将表单里的值转化为键值对.

参考资料:

 - [HTTP CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Overview)
 - [CROS](http://newhtml.net/using-cors/)
 - [XHR 规范](https://xhr.spec.whatwg.org/#event-xhr-readystatechange)


*本文更新于 2017.03.26 11:28*