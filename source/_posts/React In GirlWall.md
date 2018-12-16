title: React In GirlWall
date: 2017-01-22 15:08:24
categories: Web Develop
---

这篇博文是对于许愿墙的使用 React 的总结.希望能够提高编写 React 的效率与优雅性.
<!--more-->
##编码规范
在编码的时候基本上都有遵循 React 的规范,但是在编写方法的时候并没有遵循, 应当在组件的构造函数中编写 bind
```
class Com extends React.Components {
    constructor() {
        
        this.fn = this.fn.bind(this);
    }
    
    fn() {
        console.log("test");
    }
    
    render() {
        return (
            <div onClick={ this.fn }>
                
            </div>
        );
    }
}
```
这样的好处是在 render 函数重复执行的时候不会重新创建一个函数. 另外 *"click"* 等回调函数应写在 render 函数
上面, 关于 componentUpdate 的函数应该写在一起.
##使用 jsx 语法
在选择 Vue 与 React 框架的时候,虽说 Vue 也可以使用 jsx 语法, React 也可以使用 template, 但是如果从他们较完整的生态上考虑,往往会面临选择 JSX 还是 template 的问题.
写 template 对于书写风格上无需太多改变,并且模板是声明式的.模板很好理解,学习成本很低.
对比 jsx, DOM 结构是用 render 函数, 即将 HTML 写进 Javascript 中,好处在于能够更快地看到报错信息.以及更好地
进行调试.
jsx 语法只是 JS 语法的一个映射,所以可以使用命名空间.


应该使用变量来存样式(All In JS).但是不能表示伪类伪元素


*注*: 在使用 react 的时候千万要摆正一种思想: 你是在写 Javascript, 利用代码来构建 UI 界面,而不是在写模板,
尤其是在写列表数据渲染的时候注意 *v-for* 与 *var ul = array.map() return* 的数据,再进行 *render* 
```
<ul>
    { ul }
</ul>
```
##双向数据绑定与单向数据流
双向数据绑定的出现是为了减轻数据更改时 UI 层的变动的繁琐,而单向数据流中的思想在于数据能够跨 UI 页面共享,
状态共享,方便管理数据,两者其实并不存在竞争关系,其实可以基于项目的需求来选择数据的交互方式,不必拘泥于某个框架
的某种形式.
但是 React 不建议使用双向数据绑定,这是因为考虑到深层次的子组件更改数据时,对于其他层级或最顶级的父组件的更改数据状态极其繁琐, 如果不使用 redux 这样的工具,往往需要花费较大的功夫来实现需求且通常这样的代码并不优雅甚至暴力.
假如只需要 React(React 只是一个 UI 层工具) ,而不是用 redux, 其实可以使用双向数据绑定,而不是网上很多人所说的并不清晰说明理由的 "不建议".
###使用 mobx
因为在开发项目的时候, 需要使用数据状态管理工具, 但是时间紧迫, 来不及上手学习成本比较大的 redux, 因此采取了
学习成本比较小的 *mobx*, 但是还是需要一点代价的.因为 mobx 使用的人数比较少,因此文档和实践比较少,因此参考的
样本比较少.
下面是我做项目时找的一些参考项目与文档:

 1. [mobx-react-boilerpalte](https://github.com/mobxjs/mobx-react-boilerplate)
 2. [mobx-blog](https://mobx.js.org/faq/blogs.html)
 3. [mobx-tutorial](https://onsen.io/blog/mobx-tutorial-react-stopwatch/)

##理解虚拟DOM
如果能够理解虚拟 DOM , 能够更好地理解一些在编码的过程中遇到的重新渲染的 bug. 而关于虚拟 DOM 的内部原理
我将会在后续的博文进行学习详细讲解和结合项目分析.

##谈谈 MVVM 框架
其实以前我们准循的结构, 样式, javascript 脚本分离是不是就是正确的? 其实根据以前开发的项目,回想一下,其实这样
来说 debug 的难度并不小,因为如果需要 debug 需要了解 DOM 结构并且通读 js 脚本,这样非常耗精力, 而现在来说,基于
mvvm 框架, 将 DOM 结构与 js 脚本结合在一起,反而将调试的时间成本降低了, 我觉得这是一个非常好的征兆.
##附录

 - [Airbnb React编码规范](https://zhuanlan.zhihu.com/p/20616464?refer=FrontendMagazine)
 - 希望能够逐渐拥有自己的一套组件库,符合自己的业务需求,逐渐抛弃一些组件框架.

