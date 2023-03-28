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
        //判断是否为一元标签 ，不是就缓存element
      if (!unary) currentParent = element 
    },
    end () {
      // 省略...
    }
  })
  
  return root
}
```

修改后生成的`root`如下：

```js
root = {
  type: 1,
  tag: 'div',
  parent: undefined,
  attrsList: [],
  children: [{
    {
      type: 1,
      tag: 'span',
      parent: div,
      attrsList: [],
      children: []
    }
  }]
}
```

但这会有一个问题，假如有标签`p`与`span`同级的时候，根据上述代码逻辑，`p`标签会变成`span`标签的子对象

```html
<div>
  <span></span>
  <p></p>
</div>
```

**在解析 `p` 元素的开始标签时，由于 `currentParent` 变量引用的是 `span` 元素的描述对象，所以 `p` 元素的描述对象将被添加到 `span` 元素描述对象的 `children` 数组中，被误认为是 `span` 元素的子节点**。

而事实上 `p` 标签是 `div` 元素的子节点，这就是问题所在。

如果想解决该问题，那么就要确保一个标签结束后把`currentParent`变量重新赋值，以确保当前正在解析的标签拥有正确的父级。

```js
function parse (html) {
  let root
  let currentParent
  //缓存所有的标签
  const stack = []  
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
      if (!unary) {
        currentParent = element
        stack.push(currentParent)      }
    },
    end () {    
        //标签结束后，将其推出栈
        stack.pop()
        //修正了当前正在解析的元素的父级元素。
        currentParent = stack[stack.length - 1]    
    }  
  })
  
  return root
}
```

上述代码就 `parseHTML` 函数生成 `AST` 的基本方式，虽然不够全面，但依旧能初步得了解到了生成`AST`的基本原理，接下来就是源码的本体了



## 准备工作

由于整个`src/compiler/parser/index.js`文件都是为了创建`AST`，因此，该文件中所有内容基本都与生成`AST`有关，所以得先看一下该文件有什么东西

首先是两个关键的函数

### 关键的函数

#### `createASTElement`函数

```js
export function createASTElement (
  tag: string,
  attrs: Array<Attr>,
  parent: ASTElement | void
): ASTElement {
  return {
    type: 1,
    tag,
    attrsList: attrs,
    attrsMap: makeAttrsMap(attrs),
    parent,
    children: []
  }
}
```

用来创建一个元素的描述对象

#### `parse` 函数

该文件中最为关键的函数，结构如下

```js
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
    //一堆变量
    let root
    ....
    //一堆函数
  	function warnOnce (msg, range) {}
   	function closeElement (element) {}
	function trimEndingWhitespace (el){}
 	function checkRootConstraints (el) {}

 	parseHTML(template, {
        //一堆参数
  	 ...
    start (tag, attrs, unary, start, end) {
     
    },

    end (tag, start, end) {
     
    },

    chars (text: string, start: number, end: number) {
     
    },
    comment (text: string, start, end) {
     
    }
  })
  return root
}
```

最终生成`AST`

#### 其他函数

在 `parse` 函数的后面，定义了非常多的函数，如下：

```js
function processPre (el) {/* 省略...*/}
function processRawAttrs (el) {/* 省略...*/}
export function processElement (element: ASTElement, options: CompilerOptions) {/* 省略...*/}
function processKey (el) {/* 省略...*/}
function processRef (el) {/* 省略...*/}
export function processFor (el: ASTElement) {/* 省略...*/}
export function parseFor (exp: string): ?ForParseResult {/* 省略...*/}
function processIf (el) {/* 省略...*/}
function processIfConditions (el, parent) {/* 省略...*/}
function findPrevElement (children: Array<any>): ASTElement | void {/* 省略...*/}
export function addIfCondition (el: ASTElement, condition: ASTIfCondition) {/* 省略...*/}
function processOnce (el) {/* 省略...*/}
function processSlot (el) {/* 省略...*/}
function processComponent (el) {/* 省略...*/}
function processAttrs (el) {/* 省略...*/}
function checkInFor (el: ASTElement): boolean {/* 省略...*/}
function parseModifiers (name: string): Object | void {/* 省略...*/}
function makeAttrsMap (attrs: Array<Object>): Object {/* 省略...*/}
function isTextTag (el): boolean {/* 省略...*/}
function isForbiddenTag (el): boolean {/* 省略...*/}
function guardIESVGBug (attrs) {/* 省略...*/}
function checkForAliasModel (el, value) {/* 省略...*/}
```

这些函数都是对`el`进行进一步处理。

实际上 `el` 是元素的描述对象，如下：

```js
el = {
  type: 1,
  tag,
  attrsList: attrs,
  attrsMap: makeAttrsMap(attrs),
  parent,
  children: []
}
```

比如：

```js
function processIf (el) {
  const exp = getAndRemoveAttr(el, 'v-if')
  if (exp) {
    el.if = exp
    addIfCondition(el, {
      exp: exp,
      block: el
    })
  } else {
    if (getAndRemoveAttr(el, 'v-else') != null) {
      el.else = true
    }
    const elseif = getAndRemoveAttr(el, 'v-else-if')
    if (elseif) {
      el.elseif = elseif
    }
  }
}
```

如果有 `v-if` 属性则会在 `el` 描述对象上添加一个 `if` 属性，如下：

```js
el = {
  type: 1,
  tag,
  attrsList: attrs,
  attrsMap: makeAttrsMap(attrs),
  parent,
  children: [],
  if: {
      name:'v-if',
      value:'aavv'
  }
}
```

### 正则常量

#### 正则常量 onRE

```js
export const onRE = /^@|^v-on:/
```

用来匹配以字符 `@` 或 `v-on:` 开头的字符串，主要作用是检测标签属性名是否是监听事件的指令。

#### 正则常量 dirRE

```js
export const dirRE = /^v-|^@|^:/
```

用来匹配以字符 `v-` 或 `@` 或 `:` 开头的字符串，主要作用是检测标签属性名是否是指令

#### 正则常量 forAliasRE

```js
export const forAliasRE = /([^]*?)\s+(?:in|of)\s+([^]*)/
```

三个分组：

- 第一个分组为 `([^]*?)`

  一个惰性匹配的分组，它匹配的内容为任何字符，包括换行符等。

- 第二个分组为 `(?:in|of)`

  用来匹配字符串 `in` 或者 `of`，并且该分组是非捕获的分组

- 第三个分组为 `([^]*)`

  非惰性匹配，与第一个分组类似

正则 `forAliasRE` 用来匹配 `v-for` 属性的值，并捕获 `in` 或 `of` 前后的字符串

#### 正则常量 forIteratorRE

```js
export const forIteratorRE = /,([^,\}\]]*)(?:,([^,\}\]]*))?$/
```

该正则用来匹配 `forAliasRE` 第一个捕获组所捕获到的字符串

该正则拥有三个分组，其中两个为捕获的分组：第一个捕获组用来捕获一个不包含字符 `,``}` 和 `]` 的字符串，且该字符串前面有一个字符 `,`，如：`', index'`，第二个分组为非捕获的分组，第三个分组为捕获的分组，其捕获的内容与第一个捕获组相同。

目的：配合正则`forAliasRE`，准确匹配`v-for`的三种情况

- 第一种：

  ```js
  <div v-for="obj of list"></div>
  ```

   `forAliasRE` 正则的第一个捕获组的内容为字符串 `'obj'`

  `forIteratorRE` 正则去匹配字符串 `'obj'` 将得不到任何内容

- 第二种：

  ```js
  <div v-for="(obj, index) of list"></div>
  ```

   `forAliasRE` 正则的第一个捕获组的内容为字符串 `'(obj, index)'`，去掉左右括号则该字符串为 `'obj, index'`

  `forIteratorRE` 正则去匹配字符串 `'obj, index'` 则会匹配成功，并且 `forIteratorRE` 正则的第一个捕获组将捕获到字符串 `'index'`，但第二个捕获组捕获不到任何内容。

- 第三种：

  ```js
  <div v-for="(value, key, index) in object"></div>
  ```

  主要用于遍历对象

   `forAliasRE` 正则的第一个捕获组的内容为字符串 `'(value, key, index)'`

  使用 `forIteratorRE` 正则去匹配字符串 `'value, key, index'` 则会匹配成功，并且 `forIteratorRE` 正则的第一个捕获组将捕获到字符串 `'key'`，但第二个捕获组将捕获到字符串 `'index'`

#### 正则常量 stripParensRE

```js
const stripParensRE = /^\(|\)$/g
```

用来捕获要么以字符 `(` 开头，要么以字符 `)` 结尾的字符串，或者两者都满足

#### 正则常量 argRE

```js
const argRE = /:(.*)$/
```

用来匹配指令中的参数

```html
<div v-on:click.stop="handleClick"></div>
```

`v-on` 为指令，`click` 为传递给 `v-on` 指令的参数，`top` 为修饰符

#### 正则常量 bindRE

```js
export const bindRE = /^:|^v-bind:/
```

用来匹配以字符 `:` 或字符串 `v-bind:` 开头的字符串，主要用来检测一个标签的属性是否是绑定(`v-bind`)。

#### 正则常量 modifierRE

```js
const modifierRE = /\.[^.]+/g
```

用来匹配修饰符的，但是并没有捕获任何东西



## parse 函数创建 AST 前的准备工作

首先看一下`parse `函数的简化

```js
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  warn = options.warn || baseWarn
	//值为 options.isPreTag 函数，该函数是一个编译器选项，其作用是通过给定的标签名字判断该标签是否是 pre 标签
  platformIsPreTag = options.isPreTag || no
    //值为 options.mustUseProp 函数，该函数也是一个编译器选项，其作用是用来检测一个属性在标签中是否要使用元素对象原生的 prop 进行绑定
  platformMustUseProp = options.mustUseProp || no
  //值为 options.getTagNamespace 函数，该函数是一个编译器选项，其作用是用来获取元素(标签)的命名空间
  platformGetTagNamespace = options.getTagNamespace || no

  transforms = pluckModuleFunction(options.modules, 'transformNode')
  preTransforms = pluckModuleFunction(options.modules, 'preTransformNode')
  postTransforms = pluckModuleFunction(options.modules, 'postTransformNode')

  delimiters = options.delimiters
//定义stack初始值是一个空数组 --作用是用来修正当前正在解析元素的父级。
  const stack = []
  const preserveWhitespace = options.preserveWhitespace !== false
  let root
  let currentParent
  //用来标识当前解析的标签是否在拥有 v-pre 的标签之内
  let inVPre = false
  //用来标识当前正在解析的标签是否在 <pre></pre> 标签之内
  let inPre = false
  let warned = false

  function warnOnce (msg) {
    // 省略...
  }

  function closeElement (element) {
    // 省略...
  }

  parseHTML(template, {
     warn,
  expectHTML: options.expectHTML,
  isUnaryTag: options.isUnaryTag,
  canBeLeftOpenTag: options.canBeLeftOpenTag,
  shouldDecodeNewlines: options.shouldDecodeNewlines,
  shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
  shouldKeepComment: options.comments,
  start (tag, attrs, unary) {
    // 省略...
  },
  end () {
    // 省略...
  },
  chars (text: string) {
    // 省略...
  },
  comment (text: string) {
    // 省略...
  }
  })
  return root
}
```

4个重要的钩子函数：

- 1、`start` 钩子函数，在解析 `html` 字符串时每次遇到 **开始标签** 时就会调用该函数
- 2、`end` 钩子函数，在解析 `html` 字符串时每次遇到 **结束标签** 时就会调用该函数
- 3、`chars` 钩子函数，在解析 `html` 字符串时每次遇到 **纯文本** 时就会调用该函数
- 4、`comment` 钩子函数，在解析 `html` 字符串时每次遇到 **注释节点** 时就会调用该函数

变量解析

```js
transforms = pluckModuleFunction(options.modules, 'transformNode')
```

 `options.modules` 数组如下

```js
options.modules = [
  {
    staticKeys: ['staticClass'],
    transformNode,    genData
  },
  {
    staticKeys: ['staticStyle'],
    transformNode,    genData
  },
  {
    preTransformNode
  }
]
```

`pluckModuleFunction`函数如下

```js
export function pluckModuleFunction<F: Function> (
  modules: ?Array<Object>,
  key: string
): Array<F> {
  return modules
    ? modules.map(m => m[key]).filter(_ => _)
    : []
}
```

经过解析后

```js
options.modules.map(m => m['transformNode'])
//创建一个新数组
[
  transformNode,
  transformNode,
  undefined
]
//再经过筛选
[
  transformNode,
  transformNode,
  undefined
].filter(_ => _)
//得到以下数组
[
  transformNode,
  transformNode
]
transforms = [
  transformNode,
  transformNode
]
```

同理

```js
preTransforms = pluckModuleFunction(options.modules, 'preTransformNode')
```

```js
preTransforms = [preTransformNode]
```



```js
postTransforms = pluckModuleFunction(options.modules, 'postTransformNode')
```

```js
preTransforms = []
```

## 解析一个开始标签需要做的事情

从`parse`函数中的`start`钩子函数开始

```js
start (tag, attrs, unary) {
  // 省略...
}
```

参数：

- tag -- 标签名字 

- attrs -- 标签的属性数组

- unary -- 代表着该标签是否是一元标签的标识 

  

该函数一开就定义了一个常量

```js
const ns = (currentParent && currentParent.ns) ||platformGetTagNamespace(tag)
```

 `ns` 常量:标签的命名空间

两种情况：

- `currentParent`存在：使用父级的命名空间作为当前元素的命名空间
- `currentParent`不存在或父级元素没有命名空间：通过调用 `platformGetTagNamespace(tag)` 函数获取当前元素的命名空间

减下来是用来代码是用来判断是否兼容`IE`的`SVG`问题

```js
if (isIE && ns === 'svg') {
  attrs = guardIESVGBug(attrs)
}
```

紧接着是的代码是：

```js
let element: ASTElement = createASTElement(tag, attrs, currentParent)
if (ns) {
  element.ns = ns
}
```

为当前元素创建了描述对象，并检查当前元素是否存在命名空间 `ns`，如果存在则在元素对象上添加 `ns` 属性，其值为命名空间的值。

在往下是一段代码：

```js
 if (isForbiddenTag(element) && !isServerRendering()) {
        element.forbidden = true
        process.env.NODE_ENV !== 'production' && warn(
          'Templates should only be responsible for mapping the state to the ' +
          'UI. Avoid placing tags with side-effects in your templates, such as ' +
          `<${tag}>` + ', as they will not be parsed.',
          { start: element.start }
        )
      }
```

用来判断非服务端渲染情况下，当前元素是否是禁止在模板中使用的标签。

`isForbiddenTag` 函数

```js
function isForbiddenTag (el): boolean {
  return (
    el.tag === 'style' ||
    (el.tag === 'script' && (
      !el.attrsMap.type ||
      el.attrsMap.type === 'text/javascript'
    ))
  )
}
```

根据源码可知以下标签为被禁止的标签：

- 1、`<style>` 标签为被禁止的标签
- 2、没有指定 `type` 属性或虽然指定了 `type` 属性但其值为 `text/javascript` 的 `<script>` 标签被认为是被禁止的

继续往下看代码

```js
for (let i = 0; i < preTransforms.length; i++) {
  element = preTransforms[i](element, options) || element
}
```

`preTransforms`数组如下

```js
preTransforms = [preTransformNode]
```

通过`preTransformNode`函数对`element`进行进一步处理

接下来的代码也是对`element`进行处理的

```js

if (!inVPre) {
    processPre(element)
    if (element.pre) {
        inVPre = true
    }
}
if (platformIsPreTag(element.tag)) {
    inPre = true
}
if (inVPre) {
    processRawAttrs(element)
} else if (!element.processed) {
    processFor(element)
    processIf(element)
    processOnce(element)
}

```

上述代码之后再回顾，现在继续往下看

```js
  if (!root) {
   root = element
   if (process.env.NODE_ENV !== 'production') {
   //它的作用是用来检测模板根元素是否符合要求
          checkRootConstraints(root)
   }
}
```

这一段代码才是初始化`root`的地方

接下来一段代码是判断该标签是否已经闭合

```js
//unary 为underfined 的时候 该标签为开始标签
if (!unary) {
    //缓存当前标签 以便后续作为父标签使用
   currentParent = element
   stack.push(element)
} else {
    //如果能进入该分支的话，代表标签要进行闭合了
   closeElement(element)
}
```

`closeElement`函数如下

```js
 function closeElement(element) {
    trimEndingWhitespace(element)
    if (!inVPre && !element.processed) {
      element = processElement(element, options)
    }
    // tree management
    if (!stack.length && element !== root) {
      // allow root elements with v-if, v-else-if and v-else
      if (root.if && (element.elseif || element.else)) {
        if (process.env.NODE_ENV !== 'production') {
          checkRootConstraints(element)
        }
        addIfCondition(root, {
          exp: element.elseif,
          block: element
        })
      } else if (process.env.NODE_ENV !== 'production') {
        warnOnce(
          `Component template should contain exactly one root element. ` +
          `If you are using v-if on multiple elements, ` +
          `use v-else-if to chain them instead.`,
          { start: element.start }
        )
      }
    }
    if (currentParent && !element.forbidden) {
      if (element.elseif || element.else) {
        processIfConditions(element, currentParent)
      } else {
        if (element.slotScope) {
          // scoped slot
          // keep it in the children list so that v-else(-if) conditions can
          // find it as the prev node.
          const name = element.slotTarget || '"default"'
            ; (currentParent.scopedSlots || (currentParent.scopedSlots = {}))[name] = element
        }
        currentParent.children.push(element)
        element.parent = currentParent
      }
    }

    // final children cleanup
    // filter out scoped slots
    element.children = element.children.filter(c => !(c: any).slotScope)
    // remove trailing whitespace node again
    trimEndingWhitespace(element)

    // check pre state
    if (element.pre) {
      inVPre = false
    }
    if (platformIsPreTag(element.tag)) {
      inPre = false
    }
    // apply post-transforms
    for (let i = 0; i < postTransforms.length; i++) {
      postTransforms[i](element, options)
    }
  }
```

首先一开始看到了

```js
 trimEndingWhitespace(element)
```

其源码如下

```js
 function trimEndingWhitespace(el) {
    // remove trailing whitespace node
    if (!inPre) {
      let lastNode
      while (
        (lastNode = el.children[el.children.length - 1]) &&
        lastNode.type === 3 &&
        lastNode.text === ' '
      ) {
        el.children.pop()
      }
    }
  }
```

作用一目了然，就是为了将空文本全部清除

接下来的代码是

```js
if (!inVPre && !element.processed) {
      element = processElement(element, options)
}
```

该段代码就是判断一下这`element`是否为`v-pre`和未被加工处理过，如果是就用`processElement`进行加工

`processElement`函数如下

```js
export function processElement(
  element: ASTElement,
  options: CompilerOptions
) {
  processKey(element)

  // determine whether this is a plain element after
  // removing structural attributes
  element.plain = (
    !element.key &&
    !element.scopedSlots &&
    !element.attrsList.length
  )

  processRef(element)
  processSlotContent(element)
  processSlotOutlet(element)
  processComponent(element)
  for (let i = 0; i < transforms.length; i++) {
    element = transforms[i](element, options) || element
  }
  processAttrs(element)
  return element
}
```

这个先不管，在后面[`processElement`函数](#`processElement`函数)讲到

先看一下后续的检测

```js
//!stack.length -- 整个模板已经解析完成
//element !== root -- 不止一个根节点
 if (!stack.length && element !== root) {
     
      if (root.if && (element.elseif || element.else)) {
        if (process.env.NODE_ENV !== 'production') {
          checkRootConstraints(element)
        }
        addIfCondition(root, {
          exp: element.elseif,
          block: element
        })
      } else if (process.env.NODE_ENV !== 'production') {
          
        warnOnce(
          `Component template should contain exactly one root element. ` +
          `If you are using v-if on multiple elements, ` +
          `use v-else-if to chain them instead.`,
          { start: element.start }
        )
      }
    }
```

如果不止一个根节点的时候，判断是否最多只渲染一个根元素

```js
 if (root.if && (element.elseif || element.else)) {
        if (process.env.NODE_ENV !== 'production') {
          checkRootConstraints(element)
        }
        addIfCondition(root, {
          exp: element.elseif,
          block: element
        })
      }
```

该代码调用了 `addIfCondition` 函数

```js
export function addIfCondition(el: ASTElement, condition: ASTIfCondition) {
  if (!el.ifConditions) {
    el.ifConditions = []
  }
  el.ifConditions.push(condition)
}
```

比如

```html
<div v-if="a"></div>
<p v-else-if="b"></p>
<span v-else></span>
```

解析后结果

```js
{
  type: 1,
  tag: 'div',
  ifConditions: [
    {
      exp: {
          name:'v-else-if',
          value:'b'
      },
      block: { type: 1, tag: 'p' /* 省略其他属性 */ }
    },
    {
      exp: undefined,
      block: { type: 1, tag: 'span' /* 省略其他属性 */ }
    }
  ]
  // 省略其他属性...
}
```

实际上其解析的结果如下

```js
{
  type: 1,
  tag: 'div',
  ifConditions: [
    {      
        exp: {
        	name:'v-if',
        	value:'a'
    	},      
        block: { type: 1, tag: 'div' /* 省略其他属性 */ }    
    },   
     {
      exp: {
        	name:'v-if',
        	value:'b'
    	},
      block: { type: 1, tag: 'p' /* 省略其他属性 */ }
    },
    {
      exp: undefined,
      block: { type: 1, tag: 'span' /* 省略其他属性 */ }
    }
  ]
  // 省略其他属性...
}
```

在`processIf`函数中有,有一处代码对其进行了操作

```js
 if (exp) {
    el.if = exp
    addIfCondition(el, {
      exp: exp,
      block: el
    })
  }
```



继续往下,当前元素存在父级(`currentParent`)，并且当前元素不是被禁止的元素。

```js

if (currentParent && !element.forbidden) {
    //如果当前元素为不确定元素
      if (element.elseif || element.else) {
        processIfConditions(element, currentParent)
      } else {
         //当前元素为兄弟元素
          
          
        if (element.slotScope) {
          // scoped slot
          // keep it in the children list so that v-else(-if) conditions can
          // find it as the prev node.
          const name = element.slotTarget || '"default"'
            ; (currentParent.scopedSlots || (currentParent.scopedSlots = {}))[name] = element
        }
          //将子元素推到父元素的子数组中
        currentParent.children.push(element)
        element.parent = currentParent
      }
    }
```

如果一个标签使用 `v-else-if` 或 `v-else` 指令，那么该元素的描述对象实际上会被添加到对应的 `v-if` 元素描述对象的 `ifConditions` 数组中，而非作为一个独立的子节点，这个工作就是由如上代码中 `if` 语句块的代码完成的：

```js
if (element.elseif || element.else) {
  processIfConditions(element, currentParent)
} else {
  // 省略...
}
```

`processIfConditions`函数

```js
function processIfConditions(el, parent) {
  const prev = findPrevElement(parent.children)
  if (prev && prev.if) {
    addIfCondition(prev, {
      exp: el.elseif,
      block: el
    })
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `v-${el.elseif ? ('else-if="' + el.elseif + '"') : 'else'} ` +
      `used on element <${el.tag}> without corresponding v-if.`,
      el.rawAttrsMap[el.elseif ? 'v-else-if' : 'v-else']
    )
  }
}
```

首先看下这个，找出当前元素的前一个元素描述对象，并将其赋值给 `prev` 常量

```js
const prev = findPrevElement(parent.children)
```

`findPrevElement`函数源码

```js
function findPrevElement(children: Array<any>): ASTElement | void {
  let i = children.length
  while (i--) {
    if (children[i].type === 1) {
      return children[i]
    } else {
      if (process.env.NODE_ENV !== 'production' && children[i].text !== ' ') {
        warn(
          `text "${children[i].text.trim()}" between v-if and v-else(-if) ` +
          `will be ignored.`,
          children[i]
        )
      }
      children.pop()
    }
  }
}
```

然后查看前一个元素是否存在并确实使用了 `v-if` 指令，那么则会调用 `addIfCondition` 函数将当前元素描述对象添加到前一个元素的 `ifConditions` 数组中。

```
 if (prev && prev.if) {
    addIfCondition(prev, {
      exp: el.elseif,
      block: el
    })
  }
```

由此可知 **当一个元素使用了 `v-else-if` 或 `v-else` 指令时，它们是不会作为父级元素子节点的**，而是会被添加到相符的使用了 `v-if` 指令的元素描述对象的 `ifConditions` 数组中。

回到上面代码中

```js
if (currentParent && !element.forbidden) {
      if (element.elseif || element.else) {
        processIfConditions(element, currentParent)
      } else {
        if (element.slotScope) {
          // scoped slot
          // keep it in the children list so that v-else(-if) conditions can
          // find it as the prev node.
          const name = element.slotTarget || '"default"'
            ; (currentParent.scopedSlots || (currentParent.scopedSlots = {}))[name] = element
        }
        currentParent.children.push(element)
        element.parent = currentParent
      }
 }
```

如果一个元素使用了 `slot-scope` 特性，那么该元素的描述对象会被添加到父级元素的 `scopedSlots` 对象下，也就是说使用了 `slot-scope` 特性的元素与使用了 `v-else-if` 或 `v-else` 指令的元素一样，他们都不会作为父级元素的子节点，对于使用了 `slot-scope` 特性的元素来讲它们将被添加到父级元素描述对象的 `scopedSlots` 对象下。

### `start`函数小结

- 1、`start` 钩子函数是当解析 `html` 字符串遇到开始标签时被调用的。
- 2、模板中禁止使用 `<style>` 标签和那些没有指定 `type` 属性或 `type` 属性值为 `text/javascript` 的 `<script>` 标签。
- 3、在 `start` 钩子函数中会调用前置处理函数，这些前置处理函数都放在 `preTransforms` 数组中，这么做的目的是为不同平台提供对应平台下的解析工作。
- 4、前置处理函数执行完之后会调用一系列 `process*` 函数继续对元素描述对象进行加工。
- 5、通过判断 `root` 是否存在来判断当前解析的元素是否为根元素。
- 6、`slot` 标签和 `template` 标签不能作为根元素，并且根元素不能使用 `v-for` 指令。
- 7、可以定义多个根元素，但必须使用 `v-if`、`v-else-if` 以及 `v-else` 保证有且仅有一个根元素被渲染。
- 8、构建 `AST` 并建立父子级关系是在 `start` 钩子函数中完成的，每当遇到非一元标签，会把它存到 `currentParent` 变量中，当解析该标签的子节点时通过访问 `currentParent` 变量获取父级元素。
- 9、如果一个元素使用了 `v-else-if` 或 `v-else` 指令，则该元素不会作为子节点，而是会被添加到相符的使用了 `v-if` 指令的元素描述对象的 `ifConditions` 数组中。
- 10、如果一个元素使用了 `slot-scope` 特性，则该元素也不会作为子节点，它会被添加到父级元素描述对象的 `scopedSlots` 属性中。
- 11、对于没有使用条件指令或 `slot-scope` 特性的元素，会正常建立父子级关系。



## 处理使用了v-pre指令的元素及其子元素

在`start`钩子函数中，有一段代码是判断是否使用了`v-pre`指令的

```js
if (!inVPre) {
  processPre(element)
  if (element.pre) {
    inVPre = true
  }
}
```

这没头没尾的也不知道是什么作用，那就先看看`processPre`函数是干什么的

### `processPre`函数

```js
function processPre (el) {
  if (getAndRemoveAttr(el, 'v-pre') != null) {
    el.pre = true
  }
}
```

`processPre` 函数接收元素描述对象作为参数，通过 `getAndRemoveAttr` 函数并使用其返回值与 `null` 做比较，如果 `getAndRemoveAttr` 函数的返回值不等于 `null` 则执行 `if` 语句块内的代码，即在元素描述对象上添加 `.pre` 属性并将其值设置为 `true`。

那么`processPre`函数关键点就在于`getAndRemoveAttr`函数上了

### `getAndRemoveAttr`函数

`getAndRemoveAttr`函数代码如下

```js
export function getAndRemoveAttr (
  el: ASTElement,
  name: string,
  removeFromMap?: boolean
): ?string {
  let val
  if ((val = el.attrsMap[name]) != null) {
    const list = el.attrsList
    for (let i = 0, l = list.length; i < l; i++) {
      if (list[i].name === name) {
        list.splice(i, 1)
        break
      }
    }
  }
  if (removeFromMap) {
    delete el.attrsMap[name]
  }
  return val
}
```

参数：

- `removeFromMap` -- 一个可选参数，并且它应该是一个布尔值
- `el` -- 元素描述对象
- `name` -- 要获取属性的名字

返回值：

​	`undefined`或者获取属性的值（Object）

回到 `processPre` 函数中

```js
function processPre (el) {
  if (getAndRemoveAttr(el, 'v-pre') != null) {
    el.pre = true
  }
}
```

如果 `v-pre` 属性的值不等于 `null` 则会在元素描述对象上添加 `.pre` 属性，并将其值设置为 `true`

了解了 `precessPre` 函数的作用之后，再回到 `start` 钩子函数

```js
if (!inVPre) {
  processPre(element)
  if (element.pre) {
      inVPre = true  
  }
}
```

如果一个标签使用了`v-pre`指令，经过 `processPre` 函数处理，该元素描述对象的 `.pre` 属性值为 `true`，这时会将 `inVPre` 变量的值也设置为 `true`。当 `inVPre` 变量为真时，意味着 **后续的所有解析工作都处于 `v-pre` 环境下**，编译器会跳过拥有 `v-pre` 指令元素以及其子元素的编译过程，所以后续的编译逻辑需要 `inVPre` 变量作为标识才行

同理，在`start`钩子函数中，还有一段判断`<pre>`标签的代码

```js
if (platformIsPreTag(element.tag)) {
  inPre = true
}
```

由于`<pre> `标签内的解析行为与其他` html` 标签是不同，体现在于：

- 1、`<pre>` 标签会对其所包含的 `html` 字符实体进行解码
- 2、`<pre>` 标签会保留 `html` 字符串编写时的空白

经过上述的判断，目前已经可以得知该`element`是否存在于`pre`中或者使用`v-pre`指令，接下来的代码是对该`element`进行处理

```js
if (inVPre) {
  processRawAttrs(element)
} else if (!element.processed) {
  processFor(element)
  processIf(element)
  processOnce(element)
}
```

上述代码是通过判别`inVpre`是否为真，来进行不同的操作

例如

```html
<div v-pre v-on:click="handleClick"></div>
```

当解析如上 `html` 字符串时首先会遇到 `div` 开始标签，由于该 `div` 开始标签使用了 `v-pre` 指令，所以此时 `inVPre` 的值为真，所以 `processRawAttrs` 函数将被执行

此时的`element`如下

```js
{
    type: 1,
    tag:'div',
    attrsList: [{
        name:'v-pre',
        value:''
    },{
        name:'v-on:click',
        value:'handleClick'
    }],
    attrsMap:{
        'v-pre':'',
          'v-on:click':'handleClick'
    },
    rawAttrsMap: {},
    parent,
    children: []
  }
```



### `processRawAttrs` 函数

```js
function processRawAttrs(el) {
  const list = el.attrsList
  const len = list.length
  if (len) {
    const attrs: Array<ASTAttr> = el.attrs = new Array(len)
    for (let i = 0; i < len; i++) {
      attrs[i] = {
        name: list[i].name,
        value: JSON.stringify(list[i].value)
      }
      if (list[i].start != null) {
        attrs[i].start = list[i].start
        attrs[i].end = list[i].end
      }
    }
  } else if (!el.pre) {
    // non root node in pre blocks with no attributes
    el.plain = true
  }
}
```

首先看一下`if`部分的代码

其实部分代码的作用就是为了创建一个`value`为字符串类型的数组

```js
attrs[i] = {
  name: el.attrsList[i].name,
  value: JSON.stringify(el.attrsList[i].value)
}
```

那么现在`element`的值如下

```js
{
    type: 1,
    tag:'div',
    attrsList: [{
        name:'v-pre',
        value:''
    },{
        name:'v-on:click',
        value:'handleClick'
    }],
    attrsMap:{
        'v-pre':'',
         'v-on:click':'handleClick'
    },
     attrs: [{
        name:'v-pre',
        value:"\"\""
    },
            {
        name:'v-pre',
        value:"\"handleClick\""
    }],
    rawAttrsMap: {},
    parent,
    children: []
  }
```

 

然后看一下`else`部分

```js
function processRawAttrs (el) {
  const list = el.attrsList
  const len = list.length
  if (len) {
    // 省略...
  } else if (!el.pre) {
    // non root node in pre blocks with no attributes
    el.plain = true  }
}
```

假如 `el.attrsList` 数组的长度为 `0`，则会进入 `else...if` 分支的判断

当然有个问题是如果能进入`processRawAttrs` 函数的执行说明当前解析必然处于 `v-pre` 环境，要么是使用 `v-pre` 指令的标签自身，要么就是其子节点。

满足 `el.attrsList` 数组的长度为 `0` ，说明该元素没有任何属性，而且 `else...if` 条件的成立也说明该元素没有使用 `v-pre` 指令，这说明该元素一定是使用了 `v-pre` 指令的标签的子标签，例如：

```html
<div v-pre>
  <span></span>
</div>
```

### 总结

- 1、如果标签使用了 `v-pre` 指令，则该标签的元素描述对象的 `element.pre` 属性将为 `true`。
- 2、对于使用了 `v-pre` 指令的标签及其子代标签，它们的任何属性都将会被作为原始属性处理，即使用 `processRawAttrs` 函数处理之。
- 3、经过 `processRawAttrs` 函数的处理，会在元素的描述对象上添加 `element.attrs` 属性，它与 `element.attrsList` 数组结构相同，不同的是 `element.attrs` 数组中每个对象的 `value` 值会经过 `JSON.stringify` 函数处理。
- 4、如果一个标签没有任何属性，并且该标签是使用了 `v-pre` 指令标签的子代标签，那么该标签的元素描述对象将被添加 `element.plain` 属性，并且其值为 `true`。



## 处理使用了v-for指令的元素

上面处理完了使用`v-pre`指令的情况，下面看一下不使用`v-pre`指令会干什么

回到`start`钩子函数

```js
if (inVPre) {
  // 省略...
} else if (!element.processed) {
    //element.processed --标识着当前元素是否已经被解析过了
 processFor(element)
 processIf(element)
 processOnce(element)
}
```

首先看下`processFor`函数

### `processFor`函数源码

```js
export function processFor(el: ASTElement) {
  let exp
  if ((exp = getAndRemoveAttr(el, 'v-for'))) {
    const res = parseFor(exp)
    if (res) {
      extend(el, res)
    } else if (process.env.NODE_ENV !== 'production') {
      warn(
        `Invalid v-for expression: ${exp}`,
        el.rawAttrsMap['v-for']
      )
    }
  }
}
```

首先获取`v-for` 属性对应的属性值

```js
exp = getAndRemoveAttr(el, 'v-for')
```

例如

```js
<div v-for="obj in list"></div>
```

那么经过解析后，`exp`的值为`obj in list`

当获取到`exp`后，经该值赋予`parseFor`函数进行处理

```js
 const res = parseFor(exp)
```

### `parseFor`函数

```js
export function parseFor(exp: string): ?ForParseResult {
  const inMatch = exp.match(forAliasRE)
  if (!inMatch) return
  const res = {}
  res.for = inMatch[2].trim()
  const alias = inMatch[1].trim().replace(stripParensRE, '')
  const iteratorMatch = alias.match(forIteratorRE)
  if (iteratorMatch) {
    res.alias = alias.replace(forIteratorRE, '').trim()
    res.iterator1 = iteratorMatch[1].trim()
    if (iteratorMatch[2]) {
      res.iterator2 = iteratorMatch[2].trim()
    }
  } else {
    res.alias = alias
  }
  return res
}
```

首先进行一边正则校验，提取字符串中的参数

```js
//export const forAliasRE = /([\s\S]*?)\s+(?:in|of)\s+([\s\S]*)/

const inMatch = exp.match(forAliasRE)
```

获取到`inMatch`的值如下

```js
const inMatch = [
  'obj in list',
  'obj',
  'list'
]
```

如果匹配失败则 `inMatch` 常量的值将为 `null`。那么函数直接返回`underfind`

接下来是对`inMatch`数据进行分析处理

```js
//定义一个空对象，用于返回 
const res = {}
res.for = inMatch[2].trim()
const alias = inMatch[1].trim().replace(stripParensRE, '')
const iteratorMatch = alias.match(forIteratorRE)
```

提取循环的数组或者对象

```js
res.for = inMatch[2].trim()
```

这是`res`的值如下

```
res = {
	for:'list'
}
```

接下是处理`inMatch[1]`的值

```js
const alias = inMatch[1].trim().replace(stripParensRE, '')
```

主要是除掉多余的空格

比如：

```html
<div v-for="  obj in list"></div>
```

`inMatch[1]`的值是`' obj'`,`inMatch[1].trim()`去掉了多余的空格

至于`replace(stripParensRE, '')`是为了清楚括号的，比如：

```html
<div v-for="(obj,key,index) in list"></div>
```

经过处理后`inMatch[1]`的值为`obj,key,index`

如下是 `v-for` 指令的值与 `alias` 常量值的对应关系：

- 1、如果 `v-for` 指令的值为 `'obj in list'`，则 `alias` 的值为字符串 `'obj'`
- 2、如果 `v-for` 指令的值为 `'(obj, index) in list'`，则 `alias` 的值为字符串 `'obj, index'`
- 3、如果 `v-for` 指令的值为 `'(obj, key, index) in list'`，则 `alias` 的值为字符串 `'obj, key, index'`

得到符合规范的`alias`后，就要将其里面的值解析出来

```js
 //export const forIteratorRE = /,([^,\}\]]*)(?:,([^,\}\]]*))?$/
 const iteratorMatch = alias.match(forIteratorRE)
```

由于 `v-for` 指令的值能有不同情况，那么这里的`iteratorMatch`需要进行区分

- 如果 `alias` 字符串的值为 `'obj'`，则匹配结果 `iteratorMatch` 常量的值为 `null`
- 如果 `alias` 字符串的值为 `'obj, index'`，则匹配结果 `iteratorMatch` 常量的值是一个包含两个元素的数组：`[', index', 'index']`
- 如果 `alias` 字符串的值为 `'obj, key, index'`，则匹配结果 `iteratorMatch` 常量的值是一个包含三个元素的数组：`[', key, index', 'key', 'index']`

接下来是针对`iteratorMatch`变量进行处理

```js
//例如alias = 'obj, key, index'
//那么 iteratorMatch = [', key, index', 'key', 'index']
if (iteratorMatch) {
    //res.alias = obj
    res.alias = alias.replace(forIteratorRE, '')
    // res.iterator1 = 'key'
    res.iterator1 = iteratorMatch[1].trim()
    if (iteratorMatch[2]) {
        // res.iterator2 ='index'
      res.iterator2 = iteratorMatch[2].trim()
    }
  } else {
    res.alias = alias
  }
```

到此为止`parseFor`函数已经分析完成

那么小结一下，`parseFor`函数针对`v-for`指令不同的值，各自返回的值

- `v-for='obj in list'`

  ```js
  {
    for: 'list',
    alias: 'obj'
  }
  ```

- `v-for='(obj, index) in list'`

  ```js
  {
    for: 'list',
    alias: 'obj',
    iterator1:'index'
  }
  ```

- `v-for='(obj, key, index) in list'`

  ```js
  {
    for: 'list',
    alias: 'obj',
    iterator1:'key',
     iterator2:'index'
  }
  ```

那么回到`processFor `函数

```js
export function processFor(el: ASTElement) {
  let exp
  if ((exp = getAndRemoveAttr(el, 'v-for'))) {
    const res = parseFor(exp)
    if (res) {
      extend(el, res)
    } else if (process.env.NODE_ENV !== 'production') {
      warn(
        `Invalid v-for expression: ${exp}`,
        el.rawAttrsMap['v-for']
      )
    }
  }
}
```

获取到`res`后，如果`res`存在，那么将`res`混入到`el`中

比如：

```html
<div v-for="(obj,key,index) in list"></div>
```

那么得到的`res`如下

```js
{
  for: 'list',
  alias: 'obj',
  iterator1:'key',
  iterator2:'index'
}
```

`element`如下

```js
{
    type: 1,
    tag：'div',
    attrsList: [{
        name:'v-for',
        value:'(obj,key,index) in list'
    }],
    attrsMap: {
        'v-for':'(obj,key,index) in list'
    },
    rawAttrsMap: {},
    parent,
    children: []
  }
```

混入后，`element`如下

```js
{
    type: 1,
    tag：'div',
    attrsList: [{
        name:'v-for',
        value:'(obj,key,index) in list'
    }],
    attrsMap: {
        'v-for':'(obj,key,index) in list'
    },
    rawAttrsMap: {},
    parent,
    children: [],
    for: 'list',
  	alias: 'obj',
  	iterator1:'key',
  	iterator2:'index'
}
```

## 处理使用条件指令和v-once指令的元素

回到`start`钩子函数

```js
if (inVPre) {
  processRawAttrs(element)
} else if (!element.processed) {   
  processFor(element)
  processIf(element)
  processOnce(element)
}
```

在处理完`processFor`函数后，就是`processIf`函数和`processOnce`函数了

由于`processOnce`函数比较简单就一起过一遍

### 条件指令

`processIf`函数源码

```js
function processIf(el) {
  const exp = getAndRemoveAttr(el, 'v-if')
  if (exp) {
    el.if = exp
    addIfCondition(el, {
      exp: exp,
      block: el
    })
  } else {
    if (getAndRemoveAttr(el, 'v-else') != null) {
      el.else = true
    }
    const elseif = getAndRemoveAttr(el, 'v-else-if')
    if (elseif) {
      el.elseif = elseif
    }
  }
}
```

真的很简单，简单来说就是判别一下元素是否含有`v-if`,`v-else-if`,`v-else`,

如果有`v-if`

```js
 el.if = exp
addIfCondition(el, {
  exp: exp,
  block: el
})
```

比如

```html
<div v-if="a && b"></div>
```

那么执行完后`el`的值如下

```js
{
    type: 1,
    tag：'div',
    attrsList: [{
        name:'v-if',
        value:'a && b'
    }],
    attrsMap: {
        'v-if':'a && b'
    },
    rawAttrsMap: {},
    parent,
    children: [],
   ifConditions:[{   
       exp: {
           name:'v-if',
           value:'a && b'
       },     
       block: { type: 1, tag: 'div' /* 省略其他属性 */ }  
   }]
   if:{
      name:'v-if',
      value:'a && b' 
   }
}
```

剩下的`v-else`和`v-else-if`同理

简短的总结:

- 如果标签使用了 `v-if` 指令，则该标签的元素描述对象的 `el.if` 属性存储着 `v-if` 指令的属性值
- 如果标签使用了 `v-else` 指令，则该标签的元素描述对象的 `el.else` 属性值为 `true`
- 如果标签使用了 `v-else-if` 指令，则该标签的元素描述对象的 `el.elseif` 属性存储着 `v-else-if` 指令的属性值
- 如果标签使用了 `v-if` 指令，则该标签的元素描述对象的 `ifConditions` 数组中包含“自己”
- 如果标签使用了 `v-else` 或 `v-else-if` 指令，则该标签的元素描述对象会被添加到与之相符的带有 `v-if` 指令的元素描述对象的 `ifConditions` 数组中。



### `v-once`指令

```
function processOnce (el) {
  const once = getAndRemoveAttr(el, 'v-once')
  if (once != null) {
    el.once = true
  }
}
```

同理

## `processElement`函数

上面的处理都已经完成，那么代码回到`start`钩子函数的最末尾

```js
 if (!unary) {
        currentParent = element
        stack.push(element)
 } else {
        closeElement(element)
}
```

之前粗略得看了一下`closeElement`函数，但当时还有一段代码未详细查看

```js
if (!inVPre && !element.processed) {
      element = processElement(element, options)
}
```

这一段代码的成立条件是没有使用`v-pre`以及未完成处理，那么看一下`processElement`函数是干什么的

### 源码

```js
export function processElement(
  element: ASTElement,
  options: CompilerOptions
) {
  processKey(element)

  // determine whether this is a plain element after
  // removing structural attributes
  element.plain = (
    !element.key &&
    !element.scopedSlots &&
    !element.attrsList.length
  )

  processRef(element)
  processSlotContent(element)
  processSlotOutlet(element)
  processComponent(element)
  for (let i = 0; i < transforms.length; i++) {
    element = transforms[i](element, options) || element
  }
  processAttrs(element)
  return element
}
```

首先看一下`processKey`函数

## 处理使用了key属性的元素

### `processKey`函数源码

```js
function processKey(el) {
  const exp = getBindingAttr(el, 'key')
  if (exp) {
    if (process.env.NODE_ENV !== 'production') {
      if (el.tag === 'template') {
        warn(
          `<template> cannot be keyed. Place the key on real elements instead.`,
          getRawBindingAttr(el, 'key')
        )
      }
      if (el.for) {
        const iterator = el.iterator2 || el.iterator1
        const parent = el.parent
        if (iterator && iterator === exp && parent && parent.tag === 'transition-group') {
          warn(
            `Do not use v-for index as key on <transition-group> children, ` +
            `this is the same as not using keys.`,
            getRawBindingAttr(el, 'key'),
            true /* tip */
          )
        }
      }
    }
    el.key = exp
  }
}
```

该函数和其他处理`element`函数十分相识，都是判断有没有使用到该指令，如果有就做一个标识

但是这函数获取指令的函数`getBindingAttr`与`getAndRemoveAttr`函数作用相同，但是进行了不同的操作

### `getBindingAttr`函数

```js
export function getBindingAttr (
  el: ASTElement,
  name: string,
  getStatic?: boolean
): ?string {
  const dynamicValue =
    getAndRemoveAttr(el, ':' + name) ||
    getAndRemoveAttr(el, 'v-bind:' + name)
  if (dynamicValue != null) {
    return parseFilters(dynamicValue)
  } else if (getStatic !== false) {
    const staticValue = getAndRemoveAttr(el, name)
    if (staticValue != null) {
       // JSON.stringify 函数处理其属性值 目的就是确保将非绑定的属性值作为字符串处理，而不是变量或表达式。
      return JSON.stringify(staticValue)
    }
  }
}
```

首先获取到了绑定的属性

```js
 const dynamicValue =
    getAndRemoveAttr(el, ':' + name) ||
    getAndRemoveAttr(el, 'v-bind:' + name)
```

目的是为了兼容vue中不同的写法

```html
<div v-bind:key='1'></div>
<div :key='1'></div>
```

获取到了绑定的属性后，会执行以下代码

```js
  if (dynamicValue != null) {
    return parseFilters(dynamicValue)
  } else if (getStatic !== false) {
    const staticValue = getAndRemoveAttr(el, name)
    if (staticValue != null) {
      return JSON.stringify(staticValue)
    }
  }
```

注意 :`if` 语句的条件是在判断绑定的属性是否存在，而非判断属性值 `dynamicValue` 是否存在，因为即使获取到的属性值为空字符串，但由于空字符串不与 `null` 相等，所以 `if` 条件语句成立。只有当绑定属性本身就不存在时，此时获取到的属性值为 `undefined`，与 `null` 相等，这时才会执行 `elseif` 分支的判断。

然后关键的来了`parseFilters`函数究竟是用来干什么的？

`parseFilters` 函数的作用就像它的名字一样，是用来解析过滤器的，换句话说在编写绑定的属性时可以使用过滤器

比如：

```html
<div :key="id | featId"></div>
```

像这种用到了计算的值，为了让其拥有使用过滤器的能力，就需要使用 `parseFilters` 函数处理。

### `parseFilters`函数

```js
export function parseFilters (exp: string): string {
  let inSingle = false
  let inDouble = false
  let inTemplateString = false
  let inRegex = false
  let curly = 0
  let square = 0
  let paren = 0
  let lastFilterIndex = 0
  let c, prev, i, expression, filters

  for (i = 0; i < exp.length; i++) {
    prev = c
    c = exp.charCodeAt(i)
    if (inSingle) {
      if (c === 0x27 && prev !== 0x5C) inSingle = false
    } else if (inDouble) {
      if (c === 0x22 && prev !== 0x5C) inDouble = false
    } else if (inTemplateString) {
      if (c === 0x60 && prev !== 0x5C) inTemplateString = false
    } else if (inRegex) {
      if (c === 0x2f && prev !== 0x5C) inRegex = false
    } else if (
      c === 0x7C && // pipe
      exp.charCodeAt(i + 1) !== 0x7C &&
      exp.charCodeAt(i - 1) !== 0x7C &&
      !curly && !square && !paren
    ) {
      if (expression === undefined) {
        // first filter, end of expression
        lastFilterIndex = i + 1
        expression = exp.slice(0, i).trim()
      } else {
        pushFilter()
      }
    } else {
      switch (c) {
        case 0x22: inDouble = true; break         // "
        case 0x27: inSingle = true; break         // '
        case 0x60: inTemplateString = true; break // `
        case 0x28: paren++; break                 // (
        case 0x29: paren--; break                 // )
        case 0x5B: square++; break                // [
        case 0x5D: square--; break                // ]
        case 0x7B: curly++; break                 // {
        case 0x7D: curly--; break                 // }
      }
      if (c === 0x2f) { // /
        let j = i - 1
        let p
        // find first non-whitespace prev char
        for (; j >= 0; j--) {
          p = exp.charAt(j)
          if (p !== ' ') break
        }
        if (!p || !validDivisionCharRE.test(p)) {
          inRegex = true
        }
      }
    }
  }

  if (expression === undefined) {
    expression = exp.slice(0, i).trim()
  } else if (lastFilterIndex !== 0) {
    pushFilter()
  }

  function pushFilter () {
    (filters || (filters = [])).push(exp.slice(lastFilterIndex, i).trim())
    lastFilterIndex = i + 1
  }

  if (filters) {
    for (i = 0; i < filters.length; i++) {
      expression = wrapFilter(expression, filters[i])
    }
  }

  return expression
}
```

`parseFilters` 函数在开头定义了一些变量

#### 变量

```js
 let inSingle = false
  let inDouble = false
  let inTemplateString = false
  let inRegex = false
```

作用：

- `inSingle` 变量的作用是用来标识当前读取的字符是否在由**单引号**包裹的字符串中
- `inDouble` 变量是用来标识当前读取的字符是否在由 **双引号** 包裹的字符串中
- `inTemplateString` 变量是用来标识当前读取的字符是否在 **模板字符串** 中。
- `inRegex` 变量是用来标识当前读取的字符是否在 **正则表达式** 中。



```js
 let curly = 0
  let square = 0
  let paren = 0
```

作用：

-  `curly` 变量 --记录花括号数量

  在解析绑定的属性值时，每遇到一个左花括号(`{`)，则 `curly` 变量的值就会加一，每遇到一个右花括号(`}`)，则 `curly` 变量的值就会减一。

- `square` 变量--记录方括号数量

  在解析绑定的属性值时，每遇到一个左方括号(`[`)，则 `square` 变量的值就会加一，每遇到一个右方括号(`]`)，则 `square` 变量的值就会减一。

- `paren` 变量 -- 记录圆括号数量

  在解析绑定的属性值时，每遇到一个左圆括号(`(`)，则 `paren` 变量的值就会加一，每遇到一个右圆括号(`)`)，则 `paren` 变量的值就会减一

```js
let lastFilterIndex = 0
let c, prev, i, expression, filters
```

作用：

- 变量`lastFilterIndex`  -- 用来确定过滤器的位置，值是属性值字符串中字符的索引
- 变量 `c` -- 当前读入字符所对应的 `ASCII` 码
- 变量 `prev`--当前字符的前一个字符所对应的 `ASCII` 码
- 变量 `i`  -- 当前读入字符的位置索引
- 变量 `expression`  --  `parseFilters` 函数的返回值
- 变量 `filters` -- 数组，保存着所有过滤器函数名



接下是`parseFilters`函数的核心代码`for`循环，首先看一下简化后的代码

```js
for (i = 0; i < exp.length; i++) {
  prev = c
  c = exp.charCodeAt(i)
  if (inSingle) {
    // 如果当前读取的字符存在于由单引号包裹的字符串内，则会执行这里的代码
  } else if (inDouble) {
    // 如果当前读取的字符存在于由双引号包裹的字符串内，则会执行这里的代码
  } else if (inTemplateString) {
    // 如果当前读取的字符存在于模板字符串内，则会执行这里的代码
  } else if (inRegex) {
    // 如果当前读取的字符存在于正则表达式内，则会执行这里的代码
  } else if (
    c === 0x7C && // pipe
    exp.charCodeAt(i + 1) !== 0x7C &&
    exp.charCodeAt(i - 1) !== 0x7C &&
    !curly && !square && !paren
  ) {
    // 如果当前读取的字符是过滤器的分界线，则会执行这里的代码
  } else {
    // 当不满足以上条件时，执行这里的代码
  }
}
```

可以看到每次循环的开始，都会将上一次读取的字符所对应的 `ASCII` 码赋值给 `prev` 变量，然后再将变量 `c` 的值设置为当前读取字符所对应的 `ASCII` 码。

接下来就是一大段`if...elseif...else` 语句

先看第一个判断语句 -- 判断当前读入的字符存在于由单引号包裹的字符串内

```js
if (inSingle) {
      if (c === 0x27 && prev !== 0x5C) inSingle = false
} 
```

在`ASCII`中`0x27`代表字符'`,而`0x5C`代表`\`,目的是为了判断该字符是一个不带转译的单引号，如果判断成立，将 `inSingle` 变量的值设置为 `false`，代表接下来的解析工作已经不处于由单引号所包裹的字符串环境中了。

第二个判断语句 -- 判断当前读入的字符存在于由双引号包裹的字符串内

```js
else if (inDouble) {
    if (c === 0x22 && prev !== 0x5C) inDouble = false
} 
```

同理`0x22`代表字符`"`,如果判断成立，将 `inDouble` 变量的值设置为 `false`，代表接下来的解析工作已经不处于由双引号所包裹的字符串环境中了

第三个判断语句  -- 判断当前读入的字符存在于由模板字符串所包裹的字符串内

```js
 else if (inTemplateString) {
      if (c === 0x60 && prev !== 0x5C) inTemplateString = false
} 
```

同理`0x60`代表字符 ``` `,如果判断成立，将 `inTemplateString` 变量的值设置为 `false`，代表接下来的解析工作已经不处于模板字符串环境中了

第四个判断语句 -- 判断当前读入的字符存在于由模板字符串所包裹的字符串内

```js
else if (inRegex) {
      if (c === 0x2f && prev !== 0x5C) inRegex = false
}
```

同理`0x2f`代表字符 `/ `,如果判断成立，将 `inRegex` 变量的值设置为 `false`，代表接下来的解析工作已经不处于正则表达式的环境中了

第五个判断

```js
else if (
      c === 0x7C && // pipe
      exp.charCodeAt(i + 1) !== 0x7C &&
      exp.charCodeAt(i - 1) !== 0x7C &&
      !curly && !square && !paren
    ) {
      if (expression === undefined) {
        // first filter, end of expression
        lastFilterIndex = i + 1
        expression = exp.slice(0, i).trim()
      } else {
        pushFilter()
      }
    }
```

`0x7C`daibiao 管道符`|`,实际上这个判断条件是用来检测当前遇到的管道符是否是过滤器的分界线。

如果一个管道符是过滤器的分界线则必须满足以上条件，即：

- 1、当前字符所对应的 `ASCII` 码必须是 `0x7C`，即当前字符必须是管道符。
- 2、该字符的后一个字符不能是管道符。
- 3、该字符的前一个字符不能是管道符。
- 4、该字符不能处于花括号、方括号、圆括号之内

这里先跳过，因为还没初始化

当所有判断分支无效后，会进入最后一个分支

```js
 else {
      switch (c) {
        case 0x22: inDouble = true; break         // "
        case 0x27: inSingle = true; break         // '
        case 0x60: inTemplateString = true; break // `
        case 0x28: paren++; break                 // (
        case 0x29: paren--; break                 // )
        case 0x5B: square++; break                // [
        case 0x5D: square--; break                // ]
        case 0x7B: curly++; break                 // {
        case 0x7D: curly--; break                 // }
      }
      if (c === 0x2f) { // /
        let j = i - 1
        let p
        // find first non-whitespace prev char
        for (; j >= 0; j--) {
          p = exp.charAt(j)
          if (p !== ' ') break
        }
        if (!p || !validDivisionCharRE.test(p)) {
          inRegex = true
        }
      }
    }
```

这段 `switch` 语句的作用总结如下：

- 如果当前字符为双引号(`"`)，则将 `inDouble` 变量的值设置为 `true`。
- 如果当前字符为单引号(`‘`)，则将 `inSingle` 变量的值设置为 `true`。
- 如果当前字符为模板字符串的定义字符(```)，则将 `inTemplateString` 变量的值设置为 `true`。
- 如果当前字符是左圆括号(`(`)，则将 `paren` 变量的值加一。
- 如果当前字符是右圆括号(`)`)，则将 `paren` 变量的值减一。
- 如果当前字符是左方括号(`[`)，则将 `square` 变量的值加一。
- 如果当前字符是右方括号(`]`)，则将 `square` 变量的值减一。
- 如果当前字符是左花括号(`{`)，则将 `curly` 变量的值加一。
- 如果当前字符是右花括号(`}`)，则将 `curly` 变量的值减一。

假如

```html
<div :key="'id'"></div>
```

那么要解析的是字符串 `'id'`

该字符串的第一个字符为单引号,由于一开始所有的初始值均为`false`,因此`switch` 语句将被执行，并且 `inSingle` 变量的值将被设置为 `true`。接着会解析第二个字符 `i`，由于此时 `inSingle` 变量的值已经为真，所以如下代码将被执行：

```js
if (inSingle) {
  if (c === 0x27 && prev !== 0x5C) inSingle = false
}
```

很显然字符 `i` 所对应的 `ASCII` 码不等于 `0x27`，直接下一个字符`d`，它的情况与字符 `i` 一样，也会被跳过。直到遇到最后一个字符 `'`，该字符同样是单引号，所以此时会将 `inSingle` 变量的值设置为 `false`，意味着由单引号包裹的字符串结束了。

嗯。。。相当于什么都没有干，那这写判断目的就是为了避免误把存在于字符串中的管道符当做过滤器的分界线，例如：

```html
<div :key="'id|featId'"></div>
```

可看到绑定属性 `key` 的属性值为 `'id|featId'`，由于管道符 `|` 存在于由单引号所包裹的字符串内，所以该管道符不会被作为过滤器的分界线

同样的道理，对于存在于由双引号包裹的字符串中或模板字符串中或正则表达式中的管道符，也不会被作为过滤器的分界线。

那么现在处理需要过滤器的分支

```js
else if (
  c === 0x7C && // pipe
  exp.charCodeAt(i + 1) !== 0x7C &&
  exp.charCodeAt(i - 1) !== 0x7C &&
  !curly && !square && !paren
) {
  if (expression === undefined) {
    // first filter, end of expression
    lastFilterIndex = i + 1
    expression = exp.slice(0, i).trim()
  } else {
    pushFilter()
  }
}
```

一开始的时候`expression`为`undefined`,所以当程序在解析字符串时第一次遇到作为过滤器分界线的管道符时，将会执行如下：

```js
if (expression === undefined) {
  // first filter, end of expression
  lastFilterIndex = i + 1  
   expression = exp.slice(0, i).trim()} else {
  // 省略...
}
```

变量`lastFilterIndex`记录管道符下一个字符的位置索引，接着对字符串 `exp` 进行截取，其截取的位置恰好是索引为 `i` 的字符，也就是管道符，当然了截取后生成的新字符串是不包含管道符的，同时对截取后生成的新字符串使用 `trim` 方法去除前后空格，最后将处理后的结果赋值给 `expression` 表达式。

例如：

```html
<div :key="id | featId"></div>
```

字符串为`id | featId`，管道符索引为3，因此`lastFilterIndex`的值为4,`expression`的值为`id`.然后进入下一次循环，并又找到一个管道线，

```js
 if (expression === undefined) {
      //省略...
} else {
        pushFilter()
}
```

由于此时`expression`已经不为`underfined`，那么则进入分支执行`pushFilter`函数

`pushFilter`源码

```js
 function pushFilter() {
    (filters || (filters = [])).push(exp.slice(lastFilterIndex, i).trim())
    lastFilterIndex = i + 1
  }
```

判断有没有`filters`变量，如果没有就将其初始化为一个空数组。接着将过滤器名字推进`filters`数组中去，更新`lastFilterIndex`

例子：

```html
<div :key="id | featId | featId2"></div>
```

经过`for`循环处理后结果如下

```js
expression = 'id'
filters = ['featId']
```

但这很明显不对，上述例子中过滤器有两个，现在`filters`只有一个数组，如果想让`filters`数组数据完全正确的话，那就要用以下代码了

```js
 if (expression === undefined) {
    expression = exp.slice(0, i).trim()
  } else if (lastFilterIndex !== 0) {
    pushFilter()
  }
```

当解析结束时变量 `i` 的值就应该是字符串的长度。此时 `for` 循环也将结束，此时代码依然会执行 `elseif` 分支，再次调用 `pushFilter` 函数，这时就会把`featId2`也推到`filters`数组中去

接下来继续往后看，代码也到最关键的一步了

```js
if (filters) {
    for (i = 0; i < filters.length; i++) {
      expression = wrapFilter(expression, filters[i])
    }
  }
```

如果存在过滤器，那么就使用 `for` 循环对 `filters` 数组进行了遍历，在循环内部调用了 `wrapFilter` 函数

`wrapFilter`函数

```js
function wrapFilter(exp: string, filter: string): string {
  const i = filter.indexOf('(')
  if (i < 0) {
    return `_f("${filter}")(${exp})`
  } else {
    const name = filter.slice(0, i)
    const args = filter.slice(i + 1)
    return `_f("${name}")(${exp}${args !== ')' ? ',' + args : args}`
  }
}
```

依旧用上面的例子

```js
expression = 'id'
filters = ['featId','featId2']
```

第一次遍历,将`featId`传入到`wrapFilter`函数中，由于`featId`并非为函数，因此返回的`expression`如下

```js
expression ='_f("featId")(id)'
```

第二次循环同理，返回的`expression`如下

```js
expression ='_f("featId2")(_f("featId")(id))'
```



到此`parseFilters`函数就分析完毕了

```html
<div :key="id"></div>
```

`parseFilters`函数返回`id`



```html
<div :key="id | featId | featId2"></div>
```

`parseFilters`函数返回`_f("featId2")(_f("featId")(id))`

实际上 `_f` 函数来自于 `src/core/instance/render-helpers/resolve-filter.js` 文件，这个函数的作用就是接收一个过滤器的名字作为参数，然后找到相应的过滤器函数。当找到相应的过滤器函数之后会将表达式的值作为参数传递给该过滤器函数，同时该过滤器会返回经过处理之后的值，这个处理之后的值将作为下一个过滤器函数的参数。

再回到 `getBindingAttr` 函数

```js
export function getBindingAttr (
  el: ASTElement,
  name: string,
  getStatic?: boolean
): ?string {
  const dynamicValue =
    getAndRemoveAttr(el, ':' + name) ||
    getAndRemoveAttr(el, 'v-bind:' + name)
  if (dynamicValue != null) {
    return parseFilters(dynamicValue)  
  } else if (getStatic !== false) {
    const staticValue = getAndRemoveAttr(el, name)
    if (staticValue != null) {
      return JSON.stringify(staticValue)
    }
  }
}
```

`getBindingAttr` 函数会先获取绑定属性的属性值，如果获取成功，则会使用 `parseFilters` 函数解析该属性值，并将 `parseFilters` 函数处理后的结果作为整个函数的返回值。

再回到 `processKey` 函数

```js
function processKey(el) {
  const exp = getBindingAttr(el, 'key')
  if (exp) {
    if (process.env.NODE_ENV !== 'production') {
      if (el.tag === 'template') {
        warn(
          `<template> cannot be keyed. Place the key on real elements instead.`,
          getRawBindingAttr(el, 'key')
        )
      }
      if (el.for) {
        const iterator = el.iterator2 || el.iterator1
        const parent = el.parent
        if (iterator && iterator === exp && parent && parent.tag === 'transition-group') {
          warn(
            `Do not use v-for index as key on <transition-group> children, ` +
            `this is the same as not using keys.`,
            getRawBindingAttr(el, 'key'),
            true /* tip */
          )
        }
      }
    }
    el.key = exp
  }
}
```

总结：

- 例子一：

```html
<div key="id"></div>
```

上例中 `div` 标签的属性 `key` 是非绑定属性，所以会将它的值作为普通字符串处理，这时 `el.key` 属性的值为：

```js
el.key = JSON.stringify('id')
```

- 例子二：

```html
<div :key="id"></div>
```

上例中 `div` 标签的属性 `key` 是绑定属性，所以会将它的值作为表达式处理，而非普通字符串，这时 `el.key` 属性的值为：

```js
el.key = 'id'
```

- 例子三：

```html
<div :key="id | featId"></div>
```

上例中 `div` 标签的属性 `key` 是绑定属性，并且应用了过滤器，所以会将它的值与过滤器整合在一起产生一个新的表达式，这时 `el.key` 属性的值为：

```js
el.key = '_f("featId")(id)'
```

以上就是 `el.key` 属性的所有可能值。



## 处理使用了ref属性的元素

`processRef`源码

```js
function processRef(el) {
  const ref = getBindingAttr(el, 'ref')
  if (ref) {
    el.ref = ref
    el.refInFor = checkInFor(el)
  }
}
```

非常简单，获取`ref`的参数，如果有那么做两个标识

`checkInFor`函数 判断当前元素是否在`for`循环内部

```js
function checkInFor (el: ASTElement): boolean {
  let parent = el
  while (parent) {
    if (parent.for !== undefined) {
      return true
    }
    parent = parent.parent
  }
  return false
}
```

由以上分析可知，如果一个标签使用了 `ref` 属性，则：

- 1、该标签的元素描述对象会被添加 `el.ref` 属性，该属性为解析后生成的表达式字符串，与 `el.key` 类似。
- 2、该标签的元素描述对象会被添加 `el.refInFor` 属性，它是一个布尔值，用来标识当前元素的 `ref` 属性是否在 `v-for` 指令之内使用。

## 处理(作用域)插槽

插槽相关的形式：

- 默认插槽：

```html
<slot></slot>
```

- 具名插槽

```html
<slot name="header"></slot>
```

- 插槽内容

```html
<h1 slot="header">title</h1>
```

- 作用域插槽 - slot-scope

```html
<h1 slot="header" slot-scope="slotProps">{{slotProps}}</h1>
```

- 作用域插槽 - scope

```html
<template slot="header" scope="slotProps">
  <h1>{{slotProps}}</h1>
</template>
```

`scope` 只能使用在 `template` 标签上，并且在 `2.5.0+` 版本中已经被 `slot-scope` 特性替代。

针对不同的使用方式有不同的处理方式

### 用`slot`标签的

```js
function processSlotOutlet(el) {
  if (el.tag === 'slot') {
    el.slotName = getBindingAttr(el, 'name')
      //警告
    if (process.env.NODE_ENV !== 'production' && el.key) {
      warn(
        `\`key\` does not work on <slot> because slots are abstract outlets ` +
        `and can possibly expand into multiple elements. ` +
        `Use the key on a wrapping element instead.`,
        getRawBindingAttr(el, 'key')
      )
    }
  }
}
```



### 用slot-scope特性的

```js
function processSlotContent(el) {
  let slotScope
  if (el.tag === 'template') {
    slotScope = getAndRemoveAttr(el, 'scope')
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && slotScope) {
      warn(
        `the "scope" attribute for scoped slots have been deprecated and ` +
        `replaced by "slot-scope" since 2.5. The new "slot-scope" attribute ` +
        `can also be used on plain elements in addition to <template> to ` +
        `denote scoped slots.`,
        el.rawAttrsMap['scope'],
        true
      )
    }
    el.slotScope = slotScope || getAndRemoveAttr(el, 'slot-scope')
  } else if ((slotScope = getAndRemoveAttr(el, 'slot-scope'))) {
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && el.attrsMap['v-for']) {
      warn(
        `Ambiguous combined usage of slot-scope and v-for on <${el.tag}> ` +
        `(v-for takes higher priority). Use a wrapper <template> for the ` +
        `scoped slot to make it clearer.`,
        el.rawAttrsMap['slot-scope'],
        true
      )
    }
    el.slotScope = slotScope
  }

  // slot="xxx"
  const slotTarget = getBindingAttr(el, 'slot')
  if (slotTarget) {
    el.slotTarget = slotTarget === '""' ? '"default"' : slotTarget
    el.slotTargetDynamic = !!(el.attrsMap[':slot'] || el.attrsMap['v-bind:slot'])
    // preserve slot as an attribute for native shadow DOM compat
    // only for non-scoped slots.
    if (el.tag !== 'template' && !el.slotScope) {
      addAttr(el, 'slot', slotTarget, getRawBindingAttr(el, 'slot'))
    }
  }

  // 2.6 v-slot syntax
  if (process.env.NEW_SLOT_SYNTAX) {
    if (el.tag === 'template') {
      // v-slot on <template>
      const slotBinding = getAndRemoveAttrByRegex(el, slotRE)
      if (slotBinding) {
        if (process.env.NODE_ENV !== 'production') {
          if (el.slotTarget || el.slotScope) {
            warn(
              `Unexpected mixed usage of different slot syntaxes.`,
              el
            )
          }
          if (el.parent && !maybeComponent(el.parent)) {
            warn(
              `<template v-slot> can only appear at the root level inside ` +
              `the receiving component`,
              el
            )
          }
        }
        const { name, dynamic } = getSlotName(slotBinding)
        el.slotTarget = name
        el.slotTargetDynamic = dynamic
        el.slotScope = slotBinding.value || emptySlotScopeToken // force it into a scoped slot for perf
      }
    } else {
      // v-slot on component, denotes default slot
      const slotBinding = getAndRemoveAttrByRegex(el, slotRE)
      if (slotBinding) {
        if (process.env.NODE_ENV !== 'production') {
          if (!maybeComponent(el)) {
            warn(
              `v-slot can only be used on components or <template>.`,
              slotBinding
            )
          }
          if (el.slotScope || el.slotTarget) {
            warn(
              `Unexpected mixed usage of different slot syntaxes.`,
              el
            )
          }
          if (el.scopedSlots) {
            warn(
              `To avoid scope ambiguity, the default slot should also use ` +
              `<template> syntax when there are other named slots.`,
              slotBinding
            )
          }
        }
        // add the component's children to its default slot
        const slots = el.scopedSlots || (el.scopedSlots = {})
        const { name, dynamic } = getSlotName(slotBinding)
        const slotContainer = slots[name] = createASTElement('template', [], el)
        slotContainer.slotTarget = name
        slotContainer.slotTargetDynamic = dynamic
        slotContainer.children = el.children.filter((c: any) => {
          if (!c.slotScope) {
            c.parent = slotContainer
            return true
          }
        })
        slotContainer.slotScope = slotBinding.value || emptySlotScopeToken
        // remove children as they are returned from scopedSlots now
        el.children = []
        // mark el non-plain so data gets generated
        el.plain = false
      }
    }
  }
}
```



总结：

- 1、对于 `<slot>` 标签，会为其元素描述对象添加 `el.slotName` 属性，属性值为该标签 `name` 属性的值，并且 `name` 属性可以是绑定的。
- 2、对于 `<template>` 标签，会优先获取并使用该标签 `scope` 属性的值，如果获取不到则会获取 `slot-scope` 属性的值，并将获取到的值赋值给元素描述对象的 `el.slotScope` 属性，注意 `scope` 属性和 `slot-scope` 属性不能是绑定的。
- 3、对于其他标签，会尝试获取 `slot-scope` 属性的值，并将获取到的值赋值给元素描述对象的 `el.slotScope` 属性。
- 4、对于非 `<slot>` 标签，会尝试获取该标签的 `slot` 属性，并将获取到的值赋值给元素描述对象的 `el.slotTarget` 属性。如果一个标签使用了 `slot` 属性但却没有给定相应的值，则该标签元素描述对象的 `el.slotTarget` 属性值为字符串 `'"default"'`。



## 处理使用了is或inline-template属性的元素

```js
function processComponent(el) {
  let binding
  if ((binding = getBindingAttr(el, 'is'))) {
    el.component = binding
  }
  if (getAndRemoveAttr(el, 'inline-template') != null) {
    el.inlineTemplate = true
  }
}
```

没什么好说的

