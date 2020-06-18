# jQuery的extend

在jq中extend函数常用于拷贝对象

在jQuery官网定义中：

> Merge the contents of two or more objects together into the first object.

意思是合并两个或以上的对象内容并将其内容放到第一个对象中

也就是将多个对象的内容整合并复制到目标对象中。不过这种拷贝会有一个比较显著的问题，当两个对象中含有相同的字段是，后者将会覆盖前者，并不会进行深层次的覆盖

```javascript
//用法
/**
*@param target --- 目标对象
*@param object --- 被复制的对象
*/
jQuery.extend( target [, object1 ] [, objectN ] )
//示例：
var obj = {
	a:1,
    b:{
        b1:1,
        b2:2
    }
}

var obj2 = {
    b:{
        b1:3,
        b3:4
    },
    c:3
}
var obj3 ={
    d:4
} 
$.extend(obj,pbj2,obj3)
//结果
{
    a:1，
    b:{
      b1:3,
      b3:4
    }，
    c:3，
    d:4     
}
//相同的属性只会覆盖，不会往下深度复制
```

模拟一个extend函数

```javascript
function extend(){
    let length = arguments.length
    //以第一个作为本体
    let target = arguments[0]；
    for(let i =1;i<length;i++){
        let item = arguments[i];
        if(item !== null){
            //将obj中的所有属性一个一个地拷贝到第一个参数中
            for(let key in item){
                if(target[key] !== item[key]){
                    target[key] = item[key]
                }
            }
        }
    }
    return target
}
```

由于该模拟的函数是无法复制的，但jquery中的extend有一个专门针对深度复制的属性deep

```javascript
jQuery.extend( [deep], target, object1 [, objectN ] )

var obj1 = {
    a: 1,
    b: { b1: 1, b2: 2 }
};

var obj2 = {
    b: { b1: 3, b3: 4 },
    c: 3
};

var obj3 = {
    d: 4
}

console.log($.extend(true, obj1, obj2, obj3));

// {
//    a: 1,
//    b: { b1: 3, b2: 2, b3: 4 },
//    c: 3,
//    d: 4
// }

```

对此，对模拟extend函数进行优化

```javascript
function extend(){
    let length = arguments.length;
    //判断第一个是否为布尔值--短路运算
    let target = arguments[0||{};
    let deep = false;
	let i=1;
    //如果第一个为是boolean
    if(typeof target == 'boolean'){
        deep = target;
        //将第二个传入值赋值给目标对象
        target = arguments[i]||{};
        i++;
    }
    //如果目标不是对象是就无法进行对象赋值，因此要将给target设置为{}
    if(typeof target !== 'object'){
    	target = {}    
    }
    
    //开始遍历循环对象
    for(;i<length;i++){
        let item = arguments[i];
        if(item !== null){
            for(let key in item){
                if(deep&&item[key]&& typeof item[key] == 'object'){
                    target[key] = extend[deep,target[]key],item[key]
                }else if(item[key] !== target[key]){
                    target[key] = item[key]
                }
            }
        }
    }
    return target
}
```

目前问题是，万一键值中含有数组和函数是，复制会出错

实例：

```javascript
var obj1 = {
    a: 1,
    b: {
        c: 2
    }
}

var obj2 = {
    b: {
        c: [5],

    }
}

var d = extend(true, obj1, obj2)

{
    a: 1,
    b: {
        c: {
            0: 5
        }
    }
}
```

因此需要再次修改模拟代码

```javascript
function extend(){
    let copyIsArray;
    let length = arguments.length;
    //判断第一个是否为布尔值--短路运算
    let target = arguments[0]||{};
    let deep = false;
	let i=1;
    //如果第一个为是boolean
    if(typeof target == 'boolean'){
        deep = target;
        //将第二个传入值赋值给目标对象
        target = arguments[i]||{};
        i++;
    }
    //如果target不是函数和对象，则将target初始化为对象
    if(typeof target !== 'object' && typeof target !== 'function'){
    	target = {}    
    }
    
    //开始遍历循环对象
    for(;i<length;i++){
        let item = arguments[i];
        if(item !== null){
            for(let key in item){
                let src =target[key];
                let copy = item[key];
                //跳开无限引用
                if(copy === target){
                    continue
                }
                //$.isPlainObject(copy) -- 判断是否为纯对象
                copyIsArray = Array.isArray(copy)
                if(deep&&copy&&($.isPlainObject(copy)||copyIsArray)){
                    let clone = ''
                    if(copyIsArray){
                      copyIsArray = false;
                        clone = src&&Array.isArray(src)?src:[]
                    }else{
                        clone = src&& $.isPlainObject(src)?src:{}
                    }
                    target[key] = extend(deep,clone,copy)
                }else if(copy !=='underfined'){
                    target[key] = copy
                }
            }
        }
    }
    return target
}
```

