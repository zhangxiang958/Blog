title: 前端使用 Mock 与后端独立
date: 2017-01-23 19:55:24
categories: Web Develop
---

在项目开发期间,不应该出现开发人员有空白的时间,应该将项目中的调试与性能尽可能地去完善,但是有时候会遇到后台
还没有开发出 API 的时候,不应该干等后台人员开发出 API, 作为一名前端,应当利用 mock, 使用模拟假数据来尽早地
进入调试阶段与优化阶段.
<!--more-->
虽然 gulp 中也有像 gulp-mock 或 gulp-mock-server 这样的插件,但是从开发便利性与易上手性来说, mockjs 个人感觉更胜一筹, API 简洁友好, 使用简单,而  gulp 下的 mock 插件的 API 路径设置比较麻烦且不清晰.
推荐使用使用 [mock 工具--mockjs](https://github.com/nuysoft/Mock/wiki/Getting-Started).

###在 Ubuntu 下安装 mockjs
![](http://img.ijarvis.cn/mockjsnpm.png)
###Demo In Vue
提供一个我自己项目中使用 mock 的例子, 项目的结构其实只要根据图示进行 import 或 require 就可以使用了,简单
快捷,简直快速提高工作效率.

***项目结构***:
![](http://img.ijarvis.cn/vuePro.png)

***mock.js***:
在上面的项目结构中的 ***mock*** 文件夹中添加一个 ***mock.js***
![](http://img.ijarvis.cn/mockVue.png)

***使用 mock***:
![](http://img.ijarvis.cn/useMock.png)


***查看获取数据***:
![](http://img.ijarvis.cn/vueConsole.png)




