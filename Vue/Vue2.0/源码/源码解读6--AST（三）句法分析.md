# 句法分析 - 生成真正的AST(二)

续上文，上文已经处理了多个属性。到目前为止我们已经讲解过的属性有：

- `v-pre`
- `v-for`
- `v-if`、`v-else-if`、`v-else`
- `v-once`
- `key`
- `ref`
- `slot`、`slot-scope`、`scope`、`name`
- `is`、`inline-template`

上述属性要么使用 `getAndRemoveAttr` 函数，要么就使用 `getBindingAttr` 函数，但是无论使用哪个函数获取参数，其共同的行为是：**在获取到特定属性值的同时，还会将该属性从 `el.attrsList` 数组中移除**。那么在`el.attrsList`剩下有什么属性呢？

## processAttrs 处理剩余属性

### 剩余属性：

- `v-text`、`v-html`、`v-show`、`v-on`、`v-bind`、`v-model`、`v-cloak`

除了这些指令之外，还有部分属性的处理我们也没讲到，比如 `class` 属性和 `style` 属性，这两个属性比较特殊，因为 `Vue` 对他们做了增强，实际上在“中置处理”(`transforms` 数组)中有对于 `class` 属性和 `style` 属性的处理

### 如何处理：

`processAttrs`函数源码

```js
function processAttrs(el) {
  const list = el.attrsList
  let i, l, name, rawName, value, modifiers, syncGen, isDynamic
  for (i = 0, l = list.length; i < l; i++) {
    name = rawName = list[i].name
    value = list[i].value
    if (dirRE.test(name)) {
      // mark element as dynamic
      el.hasBindings = true
      // modifiers
      modifiers = parseModifiers(name.replace(dirRE, ''))
      // support .foo shorthand syntax for the .prop modifier
      if (process.env.VBIND_PROP_SHORTHAND && propBindRE.test(name)) {
        (modifiers || (modifiers = {})).prop = true
        name = `.` + name.slice(1).replace(modifierRE, '')
      } else if (modifiers) {
        name = name.replace(modifierRE, '')
      }
      if (bindRE.test(name)) { // v-bind
        name = name.replace(bindRE, '')
        value = parseFilters(value)
        isDynamic = dynamicArgRE.test(name)
        if (isDynamic) {
          name = name.slice(1, -1)
        }
        if (
          process.env.NODE_ENV !== 'production' &&
          value.trim().length === 0
        ) {
          warn(
            `The value for a v-bind expression cannot be empty. Found in "v-bind:${name}"`
          )
        }
        if (modifiers) {
          if (modifiers.prop && !isDynamic) {
            name = camelize(name)
            if (name === 'innerHtml') name = 'innerHTML'
          }
          if (modifiers.camel && !isDynamic) {
            name = camelize(name)
          }
          if (modifiers.sync) {
            syncGen = genAssignmentCode(value, `$event`)
            if (!isDynamic) {
              addHandler(
                el,
                `update:${camelize(name)}`,
                syncGen,
                null,
                false,
                warn,
                list[i]
              )
              if (hyphenate(name) !== camelize(name)) {
                addHandler(
                  el,
                  `update:${hyphenate(name)}`,
                  syncGen,
                  null,
                  false,
                  warn,
                  list[i]
                )
              }
            } else {
              // handler w/ dynamic event name
              addHandler(
                el,
                `"update:"+(${name})`,
                syncGen,
                null,
                false,
                warn,
                list[i],
                true // dynamic
              )
            }
          }
        }
        if ((modifiers && modifiers.prop) || (
          !el.component && platformMustUseProp(el.tag, el.attrsMap.type, name)
        )) {
          addProp(el, name, value, list[i], isDynamic)
        } else {
          addAttr(el, name, value, list[i], isDynamic)
        }
      } else if (onRE.test(name)) { // v-on
        name = name.replace(onRE, '')
        isDynamic = dynamicArgRE.test(name)
        if (isDynamic) {
          name = name.slice(1, -1)
        }
        addHandler(el, name, value, modifiers, false, warn, list[i], isDynamic)
      } else { // normal directives
        name = name.replace(dirRE, '')
        // parse arg
        const argMatch = name.match(argRE)
        let arg = argMatch && argMatch[1]
        isDynamic = false
        if (arg) {
          name = name.slice(0, -(arg.length + 1))
          if (dynamicArgRE.test(arg)) {
            arg = arg.slice(1, -1)
            isDynamic = true
          }
        }
        addDirective(el, name, rawName, value, arg, isDynamic, modifiers, list[i])
        if (process.env.NODE_ENV !== 'production' && name === 'model') {
          checkForAliasModel(el, value)
        }
      }
    } else {
      // literal attribute
      if (process.env.NODE_ENV !== 'production') {
        const res = parseText(value, delimiters)
        if (res) {
          warn(
            `${name}="${value}": ` +
            'Interpolation inside attributes has been removed. ' +
            'Use v-bind or the colon shorthand instead. For example, ' +
            'instead of <div id="{{ val }}">, use <div :id="val">.',
            list[i]
          )
        }
      }
      addAttr(el, name, JSON.stringify(value), list[i])
      // #6887 firefox doesn't update muted state if set via attribute
      // even immediately after element creation
      if (!el.component &&
        name === 'muted' &&
        platformMustUseProp(el.tag, el.attrsMap.type, name)) {
        addProp(el, name, 'true', list[i])
      }
    }
  }
}
```

先看下简化版的

```js
function processAttrs (el) {
 	const list = el.attrsList
 	let i, l, name, rawName, value, modifiers, isProp
 	for (i = 0, l = list.length; i < l; i++) {
         name = rawName = list[i].name
         value = list[i].value
         if (dirRE.test(name)) {
            // 省略...
          } else {
            // 省略...
          }
    }
}

```

首先定义了`list`作为`el.attrsList`的引用，然后后在`for`循环中分别为 `name`、`rawName` 以及 `value` 变量赋了值，其中 `name` 和 `rawName` 变量中保存的是属性的名字，而 `value` 变量中则保存着属性的值。

`if` 条件语句的判断条件:

```js
if (dirRE.test(name))
```

`dirRE` 正则 -- 匹配一个字符串是否以 `v-`、`@` 或 `:` 开头

例如：

```html
<div :custom-prop="someVal" @custom-event="handleEvent" other-prop="static-prop"></div>
```

其中 `:custom-prop` 属性和 `@custom-event` 属性将会被 `if` 语句块内的代码处理，而对于 `other-prop` 属性则会被 `else` 语句块内的代码处理

```js
if (dirRE.test(name)) {
  // mark element as dynamic
  el.hasBindings = true
  // modifiers
  modifiers = parseModifiers(name)
  if (modifiers) {
    name = name.replace(modifierRE, '')
  }
  if (bindRE.test(name)) { // v-bind    // 省略...
  } else if (onRE.test(name)) { // v-on    // 省略...
  } else { // normal directives    // 省略...
  }
} else {
  // 省略...
}
```

标识当前元素是一个动态的元素

```js
 el.hasBindings = true
```

调用 `parseModifiers` 函数，该函数接收整个指令字符串作为参数

```js
 modifiers = parseModifiers(name.replace(dirRE, ''))
```

作用就是解析指令中的修饰符，并将解析结果赋值给 `modifiers` 变量

`parseModifiers`函数源码

```js
//const modifierRE = /\.[^.\]]+(?=[^\]]*$)/g --用来全局匹配字符串中字符 . 以及 . 后面的字符

function parseModifiers(name: string): Object | void {
  const match = name.match(modifierRE)
  if (match) {
    const ret = {}
    match.forEach(m => { ret[m.slice(1)] = true })
    return ret
  }
}
```

例如：指令字符串为：`'v-bind:some-prop.sync'`

那么`match`为`[".sync"]`

那么返回`ret`如下

```js
ret = {
	sync：true
}
```

如果没有修饰符，那么`ret`为`undefined`

继续往下看

```js
 // support .foo shorthand syntax for the .prop modifier
if (process.env.VBIND_PROP_SHORTHAND && propBindRE.test(name)) {
        (modifiers || (modifiers = {})).prop = true
        name = `.` + name.slice(1).replace(modifierRE, '')
      } else if (modifiers) {
          //
        name = name.replace(modifierRE, '')
      }
```

如果`modifiers`是有值的，那么将修饰符从名字中去掉



### 解析 v-bind 指令

处理完修饰符后就到指令解析了，解析环节分为三部分，分别是对于 `v-bind` 指令的解析，对于 `v-on` 指令的解析，以及对于其他指令的解析

```js
 if (bindRE.test(name)) { // v-bind
    // 省略...
  } else if (onRE.test(name)) { // v-on
    // 省略...
  } else { // normal directives
    // 省略...
  }
```

首先看处理`v-bind`指令的

```js
if (bindRE.test(name)) { // v-bind
        name = name.replace(bindRE, '')
        value = parseFilters(value)
        isDynamic = dynamicArgRE.test(name)
        if (isDynamic) {
          name = name.slice(1, -1)
        }
        if (
          process.env.NODE_ENV !== 'production' &&
          value.trim().length === 0
        ) {
          warn(
            `The value for a v-bind expression cannot be empty. Found in "v-bind:${name}"`
          )
        }
        if (modifiers) {
          if (modifiers.prop && !isDynamic) {
            name = camelize(name)
            if (name === 'innerHtml') name = 'innerHTML'
          }
          if (modifiers.camel && !isDynamic) {
            name = camelize(name)
          }
          if (modifiers.sync) {
            syncGen = genAssignmentCode(value, `$event`)
            if (!isDynamic) {
              addHandler(
                el,
                `update:${camelize(name)}`,
                syncGen,
                null,
                false,
                warn,
                list[i]
              )
              if (hyphenate(name) !== camelize(name)) {
                addHandler(
                  el,
                  `update:${hyphenate(name)}`,
                  syncGen,
                  null,
                  false,
                  warn,
                  list[i]
                )
              }
            } else {
              // handler w/ dynamic event name
              addHandler(
                el,
                `"update:"+(${name})`,
                syncGen,
                null,
                false,
                warn,
                list[i],
                true // dynamic
              )
            }
          }
        }
        if ((modifiers && modifiers.prop) || (
          !el.component && platformMustUseProp(el.tag, el.attrsMap.type, name)
        )) {
          addProp(el, name, value, list[i], isDynamic)
        } else {
          addAttr(el, name, value, list[i], isDynamic)
        }
      }
```

首先对`name`、`value`进行处理

```js
//去掉v-bind: 或则:
name = name.replace(bindRE, '')
//使用 parseFilters 函数的返回值重新赋值 value 变量
value = parseFilters(value)
//`isProp` 变量标识着该绑定的属性是否是原生DOM对象的属性，所谓原生DOM对象的属性就是能够通过DOM元素对象直接访问的有效API，比如 `innerHTML` 就是一个原生DOM对象的属性。
isDynamic = dynamicArgRE.test(name)
```

例如：

指令字符串为 `'v-bind:some-prop.sync'`

去掉修饰符后字符串为`'v-bind:some-prop'`

在经过` name.replace(bindRE, '')`操作，`name`的值为`some-prop`



#### 处理修饰符

如果处理的字符串中带有修饰符的，那么将会进入下面的代码中：

```js
if (modifiers) {
    if (modifiers.prop && !isDynamic) {
        name = camelize(name)
        if (name === 'innerHtml') name = 'innerHTML'
    }
    if (modifiers.camel && !isDynamic) {
        name = camelize(name)
    }
    if (modifiers.sync) {
        syncGen = genAssignmentCode(value, `$event`)
        if (!isDynamic) {
            addHandler(
                el,
                `update:${camelize(name)}`,
                syncGen,
                null,
                false,
                warn,
                list[i]
            )
            if (hyphenate(name) !== camelize(name)) {
                addHandler(
                    el,
                    `update:${hyphenate(name)}`,
                    syncGen,
                    null,
                    false,
                    warn,
                    list[i]
                )
            }
        } else {
            // handler w/ dynamic event name
            addHandler(
                el,
                `"update:"+(${name})`,
                syncGen,
                null,
                false,
                warn,
                list[i],
                true // dynamic
            )
        }
    }
}
```

`v-bind` 属性为开发者提供了三个修饰符，分别是 `prop`、`camel` 和 `sync`，这也有三段`if`语句

##### 处理 `prop` 修饰符

```js
if (modifiers.prop && !isDynamic) {
    //属性名驼峰化
    name = camelize(name)
    //检查驼峰化之后的属性名是否等于字符串 'innerHtml'，如果属性名全等于该字符串则将属性名重写为字符串 'innerHTML'
    if (name === 'innerHtml') name = 'innerHTML'
}
```

##### 处理 `camel` 修饰符

```js
if (modifiers.camel && !isDynamic) {
	//属性名驼峰化
	name = camelize(name)
}
```

##### 处理 `sync` 修饰符

```js
if (modifiers.sync) {
    syncGen = genAssignmentCode(value, `$event`)
    if (!isDynamic) {
        addHandler(
            el,
            `update:${camelize(name)}`,
            syncGen,
            null,
            false,
            warn,
            list[i]
        )
        if (hyphenate(name) !== camelize(name)) {
            addHandler(
                el,
                `update:${hyphenate(name)}`,
                syncGen,
                null,
                false,
                warn,
                list[i]
            )
        }
    } else {
        // handler w/ dynamic event name
        addHandler(
            el,
            `"update:"+(${name})`,
            syncGen,
            null,
            false,
            warn,
            list[i],
            true // dynamic
        )
    }
}
```

如果 `modifiers.sync` 为真，则说明该绑定的属性使用了 `sync` 修饰符。`sync` 修饰符实际上是一个语法糖，子组件不能够直接修改 `prop` 值，通常我们会在子组件中发射一个自定义事件，然后在父组件层面监听该事件并由父组件来修改状态。比如：

```html
<template>
  <child :some-prop="value" @custom-event="handleEvent" />
</template>

<script>
export default {
  data () {
    value: ''
  },
  methods: {
    handleEvent (val) {
      this.value = val
    }
  }
}
</script>
```

为了简化该过程，我们可以在绑定属性时使用 `sync` 修饰符：

```html
<child :some-prop.sync="value" />
```

等同于

```html
<template>
  <child :some-prop="value" @update:someProp="handleEvent" />
</template>

<script>
export default {
  data () {
    value: ''
  },
  methods: {
    handleEvent (val) {
      this.value = val
    }
  }
}
</script>
```

注意事件名称 `update:someProp` 是固定的，它由 `update:` 加上驼峰化的绑定属性名称组成。所以在子组件中你需要发射一个名字叫做 `update:someProp` 的事件才能使 `sync` 修饰符生效

在 `Vue` 内部，使用 `sync` 修饰符的绑定属性与没有使用 `sync` 修饰符的绑定属性之间差异就在于：使用了 `sync` 修饰符的绑定属性等价于多了一个事件侦听，并且事件名称为 `'update:${驼峰化的属性名}'`。

如果`isDynamic`为`false`,那么会进入以下代码：

```js
syncGen = genAssignmentCode(value, `$event`)
if (!isDynamic) {
    //直接调用 addHandler 函数，在当前元素描述对象上添加事件侦听器
    addHandler(
        el,
        `update:${camelize(name)}`,
        syncGen,
        null,
        false,
        warn,
        list[i]
    )
    if (hyphenate(name) !== camelize(name)) {
        addHandler(
            el,
            `update:${hyphenate(name)}`,
            syncGen,
            null,
            false,
            warn,
            list[i]
        )
    }
}
```

`addHandler` 函数的作用实际上就是将事件名称与该事件的侦听函数添加到元素描述对象的 `el.events` 属性或 `el.nativeEvents` 属性中

首先看一下`genAssignmentCode`函数能干什么

```js
export function genAssignmentCode (
  value: string,
  assignment: string
): string {
  const res = parseModel(value)
  if (res.key === null) {
    return `${value}=${assignment}`
  } else {
    return `$set(${res.exp}, ${res.key}, ${assignment})`
  }
}
```

如上代码中 `genAssignmentCode` 的返回值即可，它返回的是一个代码字符串，可以看到如果这个代码字符串作为代码执行，其作用就是一个赋值工作



看完了对于三个绑定属性可以使用的修饰符，接下来看处理绑定属性的最后一段代码

```js
if ((modifiers && modifiers.prop) || (
    !el.component && platformMustUseProp(el.tag, el.attrsMap.type, name)
)) {
    addProp(el, name, value, list[i], isDynamic)
} else {
    addAttr(el, name, value, list[i], isDynamic)
}
```

要想进入该判断，必须先满足两个条件中的其中一个：

第一个是：该绑定的属性是原生DOM对象的属性

第二个是：

```js
!el.component && platformMustUseProp(el.tag, el.attrsMap.type, name)
```

首先`el.component`一定为`false`，`el.component` 属性保存的是标签 `is` 属性的值，如果 `el.component` 属性为假就能够保证标签没有使用 `is` 属性。

`platformMustUseProp`函数结论如下：

- `input,textarea,option,select,progress` 这些标签的 `value` 属性都应该使用元素对象的原生的 `prop` 绑定（除了 `type === 'button'` 之外）
- `option` 标签的 `selected` 属性应该使用元素对象的原生的 `prop` 绑定
- `input` 标签的 `checked` 属性应该使用元素对象的原生的 `prop` 绑定
- `video` 标签的 `muted` 属性应该使用元素对象的原生的 `prop` 绑定

`platformMustUseProp`代码

```js
const acceptValue = makeMap('input,textarea,option,select,progress')
export const mustUseProp = (tag: string, type: ?string, attr: string): boolean => {
  return (
    (attr === 'value' && acceptValue(tag)) && type !== 'button' ||
    (attr === 'selected' && tag === 'option') ||
    (attr === 'checked' && tag === 'input') ||
    (attr === 'muted' && tag === 'video')
  )
}
```



对 `v-bind` 指令的解析做一个总结：

- 1、任何绑定的属性，最终要么会被添加到元素描述对象的 `el.attrs` 数组中，要么就被添加到元素描述对象的 `el.props` 数组中。
- 2、对于使用了 `.sync` 修饰符的绑定属性，还会在元素描述对象的 `el.events` 对象中添加名字为 `'update:${驼峰化的属性名}'` 的事件。

### 解析 v-on 指令

源码如下：

```js
//export const onRE = /^@|^v-on:/
if (onRE.test(name)) { // v-on
    name = name.replace(onRE, '')
    isDynamic = dynamicArgRE.test(name)
    if (isDynamic) {
        name = name.slice(1, -1)
    }
    addHandler(el, name, value, modifiers, false, warn, list[i], isDynamic)
}
```

和`v-bind`同理，使用 `onRE` 正则去匹配指令字符串，如果该指令字符串以 `@` 或 `v-on:` 开头，则说明该指令是事件绑定，进入该判断后，首先将指令字符串中的 `@` 字符或 `v-on:` 字符串去掉，然后直接调用 `addHandler` 函数。

#### `addHandler`函数源码

看一下`addHandler`函数源码

```js
export function addHandler (
el: ASTElement,
 name: string,
 value: string,
 modifiers: ?ASTModifiers,
 important?: boolean,
 warn?: ?Function,
 range?: Range,
 dynamic?: boolean
) {
    modifiers = modifiers || emptyObject
    // warn prevent and passive modifier
    /* istanbul ignore if */
    if (
        process.env.NODE_ENV !== 'production' && warn &&
        modifiers.prevent && modifiers.passive
    ) {
        warn(
            'passive and prevent can\'t be used together. ' +
            'Passive handler can\'t prevent default event.',
            range
        )
    }

    // normalize click.right and click.middle since they don't actually fire
    // this is technically browser-specific, but at least for now browsers are
    // the only target envs that have right/middle clicks.
    if (modifiers.right) {
        if (dynamic) {
            name = `(${name})==='click'?'contextmenu':(${name})`
        } else if (name === 'click') {
            name = 'contextmenu'
            delete modifiers.right
        }
    } else if (modifiers.middle) {
        if (dynamic) {
            name = `(${name})==='click'?'mouseup':(${name})`
        } else if (name === 'click') {
            name = 'mouseup'
        }
    }

    // check capture modifier
    if (modifiers.capture) {
        delete modifiers.capture
        name = prependModifierMarker('!', name, dynamic)
    }
    if (modifiers.once) {
        delete modifiers.once
        name = prependModifierMarker('~', name, dynamic)
    }
    /* istanbul ignore if */
    if (modifiers.passive) {
        delete modifiers.passive
        name = prependModifierMarker('&', name, dynamic)
    }

    let events
    if (modifiers.native) {
        delete modifiers.native
        events = el.nativeEvents || (el.nativeEvents = {})
    } else {
        events = el.events || (el.events = {})
    }

    const newHandler: any = rangeSetItem({ value: value.trim(), dynamic }, range)
    if (modifiers !== emptyObject) {
        newHandler.modifiers = modifiers
    }

    const handlers = events[name]
    /* istanbul ignore if */
    if (Array.isArray(handlers)) {
        important ? handlers.unshift(newHandler) : handlers.push(newHandler)
    } else if (handlers) {
        events[name] = important ? [newHandler, handlers] : [handlers, newHandler]
    } else {
        events[name] = newHandler
    }

    el.plain = false
}
```

 `addHandler` 函数接收的参数，分别是

- `el`：当前元素描述对象
- `name`： 绑定属性的名字，即事件名称
- `value`：绑定属性的值，这个值有可能是事件回调函数名字，有可能是内联语句，有可能是函数表达式
- `modifiers`：指令对象
- `important`：可选参数，是一个布尔值，代表着添加的事件侦听函数的重要级别，如果为 `true`，则该侦听函数会被添加到该事件侦听函数数组的头部，否则会将其添加到尾部，
- `warn`：打印警告信息的函数，是一个可选参数
- `range`：指令在`attrsList`内对应的值
- `dynamic`：判断有没有修饰符



首先在`addHandler`函数一开始是一段这样的代码

```js
//
modifiers = modifiers || emptyObject
// warn prevent and passive modifier
/* istanbul ignore if */
if (
    process.env.NODE_ENV !== 'production' && warn &&
    modifiers.prevent && modifiers.passive
) {
    warn(
        'passive and prevent can\'t be used together. ' +
        'Passive handler can\'t prevent default event.',
        range
    )
}
```

判断`v-on`指令是否拥有修饰符对象

```js
modifiers = modifiers || emptyObject
```

如果在使用 `v-on` 指令时没有指定任何修饰符，则 `modifiers` 的值为 `undefined`，此时会使用冻结的空对象 `emptyObject` 作为替代。

接着判断如果此时如果是在非生产环境下，并且 `addHandler` 函数的第六个参数 `warn` 存在，则使用 `warn` 函数打印警告信息，提示开发者 `passive` 修饰符不能和 `prevent` 修饰符一起使用，这是因为在事件监听中 `passive` 选项参数就是用来告诉浏览器该事件监听函数是不会阻止默认行为的。

接下来是规范化“右击”事件和点击鼠标中间按钮的事件

##### 规范化“右击”事件和点击鼠标中间按钮的事件

```js
if (modifiers.right) {
    if (dynamic) {
      name = `(${name})==='click'?'contextmenu':(${name})`
    } else if (name === 'click') {
        //点击右键一般会出来一个菜单 --这本质上是触发了 contextmenu 事件。
      name = 'contextmenu'
      delete modifiers.right
    }
  } else if (modifiers.middle) {
    if (dynamic) {
      name = `(${name})==='click'?'mouseup':(${name})`
    } else if (name === 'click') {
      name = 'mouseup'
    }A
  }
```

接下来是三个判断语句,判断用了什么修饰符，源码如下： 

```js
// check capture modifier
//capture是在捕获的过程监听 -- 事件捕获
if (modifiers.capture) {
    delete modifiers.capture
    name = prependModifierMarker('!', name, dynamic)
}
//once -- 只执行一次
if (modifiers.once) {
    delete modifiers.once
    name = prependModifierMarker('~', name, dynamic)
}
/* istanbul ignore if */
//阻止默认行为
if (modifiers.passive) {
    delete modifiers.passive
    name = prependModifierMarker('&', name, dynamic)
}
```

通过判断后会修改指令名字，例如：

```html
<div @click.capture="handleClick"></div>
等价于
<div @!click="handleClick"></div>

<div @click.once="handleClick"></div>
<div @~click="handleClick"></div>

<div @click.passive="handleClick"></div>
<div @&click="handleClick"></div>
```

`addHandler` 函数继续看后面的代码

```js
let events
if (modifiers.native) {
    delete modifiers.native
    events = el.nativeEvents || (el.nativeEvents = {})
} else {
    events = el.events || (el.events = {})
}
```

定义了 `events` 变量，然后判断是否存在 `native` 修饰符，如果 `native` 修饰符存在则会在元素描述对象上添加 `el.nativeEvents` 属性，初始值为一个空对象，并且 `events` 变量与 `el.nativeEvents` 属性具有相同的引用

再往下是这样一段代码

```js
const newHandler: any = rangeSetItem({ value: value.trim(), dynamic }, range)
if (modifiers !== emptyObject) {
    newHandler.modifiers = modifiers
}
```

定义了 `newHandler` 对象，该对象初始拥有一个 `value` 属性，该属性的值就是 `v-on` 指令的属性值。

接着判断修饰符对象 `modifiers` 是否不等于 `emptyObject` -- 判断是否使用了修饰符，如果使用了修饰符，会把修饰符对象赋值给 `newHandler.modifiers` 属性。

最后：

```js

const handlers = events[name]
/* istanbul ignore if */
if (Array.isArray(handlers)) {
    important ? handlers.unshift(newHandler) : handlers.push(newHandler)
} else if (handlers) {
    events[name] = important ? [newHandler, handlers] : [handlers, newHandler]
} else {
    events[name] = newHandler
}

el.plain = false
```

举个例子

```html
<div @click.once="handleClick"></div>
```

`el`的值如下

```js
{
    type: 1,
    tag:'div',
    attrsList: [
        name:'@click.once',
        value:'handleClick'
    ],
    attrsMap: {
       '@click.once':'handleClick' 
    },
    rawAttrsMap: {},
    parent,
    children: []
 }
```

进入到`processAttrs`函数中时：

```
name = '@click.once'
value ='handleClick'
```

经过处理后得到

```js
  const list = el.attrsList

el = {
    type: 1,
    tag:'div',
    attrsList: [
        name:'@click.once',
        value:'handleClick'
    ],
    attrsMap: {
       '@click.once':'handleClick' 
    },
    rawAttrsMap: {},
    parent,
    children: [],
    hasBindings:true
 }

name = 'click.once'
value ='handleClick'
modifiers={
    once:true
}
isDynamic = false


```

由与该指令为`v-on`指令，进入`addHandler`函数

```js
 addHandler(el, name, value, modifiers, false, warn, list[i], isDynamic)
```

进到`addHandler`函数后，经过处理

```js
name ='~click'
events = {
     '~click':{
    	value:'handleClick'
   		dynamic:false
    	modifiers:{}  
		}
}
el = {
    type: 1,
    tag:'div',
    attrsList: [
        name:'@click.once',
        value:'handleClick'
    ],
    attrsMap: {
        '@click.once':'handleClick' 
    },
    rawAttrsMap: {},
    parent,
    children: [],
    hasBindings:true，
    events:{
   	 '~click':{
    	value:'handleClick'
   		dynamic:false
    	modifiers:{}  
		}
	}
}

newHandler= {
    value:'handleClick'
    dynamic:false
    modifiers:{}  
}
```

修改下模板

```html
<div @click.prevent="handleClick1" @click="handleClick2"></div>
```

同理

```js
el.events = {
  click: [
    {
      value: 'handleClick1',
      modifiers: { prevent: true }
    },
    {
      value: 'handleClick2'
    }
  ]
}
```

同理

```html
<div @click.prevent="handleClick1" @click="handleClick2" @click.self="handleClick3"></div>
```

```js
el.events = {
  click: [
    {
      value: 'handleClick1',
      modifiers: { prevent: true }
    },
    {
      value: 'handleClick2'
    },
     {
      value: 'handleClick3',
      modifiers: { self: true }
    },
  ]
}
```

接着是 `addHandler` 函数的最后一句代码

```js
el.plain = false
```

如果一个标签存在事件侦听，无论如何都不会认为这个元素是“纯”的，所以这里直接将 `el.plain` 设置为 `false`。

`addHandler` 函数作用：对于元素描述对象的影响主要是在元素描述对象上添加了 `el.events` 属性和 `el.nativeEvents` 属性



### 解析其他指令

当解析的字符串不存在`v-bind`和`v-on`指令，那么就会进入最后的判别中

```js
else { // normal directives
    name = name.replace(dirRE, '')
    // parse arg
    const argMatch = name.match(argRE)
    let arg = argMatch && argMatch[1]
    isDynamic = false
    if (arg) {
        name = name.slice(0, -(arg.length + 1))
        if (dynamicArgRE.test(arg)) {
            arg = arg.slice(1, -1)
            isDynamic = true
        }
    }
    addDirective(el, name, rawName, value, arg, isDynamic, modifiers, list[i])
    if (process.env.NODE_ENV !== 'production' && name === 'model') {
        checkForAliasModel(el, value)
    }
}
```

看下当前各个指令的处理情况

| Vue 内置提供的所有指令 | 是否已经被解析 | 解析函数       |
| ---------------------- | -------------- | -------------- |
| `v-if`                 | 是             | `processIf`    |
| `v-else-if`            | 是             | `processIf`    |
| `v-else`               | 是             | `processIf`    |
| `v-for`                | 是             | `processFor`   |
| `v-on`                 | 是             | `processAttrs` |
| `v-bind`               | 是             | `processAttrs` |
| `v-pre`                | 是             | `processPre`   |
| `v-once`               | 是             | `processOnce`  |
| `v-text`               | 否             | 无             |
| `v-html`               | 否             | 无             |
| `v-show`               | 否             | 无             |
| `v-cloak`              | 否             | 无             |
| `v-model`              | 否             | 无             |

到目前为止还有五个指令没有得到处理，分别是 `v-text`、`v-html`、`v-show`、`v-cloak` 以及 `v-model`，除了这五个 `Vue` 内置提供的指令之外，开发者还可以自定义指令，所以上面代码中 `else` 语句块内的代码就是用来处理剩余的这五个内置指令和其他自定义指令的

那么上述代码就是为了处理自定义指令和剩余的五个未被解析的内置指令

举个例子

```html
<div v-custom:arg.modif="myMethod"></div>
```

那么`name`的值为`v-custom:arg`，经过一下代码处理：

```js
name = name.replace(dirRE, '')
const argMatch = name.match(argRE)
const arg = argMatch && argMatch[1]
```

得到结果如下：

```js
name='custom:arg'
argMatch= [':arg', 'arg']
arg='arg'
```

然后对`name`进行进一步处理

```js
if (arg) {
    name = name.slice(0, -(arg.length + 1))
    if (dynamicArgRE.test(arg)) {
        arg = arg.slice(1, -1)
        isDynamic = true
    }
}
```

得到`name`的值为`custom`，得到一系列的之后会进入`addDirective`函数

```js
 addDirective(el, name, rawName, value, arg, isDynamic, modifiers, list[i])
```

以上述例子为例，参数如下

```js
addDirective(el, 'custom', 'v-custom:arg.modif', 'myMethod', 'arg',false, { modif: true },{
    name:'v-custom:arg.modif',
    value:'myMethod'
})
```

`addDirective`函数如下

```js
export function addDirective (
  el: ASTElement,
  name: string,
  rawName: string,
  value: string,
  arg: ?string,
  isDynamicArg: boolean,
  modifiers: ?ASTModifiers,
  range?: Range
) {
  (el.directives || (el.directives = [])).push(rangeSetItem({
    name,
    rawName,
    value,
    arg,
    isDynamicArg,
    modifiers
  }, range))
  el.plain = false
}
```

以上就是对自定义指令和剩余的五个未被解析的内置指令的处理，可以看到每当遇到一个这样的指令，目的是为了在元素描述对象的 `el.directives` 数组中添加一个指令信息对象，如下

```js
el.directives = [
  {
    name, // 指令名字
    rawName, // 指令原始名字
    value, // 指令的属性值
    arg, // 指令的参数
    modifiers, // 指令的修饰符
    isDynamicArg:false
  }
]
```

