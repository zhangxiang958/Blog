title: Parse 系列之 Parse Dashboard 搭建（译）
date: 2016-09-18 18:59:24
categories: Parse
---
应老师的要求，因为项目需要使用 parse-server 来作为应用的后端支撑，因此我将我学习 parse-server 的笔记
或译文放到博文中，希望可以帮助到师弟师妹的后续学习。
本次译文翻译会生硬，欢迎大家提出意见。
<!--more-->
##启动项目
需要 Node.js 4.3 以上版本与 Parse Server 2.1.4 以上版本。
##本地搭建
通过 npm 安装 dashboard.
```
npm install -g parse-dashboard
```
你可以通过像下面这样的一个填好 app ID 和 master key URL 和 name 的命令来启动应用的 dashboard。
```
parse-dashboard --appId yourAppId --masterKey yourMasterKey --serverURL "https://example.com/parse" --appName optionalName
```
你可能需要--host，--post 和 --mountPath 的 parse-dashboard 选项来设置域名，端口，安装路径。你可以使用任何
你想要的 app 名称或者使用 app ID。
启动 dashboard 之后，你可以浏览器访问 http://localhost:4040 来查看。 
###配置 Parse Dashboard
####配置文件
你可以通过配置文件在命令行中启动 dashboard。如果想这样，我们需要新建一个文件命名为 parse-dashboard-config.json，并放在你的本地 Parse Dashboard 文件目录里面，文件内容如下：
```
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ]
}
```
这样就可以通过 parse-dashboard --config parse-dashboard-config.json 命令来启动 dashboard。
####环境参数
这个只有在使用命令行启动应用的时候才可用，这里也有两个方法来使用环境参数来配置你的 dashboard。
#####多个应用
在 PARSE_DASHBOARD_CONFIG 中提供完整的 JSON 格式的配置参数，它将会被像配置文件一样解释。
#####单个应用
你也可以单独配置每一个参数。
```
HOST: "0.0.0.0"
PORT: "4040"
MOUNT_PATH: "/"
PARSE_DASHBOARD_ALLOW_INSECURE_HTTP: undefined // Or "1" to allow http
PARSE_DASHBOARD_SERVER_URL: "http://localhost:1337/parse"
PARSE_DASHBOARD_MASTER_KEY: "myMasterKey"
PARSE_DASHBOARD_APP_ID: "myAppId"
PARSE_DASHBOARD_APP_NAME: "MyApp"
PARSE_DASHBOARD_USER_ID: "user1"
PARSE_DASHBOARD_USER_PASSWORD: "pass"
PARSE_DASHBOARD_SSL_KEY: "sslKey"
PARSE_DASHBOARD_SSL_CERT: "sslCert"
PARSE_DASHBOARD_CONFIG: undefined // Only for reference, it must not exist
```
###管理多个应用 app
在同样的 dashboard 中管理多个应用是可能的。要做的只是简单地添加信息到 parse-dashboard-config.json 文件中的 apps 数组中。
你可以管理本地服务器和 Parse.com 上的 Parse Server 应用，在你的配置文件中，你需要添加 restKey 和 javascriptKey 和其他一些参数，你可以在 dashboard.parse.com 找到相关信息。设置 serverURL 到 http://api.parse.com/1：
```
{
  "apps": [
    {
      "serverURL": "https://api.parse.com/1", // Hosted on Parse.com
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "javascriptKey": "myJavascriptKey",
      "restKey": "myRestKey",
      "appName": "My Parse.Com App",
      "production": true
    },
    {
      "serverURL": "http://localhost:1337/parse", // Self-hosted Parse Server
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "My Parse Server App"
    }
  ]
}
```
###app 应用图标配置
parse dashboard 支持为每一个应用添加 icon，这样你可以更容易在列表中分辨他们。你必须使用配置文件，配置一个
iconsFolder 字段，为每个应用（包括拓展）填写 iconName 参数，这个 iconsFolder 的路径是相对于配置文件的，如果你已经全局安装了 ParseDashboard，那么你需要填写 iconsFolder 的绝对路径。为了让这个更明了，在下面的例子中， icons 是与本地配置文件相同路径下的一个文件夹。
```
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "My Parse Server App",
      "iconName": "MyAppIcon.png",
    }
  ],
  "iconsFolder": "icons"
}
```
###其他配置参数
你可以在配置文件中为每个应用设置 appNameForURL 字段在 dashboard 中控制你的应用 URl。这个让 dashboard 中使用书签或分享链接更简单。

改变应用环境到生产环境，只要在你的配置文件中简单地设置 production 为 true。 
##作为 Express 中间件使用
不仅可以通过命令行来使用 Parse Dashboard，也可以在 express 以中间件的形式来使用。
```
var express = require('express');
var ParseDashboard = require('parse-dashboard');

var dashboard = new ParseDashboard({
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ]
});

var app = express();

// make the Parse Dashboard available at /dashboard
app.use('/dashboard', dashboard);

var httpServer = require('http').createServer(app);
httpServer.listen(4040);
```
如果你想要在同一个服务器或端口上同时运行 Parse Server 和 Parse Dashbord，如下：
```
var express = require('express');
var ParseServer = require('parse-server').ParseServer;
var ParseDashboard = require('parse-dashboard');

var allowInsecureHTTP = false

var api = new ParseServer({
    // Parse Server settings
});

var dashboard = new ParseDashboard({
    // Parse Dashboard settings
}, allowInsecureHTTP);

var app = express();

// make the Parse Server available at /parse
app.use('/parse', api);

// make the Parse Dashboard available at /dashboard
app.use('/dashboard', dashboard);

var httpServer = require('http').createServer(app);
httpServer.listen(4040);
```

##部署 Parse Dashboard
###准备部署
确保 srver URL 可以被你的浏览器解析。如果你想要部署 dashboard，那么 localhost urls 将会无效。
###考虑安全
为了省略你的应用 master key 并安全部署 dashboard ，你需要使用 HTTPS 和基本认证。

如果你使用安全链接到已经部署的 dashboard 会发现的。如果你正在使用一个负载均衡或者代理服务器来更早地使用 SSL 认证来部署，那么应用将不会被安全链接。如果是这样，你可以通过 --allowInsecureHTTP=1 字段来启动 dashboard。你有责任确保你的负载均衡器和你的代理服务器只能允许 HTTPS 协议。
####基本认证配置
你可以通过添加用户名和密码到你的 parse-dashboard-config.json 文件来配置你的 Basic Authentication。
```
{
  "apps": [{"...": "..."}],
  "users": [
    {
      "user":"user1",
      "pass":"pass"
    },
    {
      "user":"user2",
      "pass":"pass"
    }
  ],
  "useEncryptedPasswords": true | false
}
```
你可以存储 plain text 或 bcrypt 两种格式的密码。为了能够使用 bcrypt 格式，你需要设置 useEncryptedPasswords
字段值为 true。你可以使用在线加密工具来对密码加密。
####分离用户的应用权限
如果你已经配置好了你的 dashboard 来管理多个应用，你可以限制应用的管理员身份。
如果想达到目的，以下面的格式更新你的 parse-dashboard-config.json 文件：
```
{
  "apps": [{"...": "..."}],
  "users": [
     {
       "user":"user1",
       "pass":"pass1",
       "apps": [{"appId": "myAppId1"}, {"appId": "myAppId2"}]
     },
     {
       "user":"user2",
       "pass":"pass2",
       "apps": [{"appId": "myAppId1"}]
     }  ]
}
```
效果将是：
当 user1 登录，他将只能在 dashboard 上管理 myAppId1 和 myAppId2。
当 user2 登录，他将只能在 dashboard 上管理 myAppId1 。
##Docker 中的应用
在 Docker 使用是非常简单的。第一步要建立镜像：
```
docker build -t parse-dashboard .
```
使用 config.json 文件来启动镜像
```
docker run -d -p 8080:4040 -v host/path/to/config.json:/src/Parse-Dashboard/parse-dashboard-config.json parse-dashboard
```
默认在 4040 端口启动项目，然而，你也可以运行管理者命令来改变。
```
docker run -d -p 80:8080 -v host/path/to/config.json:/src/Parse-Dashboard/parse-dashboard-config.json parse-dashboard --port 8080
```
如果你对 docker 不熟悉，--port 8080 将会被当作参数传入命令中 npm start -- --port 8080. 应用将会在 8080 端口中运行并在远程主机中的 80 端口运行。



