# vue打包 element-icons.woff 和element-icons.ttf 字体文件路径错误，导致icon图标显示成小方块的问题。

出现打包后icon图标变成小方块的原因只有两个

## 1.element-ui版本和element-ui/lib/theme-chalk/index.css版本不一致

解决办法：

卸载已经有的element-ui

npm uninstall element-ui

然后在安装新的

npm install element-ui

## 2.打包后的 element-icons.woff 和element-icons.ttf 字体文件路径错误

访问路径多了css/static/fonts/

结果就变成了dist/static/css/static/fonts/element-icons.535877f.woff  （错误路径）

![img](常见的vue问题/20200413154952707.png)

![img](常见的vue问题/20200413155022978.png)



3.打开本地邮箱

通过mailto协议，添加a标签

```text
mailto:johndoe@bigcorp.com // johndoe@bigcorp.com --目标又将
```

[超链接中的 mailto](https://zhuanlan.zhihu.com/p/415583405)