# 常见Css问题

## 1.文本超出省略号隐藏

### 	单行

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
```

### 	多行

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```





# 2.文字两端对齐

```html
<div>姓名</div>
<div>手机号码</div>
<div>账号</div>
<div>密码</div>
```

```css
div {
    margin: 10px 0; 
    width: 100px;
    border: 1px solid red;
    text-align: justify;
    text-align-last: justify;
}

```

![](https://user-gold-cdn.xitu.io/2019/12/10/16eeea2a9a0c03c0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# 3.盒模型

一个盒子由四个部分组成：`content`、`padding`、`border`、`margin`

分2种：

- W3C 标准盒子模型
- IE 怪异盒子模型



## 标准盒子模型

- 盒子总宽度 = width + padding + border + margin;
- 盒子总高度 = height + padding + border + margin

`width/height` 只是内容高度，不包含 `padding` 和 `border`值



## IE 怪异盒子模型

- 盒子总宽度 = width + margin;
- 盒子总高度 = height + margin;

`width/height` 包含了 `padding`和 `border`值



# 4.css选择器

`css`属性选择器常用的有：

- id选择器（#box），选择id为box的元素
- 类选择器（.one），选择类名为one的所有元素
- 标签选择器（div），选择标签为div的所有元素
- 后代选择器（#box div），选择id为box元素内部所有的div元素
- 子选择器（.one>one_1），选择父元素为.one的所有.one_1的元素
- 相邻同胞选择器（.one+.two），选择紧接在.one之后的所有.two元素
- 群组选择器（div,p），选择div、p的所有元素

使用较少的

- 伪类选择器

  ```css
  :link ：选择未被访问的链接
  :visited：选取已被访问的链接
  :active：选择活动链接
  :hover ：鼠标指针浮动在上面的元素
  :focus ：选择具有焦点的
  :first-child：父元素的首个子元素
  
  Css3中添加
  :first-of-type 父元素的首个元素
  :last-of-type 父元素的最后一个元素
  :only-of-type 父元素的特定类型的唯一子元素
  :only-child 父元素中唯一子元素
  :nth-child(n) 选择父元素中第N个子元素
  :nth-last-of-type(n) 选择父元素中第N个子元素，从后往前
  :last-child 父元素的最后一个元素
  :root 设置HTML文档
  :empty 指定空的元素
  :enabled 选择被禁用元素
  :disabled 选择被禁用元素
  :checked 选择选中的元素
  :not(selector) 选择非 <selector> 元素的所有元素
  ```

- 伪元素选择器

  ```css
  :first-letter ：用于选取指定选择器的首字母
  :first-line ：选取指定选择器的首行
  :before : 选择器在被选元素的内容前面插入内容
  :after : 选择器在被选元素的内容后面插入内容
  ```

- 属性选择器

  ```css
  [attribute] 选择带有attribute属性的元素
  [attribute=value] 选择所有使用attribute=value的元素
  [attribute~=value] 选择attribute属性包含value的元素
  [attribute|=value]：选择attribute属性以value开头的元素
  css3中新加
  [attribute*=value]：选择attribute属性值包含value的所有元素
  [attribute^=value]：选择attribute属性开头为value的所有元素
  [attribute$=value]：选择attribute属性结尾为value的所有元素
  ```

## 优先级

内联 > ID选择器 > 类选择器 > 标签选择器

## 继承属性

### 可继承属性

- 字体系列属性

- 文本系列属性

- 元素可见性

- 表格布局属性

- 列表属性

- 引用

- 光标属性

### 无继承的属性

- display
- 文本属性：vertical-align、text-decoration
- 盒子模型的属性：宽度、高度、内外边距、边框等
- 背景属性：背景图片、颜色、位置等
- 定位属性：浮动、清除浮动、定位position等
- 生成内容属性：content、counter-reset、counter-increment
- 轮廓样式属性：outline-style、outline-width、outline-color、outline
- 页面样式属性：size、page-break-before、page-break-after

# 5.em/px/rem/vh/vw区别

在`css`单位中，可以分为长度单位、绝对单位

| CSS单位      |                                        |
| ------------ | -------------------------------------- |
| 相对长度单位 | em、ex、ch、rem、vw、vh、vmin、vmax、% |
| 绝对长度单位 | cm、mm、in、px、pt、pc                 |

### （1）px

表示像素，所谓像素就是呈现在我们显示器上的一个个小点，每个像素点都是大小等同的，所以像素为计量单位被分在了绝对长度单位中

`px`的大小和元素的其他属性无关--因此是绝对单位

### （2）em

em是相对长度单位。相对于当前对象内文本的字体尺寸。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸（`1em = 16px`）

特点：

- em 的值并不是固定的
- em 会继承父级元素的字体大小
- em 是相对长度单位。相对于当前对象内文本的字体尺寸。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸
- 任意浏览器的默认字体高都是 16px

### （3）rem

rem，相对单位，相对的只是HTML根元素`font-size`的值

特点：

- rem单位可谓集相对大小和绝对大小的优点于一身
- 和em不同的是rem总是相对于根元素，而不像em一样使用级联的方式来计算尺寸

### （4）vh、vw

vw ，就是根据窗口的宽度，分成100等份，100vw就表示满宽，50vw就表示一半宽。（vw 始终是针对窗口的宽），同理，`vh`则为窗口的高度

这里的窗口分成几种情况：

- 在桌面端，指的是浏览器的可视区域
- 移动端指的就是布局视口

像`vw`、`vh`，比较容易混淆的一个单位是`%`，不过百分比宽泛的讲是相对于父元素：

- 对于普通定位元素就是我们理解的父元素
- 对于position: absolute;的元素是相对于已定位的父元素
- 对于position: fixed;的元素是相对于 ViewPort（可视窗口）

