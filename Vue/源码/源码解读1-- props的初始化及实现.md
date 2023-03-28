# 源码解读-- props的初始化及实现

源码位置 -- `src/core/instance/state.js`

props经过规范化后，数据会变成纯对象格式

比如

```javascript
props: ["someData"]
props: {
  	someData1: Number
}
//经过规范会后
props:{
	someData1:{
		type:Number
	},
	someData:{
		type:null //未定义数据类型
	}
}
```

在Vue中初始化props是在initState的函数上

```javascript
// 初始化props

 const opts = vm.$options
 if (opts.props) initProps(vm, opts.props)
//通常vm 代表的是Vue的实例对象
var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    }
})
//vm.$options -- 代表的是其配置的参数对象
vm.$options = mergeOptions(
    {
        components: {
            KeepAlive,
            Transition,
            TransitionGroup
        },
        directives: {
            model,
            show
        },
        filters: Object.create(null),
        _base: Vue
    },
    {
        el: '#app',
        data: {
            test: 1
        }
    }, Vue
)
```



## props的初始化

initProps函数

```javascript
function initProps(vm: Component, propsOptions: Object) {
    //定义了 propsData 常量 默认值为空对象
  const propsData = vm.$options.propsData || {}
  //定义了 props 常量和 vm._props 属性 默认值为空对象
  const props = vm._props = {}
  //定义了常量keys，并在 vm.$options上添加_propKeys属性 ，两者有相同的引用  初始值为空数组
  const keys = vm.$options._propKeys = []
  //判断是否为根组件  根组件没有$parent属性
  const isRoot = !vm.$parent
  //如果该组件为不是根组件 
  if (!isRoot) {
    //那么将shouldObserve 定义为false 用于关闭数据观测
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    //将 prop 的键名(key)添加到 keys 数组中
    //keys与 vm._props为相同引用，相当于把key也添加到了vm._props
    keys.push(key)
      //validateProp -- 判断key给定的prop数据是否符合预期的类型，并且返回默认值
    const value = validateProp(key, propsOptions, propsData, vm)
    //判断是否为生产环境
    if (process.env.NODE_ENV !== 'production') {
      //非生产环境 -- 用于做警告，合兴代码只有 defineReactive(props, key, value)
      const hyphenatedKey = hyphenate(key)
      //判断是否为保留的属性
      if (isReservedAttribute(hyphenatedKey) ||
        config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (!isRoot && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      //每个key都定义在了props上，相当于在vm._props定义了prop数据
   
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
      //当 key 不在组件实例对象上以及其原型链上没有定义时
      //将数据挂载到vm上
      //目的:避免子对象重复调用proxy函数进行代理
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
    //把那么将shouldObserve 打开 -- 开启数据监测，避免影响后续功能
  toggleObserving(true)
}
```



## props 的校验

分析下props的校验方法

```javascript
const value = validateProp(key, propsOptions, propsData, vm)
```

validateProp 作用是校验props是否合法，获取默认值等

传递给 `validateProp` 函数的四个参数分别是：

- `key`：`prop` 的名字
- `propsOptions`：整个 `props` 选项对象
- `propsData`：整个 `props` 数据来源对象
- `vm`：组件实例对象

假如定义了组件

```javascript
{
  name: 'someComp',
  props: {
    prop1: String
  }  
}
<some-comp prop1="str" />
 
```

validateProp接受到的参数如下

```javascript
//props的名字
key：prop1
//整个 `props` 选项对象
propsOptions:{
	prop1:{
        type:String
    }
}
// props 数据
propsData:{
    props:'str'
}
//组件实例对象
vm:vm
```

validateProp函数如下 --以上述例子举例

```javascript
export function validateProp (
  key: string,
  propOptions: Object,
  propsData: Object,
  vm?: Component
): any {
    //prop = {type:String}
  const prop = propOptions[key]
  //判断对应的 prop 在 propsData 上是否有数据 没有则为true
  const absent = !hasOwn(propsData, key)
  //value是一个变量，它的值是通过读取 propsData 得到,如果外界没有传数据，那么value = undefined
  //该例子上 value = 'str'
  let value = propsData[key]
  // 判断类型 
  //getTypeIndex 函数的作用准确地说是用来查找第一个参数所指定的类型构造函数是否存在于第二个参数所指定的类型构造函数数组中
  const booleanIndex = getTypeIndex(Boolean, prop.type)
  //prop类型中含有布尔值
  if (booleanIndex > -1) {
      //外部没有传值，prop也没有默认值
    if (absent && !hasOwn(prop, 'default')) {
      value = false
    } else if (value === '' || value === hyphenate(key)) {
       //判断值是不是为空 或者 名字由驼峰转连字符后与值为相同字符串
      const stringIndex = getTypeIndex(String, prop.type)
      
      if (stringIndex < 0 || booleanIndex < stringIndex) {
          //该判断成立由两种情况
          //1.没有定义 String 类型
          //2.虽然定义了 String 类型，但是 String 类型的优先级没有 Boolean 高 
          //示例如下--示例1
        value = true
      }
    }
  }
 //检测该 prop 的值是否是 undefined
  if (value === undefined) {
    //获取默认值
    value = getPropDefaultValue(vm, prop, key)
  //首先使用 prevShouldObserve 常量保存了之前的 shouldObserve 状态
    const prevShouldObserve = shouldObserve
  //紧接着将开关开启
    toggleObserving(true)
      //使得 observe 函数能够将 value 定义为响应式数据
    observe(value)
    //最后又还原了 shouldObserve 的状态。
    toggleObserving(prevShouldObserve)
  }
  if (
    process.env.NODE_ENV !== 'production' &&
    // skip validation for weex recycle-list child component props
    !(__WEEX__ && isObject(value) && ('@binding' in value))
  ) {
      //真正的校验工作
    assertProp(prop, key, value, vm, absent)
  }
  return value
}
```

示例1

```javascript
<!-- 值为空字符串 -->
<some-comp prop1="" />
<!-- 名字由驼峰转连字符后与值为相同字符串 -->
<some-comp someProp="some-prop" />


//定义一个名为someComp 的组件，并为props制定数据类型
//String优先级比Boolean高
{
  name: 'someComp',
  props: {
    prop1: {
      type: [String, Boolean]
    }
  }
}
prop1的值为some-prop
//Boolean 优先级比String高
{
  name: 'someComp',
  props: {
    prop1: {
      type: [Boolean, String]
    }
  }
}
prop1的值为true
```

