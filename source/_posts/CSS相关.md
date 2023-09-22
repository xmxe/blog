---
title: CSS相关
tags: 代码实战
categories: CSS
img: https://pic1.zhimg.com/v2-bdccba68ad2e37f6efc96a15f7897e2d.jpg

---


## 居中

```html
<!-- div居中,需要设置宽度-->
<div style="margin : 0 auto;width:80%"></div>
<!-- div里面的内容居中-->
<div style="margin : 0 auto;width:80%;text-align:center">
    <button></button>
</div>

<!-- 两个div设置间距 -->
<div style="margin:10px 0"></div>

<!--margin外间距是外边距，即盒子与盒子之间的距离，而padding是内边距，是盒子的边与盒子内部元素的距离。margin是用来隔开元素与元素的间距；padding是用来隔开元素与内容的间隔。margin是指从自身边框到另一个容器边框之间的距离，就是容器外距离。padding是指自身边框到自身内部另一个容器边框之间的距离，就是容器内距离。
例如两个文本框的距离使用margin，文本框的边框和文本内容之间的距离使用padding-->
<!-- 使用弹性容器 -->
<div class="container">
    <div class="centered">
    	<p>要居中的内容</p>
    </div>
</div>
```
```css
.container {
    display: flex; /* 设置为Flex容器 */
    justify-content: center; /* 水平居中 */
    align-items: center; /* 垂直居中 */
    height: 100vh; /* 设置容器高度 */
}
.centered {
  /* 可以给要居中的元素设置一些样式 */
}
```
> [42种前端常用布局方案总结※](https://mp.weixin.qq.com/s/5ZSMlbjcvaMksx4zakhgzA)
> [如何用一行CSS实现10种现代布局？](https://mp.weixin.qq.com/s/3yToJq5N8-8SXb0nPDL47g)

## n个元素等比例在一行展示

```html
<div class="image-container">
    <img src="image1.jpg" alt="Image 1" />
    <img src="image2.jpg" alt="Image 2" />
    <img src="image3.jpg" alt="Image 3" />
    <img src="image4.jpg" alt="Image 4" />
</div>
```
```css
.image-container {
    display: flex; /* 使用 Flexbox 布局 */
    flex-wrap: wrap; /* 等比例图片自动换行到下一行 */
    justify-content: space-between; /* 图片之间间距相等 */
}
img {
    width: 25%; /* 将每个图片的宽度设置为 25% 使其等比例放缩 */
    height: auto; /* 高度设置为 auto，使其自适应宽度 */
    margin-bottom: 1rem; /* 使用 margin 来设置图片之间的间距 */
    object-fit: cover; /* 自适应填充图片容器，保持图片比例 */
}
```
> 扩展**display:flex**

```css
div{
	display: flex
	flex-direction: row;/*属性决定主轴的方向（即项目的排列方向）*/
    /*
    - row（默认值）:主轴为水平方向，起点在左端。
    - row-reverse:主轴为水平方向，起点在右端。
    - column:主轴为垂直方向，起点在上沿。
    - column-reverse:主轴为垂直方向，起点在下沿。
    */
    flex-wrap: nowrap;/*属性决定了如果一条轴线排不下,如何换行*/
    /*
    - nowrap（默认）:不换行。
    - wrap:换行，第一行在上方。
    - wrap-reverse:换行，第一行在下方。
    */
    justify-content: flex-start;/*属性定义了项目在主轴上的对齐方式。*/
    /*
    - flex-start（默认值）:左对齐
    - flex-end:右对齐
    - center:居中
    - space-between:两端对齐，项目之间的间隔都相等。
    - space-around:每个项目两侧的间隔相等。
    */
    align-items: stretch;/*属性定义项目在交叉轴上如何对齐。简单来讲，假如我们将flex-direction设置为row，即主轴为行。align-items可以决定元素在列上的布局*/
    /*
    - flex-start:交叉轴的起点对齐，一行根据上边对齐。
    - flex-end:交叉轴的终点对齐，一行根据下边对齐。
    - center:交叉轴的中点对齐。
    - baseline:项目的第一行文字的基线对齐。
    - stretch（默认值）:如果项目未设置高度或设为auto，将占满整个容器的高度。
    */
    align-content: stretch;/*属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。简单来讲，假如我们将flex-direction设置为row，即主轴为行。align-content决定了出现很多行时，这些行之间怎么对齐。其有一下几个属性：*/
    /*
    - flex-start:与交叉轴的起点对齐，跟作文一样，一行一行紧挨着。
    - flex-end:与交叉轴的终点对齐，跟 flex-start类型，不过时从底部开始数。
    - center:与交叉轴的中点对齐，从中间向下向上扩散。
    - space-between:与交叉轴两端对齐，轴线之间的间隔平均分布。
    - space-around:每根轴线两侧的间隔都相等。
    - stretch（默认值）:轴线占满整个交叉轴。
    */
}

```

## 隐藏滚动条

```css
/**css隐藏滚动条**/
.class::-webkit-scrollbar{ width:0 !important /*display:none;*/}
::-webkit-scrollbar{width:0 !important /*display:none;*/}
```

## 渐变

```css
文字{
    /*
    linear-gradient是CSS中的一个渐变函数，用于在元素的背景中创建一个沿着一条直线方向的颜色渐变效果。linear-gradient函数的基本语法如下：background: linear-gradient(direction, color stop1, color stop2, ...);其中的参数解释如下：	
    - direction: 表示渐变的方向，可以是角度、关键字（top、bottom、left、right、to top left、to bottom right等）以及渐变轴线（由坐标(x1,y1)和(x2,y2)确定的一条直线，可使用两个坐标的百分比表示），也可以是任意组合;
    - color stop: 表示渐变的颜色及其所在的位置，可以定义多个颜色值，用逗号分隔。例如，color stop1可以表示位于渐变的起始点的颜色停止点，而color stop2则对应着终点的颜色停止点。

    CSS3渐变也支持透明度（transparent），可用于创建减弱变淡的效果。为了添加透明度，我们使用rgba()函数来定义颜色节点。rgba()函数中的最后一个参数可以是从0到1的值，它定义了颜色的透明度：0表示完全透明，1表示完全不透明。
    */
	background-image:-webkit-linear-gradient(bottom,#708a41,#8585a5,#4b8e9a);
    background-image: linear-gradient(top, hsla(0, 0%, 100%, .2) 1px, hsla(0, 0%, 100%, 0) 1px, hsla(0, 0%, 0%, .1) 100%);
    /*
     background-clip属性指定背景绘制区域。
       - border-box默认值。背景绘制在边框方框内（剪切成边框方框）。
       - padding-box背景绘制在衬距方框内（剪切成衬距方框）。
       - content-box背景绘制在内容方框内（剪切成内容方框）。
     */
    -webkit-background-clip:text;
    
    /**文字中填充颜色transparent:透明色**/
    -webkit-text-fill-color:transparent;
    
    /**text-stroke(文本边框)是text-stroke-width和text-stroke-color（边框填充颜色）两个属性的简写**/
    -webkit-text-stroke:6px transparent;
}
背景图{
    /*从下到上，从蓝色开始渐变、到高度40%位置是绿色渐变开始、最后以红色结束*/
    background-image: linear-gradient(0deg, blue, green 40%, red);
    /**创建一个从圆心开始，向四周渐变的径向渐变效果**/
    background: radial-gradient(circle at center, #ffafbd, #ffc3a0);
    /**创建一个从最外部向圆心渐变的径向渐变效果,farthest-corner关键字会将圆心设置在最远的角落，而不是默认的居中位置**/
    background: radial-gradient(circle farthest-corner at center, #ffafbd, #ffc3a0);
    /*
    radial-gradient是CSS中的一个渐变函数，用于在元素的背景中创建一个从一个中心向周围辐射的颜色渐变效果。
    radial-gradient函数的基本语法如下：background: radial-gradient(shape size at position, start-color, ..., last-color);其中的参数解释如下：
    shape: 表示渐变形状，可以是circle(默认)或ellipse；
    size: 表示渐变的大小
      - closest-side表示最近侧的边缘，
      - farthest-side表示最远侧的边缘，
      - closest-corner表示最近角落，
      - farthest-corner表示最远角落，
      - contain表示至少需要占满整个容器，
      - cover则表示覆盖整个容器；
    at position: 表示渐变的中心位置，可以是长度(像素或百分比)或关键字(center、top、bottom、left、right等)
    start-color和last-color: 表示渐变的起始颜色和结束颜色。可以定义多个颜色值，用逗号分隔。
    */
}

page {
    background: linear-gradient(-45deg, #ac7399, #a8a38e, #b6c24a);
    background-size: 500% 500%;
    animation: moiveAnimation 7s infinite;
}

@keyframes moiveAnimation {
    0% {
        background-position: 0% 50%
    }

    50% {
        background-position: 100% 50%
    }

    100% {
        background-position: 0% 50%
    }
}
```
> [超精美渐变色动态背景完整示例](https://blog.csdn.net/A757291228/article/details/124611342)

## 文字自动换行

```css
/*文章内容自动换行*/
#articleContent {
    /*
    - break-word:在长单词或URL地址内部进行换行。
    - normal:只在允许的断字点换行（浏览器保持默认处理）
    */
    word-wrap: break-word;
    
    /*
    - normal:使用浏览器默认的换行规则
    - break-all:允许在单词内换行允许在单词内换行。
    - keep-all:只能在半角空格或连字符处换行
    */
    word-break: break-all;
    
    /*
     - normal:忽略多余的空白，只保留一个空白（默认）
     - pre:保留空白(行为方式类似于html中的pre标签)
     - nowrap:文本不会换行，会在在同一行上继续，直到遇到br标签为止
     - pre-wrap:保留空白符序列，正常地进行换行
     - pre-line:合并空白符序列，保留换行符
     - inherit:从父元素继承white-space属性的值。
     */
    white-space: normal;
}
```
> [前端正确处理“文字溢出”的思路](https://mp.weixin.qq.com/s/7-NxE7K6QPSPLHEKW8ME8g)

## 动画@keyframes

```css
/**上下浮动动画**/
@keyframes float {
  /**0%是动画的开始时间，100%动画的结束时间。或者通过关键词"from"和"to"，等价于0%和100%。**/	
  100% {
   /*
    transform属性向元素应用2D或3D转换。该属性允许我们对元素进行旋转、缩放、移动或倾斜。应用多个属性使用空格如transform: rotate(45deg) scale(2) skew(10deg,5deg) translate(50px,90px);
    - rotate(xxdeg)(2D),rotateX()(3D),rotateY()(3D),rotateZ(180deg)：以中心为基点，deg表示旋转的角度，为负数时表示逆时针旋转
    - translate(x,y)，translateX(x)，translateY(y)：以中心为基点按照设定的x,y参数值,对元素进行进行平移。
    - scale(x,y)，scaleX(X)，scaleY(Y)：缩放基数为1，如果其值大于1元素就放大，反之其值小于1为缩小。缩放后不影响文档流,不改变原有布局,元素还是会占用,和relative定位一样,或者可以考虑zoom属性
    - skew(x,y)，skewX(x)，skewY(y)：以中心为基点，第一个参数是水平方向扭曲角度，第二个参数是垂直方向扭曲角度。
    */
   transform: translateY(20px);
   
   /**box-shadow属性可以设置一个或多个下拉阴影的框。**/
   box-shadow: 0 40px 10px -18px hsla(0, 0%, 0%, .2), 0 40px 16px -12px hsla(0, 0%, 0%, .2)
   transform-origin: right;/**(x,y)来改变元素基点**/
  }
}

/**隐藏/显示动画**/
@keyframes move{
    0%{
        transform: translate(100px, 0);
        opacity: 0;
    }
    50%{
        transform: translate(50px, 0);
        opacity: 0.5;
    }
    100%{
        transform: translate(0, 0);
        opacity: 1;
	}
}

/**使用**/
div{
    box-shadow: 0 60px 12px -18px hsla(0, 0%, 0%, .1), 0 60px 20px -12px hsla(0, 0%, 0%, .1);
    /*
     animation: name duration timing-function delay iteration-count direction;
     animation-name:规定需要绑定到选择器的keyframe名称。
     animation-duration:规定完成动画所花费的时间，以秒或毫秒计
     animation-timing-function:规定动画的速度曲线。
       - linear:动画从头到尾的速度是相同的
       - ease:默认。动画以低速开始，然后加快,在结束前变慢
       - ease-in:动画以低速开始
       - ease-out:动画以低速结束
       - ease-in-out:动画以低速开始和结束
       - cubic-bezier(n,n,n,n):在cubic-bezier函数中自己的值
     animation-delay:规定在动画开始之前的延迟。
     animation-iteration-count:规定动画应该播放的次数。默认为1次，可以填写数字
     animation-direction:规定是否应该轮流反向播放动画。如果animation-direction值是“alternate”，则动画会在奇数次数（1、3、5等等）正常播放，而在偶数次数（2、4、6等等）向后播放。如果把动画设置为只播放一次，则该属性没有效果
    */
    animation: float 1s infinite ease-in-out alternate
    /**animation: move .4s linear 1 normal**/
}
div:hover{
    /*
     animation-play-state属性规定动画正在运行还是暂停。只有两个属性可以设置：
       - paused:规定动画已暂停
       - running:规定动画正在播放
     */
	animation-play-state: paused
     
    /*
     animation-fill-mode属性规定动画在播放之前或之后，其动画效果是否可见。(规定当动画不播放时（当动画完成时或当动画有一个延迟为开始播放时）要用到的元素样式)
       - none表示等待期和完成期，元素样式都为初始状态样式，不受动画定义（@keyframes）的影响
       - both表示等待期样式为第一帧样式，完成期保持最后一帧样式
       - backwards表示等待期为第一帧样式，完成期跳转为初始样式
       - forwards表示等待期保持初始样式，完成期间保持最后一帧样式
     */
     animation-fill-mode:none;
}

/**鼠标滑过翻转**/
img:hover {
  animation: fadenum 2s;
}
@keyframes fadenum {
   100%{ transform:rotate(360deg);}
}

```

## transition

```css
/**鼠标滑动图标旋转**/
div{
    -webkit-transform: rotate(3600deg) !important;
    -moz-transform: rotate(360deg) !important;
    -o-transform: rotate(360deg) !important;
    -ms-transform: rotate(360deg) !important;
    transform: rotate(360deg) !important;
    
    /*
     transition属性是一个简写属性，用于设置四个过渡属性.
       - transition-property:规定设置过渡效果的CSS属性的名称。none没有属性会获得过渡效果。all所有属性都将获得过渡效果。property定义应用过渡效果的CSS属性名称列表，列表以逗号分隔。
       - transition-duration:规定完成过渡效果需要多少秒或毫秒。
       - transition-timing-function:规定速度效果的速度曲线。
         -- inear规定以相同速度开始至结束的过渡效果（等于cubic-bezier(0,0,1,1)）
         -- ease规定慢速开始，然后变快，然后慢速结束的过渡效果（cubic-bezier(0.25,0.1,0.25,1)）
         -- ease-in规定以慢速开始的过渡效果（等于cubic-bezier(0.42,0,1,1)）
         -- ease-out规定以慢速结束的过渡效果（等于cubic-bezier(0,0,0.58,1)）
         -- ease-in-out规定以慢速开始和结束的过渡效果（等于cubic-bezier(0.42,0,0.58,1)）。
         -- cubic-bezier(n,n,n,n)在cubic-bezier函数中定义自己的值。可能的值是0至1之间的数值。
       - transition-delay:定义过渡效果何时开始。
     */
    -webkit-transition: all .7s;
    -moz-transition: all .7s;
    -o-transition: all .7s;
    transition: all .7s;
}

a{
    /*光标样式*/
    cursor: pointer;
}
```

## 网站变灰

```css
/*在想要变灰的控件里加入此属性即可,如果想要网站整体变灰在html{}里面加入即可*/
.gray {
    -webkit-filter: grayscale(100%);
    -webkit-filter:grayscale(1);
    -moz-filter: grayscale(100%);
    -ms-filter: grayscale(100%);
    -o-filter: grayscale(100%);
    filter: grayscale(100%);
    filter: progid:DXImageTransform.Microsoft.BasicImage(grayscale=1);
    filter:url("data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\'><filter id=\'grayscale\'><feColorMatrix type=\'matrix\' values=\'0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0 0 0 1 0\'/></filter></svg>#grayscale");
}
```


## css伪类

```css
:link /*应用于未被访问过的链接*/
:hover /*应用于鼠标悬停到的元素*/
:active /*应用于被激活的元素*/
:visited /*应用于被访问过的链接，与:link互斥。*/
:focus /*应用于拥有键盘输入焦点的元素。*/

:first-child /*选择某个元素的第一个子元素*/
:last-child /*选择某个元素的最后一个子元素*/
:nth-child(n) /*匹配属于其父元素的第n个子元素，不论元素的类型*/
:nth-last-child() /*从这个元素的最后一个子元素开始算,选匹配属于其父元素的第n个子元素，不论元素的类型*/
:nth-of-type() /*规定属于其父元素的第n个指定元素*/
:nth-last-of-type() /*从元素的最后一个开始计算,规定属于其父元素的指定元素*/
:first-of-type /*选择一个上级元素下的第一个同类子元素*/
:last-of-type /*选择一个上级元素的最后一个同类子元素*/
:only-child /*选择它的父元素的唯一一个子元素*/
:only-of-type /*选择一个元素是它的上级元素的唯一一个相同类型的子元素*/
:checked /*匹配被选中的input元素，这个input元素包括radio和checkbox。*/
:empty /*选择的元素里面没有任何内容。*/
:disabled /*匹配禁用的表单元素。*/
:enabled /*匹配没有设置disabled属性的表单元素。*/
:valid /*匹配条件验证正确的表单元素。*/
:in-range /*选择具有指定范围内的值的<input>元素。*/
:invalid input:invalid /*选择所有具有无效值的<input>元素。*/
:optional input:optional /*选择不带"required"属性的<input>元素。*/
:lang(language)	p:lang(it) /*选择每个lang属性值以"it"开头的<p>元素。*/
:focus /*选择获得焦点的<input>元素。*/
:not(selector) :not(p) /*选择每个非<p>元素的元素。*/
:out-of-range input:out-of-range /*选择值在指定范围之外的<input>元素。*/
:read-only input:read-only /*选择指定了"readonly"属性的<input>元素。*/
:read-write	input:read-write /*选择不带"readonly"属性的<input>元素。*/
:required input:required /*选择指定了"required"属性的<input>元素。*/
:root root /*选择元素的根元素。*/
:target	#news:target /*选择当前活动的#news元素（单击包含该锚名称的URL）*/
```

## css伪元素
```css
::after /*例: p::after	在每个<p>元素之后插入内容。*/
::before /*例: p::before 在每个<p>元素之前插入内容。*/
::first-letter /*例: p::first-letter 选择每个<p>元素的首字母。*/
::first-line /*例: p::first-line 选择每个<p>元素的首行。*/
::selection /*例: p::selection 选择用户选择的元素部分。*/
::marker /*例: li::marker {content:'>';} 把li前面的'•'变成'>'*/
```
> [CSS伪元素](https://www.w3school.com.cn/css/css_pseudo_elements.asp)

## 21个超实用的CSS技巧

### 文档布局
仅用两行CSS，就可以创建响应式文档样式布局。

```css
.parent{
  display: grid;
  grid-template-columns: minmax(150px, 25%) 1fr;
}
```

### 自定义光标

```css
html{
  cursor:url('no.png'), auto;
}
```

### 用图像填充文本

```css
h1{
  background-image: url('images/flower.jpg');
  background-clip: text;
  color: transparent;
  background-color: white;
}
```
> 注意：使用此技术时，请始终指定background-color，因为如果由于某种原因图像未加载，可以将其用作回退值。

### 为文本添加描边效果

使用text-stroke属性可以使文本更清晰可见，因为会向文本添加描边笔触或轮廓。

```css
/* Apply a 5px wide crimson text stroke to h1 elements */
h1 {
  -webkit-text-stroke: 5px crimson;
  text-stroke: 5px crimson;
}
```

### :paused伪类

使用:paused选择器在暂停状态下设置媒体元素的样式，与:paused类似的还有:playing。

```css
/* 目前只支持Safari浏览器 */
video:paused {
  opacity: 0.6;
}
```

### 强调文本

使用text-emphasis属性将强调标记应用于文本元素。你可以指定任意字符串（包括表情符号）作为值。

```css
h1 {
  text-emphasis: "⏰";
}
```

### 首字母下沉

避免不必要的span，改用伪元素来设置内容的样式，同样，与:first-letter伪元素类似的还有:first-line伪元素。

```css
h1::first-letter{
  font-size: 2rem;
  color:#ff8A00;
}
```

### 变量回退值

```css
:root {
  --orange: orange;
  --coral: coral;
}

h1 {
  color: var(--black, crimson);
}
```

### 更改书写模式

你可以使用书写模式属性来指定文本在网站上的布局方式，垂直或水平布局。

```css
<h1>Cakes & Bakes</h1>
h1 {
  writing-mode: sideways-lr;
}
```

### 彩虹动画

为元素创建连续循环的颜色动画以吸引用户的注意力。

```css
button{
  animation: rainbow-animation 200ms linear infinite;
}

@keyframes rainbow-animation {
  to{
    filter: hue-rotate(0deg);
  }
 from{
    filter: hue-rotate(360deg);
  }
}
```

### 悬停缩放

```css
/* 定义图片容器的高度和宽度，以及设置图元溢出时隐藏 */
.img-container {
  height: 250px; width: 250px; overflow: hidden;
 }

/* 让图片填充整个容器 */

.img-container img {
  height: 100%; width: 100%; object-fit: cover; 
  transition: transform 200m ease-in;
 }

 img:hover{
  transform: scale(1.2);
 }
```

### 属性选择器

使用属性选择器根据属性选择HTML元素。

```css
<a href="">HTML</a>
<a>CSS</a>
<a href="">JavaScript</a>
/* 为每个带href的a元素设置颜色 */

a[href] {
  color: crimson;
}
```

### 剪切元素

使用clip-path属性创建有趣的视觉效果，例如将元素剪裁为自定义的三角形或六边形形状。

```css
div {
  height: 150px;
  width: 150px;
  background-color: crimson;
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}
```

### 检测属性支持

使用CSS @support rule直接在CSS中检测对CSS功能的支持。

```css
@supports (accent-color: #74992e) {
/* 如果浏览器支持，则以下代码可以运行 */
  blockquote {
    color: crimson;
  }

}
```

### CSS嵌套

CSS工作组一直在研究如何将嵌套添加到CSS中。通过嵌套，我们将能够编写更直观、更有条理、更高效的CSS。

```css
<header class="header">
  <p class="text">Lorem ipsum, dolor</p>
</header>
/* 你可以在Safari浏览器中尝试使用CSS嵌套*/
.header{
  background-color: salmon;
  .text{
    font-size: 18px;
  }
}
```

### clamp函数

clamp()函数可用于响应式和流畅的排版。

```css
/* Syntax: clamp(minimum, preferred, maximum) */
h1{
  font-size: clamp(2.25rem,6vw,4rem);
}
```

### 设置可选字段的样式

你可以使用:optional伪类设置表单字段的样式，例如输入框、下拉框和文本框，这些字段上没有必要属性。

```css
*:optional{
  background-color: green;
}
```

### 字间距属性

使用word-spacing属性指定两个单词之间的空格长度。

```css
p {
  word-spacing: 1.245rem;
}
```

### 创建渐变阴影

创建渐变阴影以提供独特的用户体验。

```css
:root{
  --gradient: linear-gradient(to bottom right, crimson, coral);
}

div {
  height: 200px;
  width: 200px;
  background-image: var(--gradient);
  border-radius: 1rem;
  position: relative;
}

div::after {
  content: "";
  position: absolute;
  inset: 0;
  background-image: var(--gradient);
  border-radius: inherit;
  filter: blur(25px) brightness(1.5);
  transform: translateY(15%) scale(0.95);
  z-index: -1;
}
```

### 更改标题位置

使用caption-side属性将表格标题放在表格的指定一侧。

### 创建文本列

使用column属性为文本元素制作漂亮的列布局。

```css
p{
  column-count: 3;
  column-gap: 4.45rem;
  column-rule: 2px dotted crimson;
}
```

## 细数那些惊艳一时的CSS属性

### position: sticky

标题在滚动的时候，会一直贴着最顶上。这种场景实际上很多：比如表格的标题栏、网站的导航栏、手机通讯录的人名首字母标题等等。

css部分
```css
.container {
    background-color: oldlace;
    height: 200px;
    width: 140px;
    overflow: auto;
}
.container div {
    height: 20px;
    background-color: aqua;
    border: 1px solid;
}
.container .header {
    position: sticky;
    top: 0;
    background-color: rgb(187, 153, 153);
}
```
html部分
```html
<div class="container">
  <div class="header">Header</div>
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

### :empty选择器

平时开发的时候数据都是通过请求接口获取的，也会存在接口没有数据的情况。这个时候正常的做法是给用户一个提示，让用户知道当前不是出bug了，而是确实没有数据。一般的做法是我们人为的判断当前数据返回列表的长度是否为0，如果为0则显示一个“暂无数据”给用户，反之则隐藏该提示。写过Vue的小伙伴是不是经常这么做：

```html
<div>
    <template v-if="datas.length">
        <div v-for="data in datas"></div>
    </template>
    <template v-else>
        <div>暂无数据</div>
    </template>
</div>
```
但是有了:empty这个选择器后，你大可以把这个活交给CSS来干。

```css
.container {
    height: 400px;
    width: 600px;
    background-color: antiquewhite;
    display: flex;
    justify-content: center;
    align-items: center;
}
.container:empty::after {
    content: "暂无数据";
}
```

### gap

日常开发中，都有用过padding和margin吧，margin一般用做边距，让两个元素隔开一点距离，但是对于一些场景下，我们很难通过计算得到一个除的尽的值，比如100px我要让3个元素等分，且每个元素隔开10px，这就很尴尬了。没关系！我们可以用gap属性，gap属性它适用于Grid布局、Flex布局以及多列布局，并不一定只是Grid布局中可以使用。比如我们要让每个元素之间隔开20px，那么使用gap我们可以这样：

```css
div{
    display: flex | grid;
    gap: 20px;
}
```

### background-clip: text

大家平时background-clip是不是都用来做一些裁切效果？你知道它还有个属性值是text吗？background-clip:text用来做带背景的文字效果，相信大家平时浏览一些网站的时候都会看到类似的实现，实际上通过CSS我们也能做到这种效果，可不要傻傻的以为都是用制图工具做的。

### user-select

网页和APP有个不同点是，网页上的字是可以通过光标选中的，而APP的不行。有的小伙伴可能会疑惑：那我网页上也用不着这个属性啊？非也非也，我们知道现在很多新的技术产生，可以在APP上嵌套webview或者是网页，比如Electron做的桌面端应用，大家没见过哪个桌面端应用是可以光标选中的吧？而user-select属性可以禁用光标选中，让网页看着和移动端一样。

### :invalid伪类

:invalid表示任意内容未通过验证的input或其他form元素。什么意思呢？举个例子。这是一个表单。

```html
<form>
  <label for="url_input">Enter a URL:</label>
  <input type="url" id="url_input" />
  <br />
  <br />
  <label for="email_input">Enter an email address:</label>
  <input type="email" id="email_input" required/>
</form>
```
我们的需求是让input当值有效时，元素颜色为绿色，无效时为红色。

```css
input:invalid {
  background-color: #ffdddd;
}
form:invalid {
  border: 5px solid #ffdddd;
}
input:valid {
  background-color: #ddffdd;
}
form:valid {
  border: 5px solid #ddffdd;
}
input:required {
  border-color: #800000;
  border-width: 3px;
}
input:required:invalid {
  border-color: #C00000;
}
```

有了:invalid属性后，我们就可以不用JS也能实现校验提示的效果了。

### :focus-within伪类

:focus-within表示一个元素获得焦点，或该元素的后代元素获得焦点，就会匹配上。

CSS
```css
form {
    border: 1px solid;
    width: 400px;
    height: 300px;
    display: flex;
    justify-content: center;
    align-items: center;
}
form:focus-within {
    box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.3);
    background-color: beige;
}
```

HTML
```html
<form>
  <input type="text" id="ipt" placeholder="请输入..." />
</form>
```
可以根据子元素的状态来改变父元素的样式，方便的很。也能玩出不少花样来。

### mix-blend-mode:difference

mix-blend-mode:difference属性描述了元素的内容应该与元素的直系父元素的内容和元素的背景如何混合。其中，difference表示差值。

```css
.mode {
    display: flex;
    justify-content: center;
    align-items: center;
    mix-blend-mode:difference;
}
.dark {
    position: relative;
    left: 6px;
    height: 24px;
    width: 24px;
    background-color: grey;
    border-radius: 50%;
}
.light {
    mix-blend-mode:difference;
    position: relative;
    left: -6px;
    height: 16px;
    width: 16px;
    border-radius: 50%;
    border: 4px solid grey;
}
```

## 超酷的纯CSS Loading效果

为保证运行正常，咱先规定下：

```css
* {
  box-sizing: border-box;
}
```

### 平滑加载

```css
<div class="progress-1"></div>
.progress-1 {
    width:120px;
    height:20px;
    background: linear-gradient(#000 0 0) 0/0% no-repeat #ddd;
    animation:p1 2s infinite linear;
}
@keyframes p1 {
    100% {background-size:100%}
}
```

1. linear-gradient(#000 0 0)你可以理解为linear-gradient(#000 0 100%)，如果还不熟悉，复制linear-gradient(#000 0 50%, #f00 50% 0)，替换原先的部分跑一下。觉得linear-gradient(#000 0 0)别扭的话，直接写#000即可。
2. 0/0%是background-position: 0;/background-size: 0;的简写。

### 按步加载

```css
<div class="progress-2"></div>
.progress-2 {
    width:120px;
    height:20px;
    border-radius: 20px;
    background: linear-gradient(orange 0 0) 0/0% no-repeat lightblue;
    animation:p2 2s infinite steps(10);
}
@keyframes p2 {
    100% {background-size:110%}
}
```

1. steps(10)是step(10, end)的简写，指明刚开始没有，所以有**第2点**的处理
2. 100% {background-size:110%}添加多一个step的百分比，上面的step是10，所以是`100% + (1/10)*100% = 110%`

### 条纹加载

```css
<div class="progress-3"></div>
.progress-3 {
    width:120px;
    height:20px;
    border-radius: 20px;
    background: repeating-linear-gradient(135deg,#f03355 0 10px,#ffa516 0 20px) 0/0% no-repeat,repeating-linear-gradient(135deg,#ddd 0 10px,#eee 0 20px) 0/100%;
    animation:p3 2s infinite;
}
@keyframes p3 {
    100% {background-size:100%}
}
```
repeating-linear-gradient(135deg,#ddd 0 10px,#eee 0 20px) 0/100%;画出灰色的斑马线条纹，repeating-linear-gradient(135deg,#f03355 0 10px,#ffa516 0 20px) 0/0% no-repeat则是进度条加载的条纹。

### 虚线加载

```css
<div class="progress-4"></div>
.progress-4 {
    width:120px;
    height:20px;
    -webkit-mask:linear-gradient(90deg,#000 70%,#0000 0) 0/20%;
    background:linear-gradient(#000 0 0) 0/0% no-repeat #ddd;
    animation:p4 2s infinite steps(6);
}
@keyframes p4 {
    100% {background-size:120%}
}
```
-webkit-mask默认有值repeat，不然遮罩不会有五个。

### 电池加载

```css
<div class="progress-5"></div>
.progress-5 {
    width:80px;
    height:40px;
    border:2px solid #000;
    padding:3px;
    background: repeating-linear-gradient(90deg,#000 0 10px,#0000 0 16px) 0/0% no-repeat content-box content-box;
    position: relative;
    animation:p5 2s infinite steps(6);
}
.progress-5::before {
    content:"";
    position: absolute;
    top: 50%;
    left:100%;
    transform: translateY(-50%);
    width:10px;
    height: 10px;
    border: 2px solid #000;
}
@keyframes p5 {
    100% {background-size:120%}
}
```
原作者对.progress-5::before伪元素实现如下：

```css
.progress-5::before {
    content:"";
    position: absolute;
    top:-2px;
    bottom:-2px;
    left:100%;
    width:10px;
    background: linear-gradient(#0000   calc(50% - 7px),#000 0 calc(50% - 5px),#0000 0 calc(50% + 5px),#000 0 calc(50% + 7px),#0000 0) left /100% 100%,linear-gradient(#000 calc(50% - 5px),#0000 0 calc(50% + 5px),#000 0) left /2px 100%,linear-gradient(#0000 calc(50% - 5px),#000 0 calc(50% + 5px),#0000 0) right/2px 100%;
    background-repeat:no-repeat;
}
```

> #0000是透明，同等transparent

### 内嵌加载

```css
<div class="progress-6"></div>
.progress-6 {
    width:120px;
    height:22px;
    border-radius: 20px;
    color: #514b82;
    border:2px solid;
    position: relative;
}
.progress-6::before {
    content:"";
    position: absolute;
    margin:2px;
    inset:0 100% 0 0;
    border-radius: inherit;
    background: #514b82;
    animation:p6 2s infinite;
}
@keyframes p6 {
    100% {inset:0}
}
```
inset:0 100% 0 0;右边内缩100%，所以在keyframes部分需要将inset设置为0。

### 珠链加载

```css
<div class="progress-7"></div>
.progress-7 {
    width:120px;
    height:24px;
    -webkit-mask:radial-gradient(circle closest-side,#000 94%,#0000) 0 0/25% 100%,linear-gradient(#000 0 0) center/calc(100% - 12px) calc(100% - 12px) no-repeat;
    background:linear-gradient(#25b09b 0 0) 0/0% no-repeat #ddd;
    animation:p7 2s infinite linear;
}
@keyframes p7 {
    100% {background-size:100%}
}
```

遮罩-webkit-mask中radial-gradient是将宽度四等份，每份以最小closest-side的边为直径画圆。

### 斑马线加载

```css
<div class="progress-8"></div>
.progress-8 {
    width:60px;
    height:60px;
    border-radius: 50%;
    -webkit-mask:linear-gradient(0deg,#000 55%,#0000 0) bottom/100% 18.18%;
    background: linear-gradient(#f03355 0 0) bottom/100% 0% no-repeat #ddd;
    animation:p8 2s infinite steps(7);
}
@keyframes p8 {
    100% {background-size:100% 115%}
}
```

对linear-gradient描绘的角度做调整，再加上蒙版。

### 水柱加载

```css
<div class="progress-9"></div>
.progress-9 {
    --r1: 154%;
    --r2: 68.5%;
    width:60px;
    height:60px;
    border-radius: 50%;
    background:
        radial-gradient(var(--r1) var(--r2) at top ,#0000 79.5%,#269af2 80%) center left,
        radial-gradient(var(--r1) var(--r2) at bottom,#269af2 79.5%,#0000 80%) center center,
        radial-gradient(var(--r1) var(--r2) at top ,#0000 79.5%,#269af2 80%) center right,
        #ccc;
    background-size: 50.5% 220%;
    background-position: -100% 0%,0% 0%,100% 0%;
    background-repeat:no-repeat;
    animation:p9 2s infinite linear;
}
@keyframes p9 {
    33%  {background-position: 0% 33% ,100% 33% ,200% 33% }
    66%  {background-position: -100% 66%,0% 66% ,100% 66% }
    100% {background-position: 0% 100%,100% 100%,200% 100%}
}
```
radial-gradient画出水平面的波动，就三个圆。var(--r1)直接调用定义好的属性值。

### 信号加载

```css
<div class="progress-10"></div>
.progress-10 {
    width:120px;
    height:60px;
    border-radius:200px 200px 0 0;
    -webkit-mask:repeating-radial-gradient(farthest-side at bottom ,#0000 0,#000 1px 12%,#0000 calc(12% + 1px) 20%);
    background: radial-gradient(farthest-side at bottom,#514b82 0 95%,#0000 0) bottom/0% 0% no-repeat #ddd;
    animation:p10 2s infinite steps(6);
}
@keyframes p10 {
    100% {background-size:120% 120%}
}
```

用repeating-radial-gradient方法画出环状的蒙版遮罩。radial-gradient从底部向上圆形渐变填充。

### 3d加载

html部分
```html
<body>
<div class="pl">
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__dot"></div>
	<div class="pl__text">Loading…</div>
</div>
</body>
```
css部分
```css
* {
  border: 0;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --bg: #454954;
  --fg: #e3e4e8;
  --fg-t: rgba(227, 228, 232, 0.5);
  --primary1: #255ff4;
  --primary2: #5583f6;
  --trans-dur: 0.3s;
  font-size: calc(16px + (20 - 16) * (100vw - 320px) / (1280 - 320));
}

body {
  background-color: var(--bg);
  background-image: linear-gradient(135deg, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.2));
  color: var(--fg);
  font: 1em/1.5 "Varela Round", Helvetica, sans-serif;
  height: 100vh;
  min-height: 360px;
  display: grid;
  place-items: center;
  transition: background-color var(--trans-dur), color var(--trans-dur);
}

.pl {
  box-shadow: 2em 0 2em rgba(0, 0, 0, 0.2) inset, -2em 0 2em rgba(255, 255, 255, 0.1) inset;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  transform: rotateX(30deg) rotateZ(45deg);
  width: 15em;
  height: 15em;
}
.pl, .pl__dot {
  border-radius: 50%;
}
.pl__dot {
  animation-name: shadow;
  box-shadow: 0.1em 0.1em 0 0.1em black, 0.3em 0 0.3em rgba(0, 0, 0, 0.5);
  top: calc(50% - 0.75em);
  left: calc(50% - 0.75em);
  width: 1.5em;
  height: 1.5em;
}
.pl__dot, .pl__dot:before, .pl__dot:after {
  animation-duration: 2s;
  animation-iteration-count: infinite;
  position: absolute;
}
.pl__dot:before, .pl__dot:after {
  content: "";
  display: block;
  left: 0;
  width: inherit;
  transition: background-color var(--trans-dur);
}
.pl__dot:before {
  animation-name: pushInOut1;
  background-color: var(--bg);
  border-radius: inherit;
  box-shadow: 0.05em 0 0.1em rgba(255, 255, 255, 0.2) inset;
  height: inherit;
  z-index: 1;
}
.pl__dot:after {
  animation-name: pushInOut2;
  background-color: var(--primary1);
  border-radius: 0.75em;
  box-shadow: 0.1em 0.3em 0.2em rgba(255, 255, 255, 0.4) inset, 0 -0.4em 0.2em #2e3138 inset, 0 -1em 0.25em rgba(0, 0, 0, 0.3) inset;
  bottom: 0;
  clip-path: polygon(0 75%, 100% 75%, 100% 100%, 0 100%);
  height: 3em;
  transform: rotate(-45deg);
  transform-origin: 50% 2.25em;
}
.pl__dot:nth-child(1) {
  transform: rotate(0deg) translateX(5em) rotate(0deg);
  z-index: 5;
}
.pl__dot:nth-child(1), .pl__dot:nth-child(1):before, .pl__dot:nth-child(1):after {
  animation-delay: 0s;
}
.pl__dot:nth-child(2) {
  transform: rotate(-30deg) translateX(5em) rotate(30deg);
  z-index: 4;
}
.pl__dot:nth-child(2), .pl__dot:nth-child(2):before, .pl__dot:nth-child(2):after {
  animation-delay: -0.1666666667s;
}
.pl__dot:nth-child(3) {
  transform: rotate(-60deg) translateX(5em) rotate(60deg);
  z-index: 3;
}
.pl__dot:nth-child(3), .pl__dot:nth-child(3):before, .pl__dot:nth-child(3):after {
  animation-delay: -0.3333333333s;
}
.pl__dot:nth-child(4) {
  transform: rotate(-90deg) translateX(5em) rotate(90deg);
  z-index: 2;
}
.pl__dot:nth-child(4), .pl__dot:nth-child(4):before, .pl__dot:nth-child(4):after {
  animation-delay: -0.5s;
}
.pl__dot:nth-child(5) {
  transform: rotate(-120deg) translateX(5em) rotate(120deg);
  z-index: 1;
}
.pl__dot:nth-child(5), .pl__dot:nth-child(5):before, .pl__dot:nth-child(5):after {
  animation-delay: -0.6666666667s;
}
.pl__dot:nth-child(6) {
  transform: rotate(-150deg) translateX(5em) rotate(150deg);
  z-index: 1;
}
.pl__dot:nth-child(6), .pl__dot:nth-child(6):before, .pl__dot:nth-child(6):after {
  animation-delay: -0.8333333333s;
}
.pl__dot:nth-child(7) {
  transform: rotate(-180deg) translateX(5em) rotate(180deg);
  z-index: 2;
}
.pl__dot:nth-child(7), .pl__dot:nth-child(7):before, .pl__dot:nth-child(7):after {
  animation-delay: -1s;
}
.pl__dot:nth-child(8) {
  transform: rotate(-210deg) translateX(5em) rotate(210deg);
  z-index: 3;
}
.pl__dot:nth-child(8), .pl__dot:nth-child(8):before, .pl__dot:nth-child(8):after {
  animation-delay: -1.1666666667s;
}
.pl__dot:nth-child(9) {
  transform: rotate(-240deg) translateX(5em) rotate(240deg);
  z-index: 4;
}
.pl__dot:nth-child(9), .pl__dot:nth-child(9):before, .pl__dot:nth-child(9):after {
  animation-delay: -1.3333333333s;
}
.pl__dot:nth-child(10) {
  transform: rotate(-270deg) translateX(5em) rotate(270deg);
  z-index: 5;
}
.pl__dot:nth-child(10), .pl__dot:nth-child(10):before, .pl__dot:nth-child(10):after {
  animation-delay: -1.5s;
}
.pl__dot:nth-child(11) {
  transform: rotate(-300deg) translateX(5em) rotate(300deg);
  z-index: 6;
}
.pl__dot:nth-child(11), .pl__dot:nth-child(11):before, .pl__dot:nth-child(11):after {
  animation-delay: -1.6666666667s;
}
.pl__dot:nth-child(12) {
  transform: rotate(-330deg) translateX(5em) rotate(330deg);
  z-index: 6;
}
.pl__dot:nth-child(12), .pl__dot:nth-child(12):before, .pl__dot:nth-child(12):after {
  animation-delay: -1.8333333333s;
}
.pl__text {
  font-size: 0.75em;
  max-width: 5rem;
  position: relative;
  text-shadow: 0 0 0.1em var(--fg-t);
  transform: rotateZ(-45deg);
}
/* Animations */
@keyframes shadow {
  from {
    animation-timing-function: ease-in;
    box-shadow: 0.1em 0.1em 0 0.1em black, 0.3em 0 0.3em rgba(0, 0, 0, 0.3);
  }
  25% {
    animation-timing-function: ease-out;
    box-shadow: 0.1em 0.1em 0 0.1em black, 0.8em 0 0.8em rgba(0, 0, 0, 0.5);
  }
  50%, to {
    box-shadow: 0.1em 0.1em 0 0.1em black, 0.3em 0 0.3em rgba(0, 0, 0, 0.3);
  }
}
@keyframes pushInOut1 {
  from {
    animation-timing-function: ease-in;
    background-color: var(--bg);
    transform: translate(0, 0);
  }
  25% {
    animation-timing-function: ease-out;
    background-color: var(--primary2);
    transform: translate(-71%, -71%);
  }
  50%, to {
    background-color: var(--bg);
    transform: translate(0, 0);
  }
}
@keyframes pushInOut2 {
  from {
    animation-timing-function: ease-in;
    background-color: var(--bg);
    clip-path: polygon(0 75%, 100% 75%, 100% 100%, 0 100%);
  }
  25% {
    animation-timing-function: ease-out;
    background-color: var(--primary1);
    clip-path: polygon(0 25%, 100% 25%, 100% 100%, 0 100%);
  }
  50%, to {
    background-color: var(--bg);
    clip-path: polygon(0 75%, 100% 75%, 100% 100%, 0 100%);
  }
}
```

### 文字旋转扭曲

整体实现思路如下：
1. 构建n个同心的div矩形。每个矩形的尺寸逐渐递减，每个矩形的旋转延迟时间也逐渐递减，实现旋转时的扭曲效果。并且每个div矩形使用伪类::after来显示文本内容。
2. 设置滤镜filter。用来设置文本的模糊效果以及明暗对比度。
3. 设置animation旋转动画。让矩形在指定的周期内做360度旋转。

html部分

```html
<body>
<div class="twist" id="textArea"></div>
<script type="text/javascript">
    var text = "CSS3_Rotate";
    var size = 860;
    var delay = 2;
    // 首先我们通过JS往容器中生成n个同心div矩形，这里我们生成42个。
    var areaCount = 42;
    var container = document.getElementById("textArea");
    for (var i = 0; i < areaCount; i++) {
        addArea(
            size - (size / areaCount) * i,
            delay - (delay / areaCount) * i,
            container
        );
    }
    function addArea(areaSize, areaDelay, parentContainer) {
        var area = document.createElement("div");
        area.setAttribute(
            "style",
            "--size:" + areaSize + "px;" + "--delay:" + areaDelay + "s;"
        );
        area.setAttribute(
            "title",
            text
        );
        parentContainer.appendChild(area);
    }
</script>
</body>
```

css部分

```css
*, *::before, *::after {
  padding: 0;
  margin: 0 auto;
  box-sizing: border-box;
}

body {
  font-family: "Lobster", cursive;
  background-color: #000;
  color: #fff;
  min-height: 100vh;
  display: grid;
  place-items: center;
  overflow:hidden;
}

.twist {
  position: relative;
  color: aquamarine;
  font-size: 78px;
  /*blur(2px)表示元素外边框2px外使用高斯模糊效果,contrast(4)设置对比度，默认是1。可以使用百分比也可以使用小数表示*/
  filter: blur(2px) contrast(4);
}
.twist > div {
  position: absolute;
  width: var(--size);
  height: var(--size);
  background-color: #000;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  overflow: hidden;
  -webkit-animation: twist 4s var(--delay) infinite ease-in-out;
          animation: twist 4s var(--delay) infinite ease-in-out;
}
.twist > div::after {
  content: attr(title);
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
@-webkit-keyframes twist {
  0% {
    transform: translate(-50%, -50%) rotateZ(0deg);
  }
  50%, 100% {
    transform: translate(-50%, -50%) rotateZ(360deg);
  }
}
/*旋转和淡入淡出的动画效果，每个div元素延迟旋转的时间在之前创建同心矩形的时候已经设置了*/
@keyframes twist {
  0% {
    transform: translate(-50%, -50%) rotateZ(0deg);
  }
  50%, 100% {
    transform: translate(-50%, -50%) rotateZ(360deg);
  }
}
```


## 相关文章

- [CSS@规则](http://c.biancheng.net/css3/at-rule.html)
- [64个超级有用的CSS资源](https://mp.weixin.qq.com/s/xYUjsf4IKYORqOLNNnEHlA)
- [HTML、CSS demo](https://www.html5tricks.com/page/26)
- [10个值得收藏的CSS资源](https://mp.weixin.qq.com/s/Sn22fn_70QHWD753eFZHiQ)
- [13个让你值得一试的CSS技巧](https://mp.weixin.qq.com/s/F22r72FW1CCYug_OtarZdQ)
- [10个顶级的CSS动画库](https://mp.weixin.qq.com/s/fkocUOGricoH4Zad4U3Wjw)
