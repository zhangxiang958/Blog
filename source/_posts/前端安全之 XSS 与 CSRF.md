title: 前端安全之 XSS 与 CSRF
date: 2017-03-06 16:49:24
categories: Web Develop

---
前端安全至关重要, 如果在前端阻拦一些安全攻击, 对于用户体验也是极大提升.
<!--more-->
##XSS
XSS 攻击,即跨站脚本攻击, 理论上,任何可输入的地方没有对数据进行处理的话,就会有 XSS 漏洞.
###理解 XSS
想要理解 XSS, 首先要知道 XSS 可以通过几种方式来进行攻击:

 1. 反射型, 攻击者需要诱导用户点击一个链接进行攻击
 2. 存储型, 通过将攻击代码储存在远程服务器中,这样所有用户在访问具有此代码数据的页面就会自动运行形成攻击,例如评论框没有过滤输入, 导致用户可以输入 ```<script>``` 这样的内含代码的标签直接运行代码.

对于反射型的攻击,往往需要诱导用户点击一个链接, 像 <<我是谁:没有绝对安全的系统>> 电影中,主角发送了一个链接
给一个喜欢猫咪的职员, 从而运行了自己的脚本, 获取了职员电脑的权限与资料.例如像用户已经登录的网页的 cookie,
攻击者得到这个 cookie, 利用这个 cookie 发送请求,就可以不需要密码直接登录账户.

###XSS 防御
对于不需要输入 HTML 的可输入域, 我们可以将输入的数据进行转义, 而对于需要输入 HTML 的输入域来说, 我们需要
对输入的数据进行解析, 然后根据白名单上的标签或者属性进行 DOM 树的构建.
XSS 攻击的本质还是 HTML 代码注入, 将用户的数据当作是 HTML 代码来执行, 误解了原本的语义.
有些人说 GET 请求比 POST 请求更加安全, 我觉得是没有道理的, 第一 GET 与 POST 都只是 HTTTP 的方法而已, 本质
上没有安不安全的说法, 第二 无论是 GET 请求的安全攻击还是 POST 请求的安全攻击都是可以通过脚本制造出来的,
而不是 POST 请求不把参数写在 URL 中就安全.
想要正确防御 XSS, 就需要列举出 XSS 的各种场景再一一解决.
从防御角度上看, XSS 防御分为用户需要输入 HTML 代码与不需要输入 HTML 代码两种.而对于不需要输入 HTML 代码的
情况,防御分为 HTML, HTML 属性, script, 事件, CSS , style, style 属性.
对于 HTML 输出的变量, 我们需要使用 HTMLEncode 将尖括号, 单引号, 双引号等等一些特殊符号转义.
对于 HTML 属性, 保证值被双引号括住, 然后通过 JavascriptEncode 来对值进行编码, JavascriptEnode 即对于数字
字母等不编码,但是特殊符号进行编码.
对于 script 标签与事件都使用 Javascript.

当我们需要处理富文本的时候, 需要将输入的 HTML 进行解析, 例如 ```<iframe>, <script>, <form>``` 这样的标签
应该严格禁止, 并且在构建 HTML 结构的时候应该只从白名单中取.
而且为了避免 XSS 诱导点击获取用户的 cookie 从而冒充用户进行操作, 我们会对一些敏感操作的 cookie, 比如登录
的 cookie 做一个 HttpOnly 的属性设置, 这样 Javascript 就没有权限获取到这些设置到 HttpOnly 的 cookie ,也就
没有办法冒充登录了.
##CSRF
CSRF 是跨站请求伪造, 原理是在 A 网站登录之后生成身份 cookie 之后登录网站 B,网站 B 利用这个 cookie 强制 A  网
站发送一个恶意请求到服务端.
在服务端, 请求无论是 GET 方法还是 POST 方法, 都能够获取 query 查询字符串来获取客户端传送过来的数据, 但是如果
post 请求还是使用 query 字段来获取数据,那么一些像利用 img 标签 src 属性形成的攻击还是无法避免, 因此我们才要
求敏感操作使用 post 请求,数据放在 body 请求,虽然不是绝对安全,但是至少能够规避一个 GET 请求的攻击方式.
###理解 CSRF
理解 CSRF 的发生,其本质就是攻击者能够预测到请求的数据参数, 只有理解了这一点, 我们才有可能进行防御手段.
###CSRF 防御
####验证码
说明请求需要带上验证码, 因此如果带上验证码, 基本上可以说这个请求是用户允许知情的.但是由于用户体验,不能给
所有操作都加上验证码.
####refer
通过 http 头部的 refer 知道发送请求的页面目前在哪个域中, 如果不是允许的域, 那么就认为是 CSRF, 但是遗憾的
是服务器并不是经常能收到 refer 字段, 因为有时候浏览器因为隐私会拒绝发送 refer 字段.
####token
使用一个足够随机的 token ,将 token 随机化并且加密, 存储在客户端的 cookie 中(设置 HttpOnly) 或者存储在服务
端的 session 中, 个人推荐 session 存储, 因为如果存储在 cookie 中的话, 页面如果有多个, 并且 token 有一定的
生命周期(只用一次), 那这样使用还是非常麻烦, 如果使用 session 存储就方便多了.这样的话使用 token ,攻击者就没
办法猜测出 token 值,从而无法攻击.

##附录
###Cookie 与 Session
cookie 具有不可跨域名性, 也就是说 A 域名下的 cookie 不能被 B 域名修改.对于浏览器来说, cookie 有 session
与 third-party 之分, session 是 cookie 在设置的时候没有设置 expires 过期时间, 而 third-party 有设置过期
时间, session cookie 是存在浏览器内存中的, 而 third-party 是存放到本地的, 而在网页未关闭之前, 请求了其他
域的资源的时候, session cookie 会加到请求头部, 而 third-party cookie 却不会(有些浏览器会), 所以我觉得在
设置登录等敏感操作的 cookie 的时候,最好加上过期时间, 防止 cookie 泄漏.
http://www.jianshu.com/p/25802021be63
###jsonp 的安全问题
为了防止恶意请求, callback 后面加上了恶意标签, 所以服务端应该需要将 callback 后面的参数进行编码, 服务端返
回时是 ```window['function']({ code: 200 });```
```
请求: http://www.test.com?callback=fun
响应: typeof window['fun'] == 'function' && window['fun']({ code: 200 });

为了防止插入恶意标签, 我们应该定义 HTTP 头部, Content-type: application/json 让数据以 json 格式进行解析.
同时对 url 进行编码.
```
检查 refer 也是一个办法.

###项目反思
例如在许愿墙中, 搜索与写祝福的文本框都没有做字符串的编码, 这两个地方明显地不需要输入 HTML 内容, 如果在搜索
框中填入的是 ```<script>alert('xss');</script>```, 而后面因为数据库没有查找到相对应的内容的时候, 显示出来
```<script>alert('xss');</script> not found``` 这样, 就会实现了 XSS 攻击, 同理, 在祝福文本框也一样, 因此我
们在做平常非富文本编辑的输入的时候, 需要对字符串进行编码, 防止 xss 的注入.
###link

 - [HTML Encode](http://blog.csdn.net/ghsau/article/details/17027893)
 - [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting)

