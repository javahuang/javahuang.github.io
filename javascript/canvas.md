# canvas

```js
function draw(){
    var canvas = document.getElementById('tutorial');
    if (canvas.getContext){
        var ctx = canvas.getContext('2d');
    }
}
```

坐标原点左上角
只支持矩形和路径（一系列点连成的线段）两种图形的绘制

- 绘制路径
  - `beginPath()` 开始绘制
  - `closePath()` 闭合路径之后，绘制回到上下文中
  - `stroke()` 通过线条绘制图形轮廓
  - `fill()` 填充路径的内容区域生成实心图形
  - `arc(x, y, radius, startAngle, endAngle, anticlockwise)`  画一个以（x,y）为圆心的以radius为半径的圆弧（圆）
    - anticlockwise true 顺时针 false 逆时针
    - startAngle 开始和结束的弧度 **弧度=(Math.PI/180)*角度** 周长=2πr  弧长=nπr/180 2π(弧度)=360°(角度)
    - 弧度以 x 轴为基准
  - 贝赛尔曲线
    - `quadraticCurveTo(cp1x, cp1y, x, y)` 绘制二次贝塞尔曲线，cp1x,cp1y为一个控制点，x,y为结束点。
    - `bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)` 绘制三次贝塞尔曲线，cp1x,cp1y为控制点一，cp2x,cp2y为控制点二，x,y为结束点。


## 参考

[canvas MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage)
