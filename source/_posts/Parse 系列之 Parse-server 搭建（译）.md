title: Parse 系列之 Parse-server 搭建（译）
date: 2016-09-11 15:13:24
categories: Parse

---
应老师的要求，因为项目需要使用 parse-server 来作为应用的后端支撑，因此我将我学习 parse-server 的笔记
或译文放到博文中，希望可以帮助到师弟师妹的后续学习。
本次译文是我第一次翻译文档，其中翻译会生硬，欢迎大家提出意见。
<!--more-->
## 开始
搭建 parse-server 最快捷简单的方式是本地运行 MongoDB 和 Parse Server。
##本地搭建 Parse Server
```
$ npm install -g parse-server mongodb-runner
$ mongodb-runner start
$ parse-server --appId APPLICATION_ID --masterKey MASTER_KEY --databaseURI mongodb://localhost/test
```
你可以使用任意字符串命名你的应用 id 和 masterKey，这些将会被用于用户信息登录 Parse Server。
输入上面的命令后，现在，你可以在你的主机运行一个本地的 Parse Server 了。

**如果想要使用远程的 MongoDB 数据库？** 在运行 Parse-server 之前将 --databaseURI DATABASE_URL 字段传入命令
中。
如果想了解更多关于配置 Parse Server 的信息，请点击[这里](https://parseplatform.github.io//docs/)。
如果想要找到关于配置的全部选项，则请输入命令 *parse-server --help*。
###保存你的第一个对象
现在，你的主机已经搭建好 Parser Server，接下来我们就要存储你的第一个对象。这里我们将会使用 REST API，但是你可以通过 Parse SDKs 更简单地做到同样的事情。运行下面的命令：
```
curl -X POST \
-H "X-Parse-Application-Id: APPLICATION_ID" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
http://localhost:1337/parse/classes/GameScore
```
你会得到一个像下面的返回信息：
```
{
  "objectId": "2ntvSpRGIK",
  "createdAt": "2016-03-11T23:51:48.050Z"
}
```
你可以通过下面的命令直接读取这个对象（确定你在得到这个对象的时候将 API 中的 *2ntvSpRGIK* 换成
你上面得到的 *objectId*）：
```
$ curl -X GET \
  -H "X-Parse-Application-Id: APPLICATION_ID" \
  http://localhost:1337/parse/classes/GameScore/2ntvSpRGIK
```
```
// Response
{
  "objectId": "2ntvSpRGIK",
  "score": 1337,
  "playerName": "Sean Plott",
  "cheatMode": false,
  "updatedAt": "2016-03-11T23:51:48.050Z",
  "createdAt": "2016-03-11T23:51:48.050Z"
}
```
通过保存每一个独立对象的 *id* 并不是理想的途径，然而，很多情况下，你会得到一个储存对象的数组，像这样：
```
$ curl -X GET \
  -H "X-Parse-Application-Id: APPLICATION_ID" \
  http://localhost:1337/parse/classes/GameScore
```
```
// The response will provide all the matching objects within the `results` array:
{
  "results": [
    {
      "objectId": "2ntvSpRGIK",
      "score": 1337,
      "playerName": "Sean Plott",
      "cheatMode": false,
      "updatedAt": "2016-03-11T23:51:48.050Z",
      "createdAt": "2016-03-11T23:51:48.050Z"
    }
  ]
}
```
想要学习更多关于在Parse Server 保存和查询对象，请查看 [Parse 文档](https://parseplatform.github.io//docs/)
##远程运行 Parse Server
只要你能够很好地明白一个项目是怎么运行的，请查阅 [Parse Server wiki](https://github.com/ParsePlatform/parse-server/wiki) 深度指南通过部署 Parse Server 提供基础服务。
通过查阅来学习更多关于附加方式来运行 Parse Server。‘
###Parse Server 应用样本
我们已经提供了基础的 [Node.js 应用](https://github.com/ParsePlatform/parse-server-example)，该应用使用了 基于 Express 下的 Parse Server 模块并且它能够轻易地部署来提供各种各样的基础服务。

 - [Heroku and mLab](https://devcenter.heroku.com/articles/deploying-a-parse-server-to-heroku)
 - [AWS and Elastic Beanstalk](http://mobile.awsblog.com/post/TxCD57GZLM2JR/How-to-set-up-Parse-Server-on-AWS-using-AWS-Elastic-Beanstalk)
 - [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-parse-server-on-ubuntu-14-04)
 - [Microsoft Azure](https://nodechef.com/blog/post/6/migrate-from-parse-to-nodechef%E2%80%99s-managed-parse-server)
 - [Pivotal Web Services](https://nodechef.com/blog/post/6/migrate-from-parse-to-nodechef%E2%80%99s-managed-parse-server)
 - [Pivotal Web Services](https://medium.com/@justinbeckwith/deploying-parse-server-to-google-app-engine-6bc0b7451d50)
 - [Back4app](http://blog.back4app.com/2016/03/01/quick-wizard-migration/)
 - [HyperDev](https://azure.microsoft.com/en-us/blog/azure-welcomes-parse-developers/)

###Parse Server + Express
你也可以创造一个 Parse Server 的实例，并且配置到新的或已经存在的 Express 网站中。
```
var express = require('express');
var ParseServer = require('parse-server').ParseServer;
var app = express();

var api = new ParseServer({
  databaseURI: 'mongodb://localhost:27017/dev', // Connection string for your MongoDB database
  cloud: '/home/myApp/cloud/main.js', // Absolute path to your Cloud Code
  appId: 'myAppId',
  masterKey: 'myMasterKey', // Keep this key secret!
  fileKey: 'optionalFileKey',
  serverURL: 'http://localhost:1337/parse' // Don't forget to change to https if needed
});

// Serve the Parse API on the /parse URL prefix
app.use('/parse', api);

app.listen(1337, function() {
  console.log('parse-server-example running on port 1337.');
});
```
了解更多关于可用的配置选项请运行 *parse-server --help*.
###日志
Parse Server 默认地打印到：

 - 控制台
 - 在日常文件中成 JSON 格式的独立一行
如果想打印在每一个请求与回应中？在启动 *parse-server* 设置 *VERBOSE* 这个环境参数。
用法：*- VERBOSE='1' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY*
如果想让日志存放在一个文件夹中？在启动 *parse-server* 设置 *PARSE_SERVER_LOGS_FOLDER* 这个环境参数。
用法：*PARSE_SERVER_LOGS_FOLDER='<path-to-logs-folder>' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY*
如果想要新建一个独立的 JSON 错误日志（针对 CloudWatch，Google Cloud Logging 等等的云服务器）？在启动 parse  server 的时候设置 *JSON_LOGS* 这个环境参数。
用法：*JSON_LOGS='1' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY*

##文档
Parse Server 的详细文档已经放到了 [wiki](https://github.com/ParsePlatform/parse-server/wiki)。Parse Server 的指南是一个很好的启动指引，如果你对于开发 Parse Server 感兴趣，[开发者文档](https://github.com/ParsePlatform/parse-server/wiki/Development-Guide) 将会帮助你搭建。
###迁移已经存在的 Parse 应用
Parse 的服务将会在 2017.1.28 全部关闭。如果你计划迁移你的应用，你需要尽快开始你的工作了。这里有几点是 Parse 主版本没有提供一致的标准的。更多详细信息请进入[迁移指南](https://parse.com/migration).
###配置
Parse Server 可以配置使用。你可以通过在运行 parse-server 的时候设置参数或下载配置 JSON 格式的文件，配置
文件通过使用 parse-server path/to/configuration.json. 如果你正在 Express 使用 Parse Server，可以通过设置
一个 ParseServer 对象来存储这些配置参数。
####基础配置参数

 - appId（必填）-这个应用的 id 是用于这个服务的主域名的。你可以使用任意字符串来定义。为了迁移应用，这个应该匹配你已经拥有的 Parse 应用。
 - masterKey（必填）- master key 是用于控制 ACL 安全的，你可以使用任意字符串命名，必须保证它的私密性！为了迁移应用，应该用你存在的 parse 应用
 - databaseURI（必填）- 这个是链接数据库的字符串，例如：mongodb://user:pass@host.com/dbname。如果你的密码有特殊字符请确认使用 URL 编码模式。
 - prot-默认端口为 1337, 设置这个参数可以使用不同的端口。
 - serverURL- 你的 parse 应用的 URL（别忘了 http：//或 https：//）。这个 URL 将会被用于云端请求 Parse Server 服务。
 - cloud - 这个值是你的云端代码的绝对路径。
 - push - 配置参数 APNS 和 GCM push。详见 [Push 入口](https://github.com/ParsePlatform/parse-server/wiki/Push)

####客户端配置参数
这些 Parse 客户端的字段对于 Parse Server 来说不再是必要的。如果你希望仍然使用他们，也许能够拒绝一些旧版本的版本的客户端请求，你可以在初始化的时候设置好这些字段。设置其中的一个字段在请求的时候就需要带上相应的字段。

 - clientKey
 - javascriptKey
 - restAPIKey
 - dotNetKey

####可用的选项

 - fileKey - 为了迁移应用，这个对于 Parse 已经部署的应用的文件请求是必要的。
 - allowClientClassCreation - 默认设置为 true，设置为 false 能让客户端不能创建类
 - enableAnonymousUsers - 默认为 true，设置为 false 是杜绝匿名用户。 
 - oauth - 用于验证第三方认证
 - facebookAppIds - 储存用户用于登录的 Facebook 应用的 IDs。
 - mountPath - 服务的安装路径，默认为 /parse
 - filesAdapter - 默认行为可以创建一个对象改变。
 - maxUploadSize - 最大的文件上传大小，默认为 20 mb
 - loggerAdapter - 默认行为可以创建一个对象改变。
 - sessionLength - 一个会话的有效时间长度，默认为 31536000 秒（1 年）
 - revokeSessionOnPasswordReset - 当用户修改密码的时候，或者通过重置密码邮件来登录的时候，如果该值为 true 则所有对话将被取消，如果你不想所有会话被删除则设置为 false
 - accoundLockout - 当有恶意用户行为企图推断用户密码的时候通过验证码和错误信息来锁定用户。
 

####日志
在启动 Parse-server 使用 PARSE_SERVER_LOGS_FOLDER 这个环境变量字段来保存你的服务日志到你想要的文件夹。
用法：
```
PARSE_SERVER_LOGS_FOLDER='<path-to-logs-folder>' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY
```
####邮箱验证与密码重置
确认用户的邮件地址并让密码重置邮件能够传到邮件服务器。作为 Parse-server 的一个模块，我们提供了一个可配值对象使用 Mailgun 来发送邮件，并将它加到你的初始化代码中。
```
var server = ParseServer({
  ...otherOptions,
  // Enable email verification
  verifyUserEmails: true,

  // if `verifyUserEmails` is `true` and
  //     if `emailVerifyTokenValidityDuration` is `undefined` then
  //        email verify token never expires
  //     else
  //        email verify token expires after `emailVerifyTokenValidityDuration`
  //
  // `emailVerifyTokenValidityDuration` defaults to `undefined`
  //
  // email verify token below expires in 2 hours (= 2 * 60 * 60 == 7200 seconds)
  emailVerifyTokenValidityDuration: 2 * 60 * 60, // in seconds (2 hours = 7200 seconds)

  // set preventLoginWithUnverifiedEmail to false to allow user to login without verifying their email
  // set preventLoginWithUnverifiedEmail to true to prevent user from login if their email is not verified
  preventLoginWithUnverifiedEmail: false, // defaults to false

  // The public URL of your app.
  // This will appear in the link that is used to verify email addresses and reset passwords.
  // Set the mount path as it is in serverURL
  publicServerURL: 'https://example.com/parse',
  // Your apps name. This will appear in the subject and body of the emails that are sent.
  appName: 'Parse App',
  // The email adapter
  emailAdapter: {
    module: 'parse-server-simple-mailgun-adapter',
    options: {
      // The address that your emails come from
      fromAddress: 'parse@example.com',
      // Your domain from mailgun.com
      domain: 'example.com',
      // Your API key from mailgun.com
      apiKey: 'key-mykey',
    }
  },

  // account lockout policy setting (OPTIONAL) - defaults to undefined
  // if the account lockout policy is set and there are more than `threshold` number of failed login attempts then the `login` api call returns error code `Parse.Error.OBJECT_NOT_FOUND` with error message `Your account is locked due to multiple failed login attempts. Please try again after <duration> minute(s)`. After `duration` minutes of no login attempts, the application will allow the user to try login again.
  accountLockout: {
    duration: 5, // duration policy setting determines the number of minutes that a locked-out account remains locked out before automatically becoming unlocked. Set it to a value greater than 0 and less than 100000.
    threshold: 3, // threshold policy setting determines the number of failed sign-in attempts that will cause a user account to be locked. Set it to an integer value greater than 0 and less than 1000. 
  },
});
```
你也可以使用社区中的其他邮件服务：

 - parse-server-postmark-adapter
 - parse-server-sendgrid-adapter
 - parse-server-mandrill-adapter
 - parse-server-simple-ses-adapter
 - parse-server-mailgun-adapter-template
 - parse-server-mailjet-adapter

###使用环境变量来配置 Parse Server
你可以使用以下的环境变量来配置你的 Parse 服务：
```
PORT
PARSE_SERVER_APPLICATION_ID
PARSE_SERVER_MASTER_KEY
PARSE_SERVER_DATABASE_URI
PARSE_SERVER_URL
PARSE_SERVER_CLOUD_CODE_MAIN
```
默认端口为 1337,如果想用不同的端口则需要设置 PORT 环境变量
```
$ PORT=8080 parse-server --appId APPLICATION_ID --masterKey MASTER_KEY
```
使用 pase-server --help 获取配置环境变量列表

####可用的模块
parse server modules（adapters）

####配置文件模块
parse server 允许开发者选择几个选项来选择文件储存方式

 - GridStoreAdapter - 使用 MongoDB 来储存
 - S3Adapter - 使用 Amazon S3 来存储
 - GCSAdapter - 使用 Google Cloud Storage 来存储

GridStoreAdapter 是默认使用的方式，不需要设置，如果你想要使用其他方式像 S3 或 Google Cloud Storage，就需要添加配置信息。