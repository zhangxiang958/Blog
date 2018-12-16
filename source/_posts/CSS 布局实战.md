title: CSS 布局实战
date: 2016-02-04 15:15:24
categories: CSS
---


CSS 布局是 CSS 开发中重要且容易被忽略的知识。它往往担任着网站布局的骨架。以下将会讲述 CSS 布局的知识及其实战，并且详细讲述其原理。
<!--more-->

CSS 布局的知识其实在重构方面非常重要，它有利于开发人员建立起一个弹性且可维护的 CSS 系统，
并避免一些无谓的 HTML 标签。其实 CSS 布局做好了，一些莫名其妙的 BUG 也会随之消失，也就代表
避免了无谓的 hack。下面切入正题：
##强大的圣杯与双飞翼布局
圣杯布局与双飞翼布局是制作网站布局的利器，无论是三列布局还是两列布局都可以轻易应付。
在视觉上，两种布局效果类似，但是在实现方法上有所不同。
###实现方法
![](http://7xns9g.com1.z0.glb.clouddn.com/shuangfeiyi.png)
这部分是作为内容区域的展示，可以在整个 div 前面加上想要的头部与页脚，构成网站布局。
先来介绍较为简单的双飞翼布局：
HTML 结构：
![](http://7xns9g.com1.z0.glb.clouddn.com/shuangfeiyicode.png)
CSS 样式：
![](http://7xns9g.com1.z0.glb.clouddn.com/shuangfeiyicss.png)
双飞翼布局运用了浮动，与负 margin 进行定位布局的。
代码中的 .main-warp 容器是用于让 main 的内容展示在合适的地方的。所有的主体容器都采取浮动，通过
margin-left 赋值为负值让 .sub 和 .extra 展示在合适的地方。
两边的容器 .sub 和 .extra 都需要定好宽度，并且 .sub 的 margin-left 为 -100%，意思是让 .sub 的
左边框与包含块的左边框重叠。同理，.extra 的 margin-left 设置为容器的宽度，意思为让 .extra 的右
边框与包含块的右边框重叠。
![](http://7xns9g.com1.z0.glb.clouddn.com/shengbei.png)
HTML 结构：
![](http://7xns9g.com1.z0.glb.clouddn.com/shengbeicode.png)
CSS 样式：
![](http://7xns9g.com1.z0.glb.clouddn.com/shengbeicss.png)
圣杯布局是不添加标签的情况下，最接近完美的布局，它使用浮动，负 margin-left 和 position 进行布局。
很多人会有点不解，为什么又使用负 margin-left 又使用定位属性，原因是，这里没有添加额外的容器去展
示内容，因此就需要主容器使用 padding 使内容正确展示，但是设置了 padding 后，.sub 和 .extra 的位置
又不正确，只能通过 position:relative; 进行位置定位。

下面提供 Demo 地址供大家阅读参考。
[Demo 地址](https://github.com/zhangxiang958/Task)

理解并运用圣杯布局与双飞翼布局，可以创作出目前大多数网站布局。
###实现原理
在实现原理上，有几个点是非常重要的。
####负 margin 
负 margin 在布局上有着很重要的作用，它的定位功能相比于 position 属性而言更为温和，相比于相对定位，它的特点是放弃元素原来在文档中的位置。
负 margin 在理解上，举个栗子：父元素的外边距在 100 px 处，而子元素的 margin-top = -15 px，这时候子元素
的外边距就会在 85 px 处。没错，在视觉上，子元素就像是被拉上去了一样。
为了帮助理解，还可以引入一个参考线的概念，元素的上与左边框为一类边框，它们以本元素以外的元素
作为基准，值为正值时，加大本元素与其他元素的距离，值为负值时，减少本元素与其他元素的距离。而
元素的下与右边框为一类边框，他们以本元素为基准，值为正值时，“挤远” 相邻元素，值为负值时，
“拉近” 其他元素。

更详细的解释请移步：[我知道你不知道的负Margin](http://www.hicss.net/i-know-you-do-not-know-the-negative-margin/)

在实战中，负 margin 有很多方面的运用，除了我们已经知道的两栏/三栏布局之外，消除我们平时块级边框
重叠的尴尬，使之视觉上只有一个边，或是消除水平列表最后一项的 margin-right 等等。其中我觉得最重要
的一点是，负 margin 很多时候可以代替浮动布局或相对定位。

####BFC
BFC, 就是 Block Formatting Context 块级格式上下文。它并不是一个 CSS 属性，它是 CSS规范中
的一个重要特性，它的目的就在于让一个元素拥有布局（引用《CSS Mastery》书中原话）。
可以理解为 BFC 是一个隔离的盒子，内部元素的属性不会影响外部的表现。它是由开发人员通过某些
手段来启用这个特性的。
那么可以通过哪些手段呢？ 对于 overflow 属性其值不为 none 的元素，浮动元素，position 属性
其值为 absolute 或 fixed 的元素，根元素，display 属性其值为 inline-block, table-cell, table-caption, flex, inline-flex 的元素都会生成 BFC。
生成了 BFC 的元素，其外边距不会折叠，其高度会将其浮动的子元素的高度计算在内（即元素对浮动
自动清除），垂直方向上在同一个 BFC 中 margin 才会重叠。

在实战中，BFC 是解决很多问题的有效手段，很多时候我们都会迫使元素拥有布局以修复问题。
比如，通过设置 overflow 触发 BFC 生成自适应的两列布局，不过这缺点是 overflow 有可能不
显示部分内容。或清除浮动，或是防止内部 margin 重叠等等。

深入理解请看此博文：[深入理解BFC和Margin Collapse](http://www.w3cplus.com/css/understanding-bfc-and-margin-collapse.html)
##常用且不可忽视的等高多列
等高列在网站设计中是常见的要求，但是在实际开发中，等高列却不是那么容易实现。
### Faux 列
在《CSS Mastery》中有提及，faux 列是一种假等高列，它利用背景图在 y 方向上的重叠，
造成等高列的假象。
```
.warpper {
    background: #fff url(images/bg.png) repeat-y 25% 0;
}
```
重点在于背景图片的位置调整，将图片移动到合适的地方再进行重复重叠。缺点在于一旦背景色
需要改变则需要重制作背景图，并且列的宽度不能改变。
###Padding 与 Margin 对冲
```
    <div class="warpper">
		<div class="box">
			<h1>andy budd</h1>
			<p>....</p>
			<div class="bottom"></div>
		</div>
		<div class="box">
			<h1>richard rutters</h1>
			<p>....</p>
			<div class="bottom"></div>
		</div>
		<div class="box">
			<h1>jeremy keith</h1>
			<p>....</p>
			<div class="bottom"></div>
		</div>
	</div>
```
```
        .warpper {
			width: 100%;
			overflow: hidden;
		}
		.box {
			float: left;
			display: inline;
			padding-top: 20px;
			padding-right: 20px;
			padding-left: 20px;
			padding-bottom: 520px;
			margin-bottom: -500px;
			margin-left: 20px;
			width: 250px;
			background: #98ac10;
		}
```
设置一个足够大的 padding-buttom，与一个足够小的负 margin-bottom 进行对冲，目的是为了让父元素
的 overflow：hidden 让各列在最高点被裁剪达到视觉上等高的效果，原理是让垂直格式化上的 padding-
bottom 和 margin-bottom 进行抵消。原因是padding 是有颜色或背景的，而用没有颜色或背景的 margin
进行遮挡。
但是这个方法的缺点是一旦需要边框，底部无法显示，我们可以通过一个 .bottom 标签通过给父元素设置
相对定位，而 .bottom 设置绝对定位模拟底部边框。
但是在不需要边框的情况下，推荐使用这种方法。
###容器模拟背景
```
     <div class="container">
        <div class="container-right">
            <div class="container-content">
                <div class="container-left">
                    <div class="left"></div>
                    <div class="content"></div>
                    <div class="right"></div>
                </div>
            </div>
        </div>
    </div>    
```
```
    .container-right {
        float: left;
        position: relative;
        width: 100%;
        overflow: hidden;
    }
    .container-content {
        float: left;
        position: relative;
        right: 40%;  /*相当于右边列的宽度*/
        width: 100%;
    }
    .container-right {
        float: left;
        position: relative;
        right: 50%; /*相当于content列的宽度*/
        width: 100%;
    }
    .left {
        float: left;
        position: relative;
        left: 72%;
        width: 30%;
        overflow: hidden;
    }
    .content {
        float: left;
        position: relative;
        left: 82%;
        width: 40%;
        overflow: hidden;
    }
    .right {
        float: left;
        position: relative;
        left: 92%;
        width: 30%;
        overflow: hidden;
    }
```
这种方法我并不提倡，因为为了造成等高列效果，加上太多繁杂的容器进行背景模拟，对网页性能
造成一定的影响，布局与内容的耦合度较高，不是理想的布局。

[参考文献](http://www.w3cplus.com/css/creaet-equal-height-columns)





