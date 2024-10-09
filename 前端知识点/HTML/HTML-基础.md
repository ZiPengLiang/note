# HTML 基础知识

## 什么是HTML?

定义：超文本标记语言(**H**yper**T**ext **M**arkup **L**anguage，简称：HTML)

作用：一种用来结构化 Web 网页及其内容的标记语言

后缀命：.html 或者 .htm

## HTML 文档详解

```html
<!-- 此行为注释内容，不会被浏览器执行。 -->
<!doctype html>
<!-- 文件的声明，告诉浏览器以什么规则来解析 -->
<html lang="en-US">
  <!-- html 网页的开始 lang表示这个网页的语言，en-US表示为英文，如果为其他地区的语言浏览器会提示是否翻译网页。-->
  <head>
    <!-- head 提供一些机器可读信息 -->
    <meta charset="utf-8" />
    <!-- 设置网页的字符编码格式 utf-8包含了所有语言编码，如果文件保存的编码与打开文件的编码不对，会出现乱码的情况。 -->
       <meta
      name="description"
      content="描述信息"
    />
    <!-- 网页的描述 -->
    <meta
      name="keywords"
      content="用于搜索引擎的关键词"
    />
    <!-- 关键词 -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0,user-scalable=no"
    />
    <!-- 设置视口信息 -->
      <!-- 网页图标 -->
    <link
      rel="icon"
      href="favicon.ico"
    />
    <title>My test page</title>
      <!-- 网页的标题 -->
  </head>
  <body>
   <!-- 网页的主体部分 -->
  </body>
</html>
<!-- 网页的结束 -->
```

- `<!DOCTYPE html>` -- 文档类型
  - html5的声明
  - 这个元素用来关联 HTML 编写规范，以供自动查错等功能所用
  - 它作用有限，可以说仅用于保证文档正常读取
- `<html></html>` --  `<html>`元素
  - HTML 页面的根元素
  - `lang` 属性 -- 标明页面的主要语种
- `<head></head>` -- `<head>`元素
  - 主要保存供机器处理的信息，而非人类可读信息。
  - 包含诸如提供给搜索引擎的关键字和页面描述、用于设置页面样式的 CSS、字符集声明等等。
- `<body></body>` -- `<body>`元素
  - 面向用户的全部内容
  - 包含文本、图像、视频、游戏、可播放的音轨或其他内容



### 什么是HTML头部

定义：HTML 头部包含 HTML `head` 元素的内容，与 `body` 元素内容不同，页面在浏览器加载后它的内容不会在浏览器中显示，它的作用是保存页面的一些元数据。

常见元数据内容：

#### `<title>`

定义：定义文档的标题，显示在浏览器的标题栏或标签页上。

作用：

- 定义了浏览器工具栏的标题
- 当网页添加到收藏夹时，显示在收藏夹中的标题
- 显示在搜索引擎结果页面的标题

```html
<title>第十五届秋季运动会</title>
```

> [!NOTE]
>
> `<title>`只应该包含文本，若是包含有标签，则它包含的任何标签都将被忽略。



#### `<base>`

定义：指定用于一个文档中包含的所有相对 URL 的根 URL。

> [!IMPORTANT]
>
> 一个文档的基本 URL，可以通过使用 [`document.baseURI`](https://developer.mozilla.org/en-US/docs/Web/API/Node/baseURI) 的 JS 脚本查询。如果文档不包含 `<base>` 元素，`baseURI` 默认为 `document.location.href`。

属性：

- `href` -- 用于文档中相对 URL 地址的基础 URL。允许绝对和相对 URL
- `target` -- 默认浏览上下文的关键字或作者定义的名称
  - `_self` (默认值) -- 载入结果到当前浏览上下文中
  - `_blank` --  载入结果到一个新的未命名的浏览上下文。
  - `_parent` -- 载入结果到父级浏览上下文（如果当前页是内联框）。如果没有父级结构，该选项的行为和`_self`一样。
  - `_top` --  载入结果到顶级浏览上下文（该浏览上下文是当前上下文的最顶级上下文）。如果没有父级，该选项的行为和_self 一样。

```html
<base target="_top" href="http://www.example.com/" />
// 链接指向https://example.com/#anchor
<a href="#anchor">Anker</a>
```

