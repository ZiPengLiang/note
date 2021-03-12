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