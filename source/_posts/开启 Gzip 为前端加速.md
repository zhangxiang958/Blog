title: 开启 Gzip 为前端加速
date: 2017-01-08 01:12:24
categories: Web Develop

---
目前来说,前端的单页面应用十分广泛,但是单页面应用有个先天的缺点是将所有的文件或页面打包在一个文件内,虽然提高
了应用的响应速度,但是却大大增加了首页的渲染时间与白屏时间.虽然目前的打包工具 webpack 支持懒加载代码分块,一
定程度上减轻了这一症状,但是当业务复杂起来,只包括首页 js 文件也会相当庞大,这时候如果能够采用 Gzip 压缩文件将
会显著减少白屏时间.
<!--more-->
由于目前 nginx 服务器使用非常广泛,加上本人平时都是使用 nginx 服务器,因此本文使用 nginx 的配置作为示例.

##开启 Gzip
###全局
如果想要在全局下开启 gzip,那么只需在 
```
/etc/nginx/nginx.conf
```
![](http://img.ijarvis.cn/gzipSet.png)
这个文件中 **http {...}** 找到 gzip 配置段,去掉前面的 **#** 符号.
###二级域名
如果想要在二级域名开启 gzip, 那么需要在该二级域名的 nginx 配置文件中,
```
/etc/nginx/sites-available/second.host
```
文件中的 **server {...}** 中写上 gzip 配置项即可.  
![](http://img.ijarvis.cn/gzipsetme.png)
##配置说明
```
##
# Gzip Setings
##

gzip on;  // 开启 gzip
gzip_disable "msie6"; //禁用 ie6 的 gzip 选项,由于 ie6 的性能差,启用 gzip 容易导致页面假死

gzip_vary on;    //http 头部,意在对于不支持 gzip 压缩的浏览器不进行压缩.
gzip_comp_level 6;  //压缩比, 0 - 9, 越高压缩时间越长, 高压缩比节省带宽消耗 CPU, 视自身的服务器性能与带宽决定. 
gzip_buffers 16 8k; //设置系统使用多少个单位去缓存 gzip 压缩数据流, 与压缩速度有关, 这里意思是原始数据以 8k 为单位的 16 倍申请内存.
gzip_min_length 1k; //最小需要 gzip 压缩的文件大小, 默认为 0, 建议设置,否则小文件反而压缩后变大.
gzip_http_version 1.1;  //只有 http 协议版本为 1.1 , gzip 才会生效
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+css text/javascript; // gzip 作用的文件类型
```

***注意**:需要开启 gzip_min_length 1k;  
当文件大小大于 1k 的时候, gzip 生效

**备注**: vary 响应首部是用于选择文档的, vary: Accpet-Encoding; 意思在于文档的类型取决于 Accpet-Encoding
字段.
##测试
测试可以使用命令行的形式
```
 curl -I -H "Accept-Encoding: gzip,deflate" "http://me.ijarvis.cn/javascripts/index.js"
```

![](http://img.ijarvis.cn/gzip.png)
可以看到有 Content-Encoding: gzip; 证明已经开启了 gzip

也可以使用在线测试工具 https://gtmetrix.com/, 输入域名即可测试.
##效果
index.js 原大小: 1.1 kb
index.html 原大小: 1.3 kb
style.css 原大小: 6.7 kb
![](http://img.ijarvis.cn/gzipRe.png)

###响应时间
没有设置 Gzip的时间:
![](http://img.ijarvis.cn/notgzipTime.png)

设置了 Gzip 的时间:
![](http://img.ijarvis.cn/gzipTime.png)

从响应时间与压缩文件大小两种角度来看, gzip 确实是优化利器.
虽然两者在时间上并没有相差太多,但是相信这是测试文件样本过小的原因,如果对于大文件,优化的结果应该是相当好的.

##开销成本
服务器消耗 CPU 来对文件进行压缩,浏览器等客户端需要解压,节约了带宽.通常是 1k 以上的文件才需要压缩.





参考: https://segmentfault.com/a/1190000006105756




