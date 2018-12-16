title: Node.js 中的模块机制
date: 2018-07-15 21:34:24
categories: Node.js

---
日常经常使用 node 中的模块，当我们遵循 dry 原则，将一些逻辑独立封装成一个可供复用的模块的时候，有可能往往会忽略 node 模块本身的知识。
<!-- more -->
## 模块的查找流程
我们在开发的时候经常使用了 require 方法, 但是可能会忽略它内部的细节. 到底 require 的时候 node 是怎么查找对应的模块的? 其实 node 在面对不同的路径标识符有不同的处理细节:
### 内置模块
像 http, net, stream 这样的模块, node 早已经将这样的模块编译为二进制模块, 在 Node 启动的时候就已经加入到内存中了，所以省略了查找定位文件与编译文件的步骤。这样的模块加载是最快的。
### 绝对路径/相对路径模块
在识别路径标识符的时候，如果标识符前面有 ./ 或 .. 或者 / 这样的字符，那么就会被当作相对路径或者是绝对路径，然后解析得到一个真实的路径之后开始查找文件，node 的模块机制会先把标识符当作是一个文件名来处理，来查找文件，并且如果省缺了后缀名会按照 .js, .json, .node 的顺序来尝试加载，如果没有找到对应的文件的话， 那么就会将这个标识符当作是一个目录来查找，node 会查找该目录下的 package.json, 解析出 json 中的 main 字段，根据 main 字段中指定的文件路径查找指定文件:
```
{
    "main": "start.js" // 当 require 的时候，会优先查看此字段，若无则查找 index.js/index.json/index.node
}
```
如果没有指定文件或者找不到指定的文件的话，然后就会在该目录下按照 index.js, index.json, index.node 的顺序来加载，如果还是没有找到的话，那么就会抛出一个无法找到文件的错误。
### NPM 模块包
如果路径标识符既不是内置模块路径，又不是相对路径或绝对路径的话， 就会被当作是 NPM 模块包来加载， 而它的加载顺序是可以使用 **module.paths** 来查看。module.paths 返回是一个数组，第一个元素是文件当前目录下的 node_modules 文件夹，越往后就是上一级目录的 node_modules 文件夹，直到查找到 home 目录下的 node_modules 文件夹。
```
// module.js
console.log(module.paths);

// 打印结果
[ 'D:\\side project\\test\\node_modules',
  'D:\\side project\\node_modules',
  'D:\\node_modules' ]
```
由此可以知道，如果文件的层级越深，那么查找逻辑也会越复杂，加载也会越慢。
在每一个查找文件路径的节点也就是各文件层级的 node_modules 文件夹下，类似于相对路径或绝对路径模块的加载逻辑，node 会在各个 node_modules 下先查找以标识符为主的分别是 .js, .json, .node 后缀的文件，如果没有找到，那么就当作是一个目录来进行来查找目录下的 package.json, 并解析出其中的 main 字段指定的文件路径，如果没有文件路径或者文件路径错误，目录下的 index.js, index.json, index.node, 如果没有查找到上述的文件，那么就会抛出一个错误。
可以看到其实相对/绝对路径的加载逻辑与 NPM 模块的加载逻辑是类似的，下面提供一个流程图来帮助理解：
![](http://ofhhdgr62.bkt.clouddn.com/module.png)

### require.resolve
知晓了上面的查找逻辑，node 模块本身提供了一个查找路径的方法：require.resolve. 这个方法可以帮助你检查这个模块是否存在。

## 模块是什么
在 Node 里面，每个模块其实是一个对象：
```
Module {
  id: '.',
  exports: {},
  parent: null,
  filename: 'D:\\side project\\test\\test.js',
  loaded: false,
  children: [],
  paths: [ 'D:\\side project\\test\\node_modules',
    'D:\\side project\\node_modules',
    'D:\\node_modules'] 
}
```
可以看到， Module 对象里面存储了很多关于模块的信息，而理解这些信息对于理解模块机制是重要的。
我们来实际看一下一个简单的模块引用的例子：
```
// lib.js
console.log('in lib', module);

// index.js
require('./lib.js');
console.log('in index', module);
```
他们打印出来的结果是:
```
in lib Module {
  id: 'D:\\side project\\test\\lib.js',
  exports: {},
  parent:
   Module {
     id: '.',
     exports: {},
     parent: null,
     filename: 'D:\\side project\\test\\index.js',
     loaded: false,
     children: [ [Circular] ],
     paths:
      [ [Array] ] },
  filename: 'D:\\side project\\test\\lib.js',
  loaded: false,
  children: [],
  paths:
   [ [Array] ] }
in index Module {
  id: '.',
  exports: {},
  parent: null,
  filename: 'D:\\side project\\test\\index.js',
  loaded: false,
  children:
   [ Module {
       id: 'D:\\side project\\test\\lib.js',
       exports: {},
       parent: [Circular],
       filename: 'D:\\side project\\test\\lib.js',
       loaded: true,
       children: [],
       paths: [Array] } ],
  paths:
   [ [Array] ] }
```
我们可以看到在模块中，模块 Module 对象会使用 parent 与 child 这一对键值对来保存父子引用关系并保存对应的模块信息对象。注意一下这里父子对象表示信息，我们看到，lib 的模块对象中的 parent 属性存储着父模块对象 index 的模块信息，而指向的父模块信息中会使用 child 属性来存储 lib 模块的模块信息，为了避免出现无限循环显示模块信息的问题，这里会使用一个 [Circular] 来代替原有的指向。
*注意这里为了简洁，将模块中的 path 数组信息省略为 [Array].*

### exports 与 module.exports
我们在模块中会使用 exports 与 module.exports 这两个变量来暴露出模块的一些属性或者方法，但是我们会发现在使用的时候，如果不小心会导致模块无法被正确暴露出需要的信息：
```
// 正确
exports.name = 'shawncheung';
exports.sex = 'man';

// 错误
exports = {
    name: 'shawncheung',
    sex: 'man'
};
```
出现上面的情况是因为没有理解 exports 与 module.exports 的关系，他们之间的关系其实是：
```
exports = module.exports = {};
```
可见，exports 只是一个 module.exports 的引用而已，如果我们对 exports 重新赋值，那么 exports 的引用指向就会改变了，就像错误示例一样，对于 exports 的修改是没有办法反应到 module.exports 上面的。可以理解为 exports 是 module.exports 的一个快捷方式。而最终模块的导出对象是以 module.exports 这个值为准的。如果像错误示例那样，最终模块暴露出去的仅仅是一个空对象而已。
### loaded
很明显, 这个属性是用于说明这个模块是否已经加载完成的. 但是疑问就在于上面的模块信息打印出来 loaded 都是 flase, 这是为什么? 这个是因为 loaded 这个值的改变是会在下一个事件循环中修改, 如果这个模块加载完成的话, 那么就会修改该值为 ture.
### 循环引用
其实这个问题也是老生常谈了, 所谓循环引用就是两个模块的相互引用, 举个简单的例子:
```
// a.js
exports.one = 'a1';
const b = require('./b');
console.log('b', b);

exports.two = 'a2';
exports.three = 'a3';

// b.js
const a = require('./a');
exports.oneInB = 'b1';
```
上面的例子里面 a.js 与 b.js 相互引用, 如果没有解决这个问题的话, 如果模块是同步执行并同时暴露出 exports 的话, 那么上面的代码就会报错, 因为其实 b 模块引入 a 模块的时候 a 模块并没有执行路逻辑. 而 CommonJs 规范解决的方式是暴露出来的 exports 默认就是一个对象, 然后模块执行到哪一步那么其他模块引入的时候就取到那一步暴露出来的 exports 对象.
那么对于上面的例子来说, b 模块取到的 a 模块的值就是:
```
{
    one: 'a1'
}
```
而 a 模块全部的 exports 是:
```
{
    one: 'a1',
    two: 'a2',
    three: 'a3'
}
```
后面的 two 和 three 没有拿到是因为 b 模块引入的时候 a 模块关于他们的逻辑还没有执行完.

而实际上对于模块**理解的重点在于，我们在 require 一个模块(比如 lib)的时候，node 做了什么工作？为什么我们没有定义 require 方法，module 对象， __firname 变量，__dirname 变量但它们却能够在模块中使用呢？**
### 文件编译执行
在 node 里面，模块都会被编译，而不同的模块类型会有不同的编译方式。在 node 里面提供了三种编译的逻辑：.js, .json, .node. 我们可以使用 require.extentions 来查看:
```
console.log(require.extentions);

{ '.js': [Function], '.json': [Function], '.node': [Function] }

.js function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  // 编译
  module._compile(stripBOM(content), filename);
}
.json function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  try {
    // 直接解析
    module.exports = JSON.parse(stripBOM(content));
  } catch (err) {
    err.message = filename + ': ' + err.message;
    throw err;
  }
}
.node function(module, filename) {
    // 使用 dlopen 进行加载执行
  return process.dlopen(module, path.toNamespacedPath(filename));
}
```
从上面的结果可以看到，模块的加载其实本质上就是利用同步的 fs.readFileSync 函数来读取其中的文件内容，然后根据不同的类型将其进行编译。
因为这里是使用同步 API, 也就是说, 如果我们使用了异步的方法来修改 exports 的值或者 module.exports 的值都不会生效.
而且, 我们编写的 js 模块都会被包裹进一个函数里面执行, 使用 require('module').wrapper 可以查看用于包装的函数:
```
[ '(function (exports, require, module, __filename, __dirname) { ',
  '\n});' ]
```
我们编写的模块会被包裹在这个函数字符串里面, 这也解释了为什么我们没有定义 require 方法, __firname, __dirname, module 等等却能够使用的原因.

## 模块的缓存
不管什么样的模块, 所有的模块都是有缓存的：来看下面的 demo
```
// test.js
console.log('test');
module.exports = {};

//index.js
require('test');
require('test');
require('test');
```
上面的例子我们直接使用 node index.js 来执行就会发现 test 字符串只打印了一次，也就是说查找执行模块只执行了一次，后面的 require 的时候，node 会直接使用模块暴露出来的 module.exports 结果进行使用。
在二次加载的时候, 会去直接读取模块缓存中加载模块暴露出来的 exports 对象, 而不是重新执行上面那样复杂的加载流程, 极大地优化了性能. 而所有缓存的模块的信息都可以在 require.cache 上面找到.

## 模块的编写
如果你的模块是一个函数, 而且你想让这个模块在 require 的时候和直接在命令行的时候都可以被执行传参的话, 那么可以借用 require.main 来判断当前文件是否被直接执行:
```
if (require.main === module) {
    func.apply(null, argv.slice[2]);
} else {
    module.exports = func;
}
```


## 参考资料

 1. [requiring-modules-in-node-js-everything-you-need-to-know](https://medium.freecodecamp.org/requiring-modules-in-node-js-everything-you-need-to-know-e7fbd119be8)

