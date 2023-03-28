# 自定义指令

**指令系统**是计算机硬件的语言系统，也叫机器语言，它是系统程序员看到的计算机的主要属性。因此指令系统表征了计算机的基本功能决定了机器所要求的能力



## 自定义指令

### 全局注册

```js
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()  // 页面加载完成之后自动让输入框获取到焦点的小功能
  }
})
```

### 局部注册

```js
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus() // 页面加载完成之后自动让输入框获取到焦点的小功能
    }
  }
}
```



### 钩子函数

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)
- `update`：所在组件的 `VNode` 更新时调用，但是可能发生在其子 `VNode` 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新
- `componentUpdated`：指令所在组件的 `VNode` 及其子 `VNode` 全部更新后调用
- `unbind`：只调用一次，指令与元素解绑时调用



所有的钩子函数的参数

- `el`：指令所绑定的元素，可以用来直接操作 `DOM`
- `binding`：一个对象，包含以下`property`：
  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`
- `vnode`：`Vue` 编译生成的虚拟节点
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用



## 应用场景

- 防抖

  ```js
  Vue.directive('throttle', {
    bind: (el, binding) => {
      let throttleTime = binding.value; // 防抖时间
      if (!throttleTime) { // 用户若不设置防抖时间，则默认2s
        throttleTime = 2000;
      }
      let cbFun;
      el.addEventListener('click', event => {
        if (!cbFun) { // 第一次执行
          cbFun = setTimeout(() => {
            cbFun = null;
          }, throttleTime);
        } else {
          event && event.stopImmediatePropagation();
        }
      }, true);
    },
  });
  // 2.为button标签设置v-throttle自定义指令
  <button @click="sayHello" v-throttle>提交</button>
  ```

- 图片懒加载

  ```js
  const LazyLoad = {
      // install方法
      install(Vue,options){
      	  // 代替图片的loading图
          let defaultSrc = options.default;
          Vue.directive('lazy',{
              bind(el,binding){
                  LazyLoad.init(el,binding.value,defaultSrc);
              },
              inserted(el){
                  // 兼容处理
                  if('IntersectionObserver' in window){
                      LazyLoad.observe(el);
                  }else{
                      LazyLoad.listenerScroll(el);
                  }
                  
              },
          })
      },
      // 初始化
      init(el,val,def){
          // data-src 储存真实src
          el.setAttribute('data-src',val);
          // 设置src为loading图
          el.setAttribute('src',def);
      },
      // 利用IntersectionObserver监听el
      observe(el){
          let io = new IntersectionObserver(entries => {
              let realSrc = el.dataset.src;
              if(entries[0].isIntersecting){
                  if(realSrc){
                      el.src = realSrc;
                      el.removeAttribute('data-src');
                  }
              }
          });
          io.observe(el);
      },
      // 监听scroll事件
      listenerScroll(el){
          let handler = LazyLoad.throttle(LazyLoad.load,300);
          LazyLoad.load(el);
          window.addEventListener('scroll',() => {
              handler(el);
          });
      },
      // 加载真实图片
      load(el){
          let windowHeight = document.documentElement.clientHeight
          let elTop = el.getBoundingClientRect().top;
          let elBtm = el.getBoundingClientRect().bottom;
          let realSrc = el.dataset.src;
          if(elTop - windowHeight<0&&elBtm > 0){
              if(realSrc){
                  el.src = realSrc;
                  el.removeAttribute('data-src');
              }
          }
      },
      // 节流
      throttle(fn,delay){
          let timer; 
          let prevTime;
          return function(...args){
              let currTime = Date.now();
              let context = this;
              if(!prevTime) prevTime = currTime;
              clearTimeout(timer);
              
              if(currTime - prevTime > delay){
                  prevTime = currTime;
                  fn.apply(context,args);
                  clearTimeout(timer);
                  return;
              }
  
              timer = setTimeout(function(){
                  prevTime = Date.now();
                  timer = null;
                  fn.apply(context,args);
              },delay);
          }
      }
  
  }
  export default LazyLoad;
  ```

  

