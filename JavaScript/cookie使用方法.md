# Cookie使用方法

## **概念**

Cookie 是一些数据, 存储于你电脑上的文本文件中。

当 web 服务器向浏览器发送 web 页面时，在连接关闭后，服务端不会记录用户的信息。

Cookie 的作用就是用于解决 "如何记录客户端的用户信息":

- 当用户访问 web 页面时，他的名字可以记录在 cookie 中。
- 在用户下一次访问该页面时，可以在 cookie 中读取用户访问记录。

存储方式 -- 名/值对形式



## 格式

document.cookie

## 属性选项

1. expires -- 设置“cookie 什么时间内有效”
2. domain -- 域名
3. path -- 路径
4. secure -- 设置`cookie`只在确保安全的请求中才会发送
5. HttpOnly -- 设置`cookie`是否能通过 `js` 去访问

在设置任一个cookie时都可以设置相关的这些属性，当然也可以不设置，这时会使用这些属性的默认值。在设置这些属性时，属性之间由一个分号和一个空格隔开。代码示例如下：

```js
"key=name; expires=Thu, 25 Feb 2016 04:18:00 GMT; domain=ppsc.sankuai.com; path=/; secure; HttpOnly"
```

### expires

`expires`选项用来设置“cookie 什么时间内有效”。`expires`其实是`cookie`失效日期，`expires`必须是 `GMT` 格式的时间（可以通过`new Date().toGMTString()`或者` new Date().toUTCString()` 来获得）。

如果没有设置该选项，则默认有效期为`session`，即会话`cookie`。这种`cookie`在浏览器关闭后就没有了

### domain 和 path

`domain`是域名，`path`是路径，两者加起来就构成了 URL，`domain`和`path`一起来限制 cookie 能被哪些 URL 访问。

`domain`和`path`2个选项共同决定了`cookie`何时被浏览器自动添加到请求头部中发送出去。如果没有设置这两个选项，则会使用默认值。`domain`的默认值为设置该`cookie`的网页所在的域名，`path`默认值为设置该`cookie`的网页所在的目录。

### secure

secure选项用来设置cookie只在确保安全的请求中才会发送。当请求是HTTPS或者其他安全协议时，包含 secure 选项的 cookie才能被发送至服务器。

默认情况下，cookie不会带secure选项(即为空)。所以默认情况下，不管是HTTPS协议还是HTTP协议的请求，cookie 都会被发送至服务端。但要注意一点，secure选项只是限定了在安全情况下才可以传输给服务端，但并不代表你不能看到这个 cookie。

下面我们设置一个 secure类型的 cookie：

```js
document.cookie = "name=huang; secure";
```

### httpOnly

用来设置cookie是否能通过 js 去访问。默认情况下，cookie不会带httpOnly选项(即为空)，所以默认情况下，客户端是可以通过js代码去访问（包括读取、修改、删除等）这个cookie的。当cookie带httpOnly选项时，客户端则无法通过js代码去访问（包括读取、修改、删除等）这个cookie。 -- 用来防止攻击



## 封装方法

```js
function setCookie(name,value,n){
	var oDate = new Date();
	oDate.setDate(oDate.getDate()+n);
	document.cookie = name+"="+value+";expires="+oDate;
}

function getCookie(name){
	var str = document.cookie;
	var arr = str.split("; ");
	for(var i = 0; i < arr.length; i++){
		//console.log(arr[i]);
		var newArr = arr[i].split("=");
		if(newArr[0]==name){
			return newArr[1];
		}
	}
}

function removeCookie(name){
	setCookie(name,1,-1);
}
```



## 缺点

1. 每个特定域名下的cookie数量有限
2.  存储量太小，只有4KB；
3. 每次HTTP请求都会发送到服务端，影响获取资源的效率；
4. 需要自己封装获取、设置、删除cookie的方法

