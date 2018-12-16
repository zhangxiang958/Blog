title: Modal In React
date: 2017-04-05 14:30:24
categories: React

---
在之前使用 React 做项目的时候, 由于时间比较紧, 对于项目中需要用到弹出框(模态框)的组件, 全部使用了 DOM 操作来
达到视觉效果, 达到目的, 但是这样代码非常不优雅, 不符合 React 的思想, 因此在完成项目之后, 仔细研究了如何在组
件中创建一个模态框 Modal.
<!--more-->
##技术难点
Dialog 的难点在于这个 div 通常需要放置到 body 标签下, 需要的时候插入到 body 底部, 不需要的时候将这个模态框移
除. 但是在 React 中, DOM 结构是以组件的形式来封装的, 那么换言之, 模态框不能脱离组件而存在于 body 中, 如果直
接将模态框放在 body 中, 又不优雅并且复用性低.
##解决方案
###组件思路
根据我自己的经验, 如果一来就说内部实现逻辑会造成混乱, 所以先说调用的形式
```
class ConfirmButton extends Component {
    constructor(props){
        super(props);
        
        this.state = {
            show: false   //控制 Modal 组件的显示或隐藏
        }
    }
    
    handleHide(){
        this.setState({ show: false });  //使用 this.state.show 来控制显隐
        this.Modal.onHide();           //或者直接调用模态框的方法
    }
    
    handleShow(){
        this.setState({ show: true });  //使用 this.state.show 来控制显隐
        this.Modal.onShow();           //或者直接调用模态框的方法
    }
    
    return (
        <button onClick={this.handleShow.bind(this)}>confirm</button>
    
        <Modal
            ref={(ref) => this.Modal = ref}>
            {/*像下面一样, modal 组件里面是 DOM 结构和封装的组件都是可以的 */}
            <div>modal</div>
            {/*
                or
                <Modalcontent hide={this.handleHide.bind(this)}/>
            */}
        </Modal>
    );    
}
```
Modal 组件就是我们实现的 modal 抽象高阶组件了, 对调用方式清晰了之后, 我们需要实现的就是 Modal 内部, 而内部
关键点在于如何将 Modal 组件下的 DOM 结构或者子组件包裹在一个容器中并插到 body 的底部呢?
###Modal 内部实现
综上, 我们会将模态框的行为抽象, 抽像成一个高阶的组件包着视觉组件.
```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';


class Modal extends Component {
    constructor(props){
        super(props);
        
        this.state = {
            state: this.props.show
        }
    }
    
    //显示 modal 的方法
    onShow(){
        this.modal = document.createElement('div');
        this.modal.style.cssText = 'position: absolute; top: 0; right: 0; bottom: 0; left: 0; background-color: rgba(0, 0, 0, 0.3)'; //在插入之前做 CSS 处理, 避免无谓的渲染, 也可以使用 css module 加入类名
        document.body.appendChild(this.modal);
        this.setState({ show: true });
        //核心方法
        this._renderLayer();
    }
    
    //隐藏 modal 的方法
    onHide(){
        ReactDOM.unmountComponentAtNode(this.modal);
        document.body.removeChild(this.modal);
        this.setState({ show: false });
    }
    
    //渲染方法
    _renderLayer(){
        //render the modal
    }
    
    render(){
        //只是行为抽象, 不需要渲染 DOM 节点
        return null;
        {/*
            or
            return React.DOM.div(this.props);
        */}
    }
}
```
上面就是简略版的 modal 组件的内部, 包含了两个方法, 在调用 modal 组件的时候, 可以使用 ref 暴露方法来调用.
而对于 _renderLayer 方法, 问题在于我们已经创建了一个 body 下的 div 了, 我们怎样才能将子组件与 DOM 结构
渲染到这个 div 中呢?
####使用 unstable_renderSubtreeIntoContainer
 答案是使用 React 中的一个不稳定的 API: ReactDOM.unstable_renderSubtreeIntoContainer(parentComponent
```
//先说明 ReactDOM.unstable_renderSubtreeIntoContainer(); 的几个参数作用
this.container = document.createElement('div');
document.body.appendChild(this.container);

_renderLayer(){
    ReactDOM.unstable_renderSubtreeIntoCiontainer(
        this,                 //一般填 this, 父组件
        this.props.children,  //组件中的子组件
        this.container,      //容器 div, 包裹子组件
        callback            //回调(可空缺)
    );
}    
```
使用这个 API,将 Modal 组件下的子组件指定地渲染到我们需要的容器中, 也就是 this.modal, 这样就可以达到目的了.
####使用 ReactDOM
在实现中发现有另外一种方法, 就是使用 ReactDOM.render();
```
_renderLayer(){
    ReactDOM.render(this.props.children, this.modal);
}
```
这样也可以指定渲染.

####一个有趣的地方
可以注意到, 上面我在 Modal 组件内部的 render 函数, 注释了可以使用 React.DOM.div(this.props);, 问题是这个有
什么作用呢? 这个 API 可以为子组件添加一个 div 容器.

总结:
我们在平时开发的时候可能会遇到一些小提示框或者小的菜单栏(也就是 popup modal 和 menus), 其实上面的两个方法就是解决这两个痛点的, 一个是 body 下的 modal, 一个是父容器加上 overflow: hidden 但是需要在父容器外展示的菜单栏.

##附录
###React 的异步更新
在实现的过程中发现, 在一个函数中, 同时对一个 DOM 对象进行移除并将对象指向 null 的时候, 会报错, 报错信息如
```
Invariant Violation: _registerComponent(…): Target container is not a DOM element

报错代码:
function onHide(){
    ReactDOM.unmountComponentAtNode(this.modal);
	document.body.removeChild(this.modal);
	this.modal = null;  //这里去掉这句
	this.setState({ show: false });
}
```
因为是异步操作, 当 this.modal 指向 null 的时候, ReactDOM 调用需要的 this.modal 已经不是一个 DOM 节点了
所以报错.


注: [solution on stackflow](http://stackoverflow.com/questions/26566317/invariant-violation-registercomponent-target-container-is-not-a-dom-elem)
link:

 - [Rendering React components to the document body](http://jamesknelson.com/rendering-react-components-to-the-document-body/)
 - [Boostrap overlay](https://github.com/react-bootstrap/react-bootstrap/blob/master/src/Overlay.js)
 - [一篇关于 React Dialog 的博文](http://www.cnblogs.com/qingguo/p/5701302.html)
