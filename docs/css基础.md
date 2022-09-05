# CSS 基础



## CSS编写格式

**1.行内样式**

**2.内嵌样式**

**3.外链样式  -- 企业开发用外链样式**

**4.导入样式**



外链样式和导入样式区别:

共同点: 都是将CSS代码写到了一个单独的文件中

不同点:

- 外链样式, 在显示网页时, 会先加载CSS文件, 再显示页面

- 导入样式, 在显示网页时, 会先显示界面, 再加载CSS文件

外链样式是通过一个HTML标签引入CSS的

而导入样式是通过@import引入CSS的, 而@import是CSS2.1推出, 所以导入样式存在兼容问题



优先级问题

行内样式的优先级最高

其它写法谁写在后面就听谁的



## CSS三大特性

### -  继承性

1.什么是继承性?

作用: 给父元素设置一些属性, 子元素也可以使用, 这个我们就称之为继承性



注意点:

1.并不是所有的属性都可以继承, 只有以`color/font-/text-/line-`开头的属性才可以继承

2.在CSS的继承中不仅仅是儿子可以继承, 只要是后代都可以继承

3.继承性中的特殊性

 	3.1 a标签的文字颜色和下划线是不能继承的

​	 3.2 h标签的文字大小是不能继承的

应用场景:

一般用于设置网页上的一些共性信息, 例如网页的文字颜色, 字体,文字大小等内容`body{}`

### - 层叠性

作用: 层叠性就是CSS处理冲突的一种能力

注意点:

层叠性只有在多个选择器选中"同一个标签", 然后又设置了"相同的属性", 才会发生层叠性

### - 优先级

1.什么是优先级?

作用:当多个选择器选中同一个标签, 并且给同一个标签设置相同的属性时, 如何层叠就由优先级来确定



2.优先级判断的三种方式

​	2.1间接选中就是指继承

​		如果是间接选中, 那么就是谁离目标标签比较近就听谁的

​	2.2相同选择器(直接选中)

​		如果都是直接选中, 并且都是同类型的选择器, 那么就是谁写在后面就听谁的

​	2.3不同选择器(直接选中)

​		如果都是直接选中, 并且不是相同类型的选择器, 那么就会按照选择器的优先级来层叠

​		`id>类>标签>通配符>继承>浏览器默认`



#### !important

1.什么是!important

作用: 用于提升某个直接选中标签的选择器中的某个属性的优先级的, 可以将被指定的属性的优先级提升为最高

注意点:

1.!important只能用于直接选中, 不能用于间接选中

2.通配符选择器选中的标签也是直接选中的

3.!important只能提升被指定的属性的优先级, 其它的属性的优先级不会被提升

4.!important必须写在属性值得分号前面

5.!important前面的感叹号不能省略

```css
*{
  color: blue !important;
  font-size:10px;
}
```



#### 权重问题

1.什么是优先级的权重?

作用: 当多个选择器混合在一起使用时, 我们可以通过计算权重来判断谁的优先级最高



2.权重的计算规则

​	2.1首先先计算选择器中有多少个id, id多的选择器优先级最高

​	2.2如果id的个数一样, 那么再看类名的个数, 类名个数多的优先级最高

​	2.3如果类名的个数一样, 那么再看标签名称的个数, 标签名称个数多的优先级最高

​	2.4如果id个数一样, 类名个数也一样, 标签名称个数也一样, 那么就不会继续往下计算了, 那么此时谁写在后面听谁的

​	也就是说优先级如果一样, 那么谁写在后面听谁的

注意点:

1.只有选择器是直接选中标签的才需要计算权重, 否则一定会听直接选中的选择器的









## 选择器

### - 标签选择器

### - 类选择器

### - id选择器

### - 后代选择器

```css
div ul li p{
  color: red;
}
```

### - 子元素选择器

```css
div>ul>li>p{
    color: purple;
}
```

**后代选择器和子元素选择器之间的区别?**



后代选择器使用空格作为连接符号

子元素选择器使用`>`作为连接符号



后代选择器会选中指定标签中, 所有的特定后代标签, 也就是会选中儿子/孙子..., 只要是被放到指定标签中的特定标签都会被选中

子元素选择器只会选中指定标签中, 所有的特定的直接标签, 也就是只会选中特定的儿子标签



**后代选择器和子元素选择器之间的共同点**



后代选择器和子元素选择器都可以使用标签名称/id名称/class名称来作为选择器



后代选择器和子元素选择器都可以通过各自的连接符号一直延续下去

选择器1>选择器2>选择器3>选择器4{}



**在企业开发中如何选择**

如果想选中指定标签中的所有特定的标签, 那么就使用后代选择器

如果只想选中指定标签中的所有特定儿子标签, 那么就使用子元素选择器



### - 交集选择器

### - 并集选择器

```css
.ht,.para{
  color: red;
}
```

### - 兄弟选择器(同级)

相邻兄弟选择器

作用: 给指定选择器后面紧跟的那个选择器选中的标签设置属性

```css
h1+p{
  color: red;
}
```

通用兄弟选择器 CSS3

作用: 给指定选择器后面的所有选择器选中的所有标签设置属性

```css

h1~p{
   color: green;
}
```

### - 序选择器



CSS3中新增的选择器最具代表性的就是序选择器

1.同级别的第几个

`:first-child` 选中同级别中的第一个标签

`:last-child` 选中同级别中的最后一个标签

`:nth-child(n)` 选中同级别中的第n个标签

`:nth-last-child(n)` 选中同级别中的倒数第n个标签

`:only-child` 选中父元素中唯一的标签

注意点: 不区分类型



2.同类型的第几个

`:first-of-type` 选中同级别中同类型的第一个标签

`:last-of-type`  选中同级别中同类型的最后一个标签

`:nth-of-type(n)` 选中同级别中同类型的第n个标签

`:nth-last-of-type(n)`  选中同级别中同类型的倒数第n个标签

`:only-of-type` 选中父元素中唯一类型的某个标签



补充

`:nth-child(odd)` 选中同级别中的所有奇数

`:nth-child(even)` 选中同级别中的所有偶数



`:nth-child(xn+y)`

x和y是用户自定义的, 而n是一个计数器, 从0开始递增



### - 属性选择器

什么是属性选择器?

作用: 根据指定的属性名称找到对应的标签, 然后设置属性



格式:

`[attribute]`

作用:根据指定的属性名称找到对应的标签, 然后设置属性

`[attribute=value]`

作用: 找到有指定属性, 并且属性的取值等于value的标签, 然后设置属性

最常见的应用场景, 就是用于区分input属性

`input[type=password]{}`



**属性取值**

1.属性的取值是以什么开头的

`[attribute|=value]` CSS2

`[attribute^=value]` CSS3

两者之间的区别:

CSS2中的只能找到value开头,并且value是被-和其它内容隔开的

CSS3中的只要是以value开头的都可以找到, 无论有没有被-隔开



2.属性的取值是以什么结尾的

`[attribute$=value]` CSS3



3.属性的取值是否包含某个特定的值得

`[attribute~=value]` CSS2

`[attribute*=value]` CSS3

两者之间的区别:

CSS2中的只能找到独立的单词, 也就是包含value,并且value是被空格隔开的

CSS3中的只要包含value就可以找到

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>21-属性选择器下</title>
    <style>
        /*
        img[alt^=abc]{
            color: red;
        }
        */
        /* 
        img[alt|=abc]{
            color: red;
        }
        img[alt$=abc]{
            color: blue;
        }
        */
        /*
        img[alt*=abc]{
            color: red;
        }
        */
        img[alt~=abc]{
            color: red;
        }
    </style>
</head>
<body>
<!--
<img src="" alt="abcdef">
<img src="" alt="abc-www">
<img src="" alt="abc ppp">
<img src="" alt="defabc">
<img src="" alt="ppp abc">
<img src="" alt="www-abc">
<img src="" alt="qq">
<img src="" alt="yy">
-->

<img src="" alt="abcwwwmmm">
<img src="" alt="wwwmmmabc">
<img src="" alt="wwwabcmmm">
<img src="" alt="www-abc-mmm">
<img src="" alt="www abc mmm">
<img src="" alt="qq">

</body>
</html>
```



### - 通配符选择器

1.什么是通配符选择器?

作用: 给当前界面上所有的标签设置属性



格式:

```css
*{
    font-size:12px;
}
```

注意点:

由于通配符选择器是设置界面上所有的标签的属性, 所以在设置之前会遍历所有的标签, 如果当前界面上的标签比较多, 那么性能就会比较差, 所以在企业开发中一般不会使用通配符选择器





### - a标签的伪类选择器

1.通过我们的观察发现a标签存在一定的状态

​	1.1默认状态, 从未被访问过

​	1.2被访问过的状态

​	1.3鼠标长按状态

​	1.4鼠标悬停在a标签上状态



2.什么是a标签的伪类选择器?

​	a标签的伪类选择器是专门用来修改a标签不同状态的样式的



3.格式

`:link` 修改从未被访问过状态下的样式

`:visited` 修改被访问过的状态下的样式

`:hover` 修改鼠标悬停在a标签上状态下的样式

`:active` 修改鼠标长按状态下的样式



4.注意点

​	4.1a标签的伪类选择器可以单独出现也可以一起出现

​	4.2a标签的伪类选择器如果一起出现, 那么有严格的顺序要求

​		编写的顺序必须要个的遵守爱恨原则 love hate

​	4.3如果默认状态的样式和被访问过状态的样式一样, 那么可以缩写





## 文字属性

### - 文字样式

```css
font-style: normal;
```

规定文字样式的属性

格式：`font-style: italic;`

取值：

`normal` ： 正常的， 默认就是正常的

`italic` :  倾斜的

快捷键：

fs `font-style: italic;`

fsn `font-style: normal;`

### - 文字粗细

```css
font-weight:bold;
```

规定文字粗细的属性

格式： `font-weight: bold;`

单词取值:

`bold` 加粗

`bolder`  比加粗还要粗

`lighter` 细线， 默认就是细线

数字取值：

`100-900`之间整百的数字

快捷键

fw font-weight:;

fwb font-weight: bold;

fwbr  font-weight: bolder;

### - 文字大小

```css
font-size:30px;
```

规定文字大小的属性

格式：`font-size: 30px;`

单位：px（像素 pixel）

注意点： 通过font-size设置大小一定要带单位， 也就是一定要写px

快捷键

fz `font-size:;`

fz30 font-size: 30px;

### - 文字字体

```css
font-family:"楷体";
```



规定文字字体的属性

格式：`font-family:"楷体";`

注意点：

- 如果取值是中文， 需要用双引号或者单引号括起来

- 设置的字体必须是用户电脑里面已经安装的字体

快捷键

ff `font-family:;`

### - 缩写font

缩写格式:

```css
font: style weight size family;
```

例如:

```css
font:italic bold 10px "楷体";
```

注意点:

1.在这种缩写格式中有的属性值可以省略

`sytle`可以省略

`weight`可以省略

2.在这种缩写格式中`style`和`weight`的位置可以交换

3.在这种缩写格式中有的属性值是不可以省略的

`size`不能省略

`family`不能省略

4.`size`和`family`的位置是不能顺便乱放的

size一定要写在family的前面, 而且size和family必须写在所有属性的最后



### - 内容补充

1.如果设置的字体不存在, 那么系统会使用默认的字体来显示

默认使用宋体



2.如果设置的字体不存在, 而我们又不想用默认的字体来显示怎么办?

可以给字体设置备选方案

格式:font-family:"字体1", "备选方案1", ...;



3.如果想给中文和英文分别单独设置字体, 怎么办?

但凡是中文字体, 里面都包含了英文

但凡是英文字体, 里面都没有包含中文

也就是说中文字体可以处理英文, 而英文字体不能处理中文

注意点: 如果想给界面中的英文单独设置字体, 那么英文的字体必须写在中文的前面



4.补充在企业开发中最常见的字体有以下几个

中文: 宋体/黑体/微软雅黑

英文: "Times New Roman"/Arial



还需要知道一点, 就是并不是名称是英文就一定是英文字体

因为中文字体其实都有自己的英文名称, 所以是不是中文字体主要看能不能处理中文

宋体 SimSun

黑体 SimHei

微软雅黑 Microsoft YaHei



## 文本属性

### - 文本装饰

文本装饰的属性

格式:`text-decoration: underline;`

取值:

`underline` 下划线

`line-through` 删除线

`overline` 上划线

`none` 什么都没有, 最常见的用途就是用于去掉超链接的下划线

快捷键:

td  `text-decoration: none;`

tdu `text-decoration: underline;`

tdl `text-decoration: line-through;`

tdo `text-decoration: overline;`

### - 文本对齐

文本水平对齐的属性

格式: `text-align: right;`

取值:

`left` 左

`right` 右

`center` 中

快捷键

ta `text-align: left;`

tar `text-align: right;`

tac `text-align: center;`



### - 文本缩进

文本缩进的属性

格式: `text-indent: 2em;`

取值:

2em, 其中em是单位, 一个em代表缩进一个文字的宽度

快捷键

ti `text-indent:;`

ti2e `text-indent: 2em;`



## 颜色属性(文字颜色)

```css
color: #769abb;
```



1.在CSS中如何通过color属性来修改文字颜色

格式: `color: 值;`

取值:

**英文单词**

一般情况下常见的颜色都有对应的英文单词, 但是英文单词能够表达的颜色是有限制的, 也就是说不是所有的颜色都能够通过英文单词来表达

**rgb**

rgb其实就是三原色, 其中r(red 红色) g(green 绿色) b(blue 蓝色)

格式: `rgb(0,0,0)`

那么这个格式中的

第一个数字就是用来设置三原色的光源元件红色显示的亮度

第二个数字就是用来设置三原色的光源元件绿色显示的亮度

第三个数字就是用来设置三原色的光源元件蓝色显示的亮度

这其中的每一个数字它的取值是0-255之前, 0代表不发光, 255代表发光, 值越大就越亮

红色: rgb(255,0,0);

绿色: rgb(0,255,0);

蓝色: rgb(0,0,255);

黑色: rgb(0,0,0);

白色: rgb(255,255,255);

在前端开发中其实并不常用黑色

只要让红色/绿色/蓝色的值都一样就是灰色

而且如果这三个值越小那么就越偏黑色, 越大就越偏白色

例如: `color: rgb(200,200,200);`



**rgba**

rgba中的rgb和前面讲解的一样, 只不过多了一个a

那么这个a呢代表透明度, 取值是0-1, 取值越小就越透明

例如: `color: rgba(255,0,0,0.2);`



**十六进制**

在前端开发中通过十六进制来表示颜色, 其实本质就是RGB

十六进制中是通过每两位表示一个颜色

例如: #FFEE00 FF表示R EE表示G 00表示B





## div和span标签

1.什么是div?

作用: 一般用于配合css完成网页的基本布局



2.什么是span?

作用: 一般用于配合css修改网页中的一些局部信息



3.div和span有什么区别?

​	1.div会单独的占领一行,而span不会单独占领一行

​	2.div是一个容器级的标签, 而span是一个文本级的标签



4.容器级的标签和文本级的标签的区别?

容器级的标签中可以嵌套其它所有的标签

文本级的标签中只能嵌套文字/图片/超链接



容器级的标签

`div h ul ol dl li dt dd ...`

文本级的标签

`span p buis strong em ins del ...`



注意点:

哪些标签是文本级的哪些标签是容器级的, 我们不用刻意去记忆, 在企业开发中一般情况下要嵌套都是嵌套在div中, 或者按照组标签来嵌套





## CSS元素的显示模式



### -  标签的显示模式

在HTML中HTML将所有的标签分为两类, 分别是容器级和文本级

在CSS中CSS也将所有的标签分为两类, 分别是块级元素和行内元素



1.什么是块级元素, 什么是行内元素?

块级元素会独占一行

行内元素不会独占一行



容器级的标签

`div h ul ol dl li dt dd ...`

文本级的标签

`span p buis stong em ins del ...`



块级元素

`p div h ul ol dl li dt dd`

行内元素

`span buis strong em ins del`



2.块级元素和行内元素的区别?

​	2.1块级元素

​		独占一行

​		如果没有设置宽度, 那么默认和父元素一样宽

​		如果设置了宽高, 那么就按照设置的来显示



​	2.2行内元素

​		不会独占一行

​		如果没有设置宽度, 那么默认和内容一样宽

​		行内元素是不可以设置宽度和高度的



​	2.3行内块级元素

​		为了能够让元素`既能够不独占一行, 又可以设置宽度和高度`, 那么就出现了行内块级元素



### - 显示模式转换

1.如何转换CSS元素的显示模式?

设置元素的display属性



2.display取值

`block` 块级

`inline` 行内

`inline-block` 行内块级



3.快捷键

di `display: inline;`

db `display: block;`

dib `display: inline-block;`





## 背景颜色

1.如何设置标签的背景颜色?

在CSS中有一个background-color:属性, 就是专门用来设置标签的背景颜色的

取值:

具体单词

`rgb`

`rgba`

`十六进制`

快捷键:

bc `background-color: #fff;`



## 背景图片

1.如何设置背景图片?

在CSS中有一个叫做`background-image: url();`的属性, 就是专门用于设置背景图片的

快捷键:

bi `background-image: url();`

注意点:

1.图片的地址必须放在url()中, 图片的地址可以是本地的地址, 也可以是网络的地址

2.如果图片的大小没有标签的大小大, 那么会自动在水平和垂直方向平铺来填充

3.如果网页上出现了图片, 那么浏览器会再次发送请求获取图片



### - 平铺方式



1.如何控制背景图片的平铺方式?

在CSS中有一个`background-repeat`属性, 就是专门用于控制背景图片的平铺方式的

取值:

`repeat` 默认, 在水平和垂直都需要平铺

`no-repeat` 在水平和垂直都不需要平铺

`repeat-x` 只在水平方向平铺

`repeat-y` 只在垂直方向平铺



快捷键

bgr `background-repeat:`



应用场景:

可以通过背景图片的平铺来降低图片的大小, 提升网页的访问速度



### - 背景定位

1.如何控制背景图片的位置?

在CSS中有一个叫做`background-position:`属性, 就是专门用于控制背景图片的位置



2.格式:

`background-position:` 水平方向 垂直方向;



3.取值

2.1具体的方位名词

水平方向: `left center right`

垂直方向: `top center bottom`



2.2具体的像素

例如: `background-position: 100px 200px;`

记住一定要写单位, 也就是一定要写px

记住具体的像素是可以接收负数的

快捷键:

bp `background-position: 0 0;`

注意点:

同一个标签可以同时设置背景颜色和背景图片, 如果颜色和图片同时存在, 那么**图片会覆盖颜色**



### - 缩写

1.背景属性缩写的格式

​	`background: 背景颜色 背景图片 平铺方式 关联方式 定位方式;`

​	快捷键:

​		bg+ `background: #fff url() 0 0 no-repeat;`

2.注意点：

​	background属性中， 任何一个属性都可以被省略

3.什么是背景关联方式？

​		默认情况下背景图片会随着滚动条的滚动而滚动， 如果不想让背景图片随着滚动条的滚动而滚动， 那么我们就可以修改背景图片和		滚动条的关联方式



4.如何修改背景关联方式？

​	在CSS中有一个叫做`background-attachment`的属性， 这个属性就是专门用于修改关联方式的

​	格式

​		`	background-attachment：scroll;`

​	取值：

​		`scroll` 默认值， 会随着滚动条的滚动而滚动

​		`	fixed` 不会随着滚动条的滚动而滚动

​	快捷键:

​		ba `background-attachment:;`



### - 背景图片与插入图片的区别

1.背景图片和插入图片区别?

​		背景图片仅仅是一个装饰, 不会占用位置

​		插入图片会占用位置



​		背景图片有定位属性, 所以可以很方便的控制图片的位置

​		插入图片没有定位属性, 所有控制图片的位置不太方便



​		插入图片的语义比背景图片的语义要强, 所以在企业开发中如果你的图片想被搜索引擎收录, 那么推荐使用插入图片



## 背景尺寸

什么是背景尺寸属性

背景尺寸属性是CSS3中新增的一个属性, 专门用于设置背景图片大小



### - 背景图片

默认

```css
background: url("images/dog.jpg") no-repeat;
```



具体像素

```css
background: url("images/dog.jpg") no-repeat;
/*
第一个参数:宽度
第二个参数:高度
*/
background-size:200px 100px;
```



百分比

```css
background: url("images/dog.jpg") no-repeat;
/*
第一个参数:宽度
第二个参数:高度
*/
background-size:100% 80%;
```



宽度等比拉伸

```css
background: url("images/dog.jpg") no-repeat;
/*
第一个参数:宽度
第二个参数:高度
*/
background-size:auto 100px;
```



高度等比拉伸

```css
background: url("images/dog.jpg") no-repeat;
/*
第一个参数:宽度
第二个参数:高度
*/
background-size:100px auto;
```



cover

```css
background: url("images/dog.jpg") no-repeat;
/*
cover含义:
1.告诉系统图片需要等比拉伸
2.告诉系统图片需要拉伸到宽度和高度都填满元素
*/
background-size:cover;
```



contain

```css
background: url("images/dog.jpg") no-repeat;
/*
cover含义:
1.告诉系统图片需要等比拉伸
2.告诉系统图片需要拉伸到宽度或高度都填满元素
*/
background-size:contain;
```



### - 背景图片定位区域属性

告诉系统背景图片从什么区域开始显示

默认情况下就是从padding区域开始显示

```css
background-origin:padding-box;
```

```css
background-origin:border-box;
```

```css
background-origin:content-box;
```



### - 多重背景图片

多张背景图片之间用逗号隔开即可

注意点:

先添加的背景图片会盖住后添加的背景图片

建议在编写多重背景时拆开编写

```css
/*background: url("images/animal1.png") no-repeat left top,url("images/animal2.png") no-repeat right top,url("images/animal3.png") no-repeat left bottom,url("images/animal4.png") no-repeat right bottom,url("images/animal5.png") no-repeat center center;*/
background-image: url("images/animal1.png"),url("images/animal2.png"),url("images/animal3.png");
background-repeat: no-repeat, no-repeat, no-repeat;
background-position: left top, right top, left bottom;
```





## 边框效果

### - 边框属性

1.什么边框?

​	边框就是环绕在标签宽度和高度周围的线条

2.边框属性的格式

​	2.1连写(同时设置四条边的边框)

​		`border: 边框的宽度 边框的样式 边框的颜色;`

​		快捷键:

​			bd+ `border: 1px solid #000;`

​		注意点:

​			1.连写格式中颜色属性可以省略, 省略之后默认就是黑色

​			2.连写格式中样式不能省略, 省略之后就看不到边框了

​			3.连写格式中宽度可以省略, 省略之后还是可以看到边框

​	2.2连写(分别设置四条边的边框)

​		`border-top`: 边框的宽度 边框的样式 边框的颜色;

​		`border-right`: 边框的宽度 边框的样式 边框的颜色;

​		`border-bottom`: 边框的宽度 边框的样式 边框的颜色;

​		`border-left`: 边框的宽度 边框的样式 边框的颜色;

​	快捷键:

​		bt+ border-top: 1px solid #000;

​		br+

​		bb+

​		bl+

```css
/*border: 5px solid blue;*/
/*border: 5px solid;*/
/*border: 5px blue;*/
/*border: solid blue;*/

border-top:5px solid blue;
border-right:10px dashed green;
border-bottom:15px dotted purple;
border-left:20px double pink;
```



​	2.3连写(分别设置四条边的边框)

```css
border-width: 上 右 下 左;
border-style: 上 右 下 左;
border-color: 上 右 下 左;
```

注意点:

1.这三个属性的取值是按照顺时针来赋值, 也就是按照上右下左来赋值, 而不是按照日常生活中的上下左右

2.这三个属性的取值省略时的规律

​	2.1`上 右 下 左 > 上 右 下 >` 左边的取值和右边的一样

​	2.2`上 右 下 左 > 上 右 >` 左边的取值和右边的一样 下边的取值和上边一样

​	2.3 `上 右 下 左 > 上 >` 右下左边取值和上边一样

3.非连写(方向+要素)

```css
border-left-width: 20px;
border-left-style: double;
border-left-color: pink;
```



### - 边框圆角

边框圆角的作用:
将原始的直角变为圆角
格式:

```css
border-radius: 100px 0px 0px 0px;
```

第一个参数: 指定左上角的半径

按照左上、右上、右下、左下的顺序



如果省略一个参数, 会变成对角的值

```css
border-radius: 100px 80px 40px;
```


如果省略两个参数, 会变成对角的值

```css
border-radius: 100px 80px;
```


如果省略三个参数, 其它角会和它一样

```css
border-radius: 100px;
```


如果指定的半径是当前元素宽高的一半, 那么就是一个圆形

```css
border-radius: 100px;
```





```css
/*单独指定某一个角的半径*/
border-top-left-radius: 100px;
border-top-right-radius: 100px;
border-bottom-left-radius: 100px;
border-bottom-right-radius: 100px;

/*如果传递了两个参数, 那么第一个参数代表水平方向的半径, 第二个参数代表垂直方向的半径*/
border-top-left-radius: 100px 50px;
border-top-right-radius: 100px 50px;
border-bottom-left-radius: 100px 50px;
border-bottom-right-radius: 100px 50px;


border-top-left-radius: 50%;
border-top-right-radius: 50%;
border-bottom-left-radius: 50%;
border-bottom-right-radius: 50%;

/*
斜杠之前的代表左上/右上/右下/左下的水平方向的半径
斜杠之后的代表左上/右上/右下/左下的垂直方向的半径
*/
border-radius: 100px 100px 100px 100px/50px 50px 50px 50px;
border-radius: 100px/50px;
```



## 边距效果

### - 内边距

1.什么是内边距?

​	边框和内容之间的距离就是内边距



2.格式

​	2.1非连写

```css
padding-top: ;
padding-right: ;
padding-bottom: ;
padding-left: ;
```

​	2.2连写

```css
padding: 上 右 下 左;
```



3.这三个属性的取值省略时的规律

​	3.1`上 右 下 左 > 上 右 下 >` 左边的取值和右边的一样

​	3.2`上 右 下 左 > 上 右 >` 左边的取值和右边的一样 下边的取值和上边一样

​	3.3`上 右 下 左 > 上 >` 右下左边取值和上边一样

注意点:

​	1.给标签设置内边距之后, 标签占有的宽度和高度会发生变化

​	2.给标签设置内边距之后, 内边距也会有背景颜色



### - 外边距

1.什么是外边距?

标签和标签之间的距离就是外边距



2.格式

​	2.1非连写

```css
margin-top: ;
margin-right: ;
margin-bottom: ;
margin-left: ;
```

​	2.2连写

```css		
margin: 上 右 下 左;
```

3.这三个属性的取值省略时的规律

​	3.1上 右 下 左 > 上 右 下 > 左边的取值和右边的一样

​	3.2上 右 下 左 > 上 右 > 左边的取值和右边的一样 下边的取值和上边一样

​	3.3上 右 下 左 > 上 > 右下左边取值和上边一样

注意点:

​	外边距的那一部分是没有背景颜色的



外边距合并现象: 

在默认布局的垂直方向上, 默认情况下外边距是不会叠加的, 会出现合并现象, 谁的外边距比较大就听谁的



## 盒子模型

1.什么是CSS盒子模型?

CSS盒子模型仅仅是一个形象的比喻, HTML中所有的标签都是盒子

结论

1.在HTML中所有的标签都可以设置

​	宽度/高度  == 指定可以存放内容的区域

​	内边距  == 填充物

​	边框  == 手机盒子自己

​	外边距  == 盒子和盒子之间的间隙

### - 盒子尺寸

1.内容的宽度和高度

​	就是通过width/height属性设置的宽度和高度

2.元素的宽度和高度

​	宽度 = 左边框 + 左内边距 + width + 右内边距 + 右边框

​	高度 同理可证

3.元素空间的宽度和高度

​	宽度 = 左外边距 + 左边框 + 左内边距 + width + 右内边距 + 右边框 + 右外边距

​	高度 同理可证

### - box-sizing (css3)

1.CSS3中新增了一个box-sizing属性, 这个属性可以保证我们给盒子新增padding和border之后, 盒子元素的宽度和高度不变
2.box-sizing属性是如何保证增加padding和border之后, 盒子元素的宽度和高度不变
	和我们前面学习的原理一样, 增加padding和border之后要想保证盒子元素的宽高不变, 那么就必须减去一部分内容的宽度和高度
3.box-sizing取值
	3.1`content-box`
		元素的宽高 = 边框 + 内边距 + 内容宽高
	3.2`border-box`
		元素的宽高 = width/height的宽高



### - 盒子居中

1.`text-align:center;`和`margin:0 auto;`区别

`text-align: center;`作用设置盒子中存储的文字/图片水平居中

`margin:0 auto;`作用让盒子自己水平居中



### - 清空默认边距

1.为什么要清空默认边距(外边距和内边距)

在企业开发中为了更好的控制盒子的宽高和计算盒子的宽高等等, 所以在企业开发中, 编写代码之前第一件事情就是清空默认的边距

2.如何清空默认的边距

格式

```css
*{
    margin: 0;
    padding: 0;
}
```

3.注意点

通配符选择器会找到(遍历)当前界面中所有的标签, 所以性能不好

企业开发中可以从这个网址中拷贝

http://yui.yahooapis.com/3.18.1/build/cssreset/cssreset-min.css



## 行高

1.什么是行高?

在CSS中所有的行都有自己的行高

注意点:

行高和盒子高不是同一个概念

行高指的是每行内容的高度

盒子高指的是元素的高度

规律:

1.文字在行高中默认是垂直居中的

2.在企业开发中我们经常将盒子的高度和行高设置为一样, 那么这样就可以保证一行文字在盒子的高度中是垂直居中的

简而言之就是: 要想一行文字在盒子中垂直居中, 那么只需要设置这行文字的"行高等于盒子的高"即可

3.在企业开发中如果一个盒子中有多行文字, 那么我们就不能使用设置行高等于盒子高来实现让文字垂直居中, 只能通过设置padding来让文字居中



注意: 

在企业开发中, 如果一个盒子中存储的是文字, 那么一般情况下我们会以盒子左边的内边距为基准, 不会以右边的内边距为基准, 因为这个右边的内边距有误差

右边内边距的误差从何而来? 因为右边如果放不下一个文字, 那么文字就会换行显示, 所以文字和内边距之间的距离就有了误差

顶部的内边距并不是边框到文字顶部的距离, 而是边框到行高顶部的距离





## 阴影效果

### - 盒子阴影

```css
/*box-shadow: 水平偏移 垂直偏移 模糊度 阴影扩展 阴影颜色 内外阴影;*/

box-shadow: 10px 10px 10px 10px skyblue;
box-shadow: 10px 10px 10px 10px skyblue inset;
```

注意点

- 盒子的阴影分为内外阴影, 默认情况下就是外阴影

- 快速添加阴影只需要编写三个参数即可`box-shadow: 水平偏移 垂直偏移 模糊度;`

默认情况下阴影的颜色和盒子内容的颜色一致

### - 文字阴影

```css
/* text-shadow: 水平偏移 垂直偏移 模糊度 阴影颜色 ;*/
text-shadow: 10px 10px 10px black;
```

## 过渡模块

过渡三要素

- 必须要有属性发生变化

- 必须告诉系统哪个属性需要执行过渡效果

- 必须告诉系统过渡效果持续时长

注意点

- 当多个属性需要同时执行过渡效果时用逗号隔开即可

- `transition-property: width, background-color;`

- `transition-duration: 5s, 5s;`

过渡属性

```css
// 属性 宽度|背景色
transition-property: width, background-color;
// 过渡时间	宽度过渡5s|背景色过渡5s
transition-duration: 5s, 5s;

// 其他属性
/*告诉系统延迟多少秒之后才开始过渡动画*/
transition-delay: 2s;


指定过渡的速度曲线
transition-timing-function 属性规定过渡效果的速度曲线。

transition-timing-function 属性可接受以下值：

ease - 规定过渡效果，先缓慢地开始，然后加速，然后缓慢地结束（默认）
linear - 规定从开始到结束具有相同速度的过渡效果
ease-in -规定缓慢开始的过渡效果
ease-out - 规定缓慢结束的过渡效果
ease-in-out - 规定开始和结束较慢的过渡效果
cubic-bezier(n,n,n,n) - 允许您在三次贝塞尔函数中定义自己的值
```

连写

过渡连写格式

- transition: 过渡属性 过渡时长 运动速度 延迟时间;

过渡连写注意点

- 和分开写一样, 如果想给多个属性添加过渡效果也是用逗号隔开即可

- 连写的时可以省略后面的两个参数, 因为只要编写了前面的两个参数就已经满足了过渡的三要素

- 如果多个属性运动的速度/延迟的时间/持续时间都一样, 那么可以简写为`transition:all 0s;`

```css
transition-property: width;
transition-duration: 5s;
/*transition: width 5s linear 0s,background-color 5s linear 0s;*/
transition: background-color 5s linear 0s;

// 等价于
transition: width 5s,background-color 5s liner 0s;
```





## 2d 转换模块



### - 基本属性 

`transform`

- 旋转 `rotate(45deg);` 

- 移动 `translate(100px, 0px);`

- 缩放 `scale(1.5);`

- 合并写法 `rotate(45deg) translate(100px, 0px) scale(1.5, 1.5);`

```css
/* 旋转角度
其中deg是单位, 代表多少度
*/
transform: rotate(45deg);

/* 移动位置
 第一个参数:水平方向
 第二个参数:垂直方向
*/
transform: translate(100px, 0px);


/* 缩放
第一个参数:水平方向
第二个参数:垂直方向
注意点:
 如果取值是1, 代表不变
 如果取值大于1, 代表需要放大
 如果取值小于1, 代表需要缩小
 如果水平和垂直缩放都一样, 那么可以简写为一个参数
*/
/*transform: scale(0.5, 0.5);*/
transform: scale(1.5);


/*
注意点:
1.如果需要进行多个转换, 那么用空格隔开
2.2D的转换模块会修改元素的坐标系, 所以旋转之后再平移就不是水平平移的
*/
transform: rotate(45deg) translate(100px, 0px) scale(1.5, 1.5);
/*transform: translate(100px, 0px);*/

```



### - 形变中心 

`transform-origin`

默认情况下所有的元素都是以自己的中心点作为参考来旋转的, 我们可以通过形变中心点属性来修改它的参考点

第一个参数:水平方向
第二个参数:垂直方向

取值有三种形式

- 具体像素
- 百分比
- 特殊关键字

```css
/*transform-origin: 200px 0px;*/
/*transform-origin: 50% 50%;*/
/*transform-origin: 0% 0%;*/
/*transform-origin: center center;*/
transform-origin: left top;
```

### - 旋转轴向

```css
/*默认情况下所有元素都是围绕Z轴进行旋转*/
transform: rotateZ(45deg);

transform: rotateX(45deg);

transform: rotateX(45deg);

/*
总结:
想围绕哪个轴旋转, 那么只需要在rotate后面加上哪个轴即可
*/
transform: rotateY(45deg);
```



## 3d转换效果



**什么是2D和3D**

2D就是一个平面, 只有宽度和高度, 没有厚度

3D就是一个立体, 有宽度和高度, 还有厚度

默认情况下所有的元素都是呈2D展现的



**如何让某个元素呈3D展现**

和透视一样, 想看到某个元素的3d效果, 只需要给他的父元素添加一个transform-style属性, 然后设置为preserve-3d即可

`transform-style: preserve-3d`



### - 正方体

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>107-3D转换模块-正方体</title>
    <style>
    *{
        margin: 0;
        padding: 0;
    }
    ul{
        width: 200px;
        height: 200px;
        border: 1px solid #000;
        box-sizing: border-box;
        margin: 100px auto;
        position: relative;
        transform: rotateY(0deg) rotateX(0deg);
        transform-style: preserve-3d;
    }
    ul li{
        list-style: none;
        width: 200px;
        height: 200px;
        font-size: 60px;
        text-align: center;
        line-height: 200px;
        position: absolute;
        left: 0;
        top: 0;
    }
    ul li:nth-child(1){
        background-color: red;
        transform: translateX(-100px) rotateY(90deg);
    }
    ul li:nth-child(2){
        background-color: green;
        transform: translateX(100px) rotateY(90deg);
    }
    ul li:nth-child(3){
        background-color: blue;
        transform: translateY(-100px) rotateX(90deg);
    }
    ul li:nth-child(4){
        background-color: yellow;
        transform: translateY(100px) rotateX(90deg);
    }
    ul li:nth-child(5){
        background-color: purple;
        transform: translateZ(-100px);
    }
    ul li:nth-child(6){
        background-color: pink;
        transform: translateZ(100px);
    }

</style>
</head>
<body>
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
</ul>
</body>
</html>
```

### - 正方体终极

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>108-3D转换模块-正方体终极</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        ul{
            width: 200px;
            height: 200px;
            border: 1px solid #000;
            box-sizing: border-box;
            margin: 100px auto;
            position: relative;
            transform: rotateY(0deg) rotateX(0deg);
            transform-style: preserve-3d;
        }
        ul li{
            list-style: none;
            width: 200px;
            height: 200px;
            font-size: 60px;
            text-align: center;
            line-height: 200px;
            position: absolute;
            left: 0;
            top: 0;
        }
        ul li:nth-child(1){
            background-color: red;
            transform: rotateX(90deg) translateZ(100px);
        }
        ul li:nth-child(2){
            background-color: green;
            transform: rotateX(180deg) translateZ(100px);
        }
        ul li:nth-child(3){
            background-color: blue;
            transform: rotateX(270deg) translateZ(100px);
        }
        ul li:nth-child(4){
            background-color: yellow;
            transform: rotateX(360deg) translateZ(100px);
        }
        ul li:nth-child(5){
            background-color: purple;
            transform: translateX(-100px) rotateY(90deg);
        }
        ul li:nth-child(6){
            background-color: pink;
            transform: translateX(100px) rotateY(90deg);
        }

    </style>
</head>
<body>
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
</ul>
</body>
</html>
```

### - 长方形

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>109-3D转换模块-长方体</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        /*
        .father{
            width: 200px;
            height: 200px;
            border: 1px solid #000;
            margin: 100px auto;
        }
        .son{
            width: 200px;
            height: 200px;
            background-color: rgba(255,0,0,0.3);
            transform: scale(1.5, 1);
        }
        */
        ul{
            width: 200px;
            height: 200px;
            border: 1px solid #000;
            box-sizing: border-box;
            margin: 100px auto;
            position: relative;
            transform: rotateY(0deg) rotateX(0deg);
            transform-style: preserve-3d;
        }
        ul li{
            list-style: none;
            width: 200px;
            height: 200px;
            font-size: 60px;
            text-align: center;
            line-height: 200px;
            position: absolute;
            left: 0;
            top: 0;
        }
        ul li:nth-child(1){
            background-color: red;
            transform: rotateX(90deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(2){
            background-color: green;
            transform: rotateX(180deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(3){
            background-color: blue;
            transform: rotateX(270deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(4){
            background-color: yellow;
            transform: rotateX(360deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(5){
            background-color: purple;
            transform: translateX(-200px) rotateY(90deg);
        }
        ul li:nth-child(6){
            background-color: pink;
            transform: translateX(200px) rotateY(90deg);
        }
    </style>
</head>
<body>
<!--
<div class="father">
    <div class="son"></div>
</div>
-->
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
    <li>6</li>
</ul>
</body>
</html>
```



### - 练习

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>110-3D转换模块-练习</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        body{
            /*想看到整个立方的近大远小效果, 就给ul的父元素添加透视*/
            perspective: 500px;
        }
        ul{
            width: 200px;
            height: 200px;
            box-sizing: border-box;
            margin: 100px auto;
            position: relative;
            transform: rotateY(0deg) rotateX(0deg);
            transform-style: preserve-3d;
            animation: sport 5s linear 0s infinite normal;
        }
        ul li{
            list-style: none;
            width: 200px;
            height: 200px;
            font-size: 60px;
            text-align: center;
            line-height: 200px;
            position: absolute;
            left: 0;
            top: 0;
        }
        ul li:nth-child(1){
            background-color: red;
            transform: rotateX(90deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(2){
            background-color: green;
            transform: rotateX(180deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(3){
            background-color: blue;
            transform: rotateX(270deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(4){
            background-color: yellow;
            transform: rotateX(360deg) translateZ(100px) scale(2, 1);
        }
        ul li:nth-child(5){
            background-color: purple;
            transform: translateX(-200px) rotateY(90deg);
        }
        ul li:nth-child(6){
            background-color: pink;
            transform: translateX(200px) rotateY(90deg);
        }
        ul li img{
            /*
            注意点:
            只要父元素被拉伸了,子元素也会被拉伸
            */
            width: 200px;
            height: 200px;
        }
        @keyframes sport {
            from{
                transform: rotateX(0deg);
            }
            to{
                transform: rotateX(360deg);
            }
        }
    </style>
</head>
<body>
<ul>
    <li><img src="images/banner1.jpg" alt=""></li>
    <li><img src="images/banner2.jpg" alt=""></li>
    <li><img src="images/banner3.jpg" alt=""></li>
    <li><img src="images/banner4.jpg" alt=""></li>
    <li></li>
    <li></li>
</ul>
</body>
</html>
```



### - 播放器上

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>111-3D播放器上</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        body{
            background: url("images/jacky/bg.jpg") no-repeat;
            background-size:cover;
        }
        ul{
            width: 200px;
            height: 200px;
            /*background-color: red;*/
            position: absolute;
            bottom: 100px;
            left: 50%;
            margin-left:-100px;
            transform-style: preserve-3d;
            /*transform: rotateX(-10deg);*/
            animation: sport 6s linear 0s infinite normal;
        }
        ul li{
            list-style: none;
            width: 200px;
            height: 200px;
            font-size: 60px;
            text-align: center;
            line-height: 200px;
            position: absolute;
            left: 0;
            top: 0;
        }
        ul li:nth-child(1){
            background-color: green;
            transform: rotateY(60deg) translateZ(200px);
        }
        ul li:nth-child(2){
            background-color: blue;
            transform: rotateY(120deg) translateZ(200px);
        }
        ul li:nth-child(3){
            background-color: yellow;
            transform: rotateY(180deg) translateZ(200px);
        }
        ul li:nth-child(4){
            background-color: pink;
            transform: rotateY(240deg) translateZ(200px);
        }
        ul li:nth-child(5){
            background-color: purple;
            transform: rotateY(300deg) translateZ(200px);
        }
        ul li:nth-child(6){
            background-color: gold;
            transform: rotateY(360deg) translateZ(200px);
        }
        ul li img{
            width: 200px;
            height: 200px;
            border: 5px solid skyblue;
            box-sizing: border-box;
        }
        @keyframes sport {
            from{
                /*
                注意点:
                1.动画中如果有和默认样式中同名的属性, 会覆盖默认样式中同名的属性
                2.在编写动画的时候, 固定不变的值写在前面, 需要变化的值写在后面
                */
                transform: rotateX(-10deg) rotateY(0deg);
            }
            to{
                transform: rotateX(-10deg) rotateY(360deg);
            }
        }
    </style>
</head>
<body>
<ul>
    <li><img src="images/jacky/1.png" alt=""></li>
    <li><img src="images/jacky/2.jpg" alt=""></li>
    <li><img src="images/jacky/3.jpg" alt=""></li>
    <li><img src="images/jacky/4.gif" alt=""></li>
    <li><img src="images/jacky/5.jpg" alt=""></li>
    <li><img src="images/jacky/6.jpg" alt=""></li>
</ul>
</body>
</html>
```



### - 播放器下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>111-3D播放器下</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        body{
            background: url("images/jacky/bg.jpg") no-repeat;
            background-size:cover;
            overflow: hidden;
        }
        ul{
            width: 200px;
            height: 200px;
            /*background-color: red;*/
            position: absolute;
            bottom: 100px;
            left: 50%;
            margin-left:-100px;
            transform-style: preserve-3d;
            /*transform: rotateX(-10deg);*/
            animation: sport 6s linear 0s infinite normal;
        }
        ul li{
            list-style: none;
            width: 200px;
            height: 200px;
            font-size: 60px;
            text-align: center;
            line-height: 200px;
            position: absolute;
            left: 0;
            top: 0;
            background-color: black;
        }
        ul li:nth-child(1){
            transform: rotateY(60deg) translateZ(200px);
        }
        ul li:nth-child(2){
            transform: rotateY(120deg) translateZ(200px);
        }
        ul li:nth-child(3){
            transform: rotateY(180deg) translateZ(200px);
        }
        ul li:nth-child(4){
            transform: rotateY(240deg) translateZ(200px);
        }
        ul li:nth-child(5){
            transform: rotateY(300deg) translateZ(200px);
        }
        ul li:nth-child(6){
            transform: rotateY(360deg) translateZ(200px);
        }
        ul li img{
            width: 200px;
            height: 200px;
            border: 5px solid skyblue;
            box-sizing: border-box;
        }
        ul:hover{
            animation-play-state: paused;
        }
        ul:hover li img{
            opacity: 0.5;
        }
        ul li:hover img{
            opacity: 1;
        }
        @keyframes sport {
            from{
                /*
                注意点:
                1.动画中如果有和默认样式中同名的属性, 会覆盖默认样式中同名的属性
                2.在编写动画的时候, 固定不变的值写在前面, 需要变化的值写在后面
                */
                transform: rotateX(-10deg) rotateY(0deg);
            }
            to{
                transform: rotateX(-10deg) rotateY(360deg);
            }
        }
        .heart{
            width: 173px;
            height: 157px;
            position: absolute;
            left: 100px;
            bottom: 100px;
            animation: move 10s linear 0s infinite normal;
        }
        @keyframes move {
            0%{
                left: 100px;
                bottom: 100px;
                opacity: 1;
            }
            20%{
                left: 300px;
                bottom: 300px;
                opacity: 0;
            }
            40%{
                left: 500px;
                bottom: 500px;
                opacity: 1;
            }
            60%{
                left: 800px;
                bottom: 300px;
                opacity: 0;
            }
            80%{
                left: 1200px;
                bottom: 100px;
                opacity: 1;
            }
            100%{
                left: 800px;
                bottom: -200px;
            }
        }
    </style>
</head>
<body>
<ul>
    <li><img src="images/jacky/1.png" alt=""></li>
    <li><img src="images/jacky/2.jpg" alt=""></li>
    <li><img src="images/jacky/3.jpg" alt=""></li>
    <li><img src="images/jacky/4.gif" alt=""></li>
    <li><img src="images/jacky/5.jpg" alt=""></li>
    <li><img src="images/jacky/6.jpg" alt=""></li>
</ul>
<img src="images/jacky/xin.png" class="heart">
<audio src="images/jacky/江哥最爱的歌.mp3" autoplay="autoplay" loop="loop"></audio>
</body>
</html>
```



## 动画模块

过渡和动画之间的异同

不同点

- 过渡必须人为的触发才会执行动画

- 动画不需要人为的触发就可以执行动画

相同点

- 过渡和动画都是用来给元素添加动画的

- 过渡和动画都是系统新增的一些属性

- 过渡和动画都需要满足三要素才会有动画效果

### - 基本属性

```css
/*1.告诉系统需要执行哪个动画*/
animation-name: lnj;

/*2.告诉系统我们需要自己创建一个名称叫做lnj的动画*/
@keyframes lnj {
    from{
        margin-left: 0;
    }
    to{
        margin-left: 500px;
    }
}

/*3.告诉系统动画持续的时长*/
animation-duration: 3s;
```

```css
div{
    width: 100px;
    height: 50px;
    background-color: red;
    /*transition-property: margin-left;*/
    /*transition-duration: 3s;*/
    /*1.告诉系统需要执行哪个动画*/
    animation-name: lnj;
    /*3.告诉系统动画持续的时长*/
    animation-duration: 3s;
}
/*2.告诉系统我们需要自己创建一个名称叫做lnj的动画*/
@keyframes lnj {
    from{
        margin-left: 0;
    }
    to{
        margin-left: 500px;
    }
}
```

### - 其他属性

告诉系统多少秒之后开始执行动画

```css
animation-delay: 2s;
```


告诉系统动画执行的速度

```css
animation-timing-function: linear;
```


告诉系统动画需要执行几次

```css
animation-iteration-count: 3;
```

告诉系统是否需要执行往返动画

取值:

- normal, 默认的取值, 执行完一次之后回到起点继续执行下一次
- alternate, 往返动画, 执行完一次之后往回执行下一次

```css
animation-direction: alternate;
```

```css
div {
    width: 100px;
    height: 50px;
    background-color: red;
    animation-name: sport;
    animation-duration: 2s;
    /*告诉系统多少秒之后开始执行动画*/
    /*animation-delay: 2s;*/
    /*告诉系统动画执行的速度*/
    animation-timing-function: linear;
    /*告诉系统动画需要执行几次*/
    animation-iteration-count: 3;
    /*告诉系统是否需要执行往返动画
    取值:
    normal, 默认的取值, 执行完一次之后回到起点继续执行下一
    alternate, 往返动画, 执行完一次之后往回执行下一次
    */
    animation-direction: alternate;
}
@keyframes sport {
    from{
        margin-left: 0;
    }
    to{
        margin-left: 500px;
    }
}
div:hover{
    /*
    告诉系统当前动画是否需要暂停
    取值:
    running: 执行动画
    paused: 暂停动画
    */
    animation-play-state: paused;
}
```

### - 动画阶段

```css
.box1 {
    width: 100px;
    height: 50px;
    background-color: red;
    position: absolute;
    left: 0;
    top: 0;
    animation-name: sport;
    animation-duration: 5s;
}
@keyframes sport {
    0%{
        left: 0;
        top: 0;
    }
    25%{
        left: 300px;
        top: 0;
    }
    50%{
        left: 300px;
        top: 300px;
    }
    75%{
        left: 0;
        top: 300px;
    }
    100%{
        left: 0;
        top: 0;
    }
}
.box2{
    width: 200px;
    height: 200px;
    background-color: blue;
    margin: 100px auto;
    animation-name: myRotate;
    animation-duration: 5s;
    animation-delay: 2s;
    /*
    通过我们的观察, 动画是有一定的状态的
    1.等待状态
    2.执行状态
    3.结束状态
    */
    /*
    animation-fill-mode作用:
    指定动画等待状态和结束状态的样式
    取值:
    none: 不做任何改变
    forwards: 让元素结束状态保持动画最后一帧的样式
    backwards: 让元素等待状态的时候显示动画第一帧的样式
    both: 让元素等待状态显示动画第一帧的样式, 让元素结束状态保持动画最后一帧的样式
    */
    /*animation-fill-mode: backwards;*/
    /*animation-fill-mode: forwards;*/
    animation-fill-mode: both;
}
@keyframes myRotate {
    0%{
        transform: rotate(10deg);
    }
    50%{
        transform: rotate(50deg);
    }
    100%{
        transform: rotate(70deg);
    }
}
```



### - 连写

动画模块连写格式

- `animation:动画名称 动画时长 动画运动速度 延迟时间 执行次数 往返动画;`

动画模块连写格式的简写

- `animation:动画名称 动画时长;`



```css
div {
    width: 100px;
    height: 50px;
    background-color: red;
    /*animation: move 3s linear 2s 1 normal;*/
    animation: move 3s;
}
@keyframes move {
    from{
        margin-left: 0;
    }
    to{
        margin-left: 500px;
        transform: rotate(45deg) translate(100px, 0px) scale(1.5, 1.5);   
    }
```



### - 云层效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>104-动画模块-云层效果</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        ul{
            height: 400px;
            background-color: skyblue;
            margin-top: 100px;
            animation: change 5s linear 0s infinite alternate;
            position: relative;
            overflow: hidden;
        }
        ul li{
            list-style: none;
            width: 400%;
            height: 100%;
            position: absolute;
            left: 0;
            top: 0;
        }
        ul li:nth-child(1){
            background-image: url("images/cloud_one.png");
            animation: one 30s linear 0s infinite alternate;
        }
        ul li:nth-child(2){
            background-image: url("images/cloud_two.png");
            animation: two 30s linear 0s infinite alternate;
        }
        ul li:nth-child(3){
            background-image: url("images/cloud_three.png");
            animation: three 30s linear 0s infinite alternate;
        }
        @keyframes change {
            from{
                background-color: skyblue;
            }
            to{
                background-color: black;
            }
        }
        @keyframes one {
            from{
                margin-left: 0;
            }
            to{
                margin-left: -100%;
            }
        }
        @keyframes two {
            from{
                margin-left: 0;
            }
            to{
                margin-left: -200%;
            }
        }
        @keyframes three {
            from{
                margin-left: 0;
            }
            to{
                margin-left: -300%;
            }
        }
    </style>
</head>
<body>
<ul>
    <li></li>
    <li></li>
    <li></li>
</ul>
</body>
</html>
```

### - 无限滚动

```css
/* infinite */
animation: move 10s linear 0s infinite normal;
```





## 颜色渐变

### 线性渐变



​      渐变不是一个新的属性, 而是一个取值

​      默认情况下线性渐变是从上至下的渐变

​      我们可以通过在颜色的前面添加to xxx修改渐变的方向

​      to top

​      to left

​      to right

```css
/*-->右上 红-->蓝*/
background-image: linear-gradient(to top right, red, blue);
```

除了可以通过关键字控制渐变的方向以外, 还可以通过角度来控制渐变的方向, 角度是按照顺时针旋转

```css
background-image: linear-gradient(200deg, red, blue);
```

30%前不渐变, 从30%开始慢慢渐变到下一个颜色

```css
background-image: linear-gradient(red 30%, blue);
background-image: linear-gradient(red 0%, red 30%,blue 30%, blue 60%, green 60%, green 100%);
background-image*: linear-gradient(to left ,red 0%, red 30%,blue 30%, blue 60%, green 60%, green 100%);
```

### 径向渐变

可以在颜色前面添加at和关键字来指定从什么位置开始渐变

```css
background-image: radial-gradient(at left top,red, blue);
```

除了可以通过关键字指定从什么位置开始渐变以外, 还可以通过指定具体像素来指定从什么位置开始渐变

```css
background-image: radial-gradient(at 50px 50px,red, blue);
```

也可以在颜色前面直接加上一个具体的像素来指定渐变的范围

```css
background-image: radial-gradient(150px ,red, blue);
```



编写规范:

如果既要指定扩散的范围, 又要指定起始位置, 那么把扩散的范围写在前面, 而起始位置写在后面



## 网页布局

什么是网页的布局方式?

​	网页的布局方式其实就是指浏览器是如何对网页中的元素进行排版的

1.标准流(文档流/普通流)排版方式

​	1.1其实浏览器默认的排版方式就是标准流的排版方式

​	1.2在CSS中将元素分为三类, 分别是块级元素/行内元素/行内块级元素

​	1.3 在标准流中有两种排版方式, 一种是垂直排版, 一种是水平排版

​		垂直排版, 如果元素是块级元素, 那么就会垂直排版

​		水平排版, 如果元素是行内元素/行内块级元素, 那么就会水平排版

2.浮动流排版方式

​	2.1浮动流是一种"半脱离标准流"的排版方式

​	2.2浮动流只有一种排版方式, 就是水平排版. 它只能设置某个元素左对齐或者右对齐

注意点:

​	1.浮动流中没有居中对齐, 也就是没有center这个取值

​	2.在浮动流中是不可以使用`margin: 0 auto;`

特点:

​	1.在浮动流中是不区分块级元素/行内元素/行内块级元素的

​		无论是级元素/行内元素/行内块级元素都可以水平排版

​	2.在浮动流中无论是块级元素/行内元素/行内块级元素都可以设置宽高

​	3.综上所述, 浮动流中的元素和标准流中的行内块级元素很像

### - 浮动元素脱标

1.什么是浮动元素的脱标?

​	脱标: 脱离标准流

​	当某一个元素浮动之后, 那么这个元素看上去就像被从标准流中删除了一样, 这个就是浮动元素的脱标

2.浮动元素脱标之后会有什么影响?

​	如果前面一个元素浮动了, 而后面一个元素没有浮动 , 那么这个时候前面一个元就会盖住后面一个元素



浮动元素排序规则

​	相同方向上的浮动元素, 先浮动的元素会显示在前面, 后浮动的元素会显示在后面

​	不同方向上的浮动元素, 左浮动会找左浮动, 右浮动会找右浮动

​	浮动元素浮动之后的位置, 由浮动元素浮动之前在标准流中的位置来确定

什么是浮动元素贴靠现象?

​	如果父元素的宽度能够显示所有浮动元素, 那么浮动的元素会并排显示

​	如果父元素的宽度不能显示所有浮动元素, 那么会从最后一个元开始往前贴靠

​	如果贴靠了前面所有浮动元素之后都不能显示, 最终会贴靠到父元素的左边或者右边



在标准流中内容的高度可以撑起父元素的高度

在浮动流中浮动的元素是不可以撑起父元素的高度的





## 定位流

1.定位流分类

​	相对定位

​	绝对定位

​	固定定位

​	静态定位

2.什么是相对定位?

​	相对定位就是相对于自己以前在标准流中的位置来移动

3.相对定位注意点

​	3.1相对定位是不脱离标准流的, 会继续在标准流中占用一份空间

​	3.2在相对定位中同一个方向上的定位属性只能使用一个

​	3.3由于相对定位是不脱离标准流的, 所以在相对定位中是区分块级元素/行内元素/行内块级元素

​	3.4由于相对定位是不脱离标准流的, 并且相对定位的元素会占用标准流中的位置, 所以当给相对定位的元素设置margin/padding等属性的时会影响到标准流的布局



4.相对定位应用场景

​	4.1用于对元素进行微调

​	4.2配合后面学习的绝对定位来使用



### - 绝对定位

1.什么是绝对定位?

​	绝对定位就是相对于body来定位

2.绝对定位注意点

​	2.1绝对定位的元素是脱离标准流的

​	2.2绝对定位的元素是不区分块级元素/行内元素/行内块级元素

规律

​	1.默认情况下所有的绝对定位的元素, 无论有没有祖先元素, 都会以body作为参考点

​	2.如果一个绝对定位的元素有祖先元素, 并且祖先元素也是定位流, 那么这个绝对定位的元素就会以定位流的那个祖先元素作为参考点

​		2.1只要是这个绝对定位元素的祖先元素都可以

​		2.2指的定位流是指绝对定位/相对定位/固定定位

​		2.3定位流中只有静态定位不行

​	3.如果一个绝对定位的元素有祖先元素, 并且祖先元素也是定位流, 而且祖先元素中有多个元素都是定位流, 那么这个绝对定位的元素会以离它最近的那个定位流的祖先元素为参考点





相对定位弊端:

​	相对定位不会脱离标准流, 会继续在标准流中占用一份空间, 所以不利于布局界面

绝对定位弊端:

​	默认情况下绝对定位的元素会以body作为参考点, 所以会随着浏览器的宽度高度的变化而变化

子元素用绝对定位, 父元素用相对定位



如何让绝对定位的元素水平居中

只需要设置绝对定位元素的left:50%;

然后再设置绝对定位元素的 margin-left: -元素宽度的一半px;



### - 固定定位

1.什么是固定定位?

​	固定定位和前面学习的背景关联方式很像, 背景定位可以让背景图片不随着滚动条的滚动而滚动, 而固定定位可以让某个盒子不随着滚动条的滚动而滚动

注意点:

​	1.固定定位的元素是脱离标准流的, 不会占用标准流中的空间

​	2.固定定位和绝对定位一样不区分行内/块级/行内块级

​	3.IE6不支持固定定位



### - z-index属性

1.什么是z-index属性?

默认情况下所有的元素都有一个默认的z-index属性, 取值是0.

z-index属性的作用是专门用于控制定位流元素的覆盖关系的

​	1.默认情况下定位流的元素会盖住标准流的元素

​	2.默认情况下定位流的元素后面编写的会盖住前面编写的

​	3.如果定位流的元素设置了z-index属性, 那么谁的z-index属性比较大, 谁就会显示在上面

注意点:

​	1.从父现象

​		1.1如果两个元素的父元素都没有设置z-index属性, 那么谁的z-index属性比较大谁就显示在上面

​		1.2如果两个元素的父元素设置了z-index属性, 那么子元素的z-index属性就会失效, 也就是说谁的父元素的z-index属性比较大谁就会			显示在上面



## 伸缩布局

```css
display:flex;
```




### - 容器属性

####  `flex-direction`（主轴方向）
`flex-direction` 属性决定主轴的方向，继而决定子项在容器中的位置。

```css
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

- `row`（默认值）：表示子项从左向右排列。此时水平方向轴为主轴。

- `row-reverse`：表示子项从右向左排列。

- `column`：表示子项从上向下排列。此时垂直方向轴为主轴。

- `column-reverse`：表示子项从下向上排列。

  

#### `flex-wrap`(子项换行)

`flex-wrap` 属性用于指定弹性布局中子项是否换行。

使用该属性，需要为弹性容器添加固定宽度，当弹性容器宽度超过子项宽度总和时，值设为 `wrap` 或 `wrap-reverse` 均不起效果

```css
.container {
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

- `nowrap`（默认值）：表示不换行，所有子项目单行排列，子项可能会溢出。

- `wrap`：表示换行，所有子项目多行排列，溢出的子项会被放到下一行，按从上向下顺序排列。

- `wrap-reverse`：所有子项目多行排列，按从下向上顺序排列。

  

#### `flex-flow`(简写)

`flex-flow` 属性是 `flex-direction` 属性和 `flex-wrap` 属性的简写形式，默认值为 `row nowrap`。

```css
.container {
  flex-flow: < 'flex-direction' > || < 'flex-wrap' >;
}
```



#### `justify-content`(主轴对齐)

`justify-content` 属性定义了子项在 **主轴**（水平方向）上的对齐方式。

仅当 `flex-direction` 为 `row` 时生效，因为 `justify-content` 仅定义子项在水平方向上的对齐方式

```css
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

- `flex-start`（默认值）：表示弹性容器子项按主轴起点线对齐
- `flex-end`：表示弹性容器子项按主轴终点线对齐
- `center`： 居中排列
- `space-between`：弹性容器子项均匀分布，第一项紧贴主轴起点线，最后一项紧贴主轴终点线，子项目之间的间隔都相等。要注意特殊情况，当剩余空间为负数或者只有一个项时，效果等同于 `flex-start`。
- `space-around`：弹性容器子项均匀分布，每个项目两侧的间隔相等，相邻项目之间的距离是两个项目之间留白的和。所以，项目之间的间隔比项目与边框的间隔大一倍。要注意特殊情况，当剩余空间为负数或者只有一个项时，效果等同于`center`。
- `space-evenly`：弹性容器子项均匀分布，所有项目之间及项目与边框之间的距离相等。



#### `align-items`(交叉轴对齐)

`align-items` 属性定义弹性容器子项在交叉轴（垂直方向）上的对齐方式。

```css
.container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

- `stretch`（默认值）：当子项未设置高度或者高度为 `atuo` 时，子项的高度设为行高。这里需要注意，在只有一行的情况下，行的高度为容器的高度，即子项高度为容器的高度。（当子项设定了高度时无法展开）
- `flex-start`：表示子项与交叉轴的起点线对齐。
- `flex-end`：表示子项与交叉轴的终点线对齐。
- `center`：表示与交叉轴的中线对齐。
- `baseline`：表示基线对齐，当行内轴与侧轴在同一线上，即所有子项的基线在同一线上时，效果等同于`flex-start`。



#### `align-content`(换行对齐)

`align-content` 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

核心是一定是盒子内部的元素超过了盒子项的宽度（默认）出现了换行，也就是有多行才可以。

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

- `stretch`（默认值）：轴线占满整个交叉轴。（当子项设定了高度时无法展开）
- `flex-start`：表示各行与交叉轴的起点线对齐。
- `flex-end`：表示各行与交叉轴的终点线对齐。
- `center`：表示各行与交叉轴的中线对齐。
- `space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。要注意特殊情况，当剩余空间为负数时，效果等同于`flex-start`。
- `space-around`：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。要注意特殊情况，当剩余空间为负数时，效果等同于`center`。

⚠️ **注意**：该属性只作用于多行的情况（`flex-warp: wrap / warp-reverse`），在只有一行的弹性容器上无效，另外该属性可以很好的处理，换行以后相邻行之间产生的间距。



#### 设定条件

- `flex-wrap`：只作用于多行情况
- `margin`（可选）：子项间的间距
- `line-height`（可选）：因为子项无高度，行高相当于给子项一个最低高度



### - 子项属性

#### `order`(子项排序)

缺省情况下，Flex 子项是按照在代码中出现的先后顺序排列的。CSS3 新增加 `order` 属性定义项目的排列顺序，是数值类型。数值越小，排列越靠前，默认为 0。

> 注意此属性设置在子项上，浏览器自动按照 `order` 的大小排序盒子，默认都是 0，如果相同的 `order` 则按照编写标签的顺序排放盒子。

```css
.item {
  order: 1;
}
```

#### `flex-grow`(子项比例)

`flex-grow` 属性定义子项的**扩展比例**，取值必须是一个单位的正整数，表示放大的比例。默认为 0，即如果存在额外空间，也不放大，负值无效。Flex 容器会根据子项设置的扩展比例作为比率来分配剩余空间

如果所有项目的 `flex-grow` 属性都为 1，则它们将等分剩余空间（如果有的话）。如果一个项目的 `flex-grow` 属性为 2，其他项目都为 1，则前者占据的剩余空间将比其他项多一倍。

一行的子盒子同时设置 `flex-grow` 属性的话，会根据设置的值的大小按比例排放子项。

`flex-grow` 属性决定了子项要占用父容器多少剩余空间

计算方式：

- 假设剩余空间 `x`（弹性容器宽度与所有弹性子项宽度总和之差）
- 假设有三个弹性子项元素，`flex-grow` 设定值分别为 `a`、`b` 和 `c`
- 每个元素可以分配的剩余空间为：`a/(a+b+c) * x`、`b/(a+b+c) * x` 和 `c/(a+b+c) * x`

假设剩余空间为 `150px`，`a`、`b` 和 `c` 的 `flex-grow` 分别为 1、2 和 3，那么 `a` 占比剩余空间：`1/(1+2+3) = 1/6`，那么 `a` 瓜分到的剩余空间宽度是 `150*(1/6)=25`，加上 `a` 原本的宽度，实际的宽度为 `<origin-width> + 25`。

#### `flex-shrink`(缩小)

如果子容器宽度超过父容器宽度，即使是设置了 `flex-grow`，但是由于没有剩余空间，就分配不到剩余空间了。这时候有两个办法：换行和压缩。由于 `flex` 默认不换行，那么压缩的话，怎么压缩呢，压缩多少？此时就需要用到 `flex-shrink` 属性了。

`flex-shrink` 属性定义了项目的**缩小比例**，默认为 1，即如果空间不足，该项目将缩小。

此时，剩余空间的概念就转化成了 **溢出空间**。

```css
.item {
  flex-shrink: <number>; /* default 1 */
}
```

如果所有项目的 `flex-shrink` 属性都为 1，当空间不足时，都将等比例缩小。如果一个项目的 `flex-shrink` 属性为 0，其他项目都为 1，则空间不足时，前者不缩小。

[flex-shrink 属性](https://tsejx.github.io/css-guidebook//layout/basic/flexible-box-layout#flex-flex-shrink)

负值对该属性无效。且如果弹性子项总和没有超出父容器，设置 `flex-shrink` 将无效。

计算方式：

- 假设三个子项的 `width` 为：`w1`、`w2`、`w3`

- 假设三个子项的 `flex-shrink` 为：`a`、`b`、`c`

- 计算总压缩权重：`sum = a * w1 + b * w2 + c * w3`

- 计算每个元素压缩率：`s1 = a * w1 / sum`、`s2 = b * w2 / sum`、`s3 = c * w3 / sum`

- 计算每个元素宽度：`width - 压缩率 * 溢出空间`

  

#### `flex-basis`

`flex-basis` 属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为 `auto`，即项目的本来大小。

```css
.item {
  flex-basis: <number> | <percentage> | auto; /* default auto */
}
```

[flex-basis 属性](https://tsejx.github.io/css-guidebook//layout/basic/flexible-box-layout#flex-flex-basis)

⚠️ **注意**：

1. 设置 `flex-grow` 进行分配剩余空间，或者使用 `flex-shrink` 进行收缩都是在 `flex-basis` 的基础上进行的；
2. 当flex-basis的值以及`width`（或者`height`）的值均为非`auto`时，
   - 若 `content` 长度同时大于 `flex-basis` 的值以及 `width`（或者 `height`）的值，此时以 `flex-basis` 与 `width`（或者 `height`）中值大的为准 ，**但是**，如果子项设置了`overflow: hidden` 或者 `overflow: auto`，此时以`flex-basis`值为准；
   - 若 `content` 长度同时小于 `flex-basis` 的值以及 `width`（或者 `height`）的值，此时以 `flex-basis` 值为准
   - 若 `content` 长度小于 `flex-basis` 的值，大于 `width`（或者 `height`）的值，此时以 `flex-basis` 值为准；
   - 若 `content` 长度大于 `flex-basis` 的值，小于 `width`（或者 `height`）的值，此时以 `content` 自身长度值为准；
3. 当 `width`（或者 `height`）的值为 `auto` 时，且内容的长度大于 `flex-basis`设置的值，此时以 `content` 自身长度值为准，且**不能进行 `flex-shrink` 收缩**。相反如果内容的长度小于 `flex-basis` 设置的值，则会使用 `flex-basis` 设置的值；
4. 当存在最小值 `min-width`（`min-height`）时，且 `flex-basis` 的值小于最小值时，宽度以最小值为准，当 `flex-basis` 的值大于最小值时，以 `flex-basis` 的值为准。

> 属性优先级：`max-width / min-width -> flex-basis -> width -> box`



#### `flex`

`flex` 属性是 `flex-grow`、`flex-shrink` 和 `flex-basis` 的简写，默认值为 `0 1 auto`。后两个属性可选。

```css
.item {
  flex: none | [ < 'flex-grow' > < 'flex-shrink' >? || < 'flex-basis' > ];
}
```

该属性有两个快捷值：`auto (1 1 auto)` 和 `none (0 0 auto)`。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

#### `align-self`

`align-self` 属性用于指定子项的对齐方式，可覆盖 `align-items` 属性。

默认值为 `auto`，表示继承父元素的 `align-items` 属性，如果没有父元素，则等同于 `stretch`。

```css
.item {
  align-self: auto || flex-start || flex-end || center || baseline || stretch;
}
```



### - 布局方向

![弹性盒子的轴](css基础.assets/flexbox-axis.566f4861.png)



 在伸缩布局中, 默认伸缩项是从左至右的排版的

主轴的排版的方向默认就是row, 默认就是从**左至右**

```css
flex-direction: row;
```

修改主轴的排版方向为从右至左

```css
flex-direction: row-reverse;
```



告诉系统把主轴的方向改为垂直方向

注意点:

- 默认情况下主轴是水平方向的, 但是也可以修改为垂直方向.只要看到`flex-direction: column/column-reverse`就代表主轴被修改为了垂直方向
- 如果将主轴修改为了垂直方向, 那么侧轴就会自动从垂直方向转换为水平方向

​      修改主轴的排版方向为从上至下

```css
flex-direction: column;
```

​      修改主轴的排版方向为从下至上

```css
flex-direction: column-reverse;
```

### - 对齐方式

#### -- 主轴对齐

`justify-content:`

用于设置伸缩项主轴上的对齐方式

如果设置为`flex-start`, 代表告诉系统伸缩项和主轴的起点对齐

```css
justify-content: flex-start;
```

```css
justify-content: flex-end;
```

居中对齐

```css
justify-content: center;
```

两端对齐

```css
justify-content: space-between;
```

环绕对齐

```css
justify-content: space-around;
```

#### --测轴(交叉轴)对齐

交叉轴垂直于主轴，所以如果你的`flex-direction` (主轴) 设成了 `row` 或者 `row-reverse` 的话，交叉轴的方向就是沿着列向下的。

如果主轴方向设成了 `column` 或者 `column-reverse`，交叉轴就是水平方向。



通过`align-items`可以修改侧轴的对齐方式

默认情况下是以侧轴的起点对齐

```css
align-items: flex-start;

align-items: flex-end;

align-items: center;
```

注意点:

​      和主轴不同的是, **侧轴没有两端对齐和环绕对齐**

基线对齐

`baseline`：表示基线对齐，当行内轴与侧轴在同一线上，即所有子项的基线在同一线上时，效果等同于`flex-start`。

```css
align-items: baseline;
```

拉伸对齐

```css
align-items: stretch;
```

- `stretch`（默认值）：当子项未设置高度或者高度为 `atuo` 时，子项的高度设为行高。这里需要注意，在只有一行的情况下，行的高度为容器的高度，即子项高度为容器的高度。（当子项设定了高度时无法展开）
- `flex-start`：表示子项与交叉轴的起点线对齐。
- `flex-end`：表示子项与交叉轴的终点线对齐。
- `center`：表示与交叉轴的中线对齐。
- `baseline`：表示基线对齐，当行内轴与侧轴在同一线上，即所有子项的基线在同一线上时，效果等同于`flex-start`。



​      如果在伸缩容器中通过 `align-items` 修改侧轴的对齐方式, 是修改所有伸缩项侧轴的对齐方式

​      如果是在伸缩项中通过 `align-self` 修改侧轴的对齐方式, 是单独修改当前伸缩项侧轴的对齐方式

​      `align-self`属性的取值和`align-items`一样



### - 换行问题



在伸缩布局中, 如果伸缩容器的宽度不够, 系统会自动压缩伸缩项的宽度, 保证所有的伸缩想都能放在伸缩容器中

如果当伸缩容器宽度不够时, 不想让伸缩项被压缩, 也可以让系统换行`flex-wrap: wrap;`

默认情况下如果伸缩容器的高度比换行之后所有伸缩项的高度还要高, 那么系统会自动将剩余空间平分之后添加给每一行

宽度不够也不换行, 默认取值`flex-wrap: nowrap;`

```css
flex-wrap: nowrap;
flex-wrap: wrap;
flex-wrap: wrap-reverse;
```

- `nowrap`（默认值）：表示不换行，所有子项目单行排列，子项可能会溢出。
- `wrap`：表示换行，所有子项目多行排列，溢出的子项会被放到下一行，按从上向下顺序排列。
- `wrap-reverse`：所有子项目多行排列，按从下向上顺序排列。

### - 换行对齐

`align-content` 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

核心是一定是盒子内部的元素超过了盒子项的宽度（默认）出现了换行，也就是有多行才可以。

也就是说**伸缩容器中的伸缩项换行**了, 那么我们就可以通过`align-content`来设置行与行之间的对齐方式

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

- `stretch`（默认值）：轴线占满整个交叉轴。（当子项设定了高度时无法展开）

- `flex-start`：表示各行与交叉轴的起点线对齐。

- `flex-end`：表示各行与交叉轴的终点线对齐。

- `center`：表示各行与交叉轴的中线对齐。

- `space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。要注意特殊情况，当剩余空间为负数时，效果等同于`flex-start`。

- `space-around`：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。要注意特殊情况，当剩余空间为负数时，效果等同于`center`。

默认情况下换行就是就是拉伸对齐

  一定要注意: 在换行中的拉伸对齐是指, 所有行的高度总和要和伸缩容器的高度一样

  所以会将多余的空间平分之后添加给每一行

```css
align-content: flex-start;
align-content: flex-end;
align-content: center;
align-content: space-between;
align-content: space-around;
align-content: stretch;
```

### - 伸缩项排序

如果想调整伸缩布局中伸缩项的顺序, 那么我们可以通过修改伸缩项的order属性来实现

默认情况下order的取值是0

如果我们设置了order属性的值, 那么系统就会按照设置的值从小到大的排序

```css
order:1;
```





## 媒体查询

内嵌形式

```css
/*
需求:
PC显示小牛(大于等于980)
pad显示小猪(小于980, 并且大于等于620)
phone显示小马(小于620)
*/
/*如果浏览器的宽度大于等于980, 那么就执行后面{}中的代码*/
@media(min-width: 980px) {
    body{
        background: url("images/animal1.png");
    }
}
/*如果浏览器的宽度大于等于620, 并且宽度小于980*/
/*注意点: and的两边必须添加空格*/
@media(min-width: 620px) and (max-width: 979px) {
    body{
        background: url("images/animal2.png");
    }
}
@media(max-width: 619px) {
    body{
        background: url("images/animal3.png");
    }
}
```

外链形式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>144-媒体查询-外链式</title>
    <link rel="stylesheet" href="144-style-pc.css" media="(min-width:980px)">
    <link rel="stylesheet" href="144-style-pad.css" media="(min-width:620px) and (max-width:979px)">
    <link rel="stylesheet" href="144-style-phone.css" media=" (max-width:619px)">

</head>
<body>

</body>
</html>
```

























  







