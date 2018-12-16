title: Vue2 throw Uncaught ReferenceError_create2 is not defined exception
date: 2017-08-12 10:45:24
categories: 虫师手记

---
在项目开发中发现在 android webview 中 vue router 在使用 webpack 打包之后的文件在某些机型像魅蓝 note, 锤子
手机会报 **Uncaught ReferenceError: _create2 is not defined**, 而在 github 上也有类似的 issues:  https://github.com/vuejs/vue/issues/4202.

## 解决方法
这个问题是 devtool 的配置引起的, 将 devtool 的配置项改为 *cheap-module-source-map* 就可以正常执行了.





