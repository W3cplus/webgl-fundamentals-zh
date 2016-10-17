# WebGL 2D 旋转（Rotation）

This post is a continuation of a series of posts about WebGL. The first <a href="webgl-fundamentals.html">started with fundamentals</a> and the previous was <a href="webgl-2d-translation.html">about translating geometry</a>.

这篇教程是一系列关于 WebGL 的教程中的一篇。第一篇教程是 <a href="../fundamentals/WebGL-Fundamentals.html" target="_blank"> WebGL 基础</a>，上一篇教程是 <a href="./WebGL-2D-Translation.html" target="_blank">关于平移几何体</a>。

I'm going to admit right up front I have no idea if how I explain this will make sense but what the heck, might as well try. First I want to introduce you to what's called a "unit circle". If you remember your junior high school math (don't go to sleep on me!) a circle has a radius. The radius of a circle is the distance from the center of the circle to the edge. A unit circle is a circle with a radius of 1.0.

在开始之前我要先承认一下我不知道该怎么通俗易懂的讲解这个内容，但不管怎么样，我都会尽力去做。首先我想给你介绍一下什么是“单位圆”。如果你还记得你初中数学（别去睡觉啊！）的内容——圆有半径。一个圆的半径是圆心到圆边的距离。单位圆是半径为 1.0 的圆。

Here's a unit circle.

这里有个单位圆。

<iframe src="http://webglfundamentals.org/webgl/unit-circle.html" width="300" height="300"></iframe>

Notice as you drag the blue handle around the circle the X and Y positions change. Those represent the position of that point on the circle. At the top Y is 1 and X is 0. On the right X is 1 and Y is 0.

注意当你拖动在圆上的蓝色控制器，X 和 Y 的值会跟着改变。它们表示这个点在圆上的位置。当这个点在圆的最上面时候 Y 是 1，X 是 0，在最右边的时候 X 是 1，Y 是 0。

If you remember from basic 3rd grade math if you multiply something by 1 it stays the same. So 123 * 1 = 123. Pretty basic, right? Well, a unit circle, a circle with a radius of 1.0 is also a form of 1. It's a rotating 1. So you can multiply something by this unit circle and in a way it's kind of like multiplying by 1 except magic happens and things rotate.

如果你还记得在基础的三年级数学中把某个数乘以 1 之后这个数依然不变，比如 123 * 1 = 123。非常基础，对吧？那么，单位圆——半径为 1.0 的圆也是 1 的一种形式。它是旋转的 1。因此你可以将某个东西乘以这个单位圆，在某种程度上，这与乘以 1 有点类似，除了会发生一些神奇的事情以及事物旋转。

We're going to take that X and Y value from any point on the unit circle and we'll multiply our geometry by them from <a href="webgl-2d-translation.html">our previous sample</a>.

我们将从单位圆上的任意一点取得 X 和 Y 的值，然后我们将它们乘以我们 <a href="./WebGL-2D-Translation.html" target="_blank"> 上一篇教程中示例</a> 的几何体。

Here are the updates to our shader.

下面是对我们着色器的更新。

```
 <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;

    uniform vec2 u_resolution;
    uniform vec2 u_translation;
    uniform vec2 u_rotation;

    void main() {
      // Rotate the position
      // 旋转位置
      vec2 rotatedPosition = vec2(
         a_position.x * u_rotation.y + a_position.y * u_rotation.x,
         a_position.y * u_rotation.y - a_position.x * u_rotation.x);

      // Add in the translation.
      // 添加平移
      vec2 position = rotatedPosition + u_translation;
      ...
    }
  </script>
```

And we update the JavaScript so that we can pass those 2 values in.

然后我们更新 Javascript 代码使得我们可以传递那两个参数的值进去。

```
  <script>
    ...
    var rotationLocation = gl.getUniformLocation(program, "u_rotation");
    ...
    var rotation = [0, 1];
    ..
    // Draw the scene.
    function drawScene() {
      // Clear the canvas.
      gl.clear(gl.COLOR_BUFFER_BIT);

      // Set the translation.
      gl.uniform2fv(translationLocation, translation);

      // Set the rotation.
      gl.uniform2fv(rotationLocation, rotation);

      // Draw the rectangle.
      gl.drawArrays(gl.TRIANGLES, 0, 18);
    }
  </script>
```

And here's the result. Drag the handle on the circle to rotate or the sliders to translate.

下面是代码运行的结果。拖动圆上的控制器进行选择或者拖动滑动器进行平移。

<iframe src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-2d-geometry-rotation.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-2d-geometry-rotation.html" target="_blank">点击这里在新窗口查看例子</a>

Why does it work? Well, look at the math.

为什么这段代码能运行？我们先来看一下数学公式。

```
  rotatedX = a_position.x * u_rotation.y + a_position.y * u_rotation.x;
  rotatedY = a_position.y * u_rotation.y - a_position.x * u_rotation.x;
```

Let's say you have a rectangle and you want to rotate it. Before you start rotating it the top right corner is at 3.0, 9.0. Let's pick a point on the unit circle 30 degrees clockwise from 12 o'clock.

假设你有一个矩形，并且你想旋转它。在你开始旋转它之前，这个矩形右上角的位置为（3.0，9.0）。让我们选择一个在单位圆中从 12 点钟的位置顺时针偏移 30 度的点。

![](http://webglfundamentals.org/webgl/resources/rotate-30.png)

The position on the circle there is 0.50 and 0.87

在圆上这个点的位置为 0.50 和 0.87

```
  3.0 * 0.87 + 9.0 * 0.50 = 7.1
  9.0 * 0.87 - 3.0 * 0.50 = 6.3
```

That's exactly where we need it to be

那刚好是我们所需要的位置

![](http://webglfundamentals.org/webgl/resources/rotation-drawing.svg)

The same for 60 degrees clockwise

顺时针旋转 60 度也是如此

![](http://webglfundamentals.org/webgl/resources/rotate-60.png)

The position on the circle there is 0.87 and 0.50

这里在圆上的位置为 0.87 和 0.50

```
  3.0 * 0.50 + 9.0 * 0.87 = 9.3
  9.0 * 0.50 - 3.0 * 0.87 = 1.9
```

You can see that as we rotate that point clockwise to the right the X value gets bigger and the Y gets smaller. If we kept going past 90 degrees X would start getting smaller again and Y would start getting bigger. That pattern gives us rotation.

你可以看到当我们顺时针向右旋转那个点时，X 的值变得越来越大而 Y 的值变得越来越小。如果我们继续选择超过 90 度，X 的值又开始变小而 Y 的值开始变大。这种模式给了我们旋转。

There's another name for the points on a unit circle. They're called the sine and cosine. So for any given angle we can just look up the sine and cosine like this.

在单位圆上的点有另一个名字。它们被称为正弦和余弦。因为对任意给定的角度，我们可以像这样查找它的正弦和余弦。

```
  function printSineAndCosineForAnAngle(angleInDegrees) {
    var angleInRadians = angleInDegrees * Math.PI / 180;
    var s = Math.sin(angleInRadians);
    var c = Math.cos(angleInRadians);
    console.log("s = " + s + " c = " + c);
  }
```

If you copy and paste the code into your JavaScript console and type `printSineAndCosignForAngle(30)` you see it prints `s = 0.49 c = 0.87` (note: I rounded off the numbers.)

如果你复制粘贴这段代码到你的 JavaScript 控制台并输入 `printSineAndCosignForAngle(30)` 你会看到它显示 `s = 0.49 c = 0.87` （注意：我去掉了数字的尾数）。

If you put it all together you can rotate your geometry to any angle you desire. Just set the rotation to the sine and cosine of the angle you want to rotate to.

如果你将上面的代码整合在一起，你可以按你想要的角度去旋转一个几何体。只需要给 `rotation` （示例代码中的旋转变量）设置你想要旋转的角度的正弦和余弦。

```
  ...
  var angleInRadians = angleInDegrees * Math.PI / 180;
  rotation[0] = Math.sin(angleInRadians);
  rotation[1] = Math.cos(angleInRadians);
```

Here's a version that just has an angle setting. Drag the sliders to translate or rotate.

如下是一个有角度设置版本的示例。通过拖动滑动器进行平移或旋转。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-rotation-angle.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-rotation-angle.html" target="_blank">点击这里在新窗口查看例子</a>

I hope that made some sense. <a href="webgl-2d-scale.html">Next up a simpler one. Scale</a>.

我希望这还是比较易懂的。<a href="./WebGL-2D-Scale.html" target="_blank">接下来是比较简单的一个内容——缩放</a>。


> ## What are radians?
> 
> Radians are a unit of measurement used with circles, rotation and angles. Just like we can measure distance in inches, yards, meters, etc we can measure angles in degrees or radians.
> 
> You're probably aware that math with metric measurements is easier than math with imperial measurements. To go from inches to feet we divide by 12. To go from inches to yards we divide by 36. I don't know about you but I can't divide by 36 in my head. With metric it's much easier. To go from millimeters to centimeters we divide by 10. To go from millimeters to meters we divide by 1000. I **can** divide by 1000 in my head.
>
> Radians vs degrees are similar. Degrees make the math hard. Radians make the math easy. There are 360 degrees in a circle but there are only 2π radians. So a full turn is 2π radians. A half turn is 1π radian. A 1/4 turn, ie 90 degress is 1/2π radians. So if you want to rotate something 90 degrees just use `Math.PI * 0.5`. If you want to rotate it 45 degrees use `Math.PI * 0.25` etc.
>
> Nearly all math involving angles, circles or rotation works very simply if you start thinking in radians. So give it try. Use radians, not degrees except in UI displays.

> ## 什么是弧度
>
> 弧度是一种用于圆、旋转和角度的测量单位。就像我们可以用英寸、码、米等来测量距离，我们也可以用角度或者弧度来测量一个角。
>
> 你很可能已经意识到在数学中使用公制单位要比使用英制单位要简单很多。从英寸到英尺我们需要除以 12。从英寸到码我们需要除以 36。我不知道你但我不会在脑海里心算除以 36。使用公制单位就要简单很多。从毫米到厘米我们除以 10。从毫米到米我们除以 1000。我 **可以** 心算除以 1000。
>
> 弧度和角度的比较跟这相似。角度使计算变得困难。弧度让计算变得简单。在一个圆中有360度但只有 2π 弧度。所以转一圈是 2π 弧度。半圈是 1π 弧度。四分之一圈，即 90 度是 1/2π 弧度。所以如果你想旋转某个东西 90 度你只需要使用 `Math.PI * 0.5`。以此类推，如果你想旋转 45 度就使用 `Math.PI * 0.25`。
>
> 几乎所有的计算都会涉及角度，如果你开始以弧度为单位进行思考，圆和旋转的原理会非常简单。所以去试一试。除了在用户界面显示时使用角度外，都使用弧度。
