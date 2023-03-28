

# 词法分析 - 为生成AST做准备

真正对模板进行编译工作的实际是 `baseCompile` 函数，而接下来我任务就是搞清楚

## `baseCompile`函数

 `baseCompile` 函数的内容。

```js
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  optimize(ast, options)
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```

`baseCompile`函数接受两个参数：

- 字符串模板（`template`）
- 选项参数（`options`）--具体参数见[编辑器参数](./编辑器参数.md)



接下来详细分析下`baseCompile`函数

```js
function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  // 调用 parse 函数将字符串模板解析成抽象语法树(AST)
  const ast = parse(template.trim(), options)
  // 调用 optimize 函数优化 ast
  optimize(ast, options)
  // 调用 generate 函数将 ast 编译成渲染函数
  const code = generate(ast, options)
  //返回参数
  return {
      //抽象语法树 --ast树
    ast,
      //渲染函数
    render: code.render,
      //静态渲染函数
    staticRenderFns: code.staticRenderFns
  }
}
```

`Vue` 的编译器分为三个阶段，即：词法分析 -> 句法分析 -> 代码生成。

在词法分析阶段 `Vue` 会把字符串模板解析成一个个的令牌(`token`)，该令牌将用于句法分析阶段，在句法分析阶段会根据令牌生成一棵 `AST`，最后再根据该 `AST` 生成最终的渲染函数，这样就完成了代码的生成。

## `parse`函数

首先了解词法分析阶段，查看Vue如何对字符串模板进行拆解

重点在于`parse`函数，位于 `src/compiler/parser/index.js` 文件

首先从整体上来看下`parse`函数

```js
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  warn = options.warn || baseWarn

  platformIsPreTag = options.isPreTag || no
  platformMustUseProp = options.mustUseProp || no
  platformGetTagNamespace = options.getTagNamespace || no
  const isReservedTag = options.isReservedTag || no
  maybeComponent = (el: ASTElement) => !!(
    el.component ||
    el.attrsMap[':is'] ||
    el.attrsMap['v-bind:is'] ||
    !(el.attrsMap.is ? isReservedTag(el.attrsMap.is) : isReservedTag(el.tag))
  )
  transforms = pluckModuleFunction(options.modules, 'transformNode')
  preTransforms = pluckModuleFunction(options.modules, 'preTransformNode')
  postTransforms = pluckModuleFunction(options.modules, 'postTransformNode')

  delimiters = options.delimiters

  const stack = []
  const preserveWhitespace = options.preserveWhitespace !== false
  const whitespaceOption = options.whitespace
  let root
  let currentParent
  let inVPre = false
  let inPre = false
  let warned = false

  function warnOnce (msg, range) {
   ...
  }

  function closeElement (element) {
   ...
  }

  function trimEndingWhitespace (el) {
   ...
  }

  function checkRootConstraints (el) {
   ....
  }

  parseHTML(template, {
    warn,
    expectHTML: options.expectHTML,
    isUnaryTag: options.isUnaryTag,
    canBeLeftOpenTag: options.canBeLeftOpenTag,
    shouldDecodeNewlines: options.shouldDecodeNewlines,
    shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
    shouldKeepComment: options.comments,
    outputSourceRange: options.outputSourceRange,
    start (tag, attrs, unary, start, end) {
      ...
    },

    end (tag, start, end) {
     ...
    },

    chars (text: string, start: number, end: number) {
     ...
    },
    comment (text: string, start, end) {
      ...
    }
  })
  return root
}
```

前面是声明变量以及各种函数，知道后面到了`parseHTML`这里才进行了操作

那么要想知道`parse`函数是具体干什么的，只能看`parseHTML`函数到底干了什么

## `html-parser.js` 文件

根据文件头部的引用关系可知 `parseHTML` 函数来自 `src/compiler/parser/html-parser.js` 文件，实际上整个 `html-parser.js` 文件所做的事情都是在做词法分析。

首先在这文件开头有一段注释

```js
/**
 * Not type-checking this file because it's mostly vendor code.
 *不检查此文件的类型，因为它是最核心的代码。
 */

/*!
 * HTML Parser By John Resig (ejohn.org)
 * Modified by Juriy "kangax" Zaytsev
 * Original code by Erik Arvidsson (MPL-1.1 OR Apache-2.0 OR GPL-2.0-or-later)
 * http://erik.eae.net/simplehtmlparser/simplehtmlparser.js
 */
```

从上述的注释中可知，`Vue`的`html parser`是老子`John Resig`的，`Vue`在该基础上做了比较多的完善操作



## 正则分析

代码正文的一开始，是两句 `import` 语句，以及定义的一些正则常量

```js
import { makeMap, no } from 'shared/util'
import { isNonPhrasingTag } from 'web/compiler/util'
import { unicodeRegExp } from 'core/util/lang'

// Regular Expressions for parsing tags and attributes
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
const dynamicArgAttribute = /^\s*((?:v-[\w-]+:|@|:|#)\[[^=]+?\][^\s"'<>\/=]*)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z${unicodeRegExp.source}]*`
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
const startTagOpen = new RegExp(`^<${qnameCapture}`)
const startTagClose = /^\s*(\/?)>/
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`)
const doctype = /^<!DOCTYPE [^>]+>/i
// #7298: escape - to avoid being passed as HTML comment when inlined in page
const comment = /^<!\--/
const conditionalComment = /^<!\[/
```

看一下不同正则的作用是干什么的

### attribute

```js
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
```

这个正则的作用是用来匹配标签的属性(`attributes`)的

该正则有五个捕获组，第一个捕获组用来匹配属性名，第二个捕获组用来匹配等于号，第三、第四、第五个捕获组都是用来匹配属性值的，同时 `?` 表明第三、四、五个分组是可选的。这样可以捕获`html`标签中的4种不同写法

- 1、使用双引号把值引起来：`class="some-class"`
- 2、使用单引号把值引起来：`class='some-class'`
- 3、不使用引号：`class=some-class`
- 4、单独的属性名：`disabled`

### ncname

```js
const ncname = `[a-zA-Z_][\\-\\.0-9_a-zA-Z${unicodeRegExp.source}]*`
```

定义了 `ncname` 的合法组成，这个正则所匹配的内容很简单：*字母或下划线开头，后面可以跟任意数量的字符、中横线和 `.`*。

`ncname` 的全称是 `An XML name that does not contain a colon (:)` 即：不包含冒号(`:`)的 XML 名称。也就是说 `ncname` 就是不包含前缀的XML标签名称。

### qnameCapture

```js
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
```

定义了 `qname` 的合法组成，由可选项的 `前缀`、`冒号` 以及 `名称` 组成，观察 `qnameCapture` 可知它有一个捕获分组，捕获的内容就是整个 `qname` 名称，即整个标签的名称

 `qname` 就是：`<前缀:标签名称>`，也就是合法的XML标签。

### startTagOpen

```js
const startTagOpen = new RegExp(`^<${qnameCapture}`)
```

用来匹配开始标签的一部分，这部分包括：`<` 以及后面的 `标签名称`，这个表达式的创建用到了上面定义的 `qnameCapture` 字符串，所以 `qnameCapture` 这个字符串中所设置的捕获分组，在这里同样适用，也就是说 `startTagOpen` 这个正则表达式也会有一个捕获的分组，用来捕获匹配的标签名称。

### startTagClose

```js
const startTagClose = /^\s*(\/?)>/
```

`startTagOpen` 用来匹配开始标签的 `<` 以及标签的名字

`startTagClose`用来匹配开始标签的闭合部分，即：`>` 或者 `/>`

### endTag

```js
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`)
```

`endTag` 这个正则用来匹配结束标签，由于该正则同样使用了字符串 `qnameCapture`，所以这个正则也拥有了一个捕获组，用来捕获标签名称。

### doctype

```js
const doctype = /^<!DOCTYPE [^>]+>/i
```

这个正则用来匹配文档的 `DOCTYPE` 标签，没有捕获组。

### comment

```js
const comment = /^<!\--/
```

用来匹配注释节点，没有捕获组

### conditionalComment

```js
const conditionalComment = /^<!\[/
```

这个正则用来匹配条件注释节点，没有捕获组。

最后很重要的一点是，大家注意，这些正则表达式有一个共同的特点，即：*他们都是从一个字符串的开头位置开始匹配的，因为有 `^` 的存在*。



## 常量分析

```js
export const isPlainTextElement = makeMap('script,style,textarea', true)
const reCache = {}

const decodingMap = {
  '&lt;': '<',
  '&gt;': '>',
  '&quot;': '"',
  '&amp;': '&',
  '&#10;': '\n',
  '&#9;': '\t',
  '&#39;': "'"
}
const encodedAttr = /&(?:lt|gt|quot|amp|#39);/g
const encodedAttrWithNewLines = /&(?:lt|gt|quot|amp|#39|#10|#9);/g
```

### `isPlainTextElement`

函数，通过 `makeMap` 函数生成的，用来检测给定的标签名字是不是纯文本标签（包括：`script`、`style`、`textarea`）

### `reCache`

 一个空的 `JSON` 对象字面量

### `decodingMap`

是一个 `JSON` 对象字面量，其中 `key` 是一些特殊的 `html` 实体，值则是这些实体对应的字符

### `encodedAttr`

正则常量

### `encodedAttrWithNewLines`

正则常量，比`encodedAttr`多了两个实体字符`$#10;`、`$#9;`

接下来是一段代码

```js
const isIgnoreNewlineTag = makeMap('pre,textarea', true)
const shouldIgnoreFirstNewline = (tag, html) => tag && isIgnoreNewlineTag(tag) && html[0] === '\n'
```

`isIgnoreNewlineTag`

函数，用来检测给定的标签是否是 `<pre>` 标签或者 `<textarea>` 标签。

`shouldIgnoreFirstNewline`

函数，作用是用来检测是否应该忽略元素内容的第一个换行符。

由于历史原因，一些元素会受到额外的限制，比如 `<pre>` 标签和 `<textarea>` 会忽略其内容的第一个换行符，所以下面这段代码是等价的：

```html
<pre>内容</pre>
```

等价于：

```html
<pre>
内容</pre>
```

`shouldIgnoreFirstNewline` 函数就很容易理解，这个函数就是用来判断是否应该忽略标签内容的第一个换行符的，如果满足：*标签是 `pre` 或者 `textarea`* 且 *标签内容的第一个字符是换行符*，则返回 `true`，否则为 `false`。



接下来是一个`decodeAttr`函数

```js
function decodeAttr (value, shouldDecodeNewlines) {
  const re = shouldDecodeNewlines ? encodedAttrWithNewLines : encodedAttr
  return value.replace(re, match => decodingMap[match])
}
```

用来解码 `html` 实体的。它的原理是利用则 `encodedAttrWithNewLines` 和 `encodedAttr` 以及 `html` 实体与字符一一对应的 `decodingMap` 对象来实现将 `html` 实体转为对应的字符。

## parseHTML

首先先看一下该函数的简化版

```js
export function parseHTML (html, options) {
    // 定义一些常量和变量
  const stack = []
  const expectHTML = options.expectHTML
  const isUnaryTag = options.isUnaryTag || no
  const canBeLeftOpenTag = options.canBeLeftOpenTag || no
  let index = 0
  let last, lastTag
  
  // 开启一个 while 循环，循环结束的条件是 html 为空，即 html 被 parse 完毕
  while (html) {
    last = html
  
    if (!lastTag || !isPlainTextElement(lastTag)) {
         // 确保即将 parse 的内容不是在纯文本标签里 (script,style,textarea)
      ....
    } else {
        // 即将 parse 的内容是在纯文本标签里 (script,style,textarea)
     ....
    }

    if (html === last) {
      options.chars && options.chars(html)
      if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
        options.warn(`Mal-formatted tag at end of template: "${html}"`, { start: index + html.length })
      }
      break
    }
  }

  // Clean up any remaining tags
  parseEndTag()

  function advance (n) {
    ....
  }

  function parseStartTag () {
    ....
  }

  function handleStartTag (match) {
    ....
  }

  function parseEndTag (tagName, start, end) {
   	 ....
   }
}
```

该函数大体能分三部分：

1. 函数开头定义的一些常量和变量
2. 一个 `while` 循环
3. `while` 循环之后定义的一些函数

### 第一部分变量及常量声明

```js
。
const stack = []
const expectHTML = options.expectHTML
const isUnaryTag = options.isUnaryTag || no
const canBeLeftOpenTag = options.canBeLeftOpenTag || no
//初始化为 0 --标识着当前字符流的读入位置
let index = 0
//last  --存储剩余还未 parse 的 html 字符串
//lastTag 则始终存储着位于 stack 栈顶的元素 
let last, lastTa
```

#### stack

空数组 

在 `while` 循环中处理` html` 字符流的时候每当遇到一个 **非一元标签**，都会将该开始标签 push 到该数组

举个例子

```html
<article><section><div></section></article>
```

当对该字符串进行`parse`时，首先会遇到`article`开始标签，并将改标签`push`到数组`stack`中，接着会遇到`section`开始标签，同理将其推进数组`stack`中，接下来遇到了`div`开始标签，同样会将其压进栈顶。此时`stack`数组如下

```js
[article,section,div]
```

再然后便会遇到 `section` 结束标签，我们知道：**最先遇到的结束标签，其对应的开始标签应该最后被压入 stack 栈**，也就是说此时 `stack` 栈顶的元素应该是 `section`，但是我们发现事实上 `stack` 栈顶并不是 `section` 而是 `div`，这说明 `div` 元素缺少闭合标签。这就是检测 `html` 字符串中是否缺少闭合标签的原理



#### expectHTML

初始化为 `options.expectHTML`，也就是 `parser` 选项中的 `expectHTML`。

#### isUnaryTag

 `options.isUnaryTag` 存在则它的值被初始化为 `options.isUnaryTag` ，否则初始化为 `no`，即一个始终返回 `false` 的函数。其中 `options.isUnaryTag` 也是一个 `parser` 选项，用来检测一个标签是否是一元标签。

#### canBeLeftOpenTag

初始化为 `options.canBeLeftOpenTag`(如果存在的话，否则初始化为 `no`)。其中 `options.canBeLeftOpenTag` 也是 `parser` 选项，用来检测一个标签是否是可以省略闭合标签的非一元标签。



### 第二部分`while`循环

先看下一下`while`循环的简化结构

```js
while (html) {
    last = html
  
    if (!lastTag || !isPlainTextElement(lastTag)) {
     ...
    } else {
      ...
    }

    if (html === last) {
      options.chars && options.chars(html)
      if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
        options.warn(`Mal-formatted tag at end of template: "${html}"`, { start: index + html.length })
      }
      break
    }
  }
```

从上述代码可以，，循环的终止条件是 `html` 字符串为空，即 `html` 字符串全部 `parse` 完毕。

在每次循环开始时，会将`html`赋值给变量`last`

```js
last = html
```

但是在循环的最后，有一个`last`和`html`比较的判断

```js
if (html === last) {
      options.chars && options.chars(html)
      if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
        options.warn(`Mal-formatted tag at end of template: "${html}"`, { start: index + html.length })
      }
      break
    }
```

如果两者相等，则说明字符串 `html` 在经历循环体的代码之后没有任何改变，此时会把 `html` 字符串作为纯文本对待。接下来我们就着重讲解循环体中间的代码是如何 `parse` html 字符串的

接下来看一下`while`循环中的中间代码，也是`while`循环中的重点

简化结构

```js
//判断是否为纯文本
//lastTag 保存了 stack 栈顶的元素，也就是 最近一次遇到的非一元标签的开始标签
//isPlainTextElement(lastTag) 是判别该lastTag是否为(script,style,textarea)
if (!lastTag || !isPlainTextElement(lastTag)) {
     ...
} else {
    //lastTag &&isPlainTextElement(lastTag) 
    //如果判断成立的话，即将parse的内容在纯文本标签内
      ...
}
```

#### 

接下来先对非纯文本代码进行分析，按惯例依旧先查看其简化后的格式

```js
if (!lastTag || !isPlainTextElement(lastTag)) {
    // html 字符串中左尖括号(<)第一次出现的位置
  let textEnd = html.indexOf('<')
  if (textEnd === 0) {
    // textEnd === 0 的情况
  }

  let text, rest, next
  if (textEnd >= 0) {
    // textEnd >= 0 的情况
  }
//不存在的状况
  if (textEnd < 0) {
    // textEnd < 0 的情况
  }

  if (options.chars && text) {
    options.chars(text)
  }
} else {
  // 省略 ...
}
```

#### textEnd 为 0 的情况

当 `textEnd === 0` 时，说明 `html` 字符串的第一个字符就是左尖括号，例如`<div>asdf</div>`

依旧是简化后的结构

```js
if (textEnd === 0) {
  // 判断是否为注释的文本
  if (comment.test(html)) {
    // 有可能是注释节点
  }
	//依旧是判断是否为注释文本
  if (conditionalComment.test(html)) {
    // 有可能是条件注释节点
  }

  // Doctype:
  const doctypeMatch = html.match(doctype)
  if (doctypeMatch) {
    // doctype 节点
  }

  // End tag:
  const endTagMatch = html.match(endTag)
  if (endTagMatch) {
    // 结束标签
  }

  // Start tag:
  const startTagMatch = parseStartTag()
  if (startTagMatch) {
    // 开始标签
  }
}
```

通过上述代码可以知道，当以左尖括号开头的字符串，其实有六种情况：

- 1、可能是注释节点：`<!-- -->`
- 2、可能是条件注释节点：`<![ ]>`
- 3、可能是 `doctype`：`<!DOCTYPE >`
- 4、可能是结束标签：`</xxx>`
- 5、可能是开始标签：`<xxx>`
- 6、可能只是一个单纯的字符串：`<abcdefg`

针对以上六种情况我们逐个来看

##### 注释节点

```js
if (comment.test(html)) {
    //获取注释结束的位置
     const commentEnd = html.indexOf('-->')
     //如果commentEnd>=0代表这的确是一个注释点
     if (commentEnd >= 0) {
     	if (options.shouldKeepComment) {
            //调用字符串的 substring 方法截取注释内容，其中起始位置是 4，结束位置是 commentEnd 的值
            //比如 <!-- 哇哈哈哈哈 -->
            //获取到的内容是 哇哈哈哈，不包含<!--和-->
         	options.comment(html.substring(4, commentEnd), index, index + commentEnd + 3)
      	}
         //当注释节点 parse 完毕后，就将已经parse完成的字符串删除掉
         //advance函数代码如下
      	advance(commentEnd + 3)
         //去掉注释点后 ，跳过此次循环，进入下一次循环
      	continue
    }
}
```

```js
function advance (n) {
    //改变当前读入的位置
    index += n
    //删除已经paser的字符串
    html = html.substring(n)
  }
```

##### 条件注释节点

```js
//判断是否为条件注释节点
if (conditionalComment.test(html)) {
    //获取知识点结尾
  const conditionalEnd = html.indexOf(']>')

  if (conditionalEnd >= 0) {
      //直接删掉，进入下一次循环
    advance(conditionalEnd + 2)
    continue
  }
}
```

##### `Doctype`节点

如果既没有命中注释节点，也没有命中条件注释节点，那么将判断是否命中 `Doctype` 节点

```js
 // Doctype:
const doctypeMatch = html.match(doctype)
if (doctypeMatch) {
  advance(doctypeMatch[0].length)
  continue
}
```

判断的方法是使用字符串的 `match` 方法去匹配正则 `doctype`，如果匹配成功 `doctypeMatch` 的值是一个数组，数组的第一项保存着整个匹配项的字符串，即整个 `Doctype` 标签的字符串，不成功则是一个null

在原则上 `Vue` 在编译的时候根本不会遇到 `Doctype` 标签。

##### 开始标签

```js
// Start tag:
//首先调用parseStartTag函数判断是否是开始标签
const startTagMatch = parseStartTag()
if (startTagMatch) {
   handleStartTag(startTagMatch)
   if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
       advance(1)
	}
    continue
}
```

首先调用`parseStartTag`函数判断是否是开始标签

###### `parseStartTag`函数处理解析

`parseStartTag`函数如下

```js
 function parseStartTag () {
     //首先会调用 html 字符串的 match 函数匹配 startTagOpen 正则
    const start = html.match(startTagOpen)
    
    //如果匹配成功这进入判断，没有之结束函数
    if (start) {
      const match = {
        tagName: start[1],
        attrs: [],
        start: index
      }
      advance(start[0].length)
      let end, attr
      while (!(end = html.match(startTagClose)) && (attr = html.match(dynamicArgAttribute) || html.match(attribute))) {
        attr.start = index
        advance(attr[0].length)
        attr.end = index
        match.attrs.push(attr)
      }
      if (end) {
        match.unarySlash = end[1]
        advance(end[0].length)
        match.end = index
        return match
      }
    }
  }
```

首先会调用 html 字符串的 match 函数匹配 startTagOpen 正则

```js
 const start = html.match(startTagOpen)
 //举个例子 -- 假如html为<div></div>
 //那么start = ['<div', 'div']
```

匹配成功，所以 `if` 语句块将被执行

```js
 const match = {
   tagName: start[1],
   attrs: [],
   start: index
}
 //将开始标签从html上剪去
advance(start[0].length)
```

声明了一个对象，并拥有三个属性

- `tagName`：它的值为 `start[1]` 即标签的名称。
- `attrs`：它的初始值是一个空数组，我们知道，开始标签是可能拥有属性的，而这个数组就是用来存储将来被匹配到的属性。
- `start`：它的值被设置为 `index`，也就是当前字符流读入位置在整个 `html` 字符串中的相对位置。

继续往下则是获取`attribute`属性代码

```js
while (
    !(end = html.match(startTagClose))
       && 
(attr = html.match(dynamicArgAttribute) || html.match(attribute))
      ) {
        attr.start = index
        advance(attr[0].length)
        attr.end = index
        match.attrs.push(attr)
}
```

该循环条件有两个：

- **没有匹配到开始标签的结束部分**

  ```js
  end = html.match(startTagClose)
  ```

  通过匹配对应结束标签，来判断有没有对应的结束部分

- **匹配到了属性**

  ```js
  attr = html.match(dynamicArgAttribute) || html.match(attribute)
  ```

  使用 `html` 字符串的 `match` 方法去匹配 `attribute`正则

简单来说，该循环的成立条件是：**没有匹配到开始标签的结束部分，并且匹配到了开始标签中的属性**

循环结束的条件是：直到遇到开始标签对应的结束标签为止

举个例子

```html
<div v-for="v in map"></div>
```

在循环体内，由于此时匹配到了开始标签的属性， `attr` 变量将保存着匹配结果

```js
attr = [
  ' v-for="v in map"',
  'v-for',
  '=',
  'v in map',
  undefined,
  undefined
]
```

在循环体内做了两件事，首先调用 `advance` 函数，将匹配到的给删除掉，然后会将此次循环匹配到的结果 `push` 到前面定义的 `match` 对象的 `attrs` 数组中

```js
//比如index =7
//在attr 上做好该字符串起始的下标
attr.start = index
//剪去匹配中的内容 -- 改变了index
advance(attr[0].length)
//在attr 上做好该字符串结束的下标
attr.end = index
match.attrs.push(attr)

attr = [
  ' v-for="v in map"',
  'v-for',
  '=',
  'v in map',
  undefined,
  undefined,
  start:7,
  end:24
]
```

这样一次循环就结束了，将会开始下一次循环，直到 **匹配到开始标签的结束部分** 或者 **匹配不到属性** 的时候循环才会停止

`parseStartTag`函数最后一个代码

```js
//首先判断了变量 end 是否为真
if (end) {
    //记录end
    match.unarySlash = end[1]
    //index已经被更新
    advance(end[0].length)
    match.end = index
    //只有解析到一个开始标签，才有返回值，不然parseStartTag 函数的返回值是underfined
    return match
  }
```

如果变量 `end` 的确存在，那么才会执行 `if` 语句块内的代码。

那么`end`是什么呢？比如

```html
//如果html是<br />
<br />
//那么end的结果
//end = ['/>', '/']

//如果html是<div>
<div>
//那么end的结果
//end = ['>', undefined]
```

所以，如果 `end[1]` 不为 `undefined`，那么说明该标签是一个一元标签。

举个例子

```html
<div v-if="isSucceed" v-for="v in map"></div>
```

一开始index假设为0，那么`parseStartTag` 函数的返回值如下

```js
match = {
  tagName: 'div',
  attrs: [
    [
      ' v-if="isSucceed"',
      'v-if',
      '=',
      'isSucceed',
      undefined,
      undefined,
        start:index // 4
        end:index // 22
    ],
    [
      ' v-for="v in map"',
      'v-for',
      '=',
      'v in map',
      undefined,
      undefined,
         start:index // 22
        end:index // 39
    ]
  ],
  start: index, // 4
  unarySlash: undefined,
  end: index //40
}
```

处理完`pareseStartTag`函数后，回到最开始的开始标签处理的地方

```js
const startTagMatch = parseStartTag()
if (startTagMatch) {
   handleStartTag(startTagMatch)
   if (shouldIgnoreFirstNewline(startTagMatch.tagName, html)) {
   advance(1)
   }
 continue
}
```

`startTagMatch`的值就是上述分析的`match`，那么就到了下一步`handleStartTag`函数了

###### `handleStartTag`函数处理解析结果

代码如下

```js
function handleStartTag(match) {
    const tagName = match.tagName
    const unarySlash = match.unarySlash

    if (expectHTML) {
      if (lastTag === 'p' && isNonPhrasingTag(tagName)) {
        parseEndTag(lastTag)
      }
      if (canBeLeftOpenTag(tagName) && lastTag === tagName) {
        parseEndTag(tagName)
      }
    }

    const unary = isUnaryTag(tagName) || !!unarySlash

    const l = match.attrs.length
    const attrs = new Array(l)
    for (let i = 0; i < l; i++) {
      const args = match.attrs[i]
      const value = args[3] || args[4] || args[5] || ''
      const shouldDecodeNewlines = tagName === 'a' && args[1] === 'href'
        ? options.shouldDecodeNewlinesForHref
        : options.shouldDecodeNewlines
      attrs[i] = {
        name: args[1],
        value: decodeAttr(value, shouldDecodeNewlines)
      }
      if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
        attrs[i].start = args.start + args[0].match(/^\s*/).length
        attrs[i].end = args.end
      }
    }

    if (!unary) {
      stack.push({ tag: tagName, lowerCasedTag: tagName.toLowerCase(), attrs: attrs, start: match.start, end: match.end })
      lastTag = tagName
    }

    if (options.start) {
      options.start(tagName, attrs, unary, match.start, match.end)
    }
  }
```

`handleStartTag` 函数用来处理开始标签的解析结果，所以它接收 `parseStartTag` 函数的返回值作为参数。

`handleStartTag` 函数的一开始定义两个常量：`tagName` 以及 `unarySlash`：

```js
const tagName = match.tagName //标签名
const unarySlash = match.unarySlash //标签结束 符号 通常是'/'或者underfined
```

再往下，定义了三个常量：

```js
//判断标签是一元标签 真为一元标签 假为二元标签
const unary = isUnaryTag(tagName) || !!unarySlash
//这里要注意的是自定义组件 
//比如<my-component />
//这个标签，它的 tagName 是 my-component ，但不是常规标签，所以isUnaryTag(tagName)返回为false
//但他又是一元标签，这是就要用到第二个判别条件： 
//!!unarySlash 
//开始标签的结束部分是否使用 '/'，如果有反斜线 '/'，说明这是一个一元标签。

const l = match.attrs.length //attribute内容的长度
const attrs = new Array(l) //新建一个数组
```

`isUnaryTag`函数 --够判断标准 `HTML` 中规定的那些一元标签

```js
export const isUnaryTag = makeMap(
  'area,base,br,col,embed,frame,hr,img,input,isindex,keygen,' +
  'link,meta,param,source,track,wbr'
)
```

接下来是通过`for`循环**格式化 `match.attrs` 数组，并将格式化后的数据存储到常量 `attrs` 中**。

```js
//以上述例子为例
/** attrs: [
    [
      ' v-if="isSucceed"',
      'v-if',
      '=',
      'isSucceed',
      undefined,
      undefined,
        start:index // 4
        end:index // 22
    ],
    [
      ' v-for="v in map"',
      'v-for',
      '=',
      'v in map',
      undefined,
      undefined,
         start:index // 22
        end:index // 39
    ]
  ],
*/
for (let i = 0; i < l; i++) {
      const args = match.attrs[i]
      //获取值
      const value = args[3] || args[4] || args[5] || ''
      //判断是否为超链接
      const shouldDecodeNewlines = tagName === 'a' && args[1] === 'href'
        ? options.shouldDecodeNewlinesForHref
        : options.shouldDecodeNewlines
      attrs[i] = {
        name: args[1],
          //将 value 传递给了 decodeAttr 函数，并使用该函数的返回值作为最终的属性值
        value: decodeAttr(value, shouldDecodeNewlines)
      }
      if (process.env.NODE_ENV !== 'production' && options.outputSourceRange) {
        attrs[i].start = args.start + args[0].match(/^\s*/).length
        attrs[i].end = args.end
      }
 }
```

这样`for`循环就没了

最后是判别是否为一元标签的操作了

```js
  if (!unary) {
      //如果开始标签是非一元标签，则将该开始标签的信息入栈，即 push 到 stack 数组中，并将 lastTag 的值设置为该标签名。
      stack.push({ tag: tagName, lowerCasedTag: tagName.toLowerCase(), attrs: attrs, start: match.start, end: match.end }) //判断是否缺少闭合标签
      lastTag = tagName //lastTag保存着 stack 栈顶的元素
    }
```

##### 结束标签

```js
const endTagMatch = html.match(endTag)
if (endTagMatch) {
    //缓存当前index
   const curIndex = index
   //删除结束标签，并修改index
   advance(endTagMatch[0].length)
    //endTagMatch[1] -- 标签名
    //curIndex -- 结束标签起始位置
    //index --结束标签结束位置
   parseEndTag(endTagMatch[1], curIndex, index)
   continue
}
```

首先调用 `html` 字符串的 `match` 函数匹配正则 `endTag`，将结果保存在常量 `endTagMatch` 中。

比如

```html
<div></div>
```

得到的`endTagMatch`是

```js
endTagMatch = [
  '</div>',
  'div'
]
```

关键点在于 `parseEndTag` 函数

目前已知`parseEndTag` 函数主要有三个作用：

- 检测是否缺少闭合标签
- 处理 `stack` 栈中剩余的标签
- 解析 `</br>` 与 `</p>` 标签，与浏览器的行为相同

```js
 function parseEndTag(tagName, start, end) {
     // pos 用于判断 html 字符串是否缺少结束标签
     //lowerCasedTagName 用来存储 tagName 的小写版
    let pos, lowerCasedTagName
    if (start == null) start = index
    if (end == null) end = index

    // Find the closest opened tag of the same type
    if (tagName) {
      lowerCasedTagName = tagName.toLowerCase()
      for (pos = stack.length - 1; pos >= 0; pos--) {
        if (stack[pos].lowerCasedTag === lowerCasedTagName) {
          break
        }
      }
    } else {
      // If no tag name is provided, clean shop
      pos = 0
    }

    if (pos >= 0) {
      // Close all the open elements, up the stack
      for (let i = stack.length - 1; i >= pos; i--) {
        if (process.env.NODE_ENV !== 'production' &&
          (i > pos || !tagName) &&
          options.warn
        ) {
          options.warn(
            `tag <${stack[i].tag}> has no matching end tag.`,
            { start: stack[i].start, end: stack[i].end }
          )
        }
        if (options.end) {
          options.end(stack[i].tag, start, end)
        }
      }

      // Remove the open elements from the stack
        
      stack.length = pos
      lastTag = pos && stack[pos - 1].tag
    } else if (lowerCasedTagName === 'br') {
      if (options.start) {
        options.start(tagName, [], true, start, end)
      }
    } else if (lowerCasedTagName === 'p') {
      if (options.start) {
        options.start(tagName, [], false, start, end)
      }
      if (options.end) {
        options.end(tagName, start, end)
      }
    }
  }
```

分析下这一段代码的作用

```js
//判断tagName是否存在
if (tagName) {
    //保存其小写版
      lowerCasedTagName = tagName.toLowerCase()
    //判断该标签在stack上对对应的是哪个开始标签，并获取其下标，并赋值给pos
      for (pos = stack.length - 1; pos >= 0; pos--) {
        if (stack[pos].lowerCasedTag === lowerCasedTagName) {
          break
        }
      }
    } else {
      // If no tag name is provided, clean shop
      pos = 0
    }
```

实际上 `pos` 变量会被用来判断是否有元素缺少闭合标签。

```js
 if (pos >= 0) {
      // Close all the open elements, up the stack
      for (let i = stack.length - 1; i >= pos; i--) {
        if (process.env.NODE_ENV !== 'production' &&
          (i > pos || !tagName) &&
          options.warn
        ) {
          options.warn(
            `tag <${stack[i].tag}> has no matching end tag.`,
            { start: stack[i].start, end: stack[i].end }
          )
        }
          //闭合标签
        if (options.end) {
          options.end(stack[i].tag, start, end)
        }
      }

      // Remove the open elements from the stack
   //最后更新 stack 栈以及 lastTag
      stack.length = pos
      lastTag = pos && stack[pos - 1].tag
    } else if (lowerCasedTagName === 'br') {
      if (options.start) {
        options.start(tagName, [], true, start, end)
      }
    } else if (lowerCasedTagName === 'p') {
      if (options.start) {
        options.start(tagName, [], false, start, end)
      }
      if (options.end) {
        options.end(tagName, start, end)
      }
    }
```

在 `if` 语句块内开启一个 `for` 循环，同样是从后向前遍历 `stack` 数组，如果发现 `stack` 数组中存在索引大于 `pos` 的元素，那么该元素一定是缺少闭合标签的，这个时候如果是在非生产环境那么 `Vue` 便会打印一句警告，告诉你缺少闭合标签。随后会调用 `options.end(stack[i].tag, start, end)` 立即将其闭合，这是为了保证解析结果的正确性。

比如

```js
stack = [article,section,div]

tagName = section
//那么pro = 1
//for (let i = stack.length - 1; i >= pos; i--) {}
//当循环到div时，i一定比pro大
```

**当`tagName`存在但是 `tagName` 没有在 `stack` 栈中找到对应的开始标签时，pros会等于-1**  

```js
if (pos >= 0) {
    ..
    } else if (lowerCasedTagName === 'br') {
      if (options.start) {
        options.start(tagName, [], true, start, end)
      }
    } else if (lowerCasedTagName === 'p') {
      if (options.start) {
        options.start(tagName, [], false, start, end)
      }
      if (options.end) {
        options.end(tagName, start, end)
      }
    }
```

当你写了 `br` 标签的结束标签：`</br>` 或 `p` 标签的结束标签 `</p>` 时，解析器能够正常解析他们，其中对于 `</br>` 会将其解析为正常的 `<br>` 标签，而 `</p>` 标签也会正常解析为 `<p></p>`。

如何处理 `stack` 栈中剩余未处理的标签的？其实就是调用 `parseEndTag()` 函数时不传递任何参数，也就是说此时 `tagName` 参数也不存在。

```js
if (tagName) {
  for (pos = stack.length - 1; pos >= 0; pos--) {
    if (stack[pos].lowerCasedTag === lowerCasedTagName) {
      break
    }
  }
} else {
  // If no tag name is provided, clean shop
  pos = 0
}
```

由于 `tagName` 不存在，所以此时 `pos` 为 `0`，我们知道在这段代码之后会遍历 `stack` 栈，并将 `stack` 栈中元素的索引与 `pos` 作对比。由于 `pos` 为 `0`，所以 `i >= pos` 始终成立，这个时候 `stack` 栈中如果有剩余未处理的标签，则会逐个警告缺少闭合标签，并调用 `options.end` 将其闭合



#### textEnd 大于等于 0 的情况

```js
  let text, rest, next
      if (textEnd >= 0) {
        rest = html.slice(textEnd)
       
        while (
          !endTag.test(rest) &&
          !startTagOpen.test(rest) &&
          !comment.test(rest) &&
          !conditionalComment.test(rest)
        ) {
          // < in plain text, be forgiving and treat it as text
          next = rest.indexOf('<', 1)
          if (next < 0) break
          textEnd += next
          rest = html.slice(textEnd)
        }
        text = html.substring(0, textEnd)
}
```

textEnd 大于等于 0 总得来说有两个情况：

- 假如字符串是`< 2`时，该字符串虽然是以`<`开头，但其并不是标签
- 假如字符串是`0<1<2`，该字符串第一个字符不是 `<` 的字符串`textEnd`大于1的

因此需要对字符串进行修饰

```js
  rest = html.slice(textEnd)
```

比如字符串是`0<1<2`,经过操作后字符串变成`<1<2`，接着开启循环

```js
//判断该字符串是否为标签
while (
  !endTag.test(rest) &&  !startTagOpen.test(rest) &&  !comment.test(rest) &&  !conditionalComment.test(rest)) {
  //确认不是标签后，检查字符串后续是否还有<，并获取其下标
    //比如 <1<2 next = 2
  next = rest.indexOf('<', 1)
  if (next < 0) break
    //记录下标
  textEnd += next
    //切割字符串，进入下一次循环
  rest = html.slice(textEnd)
}
```

循环终止后，保存其操作结果

```js
//比如 字符串是 0<1<2 ，那么最后 textEnd = 2
//那么text = 0<1 
text = html.substring(0, textEnd)
```

接下来是将`parse`成功的代码删除，并修改index

```js
 if (text) {
    advance(text.length)
}

if (options.chars && text) {
 	options.chars(text, index - text.length, index)
}
```

此时 `text` 的值为字符串 `0<1`，所以这部分字符串将被作为普通字符串处理，如果 `options.chars` 存在，则会调用该钩子函数并将字符串传递过去。

但还有一个问题，之前例子是`0<1<2`，一部分是`0<1`，这部分被作为普通文本对待，那么剩余的字符串 `<2` 呢？这部分将会在下一个整体`while`循环中处理，因此`html = <2`，第一个字符串为`<`，因此字符串索引为0，会匹配`===0`和`>=0`的状况，但是由于字符串 `<2` 既不能匹配标签，也不会被 `textEnd` 大于等于 `0` 的 `if` 语句块处理，所以代码最终会来到这里：

```js
if (html === last) {
  options.chars && options.chars(html)
  if (process.env.NODE_ENV !== 'production' && !stack.length && options.warn) {
    options.warn(`Mal-formatted tag at end of template: "${html}"`)
  }
  break
}
```

由于`html`没有经过处理，所以条件`html === last`成立，因此会执行调用了 `options.chars` 并将整个 `html` 字符串作为普通字符串处理，换句话说最终字符串 `<2` 也会作为普通字符串处理。

#### textEnd 小于 0 的情况

对于 `textEnd` 小于 `0` 的情况，处理方式很简单：

```js
if (textEnd < 0) {
  text = html
  html = ''
}
```

就将整个 `html` 字符串作为文本处理就好了。



#### 对纯文本元素的处理

上面的代码是分析了非纯文本时的情况，在while中，还有纯文本的情况出现，当lastTag存在，同时根据`isPlainTextElement`函数得知是纯文本标签(包括 `script` 标签、`style` 标签以及 `textarea` 标签)

```js
while (html) {
  last = html
  if (!lastTag || !isPlainTextElement(lastTag)) {
      // 省略...  
  } else {
      //纯文本代码
      // 省略...  
  }
  // 省略...
}
```

值得一提的是 `else` 分支的代码处理的是纯文本标签的 **内容**，并不是纯文本标签。假设 `html` 字符串如下：

```js
html = '<textarea>aaaabbbb</textarea>'
```

在解析这段字符串的时候，首先会遇到`<textarea>`标签，该标签会被正常处理被`push`进去到`stack`数组中，并同时修改`lastTag`的值为`textarea`，然后`html`会变成`aaaabbbb</textarea>`进入下一次循环，这时就会触发`if`的判别语句

```js
!lastTag || !isPlainTextElement(lastTag)
```

从而进入`else`语句中,下面是`else`语句代码

```js
let endTagLength = 0
const stackedTag = lastTag.toLowerCase()
const reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'))
const rest = html.replace(reStackedTag, function (all, text, endTag) {
	endTagLength = endTag.length
    if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
          text = text
            .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
            .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1')
     }
     if (shouldIgnoreFirstNewline(stackedTag, text)) {
          text = text.slice(1)
     }
     if (options.chars) {
         options.chars(text)
     }
     return ''
})
index += html.length - rest.length
html = rest
parseEndTag(stackedTag, index - endTagLength, index)
```

首先定义了一个变量和两个常量

```js
//用来保存纯文本标签闭合标签的字符长度
let endTagLength = 0
//纯文本标签的小写版
const stackedTag = lastTag.toLowerCase()
//一个正则表达式实例，并且使用 reCache[stackedTag] 做了缓存 --作用是用来匹配纯文本标签的内容以及结束标签的。
const reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'))
```

分析下上面的正则表达式

```js
 new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i')
```

一上述例子为例，那么`stackedTag`的值应该为`textarea`,那么这时的表示式应该是：

```
 new RegExp('([\\s\\S]*?)(</textarea[^>]*>)', 'i')
```

该正则表示式有两个分组组成：

- 用于匹配匹配纯文本标签的内容`([\\s\\S]*?)`

  `\s` 用来匹配空白符,`\S` 则用来匹配非空白符

  由于二者同时存在于中括号(`[]`)中，所以它匹配的是二者的并集，也就是字符全集

  括号后面的 `*?`，其代表懒惰模式，也就是说只要第二个分组的内容匹配成功就立刻停止匹配。

- 用来匹配纯文本标签的结束标签`(</textarea[^>]*>)`

接着代码来到了对`html`进行处理的地方

```js
/**
 * all -- 保存着整个匹配的字符串,即：aaaabbbb</textarea>
 * text -- 第一个捕获组的值 也就是纯文本标签的内容，即：aaaabbbb
 * endTag 保存着结束标签，即：</textarea>
*/ 
const rest = html.replace(reStackedTag, function (all, text, endTag) {
 	endTagLength = endTag.length
    if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') { 		text = text.replace(/<!\--([\s\S]*?)-->/g, '$1').replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1')
    }
    
    if (shouldIgnoreFirstNewline(stackedTag, text)) {
        //其作用是忽略 <pre> 标签和 <textarea> 标签的内容中的第一个换行符
    	text = text.slice(1)
    }
    if (options.chars) {
        //将纯文本标签的内容全部作为纯文本对待。
    	options.chars(text)
    }
    return ''
 })
```

简单来所，上述的函数是根据`reStackedTag`正则的匹配规则，将`html`中匹配成功的字符串替换为空字符串

比如`html`字符串为`aaaabbbb</textarea>`,那么经过处理后的`rest`的值为空字符串

在比如所`html`字符串为`aaaabbbb</textarea>ddd`,那么经过处理后`rest`的值为`ddd`

最后看一下`else`代码中最后几句

```js
//更新 index 的值，用 html 原始字符串的值减去 rest 字符串的长度
index += html.length - rest.length
//rest 常量的值赋值给 html 进入下一次循环
html = rest
//parseEndTag 函数解析纯文本标签的结束标签
parseEndTag(stackedTag, index - endTagLength, index)
```

总的来说发现对于纯文本标签的处理宗旨就是将其内容作为纯文本对待。

## 使用`paresHTML`函数

至此`paresHTML`函数就算分析完成了，那么重新回到`pares`函数上

```js
 parseHTML(template, {
    warn,
    expectHTML: options.expectHTML,
    isUnaryTag: options.isUnaryTag,
    canBeLeftOpenTag: options.canBeLeftOpenTag,
    shouldDecodeNewlines: options.shouldDecodeNewlines,
    shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
    shouldKeepComment: options.comments,
    outputSourceRange: options.outputSourceRange,
    start (tag, attrs, unary, start, end) {
      ...
    },

    end (tag, start, end) {
     ...
    },

    chars (text: string, start: number, end: number) {
     ...
    },
    comment (text: string, start, end) {
      ...
    }
  })
```

举个例子，假如我们的模板字符串如下：

```js
templateString = '<div v-for="item of list" @click="handleClick">普通文本</div>'
```

首先查看一下`start`函数的参数

```js
 parseHTML(template, {
   	....
    start (tag, attrs, unary, start, end) {
		console.log('tagName: ', tagName)
    	console.log('attrs: ', attrs)
    	console.log('unary: ', unary)
   		console.log('start: ', start)
    	console.log('end: ', end)
    	....
    },
  })
```

该函数是用来处理开始标签的

根据`parseHTML`函数对`template`模板的解析

```js
match = {
	tagName:'div',
    attrs:[{
        name:'v-for',
        value:'item of list'
    },{
        name:'@click',
        value:'handleClick'
    }],
    start:0,
    unarySlash:underfined, // unary = isUnaryTag(tagName) || !!unarySlash
    end:47
}
```

​	`start` 钩子函数将被调用，其 `console.log` 语句将得到如下输出：

- tagName

它的值为开始标签的的名字：`'div'`。

- attrs

它是一个数组，包含所有属于该标签的属性：

```js
[
  {
    name: 'v-for',
    value: 'item of list'
  },
  {
    name: '@click',
    value: 'handleClick'
  }
]
```

- unary

它是一个布尔值，代表该标签是否是一元标签，由于 `div` 标签是非一元标签，所以 `unary` 的值将为 `false`。

- start

它的值为开始标签第一个字符在整个模板字符串中的位置，所以是 `0`。

- end

注意，`end` 的值为开始标签最后一个字符在整个模板字符串中的位置加 `1`，所以是 `47`。

调用 `parseHTML` 函数时传递了 `end` 钩子函数

```js
parseHTML(templateString, {
  // ...其他选项参数
  start (tagName, attrs, unary, start, end) {
    // ...
  },
  end (tagName, start, end) {   
      console.log('tagName: ', tagName)   
      console.log('start: ', start)    
      console.log('end: ', end)  
  }
})
```

- tagName

它的值为结束标签的名字：`'div'`。

- start

它的值为结束标签在整个模板字符串中的位置，所以是：`51`。

- end

它的值同样是结束标签最后一个字符在整个模板字符串中的位置加 `1`，所以是：`57`

调用 `parseHTML` 函数时传递了 `chars` 钩子函数

```
parseHTML(templateString, {
  // ...其他选项参数
  start (tagName, attrs, unary, start, end) {
    // ...
  },
  end (tagName, start, end) {   
     //...
  },
   chars (text: string, start: number, end: number) {
      console.log('text: ', text)   
      console.log('start: ', start)    
      console.log('end: ', end)
    },
})
```

- text

它的值为：`'普通文本'`。

- start

它的值为纯文本在整个模板字符串中的位置，所以是：`47`。

- end

它的值同样是纯文本最后一个字符在整个模板字符串中的位置加 `1`，所以是：`51`



在该例子中没有调用到`comment`函数

