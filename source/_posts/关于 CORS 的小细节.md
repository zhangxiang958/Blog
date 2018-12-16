title: 关于 CORS 的小细节
date: 2017-12-09 10:24:24
categories: JavaScript

---

最近对 CORS 进行了一些小整理, 下面是对于一些小细节的总结.
<!--more-->
在处理 CORS 的时候, 如果需要带 cookie 给服务端, 那么就需要在 xhr 对象中添加属性:
```
xhr.withCredentials = true;
```
但是服务端的设置是什么呢? 服务端的设置也简单, 就是设置一个响应头部: 'Access-Control-Allow-Credentials': 'true'.
但是细节就是, 这样会跨域报错, 原因在于如果有 Access-Control-Allow-Credentials 头部, 那么 Access-Control-Control-Origin 头部就不能是 *, 必须是指定值, 这样才能设置成功.

说到 POST 请求, 一般来说服务器需要 POST 请求加上 applicaton/x-www-form-urlencoed 头部而不能是 text/plain(除了特殊需求), 不然服务端是拿不到 body 信息的.