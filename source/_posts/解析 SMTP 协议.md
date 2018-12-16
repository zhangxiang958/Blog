title: 解析 SMTP 协议
date: 2018-04-08 09:48:24
categories: Web Develop

---

SMTP 是属于应用层协议, 是基于 TCP 协议用于收发邮件的.我们常常需要在业务中使用邮件, 但是并没有对 smtp 协议有足够的了解, 我们下面就来全面地了解一下. smtp 服务器一般会开启 25 端口提供服务, 当然如果 smtp 服务器使用了安全认证也就是 ssl/tls, 那么就会开放 465 或 587 端口开放服务, 例如 smtp.qq.com.
本文相关代码仓库:https://github.com/zhangxiang958/ComputerNetworkLab/blob/master/smtp/mailer.js
<!--more-->
## SMTP 的工作流程
相信很多人在使用邮件服务的时候, 特别是在 nodejs 编写中, 都会直接使用像 nodemailer 这样的库来直接编写逻辑, 但这里并不会使用这样的库, 本着学习协议的细节的意思, 我们从零开始编写一个 smtp 客户端,要求客户端可以进行身份验证与发送多格式的邮件.
我们在最开始的时候, 需要先使用 socket 来进行 tcp 连接操作, 并且需要注意, 每个信息的末尾都要使用 \r\n 来表示该行信息结束.
```
const Net = require('net');
const socket = new Net.Socket();
const iconv = require('iconv-lite');

let data = [];
let size = 0;
// 监听 data 事件接收服务器信息
socket.on('data', (chunk) => {
    data.push(chunk);
    size += chunk.length
});
// 拼接 buffer
socket.on('end', () => {
    let buf = Buffer.concat(data, size);
    let message = iconv.decode(buf, 'utf8');
    console.log(message);
});

socket.connect({ host, port });
```
### HELO 命令
经过上面的连接操作之后, 我们会收到:
```
220 welcome
```
220 为连接成功状态码, 后面为欢迎信息, 接下来我们就需要向服务器发起一个 HELO 命令, 主要是用于标识客户端自己
身份的.
```
socket.write('HELO debugmail.io\r\n'); // debugmail.io 为 smtp 服务器域名
```
发送了 HELO 命令之后. 会收到 250 状态码, 250 状态码会在后面多次用到, 所以我们使用全局变量来存放一些命令运行后的状态, 比如已经认证了或已经发送了某些命令.
### AUTH LOGIN 命令
AUTH LOGIN 命令是用于登录 smtp 服务器的, 在上面接收到 250 状态码之后, 如果没有进行过认证的话, 那么我们就需要进行认证操作:
```
socket.write('AUTH LOGIN\r\n');
```
发送了 AUTH LOGIN 命令之后, 服务器会发送一个 334 的状态码, 334 的状态码后面跟着的信息是 base64 编码后的
Username: 和 Password:, 第一个是 Username:, 类似这样:
```
334 UGFzc3dvcmQ6
```
这个时候我们接收到 334 状态码与后面的信息之后, 需要先 base64 解码判断是需要输入帐号还是密码, 注意, 如果是需要发送帐号, 那么我们就发送帐号, 格式是 base64 格式化后的字符串:
```
socket.write(`${new Buffer(username).toString('base64')}\r\n`);
```
认证成功后, 即帐号密码都正确后, 那么服务器就会返回 235 状态码表示认证信息已经成功.
### MAIL FROM 命令
上面的认证信息登录成功后, 我们需要发送 MAIL FROM 命令, 很简单, 就是字面意思理解, 告诉 smtp 服务器此邮件是从那么邮件地址发送, 主要目的是验证该邮箱地址被不被邮件服务器支持.后面只能接一个地址
```
socket.write('MAIL FROM: <shawncheung702@gmail.com>');
```
注意, 这里的尖括号最好加上, 因为有些 smtp 服务器并不能解析不加尖括号的邮箱地址, 如果不加尖括号有可能会报错
### RCPT TO 命令
上面 MAIL FROM 成功后, 我们需要发送 RCPT TO 命令, 很简单, 就是字面意思理解, 告诉 smtp 服务器此邮件的接收地址是哪些, 能不能被邮件服务器支持, 注意, 邮件可能会被群发, 所以 RCPT TO 命令后面可能会有多个邮箱地址
```
socket.write('RCPT TO: <shawncheung702@gmail.com>');
```
同理, 这里的每个邮箱地址最好加上尖括号.
### DATA 命令
在我们发送真实邮件内容实体之前, 我们需要先发送一个 DATA 命令, 告诉服务器我们接下来需要发送邮件实体数据:
```
socket.write('DATA\r\n');
```
发送了 DATA 命令之后, 服务器接收到之后, 会返回一个状态码与信息:
```
354 Enter mail, end with "." on a line by itself
```
然后我们接收到 354 状态码之后, 就可以进行实体内容的组装发送了, 注意这里邮件所有实体内容发送完毕之后需要发送一个 . 号来标记数据已全部发送完毕.
而这里的数据, 有几个注意的地方:
1. 需要指定邮件的 Content-Type, 如果是有附件的, 那么值为 multipart/mixed, 如果没有那么就是 multipart/alternative;
2. 需要指定一个 boundary 也就是分隔符, 用于分割邮件的各个部分内容, 在每个部分的前后加上.
3. from, to 信息是需要加上的, 也即是我们通过邮件客户端看到的邮件头部信息, subject 头部是邮件的标题信息
```
const keys = Object.keys(this.content);
      const mimeVersion = '1.0';
      const boundary = `========${(Math.random() * 10000000000000).toFixed()}==`;
      socket.write(`Content-Type:multipart/mixed;\n boundary="${boundary}"${msgEnd}`);
      socket.write(`MIME-Version: ${mimeVersion}${msgEnd}`);
      for (let key of keys) {
        switch (key) {
          case 'from': // 发出地址
            socket.write(`From:${this.content[key]}${msgEnd}`);
            break;
          case 'to': // 发往地址
            socket.write(`To:${this.content[key]}${msgEnd}`);
            break;
          case 'subject': // 标题信息
            socket.write(`Subject:${this.content[key]}${msgEnd}`);
            break;
          case 'text': 处理文本信息
            socket.write(`\n--${boundary}\n`);
            socket.write(`Content-Type:text/plain; charset="utf-8"\n`);
            socket.write(`MIME-Version:${mimeVersion}\n`);
            socket.write(`\n${text}${msgEnd}`);
            break;
          case 'html': // 处理 html
            socket.write(`\n--${boundary}\n`);
            socket.write(`Content-Type:text/html; charset="utf-8"\n`);
            socket.write(`MIME-Version:${mimeVersion}\n`);
            socket.write(`\n${html}${msgEnd}`);
            break;
          case 'attachment': // 处理附件
            const filePaths = attachment;
            const fileNum = attachment.length;
            for (let path of filePaths) {
              await Mail.handleFile({ path, boundary, mimeVersion }, socket);
            }
            break;
          default:
            break;
        }
      }
      // 邮件结尾
      socket.write(`\n--${boundary}--\n${msgEnd}`);
      // 根据 354 的信息, 需要 . 进行结尾
      socket.write(`${msgEnd}.${msgEnd}`);
```
### QUIT 命令
在完成上面的步骤之后, 其实一封邮件的发送基本属于完成, 我们现在需要做的就是告诉 smtp 服务器我们的操作已经完成了, 正式结束所有操作, 所以我们需要发送一个结束命令, 也就是 QUIT:
```
socket.write('QUIIT\r\n');
```
发送了这个命令之后, 服务器收到该命令之后就会知道操作已经完成了, 那么就会返回一个状态码与信息:
```
221 Bye
```
状态码 221 就是结束状态码, Bye 是服务器的结束信息.
### 小结
我下面贴出一个真实的 smtp 客户端与服务器的沟通过程:
```
220 smtp.qq.com Esmtp QQ Mail Server

250 smtp.qq.com // 欢迎信息

334 VXNlcm5hbWU6  // 认证帐号

334 UGFzc3dvcmQ6  // 认证密码

235 Authentication successful  // 登录成功

250 Ok // mail from 命令成功

250 Ok // rcpt to 命令成功

354 End data with <CR><LF>.<CR><LF> // data 命令

250 Ok: queued as // 邮件发送等待中

221 Bye // smtp 协议沟通结束
```
如需查看更多详细代码, 可以看本人简易的 smtp 客户端代码: https://github.com/zhangxiang958/ComputerNetworkLab/blob/master/smtp/mailer.js
## SMTP 的 MIME 格式
我们知道, 一封邮件不仅仅是只可以发送文本信息, 它还可以发送 html 信息与附件.先说对于 html 文档, 我们需要在
局部的地方写上不同的 Content-Type:
```
--==========boundary==
Content-Type: text/html; charset="utf-8"
MIME-Version: 1.0

<h1>Hello World!</h1>
--==========boundary==
```
而对于附件, 我们需要在局部写上 Content-Disposition 头部, 因为我们需要的是附件, 需要可以提供下载, 所以值为
attachment, 并且指定文件名:
```
--==========boundary==
Content-Type: application/octet-stream
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="test.md"
MIME-Version: 1.0

VXNlcm5hbWU6
--==========boundary==
```
注意这里我并没有根据文件名来决定 Content-Type, 而是统一使用 application/octet-stream, 使用流的方式传文件.
文件的内容使用 base64 编码, 所以头部需要添加 Content-Transfer-Encoding 来说明编码格式.
### 邮件中的附件
邮件中的图片有两种形式, 一种可以使用 img 标签放在邮件体中的 html 内容中, 另一个就是使用附件的形式发送.对于在 html 文档中的, 就是像平时的 img 标签中使用, 而对于附件来说, 我们就可以直接通过 stream 读文件, 然后进行 base64 编码, 将图片放到协议的头部信息里面, 实现附件功能.
## 总结
通过自行编写一个 smtp 服务器, 可以收获对 node socket 编程的熟悉度以及对 smtp 协议的进一步了解.明白 smtp 服务到底是怎么跑起来的.
对于调试 smtp 协议信息, 我推荐一个工具: debugmail.io, 它不会真正地往邮箱中发送邮件, 但是它能够详细清晰地看到 smtp 协议信息.
## 参考文献
1. [用NodeJs简易实现Smtp客户端](http://superobin.iteye.com/blog/775330)
2. [SMTP协议详解以及工作过程](https://blog.csdn.net/u011981018/article/details/50931563)
3. [SMTP MIME 格式](http://blog.chinaunix.net/uid-24612247-id-4089321.html)



