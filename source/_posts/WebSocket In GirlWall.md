title: WebSocket In GirlWall
date: 2017-02-02 22:45:24
categories: Web Develop
---

WebScoket 是 HTML5 新增的全双工通信协议, 解决 HTTP 的 "拉" 协议问题,增加了服务端的 "推" , 实现实时数据传输
websocket 是浏览器支持全双工通信的协议, 协议 url 格式为 ws: url, 加密后为 wss: url.
<!--more-->
##为什么使用 WebSocket
WebSocket 在实现双向传输数据时, 会比 Comet 有优势吗? WebSocket 比 HTTP 长轮循或长连接到底好在哪里?
 WebSocket 不是需要很多长连接吗? 
在 Javascript 中有提及 Comet 这类技术用于实时传输数据,但是我认为 WebSocket 会性能比较好.
实现实时信息的方式有:

 1. 轮询 = XHR + 定时器, 浪费带宽,增加服务器负荷,很多重复的"废"数据, 但实现简单,适用于小型应用
 ```
 function createXHR(method, url){
       if(typeof XMLHttpRequest != 'undefinded') {
            return new XMLHttpRequest();
       } else if(typeof ActiveXObject != 'undefinded') {
            
            var versions = ['MSXML2.XMLHttp', 'MSXML2.XMLHttp.2.0', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp.4.0', 'MSXML2.XMLHttp.5.0', 'MSXML2.XMLHttp.6.0'];
            
            for(var i = 0; i < versions.length; i++ ) {
                
                try {
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                } catch(ex) {
                    //跳过
                }
            }
            
            return new ActiveXObject(argument.callee.activeXString);
       } else {
            alert('Your bower is not support XHR');
       }
       
       return null;
 }
 
 function sendData(method, url, data){
        var xhr = createXHR();
        
        xhr.onreadyStatechange = function(){
            if(xhr.readyState == 4) {
                if(xhr.status == 200) {
                    var data = xhr.responseText;
                }
            }
        }
        
        xhr.open(method, url, true);
        xhr.send(data);
 }
 
 setTimeout(function(){
        sendData('GET', 'http://locahost:8081', null);
 }, 5000);
 ```
 2. Comet 中的长连接 = 保持住 HTTP 的连接, 利用 XHR 的 timeout 与收到数据/连接结束后马上建立新连接. 而服务
端需要不断去确认数据是否发生变化.
```
function createXHR(method, url){
       if(typeof XMLHttpRequest != 'undefinded') {
            return new XMLHttpRequest();
       } else if(typeof ActiveXObject != 'undefinded') {
            
            var versions = ['MSXML2.XMLHttp', 'MSXML2.XMLHttp.2.0', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp.4.0', 'MSXML2.XMLHttp.5.0', 'MSXML2.XMLHttp.6.0'];
            
            for(var i = 0; i < versions.length; i++ ) {
                
                try {
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                } catch(ex) {
                    //跳过
                }
            }
            
            return new ActiveXObject(argument.callee.activeXString);
       } else {
            alert('Your bower is not support XHR');
       }
       
       return null;
 }
 
 function longPolling(method, url, data){
        var xhr = createXHR();
        
        xhr.onreadyStatechange = function(){
            if(xhr.readyState == 4) {
                if(xhr.status == 200) {
                    var data = xhr.responseText;
                } else {
                
                }
                longPolling('GET', 'http://localhost:18080', null);
            }
        }
        
        xhr.open(method, url, true);
        xhr.send(data);
 }
 longPolling('GET', 'http://localhost:18080', null);
```
3.HTTP 流:通过流模式,在数据还没有完全接收到的时候,就已经处理数据,主要是通过 XHR 的 onreadyStatechange 的
xhr.readyState == 3 就处理数据.通过监测字符串长度得知数据是否已经处理过.而服务端需要
```
//设置头部, 数据采用流模式
res.setHeader('content-type', 'multipart/octet-stream');
```
以上几个方法的性能并不好,大部分原因在于有时需要的数据是轻量的,但是 HTTP 头部信息却比数据要大,造成了损耗.
而 WebSocket 的性能消耗却比他们小,但是缺点在于兼容性差.(请看 Websocket 性能消耗章节.)
##WebSocket 使用

```
var ws = new WebSocket(url)；//url 为 ws://www.example.com/test
ws.send(string); //JSON.stringify(obj);
ws.close();

//只有 close 事件的 event 对象才有相对应的属性, wasClean(是否明确关闭连接), code(状态码), reason(服务端发送过来的信息)

//ws 不支持 DOM 2 级的监听器, 只能用 DOM 0 级的监听器
ws.onmessage = function(event){
 //接收数据
 var data = event.data;
 
}
ws.onopen = function(){
    console.log('connection opened');
}
ws.onerror = function(){
    console.log('connection error');
}
ws.onclose = function(event){
    console.log('connnection closed by' + event.reason);
}
```
使用 Socket.io 有一些小技巧:
在界面显示信息需要两个 emit, 这样可以知道所有显示的信息都是从服务器过来的,这样的好处可以确保发送成功,
并且保持接口的一致性.
监听 socket.on('to' + userId, function(){})  这样无论收到还是发送信息都可以进入这个方法.
用户离开的时候应该 delete 对象:
```
var users = {}, usocket = {};
//新增连接对象
users[userId] = userId;
usocket[userId] = socket;
```
##WebSocket 原理

```
//与常规的 HTTP 头部相比
Upgrade: websocket
Connection: Upgrade
```

websocket 利用 HTTP 进行握手,建立连接,然后就可以进行通信了. 使用 [socket.io](http://blog.fens.me/nodejs-socketio-chat/) 来跨浏览器兼容. WebSocket 通过一个长连接来达到客户端与服务端双向推送数据.



##项目实践
许愿墙有很多功能都需要服务器主动发送信息到客户端,但是问题在于服务器能不能承受,与有没有必要建立 websocket.
对于服务器能不能承受的问题不用担心, WebSocket 的性能很好, 只需 1G 内存, 1 核 CPU 处理能力就能同时连接
50,000 个用户.
###在 BAE 上的难点
在做项目的时候是部署在 BAE 上的,由于使用了多个执行单元,这样的话,用户的 socket 对象就会分别存储在不同的
执行单元中,如何在多个机器中如何找到用户? 答案是使用 redis 发布信息机制,这里不再详细阐释,请看 BAE 博客.
传送门:http://godbae.duapp.com/?p=1096.
[WebSocket 聊天室](https://cnodejs.org/topic/557a999216839d2d539361a3)到底是怎么知道自己和谁在说话,是私聊而不是广播(如何实现点对点聊天)?
我们知道,目前真正的点对点通信是不存在的, 还没有技术可以做到不通过服务器直接通信的,这样我们就知道 WebSocket
是如何实现私聊的了.老样子,我们还是通过寻找用户的 ID 或者用户名来确定用户的 socket 对象,将信息传给该用户的
socket 对象就行了.
###其他
对于实现上当然也可以使用 SSE--Server-sent Event, [SSE](http://www.cnblogs.com/goody9807/p/4257192.html) 通过创建 EventSource 对象,实现服务器 "推":
```
//客户端使用
var sse = new EventSource('http://www.example.com/test');

sse.onmessage = function(){

}


sse,onopen = function(){

}

sse.onerror = function(){

}
```
但 SSE 还是建立在 HTTP 协议上的,如果需要频繁地取小量的实时性数据, WebSocket 更胜一筹.

##反向代理影响 websocket 吗 ?
结果是影响的, WebScoket 是需要 HTTP 建立链接的.主要是通过
```
Upgrade: WebScoket
Connnection: Upgrade
```
这两个头部信息,但是问题在于这两个头部信息是逐跳头部信息,也就是说在第一条有效,后面的跳均不带这个头部信息,
而客户端是不会察觉代理的存在的,所以将 HTTP Upgrade 请求发给代理之后,代理发送给后端服务器并没有带这个头部
信息,即 WebSocket 没有真正建立起来.所以我们需要在 Nginx 反向代理配置中添加自动添加头部的设置.
```
server {
    proxy_pass http://wsbackend;
    proxy_http_version 1.1;
    proxy_set_headerUpgrade: $http_upgrade;
    proxy_set_header Connection "upgrade";
}

```
*Nginx* 官网给出的 *Nginx* 使用 *webSocket* 的例子: *[Nginx 上的 WebSocket 链接设置](https://www.nginx.com/blog/websocket-nginx/)*
##WebSocket 的性能消耗
原本担心 WebSocket 在建立很多客户端长连接之后会开销很大,但是出乎我意料的是,根据 Nginx 服务器做的性能监测,
在建立了 5000 个客户端连接, Nginx 服务器需要的开销包括不到 1G 的内存, 1 核的 CPU 处理能力. 

资料:

 - [在 Nginx 上的 NodeJS WebSocket 测试报告](https://www.nginx.com/blog/nginx-websockets-performance/)
 - [WebSocket 开销](http://crossbario.com/blog/Dissecting-Websocket-Overhead/)

链接:

 - [一篇关于 WebSocket 反向代理的博文](http://www.tuicool.com/articles/ABnyqqF)
 - [Web 即时通讯盘点](http://www.tuicool.com/articles/uINBfiZ)
 - [即时通讯网](http://www.52im.net/thread-296-1-1.html)
 - [即时通讯总结](http://www.52im.net/forum.php?mod=viewthread&tid=338)

