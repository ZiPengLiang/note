# 变换 Transformations

在学习变换之前必须要先了解2个方法，这2个方法是绘制复杂方法的基础

ctx.save() ----保存画布(canvas)的所有状态

ctx.restore() ---将 canvas 恢复到最近的保存状态的方法

Canvas状态存储在栈中，每当`save()`方法被调用后，当前的状态就被推送到栈中保存。

```javascript
 ctx.fillRect(0,0,150,150);   // 使用默认设置绘制一个矩形
  ctx.save();                  // 保存默认状态

  ctx.fillStyle = '#09F'       // 在原有配置基础上对颜色做改变
  ctx.fillRect(15,15,120,120); // 使用新的设置绘制一个矩形

  ctx.save();                  // 保存当前状态
  ctx.fillStyle = '#FFF'       // 再次改变颜色配置
  ctx.globalAlpha = 0.5;    
  ctx.fillRect(30,30,90,90);   // 使用新的配置绘制一个矩形

  ctx.restore();               // 重新加载之前的颜色状态
  ctx.fillRect(45,45,60,60);   // 使用上一次的配置绘制一个矩形

  ctx.restore();               // 加载默认颜色配置
  ctx.fillRect(60,60,30,30);   // 使用加载的配置绘制一个矩形
```

结果

![image-20191217114935077](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20191217114935077.png)



### 移动 Translating

**translate(x, y)** ---接受两个参数。x 是左右偏移量，y 是上下偏移量，用来移动 canvas 和它的原点到一个不同的位置

注意：在变形之前最好保存canvas的状态，这样回复起来用restore方法比较简单



```javascript
for (var i = 0; i < 3; i++) {
    for (var j = 0; j < 3; j++) {
        //首先将原点保存下来，当前原点在(0,0)的位置
      ctx.save();
      ctx.fillStyle = 'rgb(' + (51 * i) + ', ' + (255 - 51 * i) + ', 255)';
        //移动canvas的图像
      ctx.translate(10 + j * 50, 10 + i * 50);
        //画一个图形
      ctx.fillRect(0, 0, 25, 25);
        //将原点恢复到(0,0)的位置
      ctx.restore();
    }
  }
```

结果：

![image-20191217115808087](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20191217115808087.png)



### 旋转Rotating

rotate(angle)--以原点为中心旋转 canvas,这个方法只接受一个参数：旋转的角度(angle)，它是顺时针方向的，以弧度为单位的值

```javascript
ctx.translate(75,75);

  for (var i=1;i<6;i++){ // Loop through rings (from inside to out)
    ctx.save();
      //随机球的颜色
    ctx.fillStyle = 'rgb('+(51*i)+','+(255-51*i)+',255)';

    for (var j=0;j<i*6;j++){ // draw individual dots
        
      ctx.rotate(Math.PI*2/(i*6));
      ctx.beginPath();
      ctx.arc(0,i*12.5,5,0,Math.PI*2,true);
      ctx.fill();
    }

    ctx.restore();
  }
```

