# 句法分析 - 生成真正的AST(一)

看完`parseHTML`函数的原理后，那就回去`src/compiler/parser/index.js` 文件继续分析`parse`函数，看一下是这函数是如何生成`AST`的

## 构建抽象语法树的思路

不过在之前得看下构建抽象语法树的思路

假设`html`字符串如下

```html
<ul>
  <li>
    <span>文本</span>
  </li>
</ul>
```

那么生成的树结构应该如下

```text
├── ul
│   ├── li
│   │   ├── span
│   │   │   ├── 文本
```

用`javascript`表示的，每个节点都可以当成一个对象，那么`ul`所代表的对象如下

```js
{
  type: 1,
  tag: 'ul'
}
```

对于每个节点对象都要考虑它有没有父节点和子节点，那么加上属性`parent`和`children`属性，由于`ul`为最上级，没有父节点，所以`parent`属性为`null`

```js
{
  type: 1,
  tag: 'ul',
  parent: null,
  children: []
}
```

同时每个元素节点还可能包含很多属性(`attributes`)，因此可以添加`attrsList`属性来保存当前结点所拥有的属性

```js
{
  type: 1,
  tag: 'ul',
  parent: null,
  children: [],
  attrsList: []
}
```

接下来根据节点结构，可以一步步得得出一个`ast`树对象

```js
{
  type: 1,
  tag: 'ul',
  parent: null,
  attrsList: []
  children: [{
  	type: 1,
  	tag: 'li',
  	parent: 'ul',
   	attrsList: []
  	children: [{
  		type: 1,
  		tag: 'span',
 		parent: 'li',
 		attrsList: []
  		children: [{
  			type: 2,
  			tag: '',
 			parent: 'span',
 			attrsList: []
  			text:'文本'
		}],
	}],
  }],
}
```

这就是类似抽象语法树的对象，目前已知`pareHTML`函数能分析出模板数据，那么假设一个函数能通过`pareHTML`函数实现将模板转化为上述的抽象语法对象。

假设一个`parse`函数，如下：

```js
function parse (html) {
  let root
  //...
  return root
}
```

该函数的最终目的是返回一个对象，因此先定义一个变量`root`，并将其返回出来。函数中间的代码是为了充实变量`root`的

```js
function parse (html) {
  let root
  
  parseHTML(html, {
    start (tag, attrs, unary) {
      // 省略...
    },
    end () {
      // 省略...
    }
  })
  
  return root
}
```

假设要解析的 `html` 字符串如下：

```html
<div></div>
```

这段 `html` 字符串仅仅是一个简单的 `div` 标签，甚至没有任何子节点。

```js
function parse (html) {
  let root
  
  parseHTML(html, {
    start (tag, attrs, unary) {
        const element = {     
            type: 1,       
            tag: tag,    
            parent: null, 
            attrsList: attrs,     
            children: []   
        }     
        if (!root) root = element   
    },
    end () {
      // 省略...
    }
  })
  
  return root
}
```

根据上述的`parse`函数，那么解析出来的`root`为：

```js
root = {
  type: 1,
  tag: 'div',
  parent: null,
  attrsList: [],
  children: []
}
```

当模板有子对节点的时候，上面的函数就有问题了，例如：

```html
<div>
  <span></span>
</div>
```

解析`div`会正常解析，但解析到了`span`标签的时候，由于`root`已存在，那么将不会再次设置`root`变量，因此需要做出改变

```js
function parse (html) {
  let root
  let currentParent  
  parseHTML(html, {
    start (tag, attrs, unary) {
      const element = {
        type: 1,
        tag: tag,
        parent: currentParent,        
        attrsList: attrs,
        children: []
      }

      if (!root) {      
          root = element
      } else if (currentParent) {
          currentParent.children.push(element)
      }      
      if (!unary) currentParent = element 
    },
    end () {
      // 省略...
    }
  })
  
  return root
}
```

