title: 详谈 CSS 文本属性
date: 2015-10-25 17:25:24
categories: CSS
----
##文本属性
在网站设计中，很多时候需要对文本进行布局，因为文字在网站设计中有时候会占据较大的篇幅，因此在 CSS 中文本属性显得尤为重要。
<!--more-->
###文本框
什么是行内框？
    
![note](http://7xns9g.com1.z0.glb.clouddn.com/note.png "note")
###水平方向
####段落缩进 *text-indent*
text-indent，只作用于块级元素，作用是让元素的第一行实现缩进，它最常用的用途是处理首行缩进。它可以接受任意的长度单位，比如 em,px 等，当然也会接受百分比参数，但是使用百分比参数将会使元素缩进父元素的宽度的百分比。很多刚接触 CSS 的人会以为 text-indent 只会对元素内的文本产生作用，但是实际上不是。举个例子：
```
     p {text-indent: 10%; }
    <p>//这里的段落将会缩进父元素的宽度的 10%
        <img src = "images/star.gif " /> //因为图片是该段落的第一行，所以会缩进
        这是 前端开发者 Jarvis 张翔的 CSS 博文。
    </p>
```
text-indent 可以接受负值。这个属性可以通过继承达到意想不到的结果。但是对它进行赋负值的时候要注意，段落可能会超出屏幕的左边界，因此，最好给需要缩进的块级元素赋值左外边距或左内边距。
####水平对齐 *text-align*
text-align，是用来网站布局排版文字所用的属性。它只能运用于块级元素，能够对块级元素内的文本进行对齐排版。它接受几个参数： left , right , center , justify 等。就像字面上的意思，让文本可以靠左/右对齐，居中对齐，两端对齐。需要注意的是，居中的不是元素，而是元素内部的文本。
![text-align](http://7xns9g.com1.z0.glb.clouddn.com/css-textalign.png "text-align")
#####为何 text-align 对图片会产生效果？

###垂直方向
#### 行高 *line-height*
它的意思是文字基线之间的距离，而不是文字高度。它所控制的是行间距，而行间距是什么呢？行间距就是超出文本大小的空间，即是 line-height 与 文字大小的数值之差就是行间距。它可以运用到所有元素上。
    ![inline-height](http://7xns9g.com1.z0.glb.clouddn.com/inline-height.png "inline-height")
内容根据文字大小会生成内容区，而 line-height 指定的就是内容区上下的行间距，内容区加上行间距就是行内框。
line-height 可以接受长度单位，如 em，px 等，它可以从父元素中继承值，但是有个不好之处在于一旦继承，就不是相对于本元素的最佳选择行高，可能会出现重叠现象。所以，我们可以在子元素的样式中添加显式的属性来解决，不过这会显得麻烦，最好的办法是通过缩放因子。
```
 line-height : 1;//这就是所谓的缩放因子，就是指定的一个数。
```
这样的话，就可以通过计算子元素自身的 font-size 的大小来得出最适合的行高。
####垂直对齐
与 text-align 相对的就是 vertical-align。vertical-align 作用于行内元素与表格单元，它不可继承，使用百分数参数时参照的是元素的 line-height 的值。
#####对齐模式
vertical-align 使用 baseline 值会实现基线对齐，所谓基线对齐，就是元素基线与父元素基线对齐，如果没有基线，则是底部与基线对齐。
    使用 sub 或者是 super 将会变成上下标。
    使用 bottom 将会使元素与框内底部而不是基线进行对齐。而 top 则与 bottom 相反。
    使用 text-top 将会使元素与文本顶端对齐，与 text-bottom 相对。
    middle 是最常用的垂直居中对齐，而使用百分数则会让元素基线相对于父元素基线升高或是降低。
    使用长度对齐则是通过设置 5px 等的值进行垂直对齐，不过它不会覆盖或重叠在另一行文字之上。
![0](http://7xns9g.com1.z0.glb.clouddn.com/note.png)
####文字间隔，装饰与阴影
#####字间隔与字母间隔
字间隔，使用 word-spacing，进行字之间的空间调节。此属性接受正负值的长度单位。
但是，各位值得注意的是，这里的字不是一般意义上的字，它可能是一个或多个被空白符包围的字，这样的情况下，意味着我们如果文档中有象形文字或非罗马文字时，很可能会对这些文字不起作用，使用这个属性很可能会创建一个可读性弱的网页。字母间隔，使用 letter-spacing，同样地，它接受正负值的长度单位，调节字母间的间隔。
####文字装饰
文字装饰的属性是 text-decoration ，接受多个属性值，包括 underline , line-through , overline , none , blink 等。在众多属性中，none 属性值一般会用于取消某个元素的装饰。
text-decoration , 没有继承性，这表示如果父元素设置了文字装饰，文字装饰将与父元素颜色相同。这里可能不是很理解，举个例子：
 ![1](http://7xns9g.com1.z0.glb.clouddn.com/underline-css.png)
 
 ![2](http://7xns9g.com1.z0.glb.clouddn.com/underline-html.png)
 
 ![3](http://7xns9g.com1.z0.glb.clouddn.com/underline.png)
请注意，text-decoration 默认值为 none，效果图中 strong 标签还是有下划线，但是实际上，strong 标签是没有下划线的，只是 p 标签的下划线穿过了 strong 标签。想要让 strong 标签也有下划线，且下划线与文字颜色相同只能通过显式设置了。
####文字阴影
文字阴影，text-shadow，可以接受三组阴影设置，值设置顺序是 ： 颜色，偏移距离（长度），偏移距离（长度），模糊半径。
第一个和第二个长度值设定了阴影在元素的什么位置，如果都是正值，则在元素的右下方，如果为负值，则为元素的左上方。
![4](http://7xns9g.com1.z0.glb.clouddn.com/shadow-css.png)

![5](http://7xns9g.com1.z0.glb.clouddn.com/shadow-html.png)

![6](http://7xns9g.com1.z0.glb.clouddn.com/text-shadow.png)
####处理空白符
CSS 是怎么处理我们写文档中添加的空白符的呢？一般来说，浏览器会选择合并空白符，忽略换行符。但是我们可以通过修改属性 white-space , 来控制这样的行为。此属性接受的属性值有 pre , nowrap , pre-wrap , pre-line 等。
当设置为 pre 时，该元素就像是 pre 标签一样，不合并空白符，也不忽略换行符。当设置为 nowrap 时，会强制元素内的文字等不换行，直到遇上 br 标签为止。而 pre-wrap 与 pre-line 刚好相对，pre-wrap 会保留空白符，正常换行，但是 pre-line 就会合并空白符，正常换行。
####文字方向
text-direction，接受两个属性值：ltr 与 rtl 。第一个元素为从左往右，第二个属性是从右往左。


------



作者     张翔
2015 年 10月 25日




