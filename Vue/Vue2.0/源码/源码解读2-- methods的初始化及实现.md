# 源码解读--methods的初始化及实现

源码位置 -- `src/core/instance/state.js`

```javascript
 const opts = vm.$options
 // 初始化methods方法
 if (opts.methods) initMethods(vm, opts.methods)
```

initMethod函数代码

```javascript
function initMethods(vm: Component, methods: Object) {
  const props = vm.$options.props
  for (const key in methods) {
      //非生产环境下的校验
    if (process.env.NODE_ENV !== 'production') {
        //非函数警告
      if (typeof methods[key] !== 'function') {
        warn(
          `Method "${key}" has type "${typeof methods[key]}" in the component definition. ` +
          `Did you reference the function correctly?`,
          vm
        )
      }
        //props中有同名的参数
      if (props && hasOwn(props, key)) {
        warn(
          `Method "${key}" has already been defined as a prop.`,
          vm
        )
      }
        //vm中已声明的函数
      if ((key in vm) && isReserved(key)) {
        warn(
          `Method "${key}" conflicts with an existing Vue instance method. ` +
          `Avoid defining component methods that start with _ or $.`
        )
      }
    }
      
      //重点就这一句，上述的都是为了做警告的
      
    //判断methods[key]是不是函数，
    //如果不是，那就为空函数
    //如果是通过bind
    vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm)
  }
}
```

