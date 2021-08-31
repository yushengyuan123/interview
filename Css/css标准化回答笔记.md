# 涉及像素的一些概念

## 设备像素

设备像素又称为物理像素，设备能够显示内容的最小单位，

## css像素

css像素是web编程的概念，不同于设备像素，他是抽象的，实际并不存在

## 分辨率

**分辨率就是物理像素**

## PPI

pixels per inch所表示的是每英寸所拥有的像素（pixel）数目。、



# 什么是替换/非替换元素

替换元素：不直接显示元素的内容的元素，如img，iframe，input

替换元素尺寸：img， iframe默认宽高是300px，150px。加了图片之后默认是图片尺寸。

非替换元素：元素内容直接显示在生成框中，如div，span（对于行内非替换元素无法设置宽高）

# 什么是行内元素和块级元素

行内元素：宽度默认包裹内容，可换行。设置width和height对**行内非替换**元素不生效。**垂直padding，border有显示效果但不影响布局**（水平有影响），垂直margin无效。

如何理解border有效果，但不影响布局![image-20201118094551124](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201118094551124.png)

块级元素：元素单独成行，可设置宽高，margin，padding，如果不设magin，默认占满一行。

行内块级元素：

常见的行内元素：span

# 行内元素真的不能够设置宽高吗

说行内元素无法设置宽高这种说法不太准确，准确来说行内非替换元素不能设置宽高，行内替换元素是可以设置宽高的，例如：img标签

img标签是一个行内元素，两个img标签可以并排在一起，并且它的高度和宽度都是可以设置的

![image-20210220201436321](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210220201436321.png)

# img标签属于什么

img标签属于行内替换元素，height，margin，width，margin等都是可以用的，效果等于块级元素，**但是它是可以并排摆放的**

# img标签src是一个路劲和base64有什么区别

优点：

1. 改为base64话不会像服务端发送请求，减少http请求数（node亲测）

缺点：

1. 因为base64是一段十分长的字符串，会增加文件的体积
2. 如果你是在css中使用了背景图片的话，那么会增加css dom的解析时间

# 三个单位的区别

px，rem，em

1. px是固定的像素，一旦设置了就无法因为适应页面而改变
2. em，rem都是相对长度单位。
3. em是相对其父元素来设置字体大小的，一般都是以**父元素的font-size为标准**，**不是以他的长宽为标准**
4. 而rem是相对于根元素为标准。即<html>的font-size。rem默认情况下，等于16px。其实px是由viewport影响的，viewport的默认宽度不同，那么就会影响px的大小。有这句话和没有这句话，同样是1rem的字体大小是不一样的

```
<meta name="viewport" content="width=device-width initial-scale=1"/>
```

例如：假设viewport是980px，意思就是把浏览器分成了980分，不是分辨率分成980分。1px就相当于1/980

# font-size默认带是什么

设置为em，百分比时，**按照父元素的font-size计算**（如父元素font-size: 20px，子元素font-size: 0.5em，则子元素实际font-size为20 * 0.5 = 10px）

如果不设置：默认为1em。

# text-align

text-align: justify

多个行框左右两侧对齐，最后一行无效。

```html
<div class="span">
        <span>hehe</span>
        <span>hahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahahaha</span>
        </div>
```

![image-20201118101353843](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201118101353843.png)

# line-height



# vertical-align的理解

## 它适用于什么元素

vertical-align用于对其行内元素

``` css
display: inline-block | inline
```

## vertical-align用来对齐什么

对齐的是行盒子里面的行内元素，说明了他就是说明行内元素的对齐方式

## 几个概念

- 行盒子：

  行内元素会紧紧的靠在一起，如果放不下就会另外重启一行，所有的行都会创建行盒子，行盒子装着自己的内容，内容不同，行盒子的高度不同。

  行盒子，有自己的顶边和底边，基线，顶边和最顶部元素的顶边重合，底边和最底部元素的底边重合，基线是一个变量，它是看不见的，但是我们有办法可以知道它的位置，就是写一个x，基线和x的底部时对齐的

- 文本盒子

  围绕基线的就是行盒子里面的文本盒子，文本盒子的高度等于父元素的font-size，可以把文本盒子看作是行盒子里面没有经过对齐的行内元素，他有自己的顶边和底边，也有自己的基线（**注意和行盒子的基线区分开来**），本文盒子和基线相关，极限移动也会导致它的移动

## 行内元素和行内块元素的基线确定

行内元素：顶边和底边和自己行高的上下边重合，基线总是穿过字符高度一般以下的某一点

行内块级：有三种情况：

- 如果该元素中有内联元素，基线就是最后一行内联元素的基线
- 有内联元素但`overflow`属性值不是`visible`的行内块元素，基线就是外边距盒子的底边（中）。也就是与行内块元素的下外边界重合。
- 如果没有内联元素，它的基线就是行内块级的下边界

一段脚本可以看出overflow不是visible基线的区别：

```css
<div class="a">
    x
    <span>123</span>
    <div class="top"></div>
    <span>123</span>
</div>

        .top {
            display: inline-block;
            height: 50px;
            width: 10px;
            background-color: #da2824;
            vertical-align: baseline;
            //尝试加上overflow可以看出区别出来
            overflow: hidden;

        }

        .a span {
            vertical-align: middle;
        }
```

## vertical-align值的含义

![image-20210224212815461](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210224212815461.png)

- baseline：元素基线和行盒子基线重合，默认值就是这个
- sub：元素基线移动到和盒子基线的下方
- super：元素基线移动到和盒子基线的上方
- 具体长度值：正的就是在基线上多少个单位，负的就是在基线下方多少个单位，你也可以是百分比，**百分比的话就是相对于行高line-height的**
- middle：元素上下边界中点和行基线向上x-height的一半对齐![image-20210224212808869](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210224212808869.png)
- text-top：和父元素的内容区域的顶边对齐
- text-bottom：和父元素的内容区域的底边对齐
- top：元素的的顶边和最高元素的顶边进行重合(行盒子的文本盒子的顶边对齐)**注意不是行的顶边重合，这是不一样的，在设置了padding-top的情况下就是不一样的**
- bottom：元素的的顶边和最低元素的低边进行重合(行盒子的文本盒子的顶边对齐)**注意不是行的顶边重合，这是不一样的，在设置了padding-top的情况下就是不一样的**

## 如何让一个img标签和一段文本对齐

一段代码的区别，区别在于有一段文本是否用span进行包含，**其中加入了span的可以img和文本对齐，不加的无法对齐**

```css
<div class="a">
    x
    <img src="./img/search.png" alt="">
    123
</div>

<div class="a">
    x
    <img src="./img/search.png" alt="">
    <span>123</span>
</div>

.a span {
     vertical-align: middle;
}

img {
    display: inline-block;
    height: 20px;
    width: 20px;
    vertical-align: middle;
}
```

原因在于：不加span标签，默认`vertical-align: baseline;`和基线对齐，但是img设置了 `vertical-align: middle;`根据middle规则，那么别的一些字符长度和x不一样的，那么就会造成看起来不对齐的情况

但是加了span就能够应用到 `vertical-align: middle;`根据middle规则重新调整位置，基线稍微往下移动就对齐了

## 行盒子基线的移动

这是一个坑，因为行盒子里面的元素是基于基线的，那么如果有一些行为会让极限移动那么就会导致导致其他元素的内容也发生移动。

这种情况就是，有一个很高的盒子，已经顶住了上下边，不能在移动了，此时要和基线对齐，**那么只能去移动基线，不能移动盒子**

```css
<div class="a">
    x
    <div class="top"></div>
    <span>123</span>
</div>

        .top {
            display: inline-block;
            height: 50px;
            width: 10px;
            background-color: #da2824;
          	//尝试调节一下align就知道什么回事了
            vertical-align: text-top;

        }

        .a span {
            vertical-align: middle;
        }
```



# IE盒模型和标准盒子模型

标准盒模型（从内到外）：

1. 内容
2. 内边距padding
3. 边界border
4. 外边距margin

高度是指内容的高度，宽度是指内容的宽度

![image-20201118143315116](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201118143315116.png)

IE盒模型

1. 内容
2. 内边距padding
3. 边界border
4. 外边距margin

和标准盒模型不同的是，**高度是指内容高度 + 内边距 + border，宽度也是如此。**

![image-20201118143629205](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201118143629205.png)

举一个例子：

```css
<style>
    .div1 {
            width: 100px;
            height: 80px;
            border: 10px solid #000;
            padding: 20px;
            background-color: red;
            margin: 50px;
        }
</style>
```

在标准和模型中，这个盒子占据的空间：

宽：100 + 2 * 20 + 2 * 10 + 2 * 50 = 260

高：80 + 20 * 2+10 * 2+50 * 2=240

盒子实际大小：

宽：100 + 2 * 20 + 2 * 10  = 160

高：80 + 20 * 2+10 * 2=140



IE中盒子：

宽：100 + 2 * 50 = 200

高：80 + 50 * 2= 180

盒子实际大小：100 80

# 颜色填充

```
background-color
```

这个属性填充的地方是内容和内边距区域，当然你可以去设置不填充padding区域

# 文档流是什么

文档流指的是在排版过程中，元素会自动从左向右，从上往下的流式排列。并最终窗体自上而下分成一行行，并在每行中按从左至右的顺序排放元素。脱离文档流就是打乱这个排列

# 浮动

## 浮动的问题

1. 浮动元素会脱离文档流
2. 会造成父级元素高度，坍塌。

给元素添加一个float属性，就会让元素脱离文档流，如果元素的下方原本有一个元素，那么下方的元素就会占据浮动元素原本的位置。因为这两个东西不是在同一个层级中的。

## 对于clear的理解

clear意思就是不想哪边出现浮动就清除那边，同时clear只会影响到自身，不会影响到别人，如果你想谁的位置发生移动你就需要在谁的身上加上clear，

一般：float：left使用clear：left进行清除，right的话对应使用right进行清除

```html
    <div class="con"></div>

    <div class="outer"></div>

    <div class="test"></div>
    
        .con {
            height: 100px;
            width: 200px;
            background-color: black;

        }

        .outer {
            height: 100px;
            width: 10rem;
            background-color: #898b93;
            float: left;
        }

        .test {
            height: 100px;
            width: 20rem;
            background-color: green;
            font-size: 30px;
            //我想test不和outer浮动重叠那么我就需要clear test元素
            //因为我需要移动test元素所以clear需要设置在他的身上
            //不能在outer中设置clear:right，因为你不是移动outer，你在他身上clear是没有用的
            clear: left;
        }
```



## 浮动清除方式

1. 下面的元素添加clear方式清除

2. 伪元素的形式，在父元素最后加一个after伪元素，通过在伪元素中清除浮动，通过设置伪元素为display:block从而把高度撑开，其实原理和方法1是一样的

3. 父元素同样浮动，那么它们就在同一层了。
4. 设置`display：table`(`BFC`)
5. 什么都不用做，父元素直接加`overflow:hidden`(`BFC`)

## 浮动清除原理

1. clear：使用了clear属性的元素，只能够影响自身。你在两个浮动元素下方加入clear元素。那么clear元素判断自己左右两边不能出现浮动，那么它只能排在两个浮动元素的下方，然后顺带把父元素的高度也撑开了。
2. `BFC`,`BFC`有一个特性就是，**内部元素再怎么翻江倒海，都不会影响外部的元素。**那么你通过某些属性触发了BFC，自然就会消除浮动带来的影响



## `BFC`

`BFC`含义就是块级格式化上下文。

`BFC`元素的特性原则就是：**内部元素再怎么翻江倒海，都不会影响外部的元素。**

`BFC`特性

1. 块级格式化上下文会阻止外部盒子的外边距叠加
2. 块级格式化上下文会阻止外部浮动盒子和BFC发生重叠
3. 块级格式化上下文在计算高度的时候，内部的浮动元素的高度也会计算在内，所以这里就解决了浮动的高度坍塌问题

换句话说就是创建`BFC`元素就是一个独立的盒子，里面的子元素不会在布局上影响外面的元素，同时`BFC`依然属性文档流，**所以解决了浮动**。

## `BFC`触发条件

```css
float 除了none以外的值 
 
overflow 除了visible 以外的值（hidden，auto，scroll ） 
 
display (table-cell，table-caption，inline-block) 
 
position（absolute，fixed） 
 
fieldset元素
```

是设置了这些属性元素创建了块级格式化上下文，而不是他成为了块级格式化上下文

## BFC应用

1. 阻止外边距合并，下面例子正常情况下，外边距会合并在一起，解决办法可以把其中一个变为BFC就可以了

   ```html
           * {
               margin: 0;
               padding: 0;
           }
           .box1 {
               width: 100px;
               height: 100px;
               margin: 50px;
               background-color: #ff8000;
           }
           .box2 {
           		//display: inline-block;加上这句化就能够阻止外边距合并
               width: 100px;
               height: 100px;
               margin: 50px;
               background-color: #ff0000;
           }
           
   <div class="box1"></div>
   <div class="box2"></div>
   ```

2. 两栏自适应布局，正常情况下两个盒子重叠摆放，可以把其中一个盒子创建BFC，从而让两个盒子分开摆放，原因在于BFC不会重叠浮动元素

   ```css
   * {
       margin: 0;
       padding: 0;
   }
   
   .box1 {
       width: 100px;
       height: 100px;
       float: left;
       background-color: #ff8000;
   
   }
   
   .box2 {
       width: 200px;
       height: 200px;
       background-color: #ff0000;
       //不加这句话，那么两个box就会重叠摆放在一起，加了这句话两个盒子就可以分开摆放
       overflow: hidden;
   }
   
   <div class="box1"></div>
   <div class="box2"></div>
   ```

3. 清除浮动





# 伪类和伪元素区别

伪元素：伪元素类似于真的在dom上加了一个元素一样，但实际并不，所以它叫做伪元素

伪类：一种选择器被选中的元素处于某一种特定的状态，它们表现得好像就是你在他身上加了一个class一样，用来帮助我们更容易就能够操作到元素的css。

总结来说，伪元素是类似于增加了dom，伪类是为了让我们更好的操作元素的css，伪类看上去添加了元素，伪元素并没有添加元素

举一个例子：

这里用伪类 `:first-child` 和伪元素 `:first-letter` 来进行比较。

```
p>i:first-child {color: red}
<p>
    <i>first</i>
    <i>second</i>
</p>
```

![请输入图片描述](http://segmentfault.com/img/bVcccr)//伪类 `:first-child` 添加样式到第一个子元素
如果我们不使用伪类，而希望达到上述效果，可以这样做：

```
.first-child {color: red}
<p>
    <i class="first-child">first</i>
    <i>second</i>
</p>
```

即我们给第一个子元素添加一个类，然后定义这个类的样式。那么我们接着看看为元素：

```
p:first-letter {color: red}
<p>I am stephen lee.</p>
```

![请输入图片描述](http://segmentfault.com/img/bVcccs)//伪元素 `:first-letter` 添加样式到第一个字母
那么如果我们不使用伪元素，要达到上述效果，应该怎么做呢？

```
.first-letter {color: red}
<p><span class='first-letter'>I</span> am stephen lee.</p>
```

即我们给第一个字母添加一个 `span`，然后给 `span` 增加样式。
两者的区别已经出来了。那就是：

> **伪类的效果可以通过添加一个实际的类来达到，而伪元素的效果则需要通过添加一个实际的元素才能达到，这也是为什么他们一个称为伪类，一个称为伪元素的原因。**

# 有哪些伪元素或者伪类

伪类：

- :hover
- :nth-child(n)，n可以为一个数字，一个关键字或一个公式，想匹配奇数的话就是2n + 1，偶数的话就是2n
- nth-of-type(n)，匹配父类某种类型的第n个元素 p:nth-of-type(2)全局匹配第二个元素
- :fist-child
- :last-child

伪元素：

- ::after
- ::before
- ::first-letter
- ::first-line

# css选择器系列

## 一些常用的选择器

1. 元素选择器，就是直接标签名称`html {}`

2. 选择器分组（就是和） `h2, p {}`

3. 类选择器

4. id选择器

5. 属性选择器

   ```
   <p class="important warning">This paragraph is a very important warning.</p>
   
   p[class="important warning"] {color: red;}
   ```

6. 后代选择器

   ```
   h1 em {color:red;}
   ```

7. 子元素选择器

   ```
   //选择h1后面的strong
   h1 > strong {color:red;}
   ```

8. 伪类，给元素添加上特殊的效果

   ```
   :hover这些
   
   :first-child
   
   :active
   ```

9. 伪元素

   ```
   //用于向文本的首行设置特殊样式
   p:first-line
     {
     color:#ff0000;
     font-variant:small-caps;
     }
     //给文本首字母添加特殊的样式
     p:first-letter
     {
     color:#ff0000;
     font-size:xx-large;
     }
     
     ::befer
     
     ::after
   ```

## css选择器有多少类

- 简单选择器
- 符合选择器
- 复杂选择器

简单选择器：

- id选择器

- 类选择器

- 属性选择器，某类元素有某种属性，希望把这一类元素选出来

  ```
  planet[moons] {color:red;}
  
  <planet>Venus</planet>
  <planet moons="1">Earth</planet>
  <planet moons="2">Mars</planet>
  ```

- 伪类

- 伪元素

复合选择器：复合选择器有多个简单选择器构成，只要多个简单选择器挨着写就构成了符合选择器

- <简单选择器> <简单选择器> <简单选择器>
- 交集选择器：第一个是标签选择器第二个是属性选择器，p.one，意思就是类名为one的p标签
- 并集选择器： .one，p，.test{}
- 后代选择器：.one p {}
- 子元素选择器：.demo > h3

复杂选择器：复合选择器中间使用一些连接符连接起来就形成了复杂选择器

<复合选择器>  <复合选择器> —— 子孙选择器，单个元素必须要有空格左边的一个父级节点或者祖先节点

<复合选择器> ">" <复合选择器> —— 父子选择器，必须是元素直接的上级父元素

<复合选择器> "~" <复合选择器> —— 邻接关系选择器

<复合选择器> "+" <复合选择器> —— 邻接关系选择器

<复合选择器> "||" <复合选择器> —— 双竖线是 Selector Level 4 才有的，当我们做表格的时候可以选中每一个列



# css样式优先级

## 继承优先级

1. 最近的祖先样式比其他祖先高
2. 直接样式优先级高于祖先

## 选择器优先级别

内联样式(style) > id选择器 > 类选择器 = 属性选择器 = 伪类选择器 > 标签选择器 = 伪元素选择器

## css优先级的计算

优先级是有一个四元组确定的，（A,B,C,D)

A：出现内联样式，A加1

B：出现id选择器 + 1

C：出现类选择器，属性选择器，伪类选择器 + 1

D：出现标签选择器，伪元素选择器 + 1

比较原则：从左向右进行比较，左边位数大的优先级高，如果四位都是相同的，那么后面覆盖前面。



一个列子：

```css
<div class="nav-list" id="nav-list">
	<div class="item">nav1</div>
	<div class="item">nav2</div>
</div>

//(0,1,1,0) 显示这个
#nav-list .item {
	color: #f00;
}
(0,0,2,0)
.nav-list .item {
	color: #0f0;
}





<div id="box1" class="c1">
    <div id="box2" class="c2">
        <div id="box3" class="c3">
            文字
        </div>
    </div>
</div>

<style>
				(0,0,2,1)
        .c1 .c2 div{
            color: blue;
        }
        (0,1,0,1)
        #box1 div {
            color:yellow;
        }
        (0,1,0,1) 后面覆盖前面 显示绿色
        div #box3 {
            color:green;
        }

    </style>
```

## important特殊情况

从上面的优先级看到内联的优先级是最强大的，但是有没有办法覆盖它，有的

通过!important就可以覆盖他

```
//如果你是这么写的，那么内联的确认就已经达到不可覆盖的地步
<div class="app" style="color:#f00!important">666</div>
```



## 层叠优先级

内联样式 > 内部样式表 > 外部样式表  > 浏览器缺省

# `css`的resetting和normalizing

这两个应该算是一个`CSS`的库。就是用来**修改浏览器的一些初始化样式**

# z-index

https://juejin.cn/post/6844903667175260174

它可以作用于`postion: relative | absoulte | fixed`注意postion都是可以的。

原理：z-index的值可以控制定位元素在垂直于显示屏幕方向（z轴）上的堆叠顺序。

## 父元素真的不能覆盖子元素吗

如果说，你是这么写的。

```html
.parent {
            height: 200px;
            width: 200px;
            position: relative;
            background-color: yellow;
            z-index: 100;
        }

        .son1 {
            height: 100px;
            width: 100px;
            position: absolute;
            background-color: aqua;
            z-index: -1;
        }

        .son2 {
            height: 100px;
            width: 100px;
            top: 50px;
            left: 50px;
            position: absolute;
            background-color: red;
            z-index: -1;
        }
        
<div class="parent">
    <div class="son1"></div>
    <div class="son2"></div>
</div>
```

这样写确实没有效果。

假如你把子元素的z-index设置为负值，父元素不设置z-index就有效果的。说明父元素和子元素还是可以比较。

![image-20201119104221530](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201119104221530.png)

## 原理

z-index的堆叠顺序不仅仅由离距离屏幕z轴方向的距离所决定。

实际上这个层叠顺序是由层叠上下文，层叠等级等共同决定。

### 层叠上下文

层叠上下文是一个三维的概念，每个盒模型的位置都是三维的，分别是平面画布上的x轴，y轴以及表示层叠的z轴。如果一个元素是层叠上下文的元素，可以理解为这个元素在Z轴上就"高人一等"。具体的表现就是里屏幕更加近

### 层叠等级

1. 在同一个层叠上下文中(可以说你是z-index里面的子元素)。它描述定义的是该层叠上下文中的层叠上下文元素在Z轴上的上下顺序。
2. 在其他普通元素中，它描述定义的是这些普通元素在Z轴上的上下文顺序。

我们比较的时候应该需要比较同一个层叠上下文的

### 如何产生层叠上下文

一般层叠上下文由一些特定的`css`属性产生，一般有三种方法：

1. HTML中的根元素<html></html>本身就具有层叠上下文，称为根层叠上下文
2. 普通元素设置position属性，且不是static，并且设置z-index
3. `css3`中的新属性也可以产生层叠上下文

例子：

```html
<style>
  div {
    width: 100px;
    height: 100px;
    position: relative;
  }
  .box1 {
    z-index: 2;
  }
  .box2 {
    z-index: 1;
  }
  p {
    position: absolute;
    font-size: 20px;
    width: 100px;
    height: 100px;
  }
  .a {
    background-color: blue;
    z-index: 100;
  }
  .b {
    background-color: green;
    top: 20px;
    left: 20px;
    z-index: 200;
  }
  .c {
    background-color: red;
    top: -20px;
    left: 40px;
    z-index: 9999;
  }
</style>

<body>
  <div class="box1">
    <p class="a">a</p>
    <p class="b">b</p>
  </div>

  <div class="box2">
    <p class="c">c</p>
  </div>
</body>


```

![image-20201119111941912](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201119111941912.png)

解释：因为`box1`中的元素z-index > `box2`中的z-index。所以无论`box2`的子元素z-index设置得多大，都无法遮盖`box1`的元素。这就符合了同层比较的原则

## 层级判断口诀

1. 先看是不是在同一个层叠上下文中（意思是不是同一个父元素）
2. 如果是的话直接看z-index就好了。
3. 如果不是，看这两个层叠上下文的z-index是否相同。不同的话，z-index大的所有元素包括子元素都会在z-index小的之上。如果相同的话，`dom`节点靠后的在靠前的之上。所有元素包括子元素，无论你子元素z-index多小，也是在上层。



# position系列

## postion有什么值，各自作用是什么

1. static 静态定位 对象遵循标准文档流，top，right，bottom，left失效
2. relative  对象遵循标准文档流中？？，但是元素在文档中本身也会占据位置，他也是依赖top，right，bottom，left进行定位，但**是这个left这些东西不会改变元素原来在文档流中的定位，意思就是说你调整他不会挤压别的元素，但是改变margin和padding会挤压别人**，同时可以使用z-index设置层级
3. absolute 绝对定位，对象脱离文档流，使用top，right，bottom，left等属性进行绝对定位，同时可以使用z-index设置层级
4. fixed 固定定位，对象脱离文档流，通过top，right，bottom，left进行定位，同时可以使用z-index设置层级
5. sticky 粘性定位 这个我基本没用过 ，这个属性的作用就是：它配合滚动，就是一个元素设置了这个属性，如果滚动的时候这个元素离开的屏幕视野，那么他就会粘在屏幕上方，我一直以为这种效果需要`js`实现。

## position sticky了解多少

它被称为粘性定位元素，简单理解就是，在目标区域内它的行为就像relative；在滑动过程中，某个元素的距离达到sticky粘性定位的要求时，这时它的效果相当于fixed，就是他结合了relative和fixed的特性

sticky使用条件：

1. 父元素不能设置overflow:hidden或者overflow:auto属性。
2. 必须指定，top，left，right4个值之一
3. 父元素的高度不能低于sticky元素的高度
4. sticky元素仅在其父元素内生效，它的top是基于吸顶之后距离上面的距离

对于sticky设置z-index是无效的

## sticky实现吸顶效果

但是他有一定的问题，兼容问题

```html
<div class="parent">
    <div class="top"></div>
    <div class="sticky-con">标题</div>
    <div class="bottom-con"></div>
</div>

        .parent {}

        .sticky-con {
            position: sticky;
            top: 0px;
            height: 50px;
            background-color: #3B67A0;
            line-height: 50px;
        }

        .bottom-con {
            height: 2000px;
        }

        .top {
            height: 300px;
        }
```

## 通过js实现吸顶效果

思路：

1. 侦听window.scroll属性
2. 定义scroll事件处理程序，没触发一次，计算scollTop于元素的的offsetTop  - offsetHeight差值。
3. 如果符合，就改变元素的postion为fixed

## postion的left这些定位是怎么相对的

如果父容器没有设置postion非static的值的话，那么就是相对于全局窗口进行定位，如果说设置了非static的值，那么就是相对于父容器进行位置偏移

## relative是相对于谁的

**相对自身**。使用relative定位的元素，其相对的是自身进行偏移。

**无侵入性**。使用relative定位的元素，可以理解为产生了"幻影"，其真身依然在原来的位置上，所以并不会影响页面中其他的元素的布局。本例中，`使用relative`这几个字依然在原来的位置上，而`使用margin`这几个字则偏移了原来的位置。



# margin系列

## margin设置负值是什么效果

负值的话会让元素向相反的方向进行移动，在居中的时候用到过它

# tabel布局

父元素开启设置{ display: table }就能够开启tabel布局

## tabel布局常见的属性

1. tabel-row，可以模仿列表的行
2. tabel-cell，可以模仿列表的列
3. tabel-caption，模仿标题，只要你放在tabel下，那么无论的title放在html那个位置他都可以放在最上方



## cell和column区别

cell是真正意义上的列， table-column相当于col标签用来控制列的行为，并不是真正用来显示标签的

## tabel布局的应用

1. 响应式布局

2. 动态垂直居中对齐

3. 动态水平居中对齐

   

# box-sizing

这个属性可以让你选择是以IE和模型还是`W3C`盒模型。他有两个属性值。

```
border-box就是ie盒模型
content-box就是W3C盒模型

```

```css
.parent {
    box-sizing: content-box;
    height: 200px;
    width: 200px;
    padding: 10px;
    border: 2px solid;
}
```

![image-20201119141918180](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201119141918180.png)



```css
.parent {
    box-sizing: border-box;
    height: 200px;
    width: 200px;
    padding: 10px;
    border: 2px solid;
}
```

![image-20201119141900202](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201119141900202.png)

IE盒子模型有一个优点，就是它不会导致box的总尺寸发生改变。

# zoom和scale

两个属性都是可以对元素进行放大。但是它们用法是有差别的。

1. zoom是IE发明的，它对浏览器的兼容性不太好。scale兼容性强。
2. zoom的放大是以(0,0)为基准，scale是以中心点放大。
3. zoom放大之后，会占用体积。导致页面回流，重渲染消耗性能。scale放大不会占用位置。性能较高。

# `clientWidth innerWidth`

上面两者都能获取网页区域大小，它们的区别在于，前者不包含滚动条，后者包含。



# 水平居中

方法：

1. 如果是块级元素的话，margin: auto.

2. 如果是一个元素在一个元素内部。也可以用margin: auto。

3. 弹性布局。如果是一个元素在一个元素内部。父元素设置

   ```css
   display: flex;
   justify-content: center;
   flex-direction: row;
   ```

4. 绝对定位下：left: 50%。margin-left: -width / 2。

5. 子元素置为`display: inline-block`; 父元素: text-align: center

6. 行内元素:直接 text-align: center.

7. 有种类似于绝对定位的方式。使用transform

           .parent {
                   background-color: yellow;
                   width: 100px;
                   height: 100px;
                   position: relative;
            }
           
           .child {
               position: absolute;
               height: 100px;
               background-color: red;
               left: 50%;
               transform: translateX(-50%);
           }

8. 绝对定位下利用`margin:auto`。实现水平垂直居中

   

## margin: auto原理

一个块级元素，占满一行。如果设置宽度，使用外边距填充完一行。
不设置宽度默认宽度100%。
margin: auto的意思就是说把剩下的空间分配给对应设置的边，直接指定margin，就是说剩下的边平均分配剩下的外边距，就达到了居中的效果。
如果你单侧设置了margin-left:auto那么剩下的空间都会分配给左边。
分配原则：
(1) 如果一侧定值，一侧auto，则auto为剩余空间大小 (2) 如果两侧均是auto，则平分剩余空间

## margin: auto局限

没有宽度，浮动，绝对定位的盒子失效。



# 垂直居中

方法：

2. ```css
   display: flex;
   align-items: center;
   ```

3. 绝对定位

   ```
   top: 50%;
   margin-top: -width / 2
   ```

4. 利用transform同理上面

5. 单行文字/行内元素/行内块级 垂直居中line-height

5. 多行文字，也可以使用line-height：高度 / 行数

6. 对于inline-block的垂直居中，原理就是利用vertical-align，l**ine-height改变了行盒子的基线**是的他在行的最中心，根据vertical-align：middle于x的对应规则使得图片垂直居中

   ```
   <div id="parent">
       x
       <div class="inline"></div>
   </div>
   
           #parent{
               height: 150px;
               line-height: 150px;
               background-color: #da2824;
               font-size: 16px;
           }
   
           .inline {
           		//必须要是inline-block
               display: inline-block;
               height: 100px;
               width: 100px;
               vertical-align: middle;
               background-color: #898b93;
           }
   ```

   

7. 利用table布局

   ```css
   <div id="parent">
       <div id="son"></div>
   </div>
   
           #parent{
               height: 100px;
               width: 100px;
               display: table-cell;
               vertical-align: middle;
               background-color: #da2824;
           }
   
           #son {
               height: 50px;
               width: 50px;
               background-color: #898b93;
           }
   ```

   



# display:flex

display: flex是弹性布局。他决定了元素如何在页面中排列。在不同设备和尺寸下都能够可预测的展现出来。

它能够扩展和收缩flex容器内的元素，以最大的限度填充可用空间。

# flex

https://juejin.cn/post/6844904016439148551

http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html

## 理解

flex意思是弹性布局。子元素根据父级元素的尺寸进行自适应。

当父元素宽度足够的时候，子元素会按照之前设定的宽的值进行显示。

但是假如父级元素宽度不够的时候，该**子元素的实际宽度是(设置的该子元素的宽度/设置的子元素宽度之和)*父元素的宽度。**

```html
.parent {
    background-color: yellow;
		width: 400px;
    height: 100px;
    display: flex;
}

.box1 {
    height: 80px;
    width: 100px;
}

.box2 {
    height: 80px;
    width: 200px;
}

<div class="parent">
    <div class="box1"></div>
    <div class="box2"></div>
    <div class="box1"></div>
    <div class="box1"></div>
</div>

//这个例子中显然子元素超出了父元素宽度，子元素就会按照比例主动压缩。

```

![image-20201121120634655](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201121120634655.png)

## inline-flex

`inline-flex`将对象作为内联块级弹性伸缩盒显示，内联的特点就是元素的宽度由内容撑开，当你不设置宽度的时候那么宽度就是由子元素撑开，如果你设置宽度，那么宽度还是按flex那样

inline-flex和flex不同的地方。**我们在flex下如果父元素不设置宽度那么宽度默认100%。**

**那么如果在这种情况下不设置宽度，默认被子元素撑开。**

**但是虽然是内联的，但是依旧可以设置宽度**

## 对于flex-wrap理解

flex-wrap有两个值wrap，no-wrap。它可以在让flex盒子是以单行呈现还是以多行呈现。

当你设置nowrap就是强制不换行（即使超出了盒子宽度或者高度），此时flex就是一个single-line flex容器

当你设置wrap的时候就是换行，此时盒子就是一个multi-line flex容器，当宽度或者高度不高的时候就会进行换行。

注意这里的换行是一个广义的概念，**就是不单只有水平方向上的换行，还有垂直方向上的换行**，垂直方向上就是当你设置flex-direction：column的时候，如果高度不够，就会自动开启下一列。

这个属性还会对其他的属性的效果产生影响，例如align-content和align-center

## justify-content的误区

可能我们会认为justify-content属性只会在flex-direction: row;的时候产生水平居中的效果，在row的情况下是没有用的。

实际情况是column情况下也是有用的，因为column情况下主轴是垂直方向上的。当设置column，元素则会竖直垂直居中摆放。

## justify-content有多少种效果

https://www.runoob.com/try/playit.php?f=playcss_justify-content&preval=space-around

1. flex-start：从头开始排列，不间隔
2. flex-end：从尾巴开始排列，不间隔
3. center：居中排列
4. space-between，两边不空，中间相等间隔排列
5. space-around：前后左右都有间隔



## align-items有多少种效果

1. stretch 元素被拉伸以适应容器
2. center，垂直居中排列
3. flex-start，从头排列
4. flex-end，从尾巴开始排列
5. baseline 元素位于容器的基线

## align-content有多少种效果

align-content感觉就是垂直方向上的justify-content，他的效果和justify-content十分相似。只是在水平方向上改为垂直方向上

1. flex-start：从头开始排列，不间隔
2. flex-end：从尾巴开始排列，不间隔
3. center：居中排列
4. space-between，两边不空，中间相等间隔排列
5. space-around：前后左右都有间隔
6. stretch：拉伸

## align-content和items

items。可以设定多行或者单行的垂直居中。flex-wrap为nowrap或者wrap的时候都是有效的。

align-content：**只能够设置多行的位置，当只有当行的时候就是flex-wrap为nowrap的时候，这个属性是不起作用的。**

```css
<div class="parent">
    <div class="box1"></div>
    <div class="box1"></div>
</div>

.parent {
            height: 200px;
            background-color: yellow;
            display: flex;
            align-content: center;
            //如果没有加这个那么单行content无效。
  					//多行才有效
            flex-wrap: wrap;
}

.box1 {
            height: 80px;
            width: 100px;
            background-color: red;
}
```

## flex的理解

flex是一个符合属性。

它是flex-grow flex-shrink flex-basis组合。

### flex-grow

默认为0，用于决定项目在有**剩余空间的情况下是否放大。默认不放大。**

**如果设置了放大那么在设置了宽度的情况下也是会放大的。**

```
flex 1
flex 2
//可以这么理解。
flex 1假如有剩余空间，那么我要占剩余空间得1份。所以现有长度就是原长 + 新增
flex 2假如有剩余，我要占用两份 
//所以说你的子项目中如果都设置了flex的话，你原来设置的宽度就没有用了。
```

![image-20210211161534903](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210211161534903.png)

![image-20210211161526603](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210211161526603.png)

### flex-shrink

默认为1 用于决定项目在空间不足时是否缩小，默认项目都是1。**即使设置了宽度也会缩小**。

设置flex-shrink为:0就是不缩小。

压缩的计算方式：

三个flex item元素的width: w1, w2, w3

三个flex item元素的flex-shrink：a, b, c

计算总压缩权重： sum = a * w1 + b * w2 + c * w3

计算每个元素压缩率： S1 = a * w1 / sum，S2 =b * w2 / sum，S3 =c * w3 / sum

计算每个元素宽度：width - 压缩率 * 溢出空间



### flex-basis

默认auto 用于设置项目宽度，项目保持默认宽度，就是width的宽度。

但是如果同时设置了flex-basis和width，flex-basis的权重比width属性高，因此表现为flex-basis的宽度。

假如flex-basis和其他元素的宽度超过父容器，则按照width和flex-basis的比例计算

![image-20201123130647621](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201123130647621.png)



# grid栅格布局

http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html

## 一些概念

容器：采用grid布局的区域就叫做容器

项目：放在容器里面的一些元素就叫做项目

行和列：个人觉得一个项目可以构成一个行或者一列

单元格：行和列交叉区域就叫做单元格，3行3列会产生9个单元格

网格线：划分网格的线叫做网格线，水平网格线划分出行，垂直网格线划分出列，n行有n + 1根水平网格线，m列有m + 1根垂直网格线。

## 容器属性总结

1. display: grid | inline-grid 开启行内的网格还是块级网格

2. grid-template-columns / grid-template-rows：规定网格的一行或者一列有多少个项目
   - 可以写死10px 10px，写成auto关键字，就是交给浏览器去决定他的高度
   - 可以使用repeat函数避免重复写，repeat函数参数
   - 基础(3, 10px)表示三行，三个元素，每个元素都是10px
   - auto-fill，不知道一行有多少列，你尽量得去把他排满
   - fr关键字，等于项目的比例 `grid-template-columns: 1fr 1fr;`意思就是宽度1比1
   - minmax，定义长度范围
   - 百分比
   
3.  grid-row-gap / column：定义行于行的间距或者列于列的间距

4.  grid-template-areas，进行区域划分，对于一些不需要利用的空间，用.进行表示

    ```css
    grid-template-areas: 'a a a'
                         'b b b'
                         'c c c';
    ```

5. grid-auto-flow：他可以决定项目是先行后列摆放还是先列后行摆放，有一个row dense没有完全理解

6.  justify-items 属性， align-items 属性， place-items 属性（是前面两个属性的合并） 这个类似于flex属性，它是规定**单元格里面**内容的摆放。如果单元格设置宽度100%的话就看不出什么效果，小于100%的时候才看得出效果

7. justify-content 属性， align-content 属性， place-content 属性这三个属性是针对放在容器中的网格在容器的位置，和上面那个需要区别开来，上面那个是针对单元格的

8. grid-auto-rows / columns：有时候我们通过grid-column-start属性指定了某一些项目在网格的外部，就是超出来开始的定义的行列数，那么这些多余网格的大小就可以使用grid-auto-rows进行设置，如果不指定这些多余的那么就根据里面的内容决定大小

## 项目属性总结

1.  grid-column-start 属性， grid-column-end 属性， grid-row-start 属性， grid-row-end 属性 这四个属性可以指定项目的位置，从哪里开始从那里结束
2. grid-column 属性， grid-row 属性 是上面对应属性的合并
3. grid-area 指定项目放在哪一个区域，和上面容器属性有一个分区相关
4. justify-self



## 容器属性

display: grid | inline-grid;

```css
.container {
  display: grid | inline-grid;
  //加了inline之后就能够向行内元素一样，不加就是块级
}
```

grid-template-columns: 100px 100px 100px;这个属性设置一行有多少个元素，同时设置每一个行的元素的宽度，你写了多少个就会有多少列，有时觉得一个个些麻烦的话，那么可以使用repeat函数,第一个参数是项目数，第二个参数是项目宽度。

grid-template-rows，同理

```css
grid-template-columns: 100px 100px 100px;
grid-template-columns: repeat(3, 33.33%);
grid-template-rows
```

**auto-fill 关键字**

有时单元格的大小是不确定的，如果你希望京可能容纳更多的单元格，你可以使用auto-fill 关键字

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, 100px);
}
```

**fr 关键字**

为了方便表示比例关系，网格布局提供了fr关键子，2fr和1fr的两倍类似于flex布局的flex属性

```css
grid-template-columns: 1fr 2fr;
```

**行列间距**

`grid-row-gap`属性设置行与行的间隔（行间距），`grid-column-gap`属性设置列与列的间隔（列间距）。

> ```css
> .container {
>   grid-row-gap: 20px;
>   grid-column-gap: 20px;
> }
> ```

`grid-gap`属性是`grid-column-gap`和`grid-row-gap`的合并简写形式，语法如下。

> ```css
> grid-gap: <grid-row-gap> <grid-column-gap>;
> ```

因此，上面一段 CSS 代码等同于下面的代码。

> ```css
> .container {
>   grid-gap: 20px 20px;
> }
> ```

## 如何设置单元格水平垂直居中

## 如何让项目行跨几个单元格

# transform



了解以下3d的属性

## 透视perspective

可以这里理解这个属性。这个属性是视距的意思。就是说物理距离人眼睛的距离

假如这个值调得很小，那么你看到的这个物体就会很大，**那么就能够看到它微小的变换细节**。假如说你调节得很大的话，**这个物体发生一些微小的变化你是看不出来的。**

假如一个物体沿x轴转了30度。如果perspective很大，你是完全看不出来他转了30度。

假如他只转了1度，perspective很小的情况下，你才看得出来它旋转了1度。

#### transform-style

控制子元素是否开启三维立体环境，代码写给父级，但是影响的是子盒子



# css如何绘制一个三角形原理

原理就是看看一个div盒子长什么样的

```html
.sanjiaoxing {
    height: 100px;
    width: 100px;
    border-left: 50px solid yellow;
    border-right: 50px solid red;
    border-bottom: 100px solid blue;
    border-top: 100px solid green;
}

<div class="sanjiaoxing"></div>
```

![image-20210307144541430](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210307144541430.png)

所以关键就是`height: 0`和`width：0`是形成一个三角形的关键

# BFC

名称：块级格式化上下文

三个产生具有`BFC`**资质**（具有资质但还不是BFC元素）的属性。

```css
display: block | table | list-item
```

给元素加上这些属性才能够成为BFC：

1. overflow不为：visible
2. position: absolute, relative, fixed 
3. `display: inline-block  flex inline-flex table-cell, table-caption`
4. float不为none

## BFC特性

1. 盒子从顶端一个接一个的垂直排列下来
2. 属于同一个BFC的垂直距离由margin决定，但是同一个BFC的margin会合并
3. BFC在计算高度的时候会计算浮动的高度和定位的高度
4. BFC元素不和浮动元素产生交集，而是紧贴浮动元素边缘

## 主要用途

1. 解决float高度坍塌问题

2. 解决边距合并问题。（需要在元素外面包裹一层，并且让他成为BFC）

   ```css
            .bfc {
               height: 500px;
               width: 200px;
               background-color: #da2824;
               overflow: hidden;
           }
   
           .children1 {
               height: 100px;
               width: 150px;
               background-color: yellow;
               margin-bottom:  20px;
               overflow: hidden;
           }
   
           .bfc div:nth-child(2) {
               display: block;
               overflow: hidden;
           }
   
           .bfc div:nth-child(2) div{
               height: 100px;
               width: 150px;
               background-color: purple;
               margin-top: 20px;
           }
    
    <div class="bfc">
           <div class="children1"></div>
           <div>
               <div class="children2"></div>
           </div>
    </div>
   ```

3. 制作右侧自适应盒子

   ```css
   .bfc {
               height: 500px;
               background-color: #da2824;
           }
   
           .children1 {
               height: 100px;
               width: 150px;
               background-color: yellow;
               float: left;
           }
   
           .bfc div:nth-child(2) {
               display: block;
               overflow: hidden;
           }
   
           .bfc div:nth-child(2) div{
               height: 100px;
               width: 150px;
               background-color: purple;
           }
           
           
   <div class="bfc">
           <div class="children1"></div>
           <div>
               <div class="children2"></div>
           </div>
   </div>
   ```

   

# IFC

名称： 行级格式化上下文

- 内部的盒子会在水平方向，一个个地放置(默认就是IFC)；
- IFC的高度，由里面最高盒子的高度决定(里面的内容会撑开父盒子）；
- 当一行不够放置的时候会自动切换到下一行；

# 硬件加速原理

硬件加速就是把渲染过程交给GPU处理而不是使用自带的比较慢的渲染器

原理：

浏览器接收到页面文档后，会将文档中的标记语言解析为DOM树。DOM树和CSS结合后形成浏览器构建页面的渲染树。渲染树中包含大量的渲染元素，每个渲染元素会被分到一个图层中，每个图层又会被加载到GPU形成渲染纹理，而图层在GPU中transform是不会触发repaint的，最终这些使用transform的图层都会由独立的合成器进程进行处理, CSS transform会**「创建了一个新的复合图层，可以被GPU直接用来执行transform操作」**。

开启硬件加速方法：

- 3D或者CSS transform

- <video>和<canvas>标签

- `css filters(滤镜效果)`

- 元素覆盖时，比如使用了z-index属性

# 节流和防抖

节流就是：对于一些会频繁触发的事件，例如html5的touchmove事件，只要你频繁滑动就会频繁触发，如果你在事件的处理程序中又是做了些高运算量的东西，那么对于性能的消耗是十分大的。

那么这个时候就需要进行节流，节流就是防止一段时间内事件的频繁触发：

```js
function throttle(func, timeout) {
  let previous = new Date()
  let timer = null

  return function () {
    let now = new Date()
    let trigger = timeout - (now - previous)
    let args = Array.prototype.call(...arguments)

    if (trigger <= 0) {
      //可以触发
      clearTimeout(timer)
      timer = null

      previous = new Date()

      func.call(this, ...args)
    } else if (!timer) {

      timer = setTimeout(() => {
        clearTimeout(timer)
        timer = null

        func.call(this, ...args)
        previous = new Date()
      }, trigger)
    }
  }
}
```



防抖：短时间内多次触发通过一个函数，**只执行最后的一次**。简单实现

```js
function debounce(func, timeout, immediate = true) {
  let timer = null

  return function () {
    let args = Array.prototype.call(...arguments)
    let now = immediate && !timer
    clearTimeout(timer)

    timer = setTimeout(() => {
      timer = null

      !immediate ? func.call(this, ...args) : null
    }, timeout)

    now ? func.call(this, ...args) : null
  }
}
```

防抖和节流有相同也有不同的地方，节流是只要触发，一段时间内就不会再触发，时间内触发他不会对计时器进行清空。防抖是，一段时间内如果重新触发，都会重新计算时间，以最后一次触发事件为准。

# 回流和重绘

回流被造成重绘，重绘不一定会造成回流。

重绘就是，例如你只是单纯改变了某个元素的背景颜色，那么浏览器就需要重新去描绘你改变的颜色，它的改变不会影响到其他的元素

回流：当浏览器发现某个部分发生了变化影响了布局，虽然只改变了他，但是他改变的同时也影响了别人，那么浏览器就需要从根节点重新计算所有节点的几何信息，倒回去重新渲染，那么这个过程就叫做回流。

造成回流的原因在于，例如：我们改变了某个元素的位置信息，那么这个元素的位置改动可能就会造成其他元素位置的移动，那么一个元素发生移动，那么浏览器都会重新计算其他的元素位置信息是否会被他影响，这就造成了回流。显然这个是一个十分消耗性能的操作。

这些操作会造成回流：

1. 增加，删除dom
2. 首屏渲染
3. 操作class属性
4. 脚本操作dom
5. 计算offsetWidth和offsetHeight
6. 设置style，clientWidth，getComputedStyle(等等和集合相关的属性

flush队列，在浏览器的内部有优化策略，如果调一个回流一个，那么浏览器肯定是受不了的。那么他需要维护一个队列，把一些引起回流，重绘的操作放在队列中，等队列中的操作到了一定数量之后，浏览器再对flush队列进行一个批处理。那么就能够让多次回流操作变成一次。

但是对于某些属性，提前触发浏览器的回流机制，以便获得最准确的信息，例如`offset*`系列的属性。`client*`几何系列。height，width等。

回流的优化：

1. 尽量将动画应用到postion为absolute，fixed元素上。
2. 尽量避免频繁操作dom，应该把多次的dom操作合并成为一次dom操作，例如利用文档碎片。
3. 避免频繁调用那些会强制flush队列清空的几何位置属性
4. 不要频繁操作样式，最后一次性将class重写，然后一次性改变class属性



如果避免回流：

1. 不要频繁去调用那么位置api，什么left，right这些，这些都会造成回流
2. 使用文档随pian

# display:none与visibility:hidden

https://juejin.cn/post/6844903686313869320

display:none设置后的元素不会占据页面的空间，但是它依旧会存在于文档流中，元素不会被完全删除。

在浏览器生成render数的时候，对应的display:none属性的元素没有生成对应的盒子模型，因此后续的布局，渲染就不关它的事情。

visibility:hidden，虽然隐藏了，但是它不会依旧会占据着空间位置

# css3动画了解多少

主要涉及几个属性：transition animation结合keyframe机制 transform

- transition： 变化属性 | 动画持续时间 | 变化速率 | 延迟。transition需要知道变化的起点和终点，同时需要事件进行触发，例如hover，点击等等，根据起点和终点就能够计算中间的值完成过度。

- animation：keyframe名称 | 动画持续时间 | 速率曲线 | 延迟 | 动画播放次数 | 是否应该轮询反向播放动画。animation需要配合keyframe来用，keyframe让开发者能够控制动画的关键帧。

  ```css
  <div class="one"></div>
  
  
  .one {
          width: 100px;
          height: 100px;
          margin: 200px auto;
          background-color: #cd4a48;
          position: relative;
          animation: moveHover 5s ease-in-out 0.2s;
  
      }
  
  
      @keyframes moveHover {
          0% {
              top: 0px;
              left: 0px;
              background: #cd4a48;
          }
          50% {
              top: 200px;
              left: 200px;
              background:#A48992;
          }
          100% {
              top: 350px;
              left:350px;
              background: #FFB89A;
          }
      }
  
  ```

- transform：主要包括平移 旋转 扭曲skew 缩放scale和矩阵变换



# js动画了解多少

1. setTimeInterval
2. requestAnimation
3. 



# 面试各种布局的写法

https://juejin.cn/post/6844903574929932301

## 两列布局系列

html代码

```html
    <div class="flex-box">
        <div class="left-con"></div>
        <div class="right-con"></div>
    </div>
```

### 左列定宽，右列自适应

这个一共有5种方法可以实现

1. 方法1：flex

   ```
   .flex-box {
       display: flex;
       height: 200px;
       width: 300px;
   }
   
   .left-con {
       width: 200px;
       background-color: #da2824;
   }
   
   .right-con {
       flex: 1;
       background-color: #898b93;
   }
   ```

2. 方法2：float overflow 触发BFC

   ```css
           .flex-box {
               height: 200px;
               width: 300px;
           }
   
           .left-con {
               float: left;
               height: 100%;
               width: 200px;
               background-color: #da2824;
           }
   
           .right-con {
               height: 100%;
               //bfc
               overflow: hidden;
               background-color: #898b93;
           }
   ```

3. 方法3：float + margin-left 这种原理是一个掩人耳目的写法，本质上因为float了，那么右边的元素占100%，然后你设置了margn-left，使得右边元素框变小了，看上去

   ```css
           .flex-box {
               height: 200px;
               width: 300px;
           }
   
           .left-con {
               float: left;
               height: 100%;
               width: 200px;
               background-color: #da2824;
           }
   
           .right-con {
               height: 100%;
               margin-left: 200px;
               background-color: #898b93;
           }
   ```

4. 利用绝对定位实现，原理和margin-left相当，不些代码了

5. table 布局

   ```
           .flex-box {
               height: 200px;
               width: 300px;
               display: table;
           }
   
           .left-con {
               display: table-cell;
               height: 100%;
               width: 200px;
               background-color: #da2824;
           }
   
           .right-con {
               display: table-cell;
               height: 100%;
               background-color: #898b93;
           }
   ```
   
6. 还有一种不是很好的办法，如果事sass环境可以把左侧的宽定位一个变量，右侧使用calc减去左边



### 左列自适应，右列定宽

这个和上面一样，换个方向就好

### 一列不定，一列自适应

1. 方法1：flex

   ```
   .flex-box {
       display: flex;
       height: 200px;
       width: 300px;
   }
   
   .left-con {
       background-color: #da2824;
   }
   
   .right-con {
       flex: 1;
       background-color: #898b93;
   }
   ```

2. 方法2：float实现，**这种的原理是一个掩人耳目的写法，本质上右边并不是自适应的，而是默认就占了100%，然后被浮动元素盖住了**

   ```
           .flex-box {
               height: 200px;
               width: 300px;
           }
   
           .left-con {
               float: left;
               height: 100%;
               background-color: #da2824;
           }
   
           .right-con {
               height: 100%;
               background-color: #898b93;
           }
   ```

3. grid 布局

   ```
   .flex-box {
       height: 200px;
       width: 300px;
       display: grid;
       grid-template-columns: auto 1fr;
   }
   
   .left-con {
       background-color: #da2824;
   }
   
   .right-con {
       background-color: #898b93;
   }
   ```



## 三列布局系列

### 两列定宽,一列自适应

1. 方法1：flex方案

   ```
           .flex-box {
               height: 200px;
               width: 300px;
               display: flex;
           }
   
           .left-con {
               float: left;
               height: 100%;
               width: 100px;
               background-color: #da2824;
           }
   
           .middle-con {
               height: 100%;
               width: 100px;
               background-color: purple;
           }
   
           .right-con {
               flex: 1;
               height: 100%;
               background-color: #898b93;
           }
   ```

2. float + margin实现和两列类似

   ```
   #parent{
       min-width: 310px; /*100+10+200,防止宽度不够,子元素换行*/
   }
   #left {
       margin-right: 10px;  /*#left和#center间隔*/
       float: left;
       width: 100px;
       height: 500px;
       background-color: #f00;
   }
   #center{
       float: left;
       width: 200px;
       height: 500px;
       background-color: #eeff2b;
   }
   #right {
       margin-left: 320px;  /*等于#left和#center的宽度之和加上间隔,多出来的就是#right和#center的间隔*/
       height: 500px;
       background-color: #0f0;
   }
   
   ```

3. float + overflow bfc

   ```css
           .flex-box {
               height: 200px;
               width: 300px;
           }
   
           .left-con {
               float: left;
               height: 100%;
               width: 100px;
               background-color: #da2824;
           }
   
           .middle-con {
               float: left;
               height: 100%;
               width: 100px;
               background-color: purple;
           }
   
           .right-con {
               overflow: hidden;
               height: 100%;
               background-color: #898b93;
           }
   ```

4. table table-cell结合和两列一样，不贴代码了

5. 使用grid实现

   ```css
   .flex-box {
       height: 400px;
       width: 300px;
       display: grid;
       grid-template-columns: 50px 50px 1fr;
   }
   
   .left-con {
       background-color: #da2824;
   }
   
   .middle-con {
       background-color: purple;
   }
   
   .right-con {
       background-color: #898b93;
   }
   ```

### 两侧定宽,中间自适应

1. 方法1： flex布局，本质还是用到了flex：1，不贴代码了

2. tabel布局

3. grid布局

4. 使用浮动实现

   ```
   
   ```

### 垂直的两边固定，中间自适应

方法1：flex实现

```css
        .flex-box {
            height: 400px;
            width: 100%;
            display: flex;
          	//关键在于这句话，转转方向才行
            flex-direction: column;
        }

        .left-con {
            height: 100px;
            width: 100%;
            background-color: #da2824;
        }

        .middle-con {
            flex: 1;
            width: 100%;
            background-color: purple;
        }

        .right-con {
            height: 100px;
            background-color: #898b93;
        }
```

## 五点布局

- 方法1：可以用最笨的方法去实现

- 方法2：使用grid布局

  ```css
  #container{
      height: 500px;
      width: 500px;
      border: 1px solid black;
      display: grid;
      grid-template-columns: 100px 100px 100px;
      grid-template-rows: 100px 100px 100px;
      grid-template-areas: 'a . c'
                           '. e .'
                          'g . i';
  }
  
  .item {
      height: 100%;
      width: 100%;
      font-size: 2em;
      text-align: center;
      border: 1px solid #e5e4e9;
      justify-self: center;
      align-self: center;
  }
  
  .item-1 {
      background-color: #ef342a;
      grid-column-start: a;
  }
  
  .item-2 {
      background-color: #f68f26;
      grid-column-start: c;
  }
  
  .item-3 {
      background-color: #4ba946;
      grid-column-start: e;
  }
  
  .item-4 {
      background-color: #0376c2;
      grid-column-start: g;
  }
  
  .item-5 {
      background-color: #c077af;
      grid-column-start: i;
  }
  
  <div id="container">
      <div class="item item-1">1</div>
      <div class="item item-2">2</div>
      <div class="item item-3">3</div>
      <div class="item item-4">4</div>
      <div class="item item-5">5</div>
  </div>
  ```

## 多列布局

通用html

```html
<div class="flex-box">
    <div class="box">12356</div>
    <div class="box">12356</div>
    <div class="box">12356</div>
    <div class="box">12356</div>
</div>
```

### 等宽布局

方法1：flex方案

```css
        .flex-box {
            height: 400px;
            width: 100%;
            display: flex;
        }

        .box {
            height: 100%;
            flex: 1;
        }
```

方法2：float + width + border-box：百分比实现，这个是加大难度的版本，元素间有边距

```css
        .flex-box {
            height: 400px;
            width: 100%;
            //看起来是整体居中
            margin-left: -12.5px;
        }

        .box {
            float: left;
            height: 100%;
            width: 25%;
            //使用ie盒子
            box-sizing: border-box;
            padding-left: 25px;
            //填写只填充内容部分
            background-clip: content-box;
            background-color: #898b93;
        }
```

方法3：基于table table-cell实现

```css
.flex-box {
    height: 400px;
    width: 100%;
    display: table;
}

.box {
    display: table-cell;
    height: 100%;
    background-color: #898b93;
}
```