# Css篇

## 	了解盒模型吗

CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：`外边距（margin）`、`边框（border）`、`内边距（padding）`、`实际内容（content）`四个属性。 CSS盒模型：**标准模型 + IE模型**

标准盒子模型：宽度=内容的宽度（content）+ border + padding

低版本IE盒子模型：宽度=内容宽度（content+border+padding），如何设置成 IE 盒子模型:

```css
box-sizing: border-box;
```




## 清除浮动有哪些方法？

不清楚浮动会发生高度塌陷：浮动元素父元素高度自适应（父元素不写高度时，子元素写了浮动后，父元素会发生高度塌陷）

- clear清除浮动（添加空div法）在浮动元素下方添加空div,并给该元素写css样式：{clear:both;height:0;overflow:hidden;}
- 给浮动元素父级设置高度
- 父级同时浮动（需要给父级同级元素添加浮动）
- 父级设置成inline-block，其margin: 0 auto居中方式失效
- 给父级添加overflow:hidden 清除浮动方法
- 万能清除法 after伪类 清浮动（现在主流方法，推荐使用）

```css
.float_div:after{
  content:".";
  clear:both;
  display:block;
  height:0;
  overflow:hidden;
  visibility:hidden;
}
.float_div{
  zoom:1
}
```

## CSS浮动怎么理解的

浮动的意义：设置了浮动属性的元素会脱离普通标准流的控制，移动到其父元素中指定的位置的过程，将块级元素放在一行，浮动会脱离标准流，不占位置，会影响标准流，浮动只有左右浮动，不会出现上下浮动

### **特性：**

**1、浮动的元素脱离了标准文档流，摆脱块级元素和行内元素的限制**

**2、浮动的元素存在相互贴靠的效果，当宽度不够的时候，会出现自动换行**

**3、浮动的元素虽然脱离了标准文档流，但是没有脱离文本流，出现被字包围的效果**

**4、浮动之后的元素会存在收缩的效果，当一个块级元素没有设置宽度的时，当块级元素浮动之后，就会失去高度**

**5、当父元素不设置高度的时候，多个子元素的高度和撑起了父元素的高度；当设置浮动后，子元素最高的高度撑起了父元素的高度。**



### **弊端：**

**1、高度塌陷**

**当子元素同时设置浮动后，父元素失去支撑，父元素的高度消失，缩成一条线。**

解决办法：在父元素失去高度，发生塌陷之后，可以给父元素添加高度或者设置overflow:hidden的方法进行解决高度塌陷的问题。

**2、页面结构的不稳定性，子元素浮动，导致标准文档流出现空白区域。**

解决办法：clear:both; 去进行解决，这也是称之为隔墙法。



## 绝对定位相对定位怎么理解

### 绝对定位 --absolute

脱离了标准文档流的

参照物：父元素，假如父元素没有就一直往上找，知道body

### 相对定位 --relative

没有脱离标准文档流

参照物：自身



## 块元素和行内元素什么区别

1.排布上

​	行内元素能能多个在一行显示

​	块元素独占一行

2.内容上

​	行内元素：文本或者其它行内元素 --无法包含块级元素

​	块级元素：包含行内元素和块级元素

3.属性上 -- 盒模型属性

​	行内元素设置width无效，height无效(可以设置line-height)，margin上下无效，padding上下无效



inline-block -- 行内块级元素

​	既具有 block 元素可以设置宽高的特性，同时又具有 inline 元素默认不换行的特性



## Css如何实现盒子水平垂直居中

- 利用定位+margin:auto
- 利用定位+margin:负值
- 利用定位+transform
- table布局 -- 不建议使用
- flex布局
- grid布局



## src和href的区别

**href**标识超文本引用，用在**link**和**a**等元素上，**href**是引用和页面关联，是在当前元素和引用资源之间建立联系

**src**表示引用资源，表示替换当前元素，用在**img**，**script**，**iframe**上，src是页面内容不可缺少的一部分。



## 常见的浏览器内核和前缀有哪些？微信的浏览器内核是什么

**谷歌**--以前是Webkit内核，现在是Blink内核。

**IE**--Trident内核

**火狐**--Gecko内核

**Safari** --Webkit内核

**Opera** -- 最初Presto内核，后来是Webkit，现在是Blink内核;

微信浏览器内核：X5 Blink内核

前缀：

WebKit内核   -webkit-

Gecko内核     -moz-

Trident内核     -ms-

Presto内核      -o-



## html5有哪些新特性、移除了那些元素？如何处理HTML5新标签的浏览器兼容问题？如何区分HTML和HTML5？ 

### 新特性

- 拖拽释放(Drag and drop) API 
- 语义化更好的内容标签（header,nav,footer,aside,article,section）
- 音频、视频API(audio,video)
- 画布(Canvas) API
- 地理(Geolocation) API
- 本地离线存储 localStorage 长期存储数据，浏览器关闭后数据不丢失；
- sessionStorage 的数据在浏览器关闭后自动删除
- 表单控件，calendar、date、time、email、url、search 
- 新的技术webworker, websocket, Geolocation

### 移除元素

纯表现的元素：basefont，big，center，font, s，strike，tt，u；

对可用性产生负面影响的元素：frame，frameset，noframes；

### 如何处理

支持HTML5新标签：
IE8/IE7/IE6支持通过document.createElement方法产生的标签，
可以利用这一特性让这些浏览器支持HTML5新标签，
浏览器支持新标签后，还需要添加标签默认的样式；

当然最好的方式是直接使用成熟的框架、使用最多的是html5shim框架

```html
   <!--[if lt IE 9]> 
   <script> src="http://html5shim.googlecode.com/svn/trunk/html5.js"</script> 
   <![endif]--> 
```

### 如何区分

DOCTYPE声明\新增的结构元素\功能元素

## 语义化标签、作用

语义化标签--简单明了地知道该标签的作用

作用：

1. 有利于SEO，搜索引擎根据标签来确定上下文和各个关键字的权重
2. 有利于开发和维护，语义化更具可读性，代码更好维护，与CSS3关系更和谐。
3. 易于用户阅读，样式丢失的时候能让页面呈现清晰的结构。
4. 兼容性更好，支持更多的网络设备

## Css3动画有哪些

1. transition 实现渐变动画

2. transform 转变动画

   translate：位移

   scale：缩放

   rotate：旋转

   skew：倾斜

   配合`transition`过度使用

   transform`不支持`inline`元素，使用前把它变成`block

3. animation 实现自定义动画

   通过@keyframes定义关键帧

   ```css
   
   @keyframes rotate{
       0%{
           transform: rotate(0deg);
       }
       50%{
           transform: rotate(180deg);
       }
       100%{
           transform: rotate(360deg);
       }
   }
   animation: rotate 2s;
   ```

   

## CSS预处理器

sass、less、stylus

特性：

- 变量

  ```css
  sass
  $mainColor: #0982c1;
  
  less
  @mainColor: #0982c1;
  
  Stylus
  Stylus对变量名没有任何限定但不能用@开头
  mainColor = #0982c1
  siteWidth = 1024px
  $borderStyle = dotted
  ```

- 作用域	

- 混合(Mixins)

  ```css
  sass
  @mixin error($borderWidth:2px){
    border:$borderWidth solid #f00;
    color: #f00;
  }
  /*调用error Mixins*/
  .generic-error {
    @include error();/*直接调用error mixins*/
  }
  
  less
  /*声明一个Mixin叫作“error”*/
  .error(@borderWidth:2px){
    border:@borderWidth solid #f00;
    color: #f00;
  }
  /*调用error Mixins*/
  .generic-error {
    .error();/*直接调用error mixins*/
  }
  
  Stylus
  /*声明一个Mixin叫作“error”*/
  error(borderWidth=2px){
    border:borderWidth solid #f00;
    color: #f00;
  }
  /*调用error Mixins*/
  .generic-error {
    error();/*直接调用error mixins*/
  }
  ```

- 嵌套（Nesting）



通过import导入文件

```css
@import "reset.css";
```



## CSS优化、提高性能的方法有哪些？

- 合并css文件，减少css文件数量
- 减少嵌套，最好不要大于三层
- 不在ID选择器钱进行嵌套，浪费性能
- 建立公共样式类，比如flex、清除浮动等
- 减少通配符*或者类似[hidden="true"]这类选择器的使用
- 灵活运用css的继承机制
- 拆分出公共css文件
- 不用css表达式
- 减少css的重置
- 图片使用雪碧图、精灵图等
- css压缩
- GZIP压缩

## 让Chrome支持小于12px的文字

针对chrome浏览器,加webkit前缀，用transform:scale()这个属性进行放缩.

```css
span{
    font-size: 12px;
    display: inline-block;
    -webkit-transform:scale(0.8);
}
```

## CSS3有哪些新特性

新增选择器 p:nth-child（n）{color: rgba（255, 0, 0, 0.75）}

弹性盒模型 display: flex;

多列布局 column-count: 5;

媒体查询 @media （max-width: 480px） {.box: {column-count: 1;}}

个性化字体 @font-face{font-family:BorderWeb;src:url（BORDERW0.eot）；}

颜色透明度 color: rgba（255, 0, 0, 0.75）；

圆角 border-radius: 5px;

渐变 background:linear-gradient（red, green, blue）；

阴影 box-shadow:3px 3px 3px rgba（0, 64, 128, 0.3）；

倒影 box-reflect: below 2px;

文字装饰 text-stroke-color: red;

文字溢出 text-overflow:ellipsis;

背景效果 background-size: 100px 100px;

边框效果 border-image:url（bt_blue.png） 0 10;

旋转 transform: rotate（20deg）；

倾斜 transform: skew（150deg, -10deg）；

位移 transform:translate（20px, 20px）；

缩放 transform: scale（。5）；

平滑过渡 transition: all .3s ease-in .1s;

动画 @keyframes anim-1 {50% {border-radius: 50%;}} animation: anim-1 1s;

## ::before和:after中双冒号和单冒号有什么区别

单冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。

伪类是选择器的一种，它用于选择处于特定状态的元素

伪元素像往标记文本中加入全新的HTML元素一样



## 响应式布局

响应式布局（Responsive design），意在实现不同屏幕分辨率的终端上浏览网页的不同展示方式。

### 步骤

#### 1. 设置 Meta 标签

```
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

#### 2. 通过媒介查询来设置样式 Media Queries

```css
@media screen and (max-width: 980px) {
  #head { … }
  #content { … }
  #footer { … }
}
```

#### 3. 设置多种试图宽度

```css
/** iPad **/
@media only screen and (min-width: 768px) and (max-width: 1024px) {}
/** iPhone **/
@media only screen and (min-width: 320px) and (max-width: 767px) {}
```



## 各种兼容性的处理方法

### **不同浏览器的标签默认的样式**

```css
*{ margin: 0; padding: 0; }
```

### **IE6 双倍边距的问题**

设置 ie6 中设置浮动，同时又设置 margin，会出现双倍边距的问题

```css
display: inline;
```

### **解决 IE9 以下浏览器不能使用 opacity**

```css
opacity: 0.5;
filter: alpha(opacity = 50);
filter: progid:DXImageTransform.Microsoft.Alpha(style = 0, opacity = 50);
```

### **图片默认有间距**

使用float属性为img布局

###  **li之间有间距**

解决方法：li 设置vertical-align:middle;



## link和@import的区别

1. link是XHTML标签，除了加载CSS外，还可以定义RSS等其他事务；@import属于CSS范畴，只能加载CSS。
2. link引用CSS时，在页面载入时同时加载；@import需要页面网页完全载入以后加载。
3. link是XHTML标签，无兼容问题；@import是在CSS2.1提出的，低版本的浏览器不支持。
4. ink支持使用Javascript控制DOM去改变样式；而@import不支持。



# JavaScript篇

##  Ajax同步和异步怎么理解的

### 区别：

ajax异步请求：异步请求就当发出请求的同时，浏览器可以继续做任何事，Ajax发送请求并不会影响页面的加载与用户的操作，相当于是在两条线上，各走各的，互不影响。

ajax同步请求：同步请求即是当前发出请求后，浏览器什么都不能做，必须得等到请求完成返回数据之后，才会执行后续的代码。

### 同步与异步适用的场景

看需要的请求的数据是否是程序继续执行必须依赖的数据



## Js面相对象是怎么理解的 

### 特性：

- 封装性
- 继承性
- [多态性]抽象



### 设计思想：

- 抽象出 Class(构造函数)
- 根据 Class(构造函数) 创建 Instance
- 指挥 Instance 得结果



### 创建对象：

#### 简单方式

直接通过 `new Object()` 创建：

```js
var person = new Object()
person.name = 'Jack'
person.age = 18

person.sayName = function () {
  console.log(this.name)
}
```

缺点：代码太过冗余，复用性低

#### 工厂函数

```js
function createPerson (name, age) {
  return {
    name: name,
    age: age,
    sayName: function () {
      console.log(this.name)
    }
  }
}
```

生成实例对象：

```js
var p1 = createPerson('Jack', 18)
var p2 = createPerson('Mike', 18)
```

缺点：却没有解决对象识别的问题

#### 构造函数

```js
function Person (name, age) {
  this.name = name
  this.age = age
  this.sayName = function () {
    console.log(this.name)
  }
}

var p1 = new Person('Jack', 18)
p1.sayName() // => Jack

var p2 = new Person('Mike', 23)
p2.sayName() // => Mike
```

缺点：耗内存

优化： 原型 ,**把所有对象实例需要共享的属性和方法**直接定义在 `prototype` 对象上



```js
function Person (name, age) {
  this.name = name
  this.age = age
}

console.log(Person.prototype)

Person.prototype.type = 'human'

Person.prototype.sayName = function () {
  console.log(this.name)
}

var p1 = new Person(...)
var p2 = new Person(...)

console.log(p1.sayName === p2.sayName) // => true

```

## apply和call的区别 

### 定义：

**重新定义函数的执行环境**，也就是`this`的指向

### 语法：

#### call()

调用一个对象的方法，**用另一个对象替换当前对象**，可以继承另外一个对象的属性

```js
Function.call(obj[, param1[, param2[, [,...paramN]]]]);
```

- `obj`：这个对象将代替`Function`类里`this`对象
- `params`：一串参数列表

#### apply()

```js
Function.apply(obj[, argArray]);
```

- `obj`：这个对象将代替`Function`类里`this`对象
- `argArray`：这个是数组，它将作为参数传给`Function`

### 不同点

**接收参数的方式不同**：

- **apply()方法**接收两个参数，一个是函数运行的作用域（`this`），另一个是参数数组。
- **call()方法**不一定接受两个参数，第一个参数也是函数运行的作用域（`this`），但是传递给函数的参数必须列举出来。



