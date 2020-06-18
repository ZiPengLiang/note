# 迭代器和for of

### 迭代器

所谓迭代器，其实就是一个具有 next() 方法的对象，每次调用 next() 都会返回一个结果对象，该结果对象有两个属性，value 表示当前的值，done 表示遍历是否结束

```javascript
function create(items){
    let i=0
    return {
        next:function(){
            var done = i >=items.length;
            var value = !dome ？items[i++]；underfined；
            return {
                done:done,
                value:value
            }
        }
    }
}


let iterator = create([1,2,3])
console.log(iterator.next()); // { done: false, value: 1 }
console.log(iterator.next()); // { done: false, value: 2 }
console.log(iterator.next()); // { done: false, value: 3 }
console.log(iterator.next()); // { done: true, value: undefined }
```



### for of

可以i循环可迭代对象（包括Array,Set，Map，String，TypedArray，arguments对象）

```javascript
let obj = {
    value:1,
    num:2,
    name:'123'
}
//直接遍历对象会报错
for(let value in obj){
    console.log(key,value)
}
// obj is not iterable at <anonymous>
```

那怎么才可以遍历呢？

这就要说到Iterator接口了，当一种数据结构在部署了iterator接口后，该数据结构便可遍历

ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性，或者说，一个数据结构只要具有 Symbol.iterator 属性，就可以认为是"可遍历的"（iterable）。

```javascript
const obj = {
    value: 1
};

obj[Symbol.iterator] = function() {
    return createIterator([1, 2, 3]);
};

for (value of obj) {
    console.log(value);
}
```

