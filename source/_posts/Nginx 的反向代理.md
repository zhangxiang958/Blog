title: Nginx 的反向代理
date: 2017-01-20 22:17:24
categories: Web Develop
---

在做项目的时候,有时候需要自己部署 *Node.JS* 在 Linux 服务器上, 但是由于基于 *Express* 的 *Node.JS* 监听某个
端口,但是一般来说不会监听 80 端口, 因为 80 端口是 HTTP 的默认端口, 在设置站点的时候需要将发送到 80 端口的数
据转送到 *NodeJS* 监听的端口去,这时候就需要设置反向代理了.
<!--more-->
##什么是反向代理
从结构上说,与正向代理相反的就是反向代理,从功能与原理上,正向代理与反向代理却是不一样的东西.正向代理对服务端屏
弊客户端,反向代理对客户端屏蔽服务端.其实正向代理与我们使用的 VPN 原理类似,在国外架设一个服务器可以访问国外的
网站,我们只需访问这台服务器,通知这台服务器去访问 Google, Facebook 等网站,然后将结果发送给我们.
而反向代理在负载均衡中起到关键的作用, 负责将接收到的请求,转接到相对应的服务器上,其中起到一部分安全,高性能的
作用.
推荐看看知乎上关于反向代理的比喻. 传送门: [反向代理](https://www.zhihu.com/question/24723688).
##如何开启反向代理
这里的反向代理的作用是将传到 Nginx 服务器的 HTTP 请求, 转送到相对应端口的我们建立起来的 NodeJS 服务器.

 1. 登录服务器,并以管理员身份运行
 2. 移动到 nginx 文件夹
    ![](http://img.ijarvis.cn/nginx.png)
 3. 在 site-available 文件夹找到所要设置反向代理的站点的配置文件
 4. 在 server 中填入如下字段
    ![](http://img.ijarvis.cn/server80.png)
    ![](http://img.ijarvis.cn/nginxF.png)
```
    proxy_pass http://localhost:8004;  //NodeJS 服务器监听 8004 端口
    proxy_http_version 1.1;  //HTTP 版本为 1.1
    proxy_set_header Upgrade $http_upgrade; 
    proxy_set_header Host $host; 
    proxy_cache_bypass $http_upgrade;  
```
 5. 重启 Nginx 服务器,访问 easyread.ijarvis.cn 就可以访问到我们的 NodeJS 了
 

##附录 
###配置中的 *$http_upgrade* 有什么作用?
使用 HTTP 协议升级到 Upgrade ,支持 Websocket 协议.
http://nginx.org/en/docs/http/websocket.html






