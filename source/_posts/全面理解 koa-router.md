title: 全面理解 koa-router
date: 2018-06-03 18:13:24
categories: Node.js

---
koa 框架一直都保持着简洁性, 它只对 node 的 HTTP 模块进行了封装, 而在真正实际使用, 我们还需要更多地像路由这样的模块来构建我们的应用, 而 koa-router 是常用的 koa 的路由库. 这里通过解析 koa-router 的源码来达到深入学习的目的.
<!-- more -->
## 深入浅出路由模块
在 node 原生里面, 如果我们需要实现路由功能, 那么就可以像下面这样编写代码:
```
const http = require('http');
const { parse } = require('url');

const server = http.createServer((req, res) => {
    let { pathname } = parse(req.url);
    
    if (pathname === '/') {
        res.end('index page');
    } else if (pathname === '/test') {
        res.end('test page');
    } else {
        res.end('router is not found');
    }
});

server.listen(3000);
```
上面的代码通过解析原生 *request IncomingMessage* 的 url 属性, 利用 *if...else* 判断路径返回不同的结果.
但是上面的代码缺点也很明显, 如果路由过多, *if...else* 的分支也会越庞大, 不利于代码的维护与多人合作.因此,我们需要一个特定的路由模块来统一地模块化地解决路由功能的问题.
如果是使用 koa-router 的话, 那么可以借助下面的代码来简单建立一个 koa-router 库的使用 demo:
```
const Koa = require('koa');
const KoaRouter = require('koa-router');

const app = new Koa();
// 创建 router 实例对象
const router = new KoaRouter();

//注册路由
router.get('/', async (ctx, next) => {
  console.log('index');
  ctx.body = 'index';
});

app.use(router.routes());  // 添加路由中间件
app.use(router.allowedMethods()); // 对请求进行一些限制处理

app.listen(3000);
```
运行上面的代码, 访问根路由 '/' 我们可以看到返回数据为 'index', 这说明路由已经基本生效了.
我们来看上面的代码, 使用 koa-router 第一步就是新建一个 router 实例对象:
```
const router = new KoaRouter();
```
然后在构建应用的时候, 我们的首要目标就是创建多个 CGI 接口以适配不同的业务需求, 那么接下来就需要注册对应的路由:
```
router.get('/', async (ctx, next) => {
  console.log('index');
  ctx.body = 'index';
});
```
上面的示例使用了 GET 方法来进行注册根路由, 实际上不仅可以使用 GET 方法, 还可以使用 POST, DELETE, PUT 等等
node 支持的方法.
然后为了让 koa 实例使用我们处理后的路由模块, 我们需要使用 routes 方法将路由加入到应用全局的中间件函数中:
```
app.use(router.routes());  // 添加路由中间件
app.use(router.allowedMethods()); // 对请求进行一些限制处理
```
## 源码架构与解析
通过上面的代码, 我们已经知道了 koa-router 的简单使用, 接下来我们需要深入到代码中, 理解它是怎么做到匹配从
客户端传过来的请求并跳转执行对应的逻辑的.在此之前我们先看一下代码的结构图:
![](http://img.ijarvis.cn/%E7%BB%93%E6%9E%84%E5%9B%BE.png)
### Router & Layer
第一步, 我们需要新建一个 Router 的实例对象, 而对于一个 Router 的实例来说理解其属性是至关重要的.
```
function Router(opts) {
  if (!(this instanceof Router)) {
    return new Router(opts);
  }

  this.opts = opts || {};
  this.methods = this.opts.methods || [
    'HEAD',
    'OPTIONS',
    'GET',
    'PUT',
    'PATCH',
    'POST',
    'DELETE'
  ];

  this.params = {};
  this.stack = [];
};
```
可以看到, 实际有用的属性不过 3 个, 分别是 methods 数组, params 对象, stack 数组. methods 数组存放的是允许
使用的 HTTP 方法名, 会在 Router.prototype.allowedMethods 方法中使用, 我们在创建 Router 实例的时候可以进行配置, 允许使用哪些方法. 而对于 params 对象它存储的是键为参数名与值为对应的参数校验函数, 这样是为了通过在全局存储参数的校验函数, 方便在注册路由的时候为路由的中间件函数数组添加校验函数. 对于 stack 数组, 则是存储每一个路由, 也就是 Layer 的实例对象, 每一个路由都相当于一个 Layer 实例对象.
对于 Layer 类来说, 创建一个实例对象用于管理每个路由:
```
function Layer(path, methods, middleware, opts) {
  this.opts = opts || {};
  // 路由命名
  this.name = this.opts.name || null;
  // 路由对应的方法
  this.methods = [];
  // 路由参数名数组
  this.paramNames = [];
  // 路由处理中间件数组
  this.stack = Array.isArray(middleware) ? middleware : [middleware];
  // 存储路由方法
  methods.forEach(function(method) {
    var l = this.methods.push(method.toUpperCase());
    if (this.methods[l-1] === 'GET') {
      this.methods.unshift('HEAD');
    }
  }, this);

  // 将添加的回调处理中间件函数添加到 Layer 实例对象的 stack 数组中
  this.stack.forEach(function(fn) {
    var type = (typeof fn);
    if (type !== 'function') {
      throw new Error(
        methods.toString() + " `" + (this.opts.name || path) +"`: `middleware` "
        + "must be a function, not `" + type + "`"
      );
    }
  }, this);

  this.path = path;
  this.regexp = pathToRegExp(path, this.paramNames, this.opts);

  debug('defined route %s %s', this.methods, this.opts.prefix + this.path);
};
```
我们可以看到, 对于 Layer 的实例对象, 核心的逻辑还是在于将 path 转化为正则表达式用于匹配请求的路由,  然后将路由的处理中间件添加到 Layer 的 stack 数组中. 注意这里的 stack 和 Router 里面的 stack 是不一样的, Router 的 stack 数组是存放每个路由对应的 Layer 实例对象的, 而 Layer 实例对象里面的 stack 数组是存储每个路由的处理函数中间件的, 换言之, 一个路由可以添加多个处理函数.
![](http://img.ijarvis.cn/router&layer.png)
### method 相关函数
所谓 method 就是 HTTP 协议中或者说是在 node 中支持的 HTTP 请求方法.其实我们可以通过打印 node 中的 HTTP 的方法来查看 node 支持的 HTTP method:
```
require('http').METHODS; // ['ACL', ...., 'GET', 'POST', 'PUT', ...]
```
在 koa-router 里面的体现就是我们可以通过在 router 实例对象上调用对应的方法函数来注册对应的 HTTP 方法的路由而且每个方法的核心逻辑都类似, 就是将传入的路由路径与对应的回调函数绑定, 所以我们可以遍历一个方法数组来快速构建原型的 method 方法:
```
methods.forEach(function (method) {
  Router.prototype[method] = function (name, path, middleware) {
    var middleware;
    // 判断有没有传入 name 参数, 如果有则处理参数个数问题
    if (typeof path === 'string' || path instanceof RegExp) {
      middleware = Array.prototype.slice.call(arguments, 2);
    } else {
      middleware = Array.prototype.slice.call(arguments, 1);
      path = name;
      name = null;
    }
    // 注册路由
    this.register(path, [method], middleware, {
      name: name
    });

    return this;
  };
});
```
上面函数中先判断 path 是否是字符串或者正则表达式是因为注册路由的时候还可以为路由进行命名(命名空间方便管理), 然后准确地获取回调的函数数组(注册路由可以接收多个回调), 这样如果匹配到某个路由, 回调函数数组中的函数就会依次执行. 留意到每个方法都会返回对象本身, 也就是说注册路由的时候是可以支持**链式**调用的.
此外, 我们可以看到, 每个方法的核心其实还是 register 函数, 所以我们下面看看 register 函数的逻辑.
### Router.prototype.register
register 是注册路由的核心函数, 举个例子, 如果我们需要注册一个路径为 *'/test'* 的接收 GET 方法的路由, 那么:
```
router.get('/test', async (ctx, next) => {});
```
其实它相当于下面这段代码:
```
router.register('/test', ['GET'], [async (ctx, next) => {}], { name: null });
```
我们可以看到, 函数将路由作为第一个参数传入, 然后方法名放入到方法数组中作为第二个参数, 第三个函数是路由的回调数组, 其实每个路由注册的时候, 后面都可以添加很多个函数, 而这些函数都会被添加到一个数组里面, 如果被匹配到, 就会利用中间件机制来逐个执行这些函数. 最后一个函数是将路由的命名空间传入.
这里避免篇幅过长, 不再陈列 register 函数的代码, 请移步 [koa-router 源码仓库关于 register 函数部分](https://github.com/alexmingoia/koa-router/blob/master/lib/router.js#L537) 查看.
register 函数的逻辑其实也很简单, 因为核心的代码全部都交由 Layer 类去完成了, register 函数只是负责处理 path 如果是数组的话那么需要递归调用 register 函数, 然后新建一个 Layer 类的实例对象, 并且检查在注册这个路由之间有没有注册过 param 路由参数校验函数, 如果有的话, 那么就使用 Layer.prototype.param 函数将校验函数加入到路由的中间件函数数组前面.
### Router.prototype.match
通过上面的模块, 我们已经注册好了路由, 但是, 如果请求过来了, 请求是怎么匹配然后进行到相对应的处理函数去的呢? 答案就是利用 match 函数.先看一下 match 函数的代码:
```
Router.prototype.match = function (path, method) {
  // 取所有路由 Layer 实例
  var layers = this.stack;
  var layer;
  // 匹配结果
  var matched = {
    path: [],
    pathAndMethod: [],
    route: false
  };
  // 遍历路由 Router 的 stack 逐个判断
  for (var len = layers.length, i = 0; i < len; i++) {
    layer = layers[i];

    debug('test %s %s', layer.path, layer.regexp);
    // 这里是使用由路由字符串生成的正则表达式判断当前路径是否符合该正则
    if (layer.match(path)) {
      // 将对应的 Layer 实例加入到结果集的 path 数组中
      matched.path.push(layer);
      // 如果对应的 layer 实例中 methods 数组为空或者数组中有找到对应的方法
      if (layer.methods.length === 0 || ~layer.methods.indexOf(method)) {
        // 将 layer 放入到结果集的 pathAndMethod 中
        matched.pathAndMethod.push(layer);
        // 这里是用于判断是否有真正匹配到路由处理函数
        // 因为像 router.use(session()); 这样的中间件也是通过 Layer 来管理的, 它们的 methods 数组为空
        if (layer.methods.length) matched.route = true;
      }
    }
  }

  return matched;
};
```
通过上面返回的结果集, 我们知道一个请求来临的时候, 我们可以使用正则来匹配路由是否符合, 然后在 path 数组或者 pathAndMethod 数组中找到对应的 Layer 实例对象.
### Router.prototype.routes(middlewares)
如果根据一开始的 demo 例子, 在上面注册好了路由之后, 我们就可以使用 router.routes 来将路由模块添加到 koa 的中间件处理机制当中了. 由于 koa 的中间件插件是以一个函数的形式存在的, 所以 routes 函数返回值就是一个函数:
```
Router.prototype.routes = Router.prototype.middleware = function () {
  var router = this;

  var dispatch = function dispatch(ctx, next) {
    ...
  };

  dispatch.router = this;

  return dispatch;
};
```
我们可以看到返回的 dispatch 函数在 routes 内部形成了一个闭包, 并且按照 koa 的中间件形式编写函数.对于 dispatch 函数内部逻辑就如下:
```
var dispatch = function dispatch(ctx, next) {
    debug('%s %s', ctx.method, ctx.path);
    
    var path = router.opts.routerPath || ctx.routerPath || ctx.path;
    // 根据 path 值取的匹配的路由 Layer 实例对象
    var matched = router.match(path, ctx.method);
    var layerChain, layer, i;
    
    if (ctx.matched) {
      ctx.matched.push.apply(ctx.matched, matched.path);
    } else {
      ctx.matched = matched.path;
    }
    
    ctx.router = router;
    // 如果没有匹配到对应的路由模块, 那么就直接跳过下面的逻辑
    if (!matched.route) return next();
    // 取路径与方法都匹配了的 Layer 实例对象
    var matchedLayers = matched.pathAndMethod
    var mostSpecificLayer = matchedLayers[matchedLayers.length - 1]
    ctx._matchedRoute = mostSpecificLayer.path;
    if (mostSpecificLayer.name) {
      ctx._matchedRouteName = mostSpecificLayer.name;
    }
    // 构建路径对应路由的处理中间件函数数组
    // 这里的目的是在每个匹配的路由对应的中间件处理函数数组前添加一个用于处理
    // 对应路由的 captures, params, 以及路由命名的函数
    layerChain = matchedLayers.reduce(function(memo, layer) {
      memo.push(function(ctx, next) {
        // captures 是存储路由中参数的值的数组
        ctx.captures = layer.captures(path, ctx.captures);
        // params 是一个对象, 键为参数名, 根据参数名可以获取路由中的参数值, 值从 captures 中拿
        ctx.params = layer.params(path, ctx.captures, ctx.params);
        ctx.routerName = layer.name;
        return next();
      });
      return memo.concat(layer.stack);
    }, []);
    // 使用 compose 模块将对应路由的处理中间件数组中的函数逐个执行
    // 当路由的处理函数中间件函数全部执行完, 再调用上一层级的 next 函数进入下一个中间件
    return compose(layerChain)(ctx, next);
};
```
### Router.prototype.allowedMethod
对于 allowedMethod 方法来说, 它的作用就是用于处理请求的错误, 所以它作为路由模块的最后一个函数来执行.同样地, 它也是以一个 koa 的中间件插件函数的形式出现, 同样在函数内部形成了一个闭包:
```
Router.prototype.allowedMethods = function (options) {
  options = options || {};
  var implemented = this.methods;

  return function allowedMethods(ctx, next) {
    ...
  };
};
```
上面的代码很简单, 就是保存 Router 配置中允许的 HTTP 方法数组在闭包内部
```
return function allowedMethods(ctx, next) {
    // 从这里可以看出, allowedMethods 函数是用于在中间件机制中处理返回结果的函数
    // 先执行 next 函数, next 函数返回的是一个 Promise 对象
    return next().then(function() {
      var allowed = {};
      // allowedMethods 函数的逻辑建立在 statusCode 没有设置或者值为 404 的时候
      if (!ctx.status || ctx.status === 404) {
        // 这里的 matched 就是在 match 函数执行之后返回结果集中的 path 数组
        // 也就是说请求路径与路由正则匹配的 layer 实例对象数组
        ctx.matched.forEach(function (route) {
          // 将这些 layer 路由的 HTTP 方法存储起来
          route.methods.forEach(function (method) {
            allowed[method] = method;
          });
        });
        // 将上面的 allowed 整理为数组
        var allowedArr = Object.keys(allowed);
        // implemented 就是 Router 配置中的 methods 数组, 也就是允许的方法
        // 这里通过 ~ 运算判断当前的请求方法是否在配置允许的方法中
        // 如果该方法不被允许
        if (!~implemented.indexOf(ctx.method)) {
          // 如果 Router 配置中配置 throw 为 true
          if (options.throw) {
            var notImplementedThrowable;
            // 如果配置中规定了 throw 抛出错误的函数, 那么就执行对应的函数
            if (typeof options.notImplemented === 'function') {
              notImplementedThrowable = options.notImplemented(); // set whatever the user returns from their function
            } else {
            // 如果没有则直接抛出 HTTP Error
              notImplementedThrowable = new HttpError.NotImplemented();
            }
            // 抛出错误
            throw notImplementedThrowable;
          } else {
            // Router 配置 throw 为 false
            // 设置状态码为 501
            ctx.status = 501;
            // 并且设置 Allow 头部, 值为上面得到的允许的方法数组 allowedArr
            ctx.set('Allow', allowedArr.join(', '));
          }
        } else if (allowedArr.length) {
          // 来到这里说明该请求的方法是被允许的, 那么为什么会没有状态码 statusCode 或者 statusCode 为 404 呢?
          // 原因在于除却特殊情况, 我们一般在业务逻辑里面不会处理 OPTIONS 请求的
          // 发出这个请求一般常见就是非简单请求, 则会发出预检请求 OPTIONS
          // 例如 application/json 格式的 POST 请求
          
          // 如果是 OPTIONS 请求, 状态码为 200, 然后设置 Allow 头部, 值为允许的方法数组 methods
          if (ctx.method === 'OPTIONS') {
            ctx.status = 200;
            ctx.body = '';
            ctx.set('Allow', allowedArr.join(', '));
          } else if (!allowed[ctx.method]) {
          // 方法被服务端允许, 但是在路径匹配的路由中没有找到对应本次请求的方法的处理函数
            // 类似上面的逻辑
            if (options.throw) {
              var notAllowedThrowable;
              if (typeof options.methodNotAllowed === 'function') {
                notAllowedThrowable = options.methodNotAllowed(); // set whatever the user returns from their function
              } else {
                notAllowedThrowable = new HttpError.MethodNotAllowed();
              }
              throw notAllowedThrowable;
            } else {
              // 这里的状态码为 405
              ctx.status = 405;
              ctx.set('Allow', allowedArr.join(', '));
            }
          }
        }
      }
    });
};
```
值得注意的是, Router.methods 数组里面的方法是服务端需要实现并支持的方法, 如果客户端发送过来的请求方法不被允许, 那么这是一个服务端错误 501, 但是如果这个方法被允许, 但是找不到对应这个方法的路由处理函数(比如相同路由的 POST 路由但是用 GET 方法来获取数据), 这是一个客户端错误 405.
### Router.prototype.use
use 函数就是用于添加中间件的, 只不过不同于 koa 中的 use 函数, router 的 use 函数添加的中间件函数会在所有路由执行之前执行.此外, 它还可以对某些特定路径的进行中间件函数的绑定执行.
```
Router.prototype.use = function () {
  var router = this;
  // 中间件函数数组
  var middleware = Array.prototype.slice.call(arguments);
  var path;

  // 支持同时为多个路由绑定中间件函数: router.use(['/use', '/admin'], auth());
  if (Array.isArray(middleware[0]) && typeof middleware[0][0] === 'string') {
    middleware[0].forEach(function (p) {
      // 递归调用
      router.use.apply(router, [p].concat(middleware.slice(1)));
    });
    // 直接返回, 下面是非数组 path 的逻辑
    return this;
  }
  // 如果第一个参数有传值为字符串, 说明有传路径
  var hasPath = typeof middleware[0] === 'string';
  if (hasPath) {
    path = middleware.shift();
  }
    
  middleware.forEach(function (m) {
    // 如果有 router 属性, 说明这个中间件函数是由 Router.prototype.routes 暴露出来的
    // 属于嵌套路由
    if (m.router) {
      // 这里的逻辑很有意思, 如果是嵌套路由, 相当于将需要嵌套路由重新注册到现在的 Router 对象上
      m.router.stack.forEach(function (nestedLayer) {
        // 如果有 path, 那么为需要嵌套的路由加上路径前缀
        if (path) nestedLayer.setPrefix(path);
        // 如果本身的 router 有前缀配置, 也添加上
        if (router.opts.prefix) nestedLayer.setPrefix(router.opts.prefix);
        // 将需要嵌套的路由模块的 stack 中存储的 Layer 加入到本 router 对象上
        router.stack.push(nestedLayer);
      });
      // 这里与 register 函数的逻辑类似, 注册的时候检查添加参数校验函数 params
      if (router.params) {
        Object.keys(router.params).forEach(function (key) {
          m.router.param(key, router.params[key]);
        });
      }
    } else {
      // 没有 router 属性则是常规中间件函数, 如果有给定的 path 那么就生成一个 Layer 模块进行管理
      // 如果没有 path, 那么就生成通配的路径 (.*) 来生成 Layer 来管理
      router.register(path || '(.*)', [], m, { end: false, ignoreCaptures: !hasPath });
    }
  });

  return this;
};
```
通过上面我们就清楚, 在 koa-router 里面, 它将所有的路由与所有路由都适用的中间件函数都看做 Layer, 通过 Layer 来处理, 然后将他们的回调函数存储在 Layer 实例本身的 stack 数组中, 然后全局的 router 实例对象的 stack 数组存放所有的 Layer 达到全局管理的目的.
## router 处理请求的流程
上面就是 koa-router 的核心 API, 下面我们通过一张图来总结一下, 看一下当一个请求来临, koa-router 是如何处理的:
![一个请求处理流程图](http://img.ijarvis.cn/%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B%E5%9B%BE.png)
## 附录
### 为什么需要在 GET 请求放一个 HEAD 请求 ?
我们可以看到在 Layer 的构建函数里面, 在对于 methods 的处理中, 会进行判断如果该请求为 GET 请求, 那么就需要在 GET 请求前面添加一个 HEAD 方法, 其原因在于 HEAD 方法与 GET 方法基本是一致的, 所以 koa-router 在处理 GET 请求的时候顺带将 HEAD 请求一并处理, 因为两者的区别在于 HEAD 请求不响应数据体.




