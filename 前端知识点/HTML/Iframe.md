# iframe 基本运用

作用：它能够将另一个 HTML 页面嵌入到当前页面中。

示例：

```html
<iframe src="https://www.example.com"
        width="100%" height="500"
        allowfullscreen sandbox>
  <p><a href="https://www.example.com">点击打开嵌入页面</a></p>
</iframe>
```



## 关键应用：

### 1）`iframe`高度根据内容自动适配

示例：以下面例子为例

```vue
<iframe src="https://www.example.com" width="100%" ref='iframeContentRef'></iframe>
```

#### 不跨域

```vue
<script setup>
const iframeContentRef = ref(null);
const setIframeHeight = (iframe) => {
    if (iframe) {
        // 获取iframe里面的内容
        var iframeWin = iframe.contentWindow || iframe.contentDocument.parentWindow;
        if (iframeWin.document.body) {
        	iframe.height = iframeWin.document.documentElement.scrollHeight || iframeWin.document.body.scrollHeight;
        }
    }
};
setIframeHeight(iframeContentRef.value)
</script>
```

#### 跨域

因为跨域的原因，`iframe`直接访问内容的`document`,因此只能通过取巧的方式来获取内容页面的高度

子页面如下例子：`www.children.com`

```vue
<template>
    <div class="detail-content" ref="detailContentRef"></div>
</template>
<script setup>
import { useWatchResize } from '@/libs/ai/watchResize';
const route = useRoute();
const id = computed(() => route.query.id);
let observer = null;
const detailContentRef = ref(null)
onMounted(async () => {
    let { initObserver } = useWatchResize(id.value);
    observer = initObserver(detailContentRef.value);
});
onBeforeUnmount(() => {
    if (observer) {
        observer.disconnect();
    }
});
</script>
```

`watchResize.js` 内容如下

```js
export const useWatchResize = (id) => {
    let observer = null;
    // 事件防抖
    const debounce = (fn, wait) => {
        let timeout;
        return function executedFunction(...args) {
            const later = () => {
                clearTimeout(timeout);
                fn(...args);
            };
            clearTimeout(timeout);
            timeout = setTimeout(later, wait);
        };
    };

    const handleResize = (entries) => {
        if (!entries.length) return;
        const { height } = entries[0].contentRect;
        const messageData = {
            content: {
                height: parseInt(height) + 10,
                id: id,
            },
            dataType: 'Height',
            districtData: entries.length > 0 ? entries[0].contentRect : null,
        };
		// 通过postMessage的方式来给父页面传递高度信息
        window.parent.postMessage(messageData, '*');
    };
    const initObserver = (dom) => {
        observer = new ResizeObserver(debounce(handleResize, 30));
        observer.observe(dom);
        return observer;
    };
    return {
        initObserver,
    };
};

```

父页面:

```vue
<template>
    <div class="father-content">
    	<iframe :src="url" width="100%" :height='iframeHeight'></iframe>
    </div>
</template>
<script setup>
const url= ref()
const id = ref()
const iframeHeight = ref('1000px')
const iframeRef = ref(null)
const handleMessage = (event: MessageEvent) => {
    // 根据id 判断是否当前iframe
    if (
      event.data.dataType === 'Height' &&
      id.value === event.data.content.id
    ) {
      iframeHeight.value = event.data.content.height + 'px';
      if (iframeRef.value) {
         // 根据内容高度修改iframe的高度
        iframeRef.value.current.height = iframeHeight.value;
      }
    }
  };
onMounted(async () => {
    id.value = new Date().getTime().toString()
    url.value = `www.children.com?id=${id.value}`
    // 父页面监听子页面加载完后返回的信息
    window.addEventListener('message', handleMessage);
});
onBeforeUnmount(() => {
    window.removeEventListener('message', handleMessage);
});
</script>
```



### 2） 如何判断iframe内容加载完成

通过`iframe`内容加载完成后会触发`iframe`的`onload`事件

```vue
<template>
    <div style="width: 100%; height: 100%" v-loading="loading">
        <iframe style="width: 100%; height: 100%; border: 0" @load="handleIframeload" :src="url"></iframe>
    </div>
</template>
<script setup>
const loading = ref(true);
const handleIframeload = () => {
    loading.value = false;
};
const url = computed(() => {
    return 'www.children.com';
});
</script>
<style scoped lang="scss"></style>

```

