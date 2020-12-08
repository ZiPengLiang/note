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

