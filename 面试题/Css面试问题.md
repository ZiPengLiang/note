## 了解盒模型吗

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


