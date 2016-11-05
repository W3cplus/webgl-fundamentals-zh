# WebGL 2D 矩阵（Matrices）

This post is a continuation of a series of posts about WebGL. The first <a href="webgl-fundamentals.html">started with fundamentals</a> and the previous was <a href="webgl-2d-scale.html">about scaling 2D geometry</a>.

这篇教程是一系列关于 WebGL 的教程中的一篇。第一篇教程是 <a href="../fundamentals/WebGL-Fundamentals.html" target="_blank"> WebGL 基础</a>，上一篇教程是 <a href="./WebGL-2D-Scale.html" target="_blank">关于缩放 2D 几何体</a>。

In the last 3 posts we went over how to <a href="webgl-2d-translation.html">translate geometry</a>, <a href="webgl-2d-rotation.html">rotate geometry</a>, and <a href="webgl-2d-scale.html">scale geometry</a>. Translation, rotation and scale are each considered a type of 'transformation'. Each of these transformations required changes to the shader and each of the 3 transformations was order dependent. In <a href="webgl-2d-scale.html">our previous example</a> we scaled, then rotated, then translated. If we applied those in a different order we'd get a different result.

在前面的 3 篇教程中我们已经详细讲解了如何 <a href="./WebGL-2D-Translation.html" target="_blank">平移几何体</a>、<a href="./WebGL-2D-Rotation" target="_blank">旋转几何体</a>，以及 <a href="./WebGL-2D-Scale.html" target="_blank">缩放几何体</a>。平移、旋转和缩放都被认为是 “变换” 的一种类型。每一种变换都需要改变着色器并且这三种变换都对顺序有依赖性。在 <a href="./WebGL-2D-Scale.html" target="_blank">我们上一个例子中</a> 我们先缩放，再旋转，最后再平移。如果我们用不同的顺序去执行，我们会得到不用的结果。

<!--more-->
For example here is a scale of 2, 1, rotation of 30 degrees, and translation of 100, 0.

例如下面的例子中，X 和 Y 的方向的缩放系数分别为 2 和 1，顺时针旋转 30 度，然后在 X 和 Y 方向分别平移 100 个单位和 0 个单位。

![](http://wiki.jikexueyuan.com/project/webgl/images/transformation1.png)

And here is a translation of 100,0, rotation of 30 degrees and scale of 2, 1

而如下是先在 X 和 Y 方向分别平移 100 个单位和 0 个单位，再顺时针旋转 30 度，最后再进行缩放，X 和 Y 方向的缩放系数分别为 2 和 1。

![](http://wiki.jikexueyuan.com/project/webgl/images/transformation2.png)

The results are completely different. Even worse, if we needed the second example we'd have to write a different shader that applied the translation, rotation, and scale in our new desired order.

结果完全不同。更糟糕的是，如果我们想要的是第二个例子那样的，我们必须重写一个新的不同的着色器，按照我们想要的平移、旋转、缩放顺序去执行。

Well, some people way smarter than me figured out that you can do all the same stuff with matrix math. For 2D we use a 3x3 matrix. A 3x3 matrix is like a grid with 9 boxes:

然而，有些比我聪明的人指出你可以用矩阵数学来做一模一样的事情。对于 2D 我们使用一个 3×3 的矩阵。一个 3×3 的矩阵像一个有着 9 个格子的网格：

<style>
.glocal-center { text-align: center; } 
.glocal-center-content { margin-left: auto; margin-right: auto; } 
.glocal-mat td, .glocal-b { border: 1px solid black; text-align: left;} .glocal-mat td { text-align: center; } 
.glocal-border { border: 1px solid black; } 
.glocal-sp { text-align: right !important;  width: 8em;} 
.glocal-blk { color: black; background-color: black; } 
.glocal-left { text-align: left; } 
.glocal-right { text-align: right; }
</style>
<div class="glocal-center">
  <table class="glocal-center-content glocal-mat">
    <tr><td>1.0</td><td>2.0</td><td>3.0</td></tr>
    <tr><td>4.0</td><td>5.0</td><td>6.0</td></tr>
    <tr><td>7.0</td><td>8.0</td><td>9.0</td></tr>
  </table>
</div>

To do the math we multiply the position down the columns of the matrix and add up the results. Our positions only have 2 values, x and y, but to do this math we need 3 values so we'll use 1 for the third value.

进行数学计算的方式是我们将坐标依次乘以矩阵的列，并将结果相加。我们的坐标只有两个值，x 和 y，但是在这个计算中我们需要三个值，所以我们设置第三个值为 1。

In this case our result would be

在这个例子中我们的结果将是：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col class="glocal-b"/>
    <tr><td class="glocal-right">newX&nbsp;=&nbsp;</td><td>x&nbsp;*&nbsp;</td><td class="glocal-border">1.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td>x&nbsp;*&nbsp;</td><td class="glocal-border">2.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td>x&nbsp;*&nbsp;</td><td class="glocal-border">3.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>y&nbsp;*&nbsp;</td><td class="glocal-border">4.0</td><td class="glocal-left">&nbsp;+</td><td></td><td>y&nbsp;*&nbsp;</td><td class="glocal-border">5.0</td><td class="glocal-left">&nbsp;+&nbsp;</td><td></td><td>y&nbsp;*&nbsp;</td><td class="glocal-border">6.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>1&nbsp;*&nbsp;</td><td>7.0</td><td>&nbsp;</td><td></td><td>1&nbsp;*&nbsp;</td><td>8.0</td><td>&nbsp;&nbsp;</td><td></td><td>1&nbsp;*&nbsp;</td><td>9.0</td><td>&nbsp;</td></tr>
  </table>
</div>

You're probably looking at that and thinking "WHAT'S THE POINT?" Well, let's assume we have a translation. We'll call the amount we want to translate by tx and ty. Let's make a matrix like this

你可能看着上面的操作然后在想 “重点在哪里”？让我们假设我们有一个平移操作。我们将要执行的平移量分别为 tx 和 ty。我们可以构造出像下面那样的矩阵：

<div class="glocal-center">
  <table class="glocal-center-content glocal-mat">
    <tr><td>1.0</td><td>0.0</td><td>0.0</td></tr>
    <tr><td>0.0</td><td>1.0</td><td>0.0</td></tr>
    <tr><td>tx</td><td>ty</td><td>1.0</td></tr>
  </table>
</div>

And now check it out

然后现在进行计算

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">1.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td class="glocal-left">&nbsp;+</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">1.0</td><td class="glocal-left">&nbsp;+&nbsp;</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>tx</td><td>&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>ty</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

If you remember your algebra, we can delete any place that multiplies by zero. Multiplying by 1 effectively does nothing so let's simplify to see what's happening

如果你还记得代数学，我们可以删掉所有与 0 相乘的地方。乘以 1 在实际上并没有什么作用，所以让我们简化一下计算然后看看会发生什么：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">1.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td class="glocal-blk">x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td class="glocal-blk">x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td class="glocal-blk">y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk glocal-left">&nbsp;+</td><td></td><td>y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">1.0</td><td class="glocal-left">&nbsp;+&nbsp;</td><td></td><td class="glocal-blk">y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td>tx</td><td>&nbsp;</td><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td>ty</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

or more succinctly

或者更加简洁的：

```
newX = x + tx;
newY = y + ty;
```

And extra we don't really care about. That looks surprisingly like <a href="webgl-2d-translation.html">the translation code from our translation example</a>.

我们并不会去在意 `extra` 变量。上面的式子看起来跟 <a href="./WebGL-2D-Translation.html" target="_blank">我们平移教程的例子中的平移代码</a> 惊奇的相似。

Similarly let's do rotation. Like we pointed out in the rotation post we just need the sine and cosine of the angle at which we want to rotate, so

同样地，旋转也是这么做。就像在旋转那篇教程中指出的我们只需要我们想要旋转的角度的正弦和余弦，所以

```
  s = Math.sin(angleToRotateInRadians);
  c = Math.cos(angleToRotateInRadians);
```

And we build a matrix like this

然后我们构造如下的矩阵：

<div class="glocal-center">
  <table class="glocal-center-content glocal-mat">
    <tr><td>c</td><td>-s</td><td>0.0</td></tr>
    <tr><td>s</td><td>c</td><td>0.0</td></tr>
    <tr><td>0.0</td><td>0.0</td><td>1.0</td></tr>
  </table>
</div>

Applying the matrix we get this

对这个矩阵执行我们上面提到的操作：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">c</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">-s</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">s</td><td class="glocal-left">&nbsp;+</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">c</td><td class="glocal-left">&nbsp;+&nbsp;</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>0.0</td><td>&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>0.0</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

Blacking out all multiply by 0s and 1s we get

将乘以 0 和乘以 1 的部分涂黑：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">c</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">-s</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td class="glocal-blk">x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">s</td><td class="glocal-left glocal-blk">&nbsp;+</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">c</td><td class="glocal-left glocal-blk">&nbsp;+&nbsp;</td><td></td><td class="glocal-blk">y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">0.0</td><td>&nbsp;</td><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">0.0</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

And simplifying we get

在简化之后，我们得：

```
  newX = x *  c + y * s;
  newY = x * -s + y * c;
```

Which is exactly what we had in our <a href="webgl-2d-rotation.html">rotation sample</a>.

这实际上跟我们的 <a href="./WebGL-2D-Rotation.html" target="_blank">旋转例子</a> 中用到的式子一样。

And lastly scale. We'll call our 2 scale factors sx and sy

最后是缩放。我们设两个缩放因子分别为 sx 和 sy。

And we build a matrix like this

然后我们构造如下的矩阵：

<div class="glocal-center">
  <table class="glocal-center-content glocal-mat">
    <tr><td>sx</td><td>0.0</td><td>0.0</td></tr>
    <tr><td>0.0</td><td>sy</td><td>0.0</td></tr>
    <tr><td>0.0</td><td>0.0</td><td>1.0</td></tr>
  </table>
</div>

Applying the matrix we get this

对这个矩阵执行我们上面提到的操作：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">sx</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td class="glocal-left">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td class="glocal-left">&nbsp;+</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">sy</td><td class="glocal-left">&nbsp;+&nbsp;</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">0.0</td><td>&nbsp;+</td></tr>
    <tr><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>0.0</td><td>&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>0.0</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td>&nbsp;*&nbsp;</td><td>1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

which is really

实际上只需要计算：

<div class="glocal-center">
  <table class="glocal-center-content">
    <col/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/><col/><col class="glocal-sp"/><col/><col/><col class="glocal-b"/>
    <tr><td>newX&nbsp;=&nbsp;</td><td>x</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">sx</td><td class="glocal-left glocal-blk">&nbsp;+</td><td class="glocal-right">newY&nbsp;=&nbsp;</td><td class="glocal-blk">x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-left glocal-blk">&nbsp;+</td><td class="glocal-right">extra&nbsp;=&nbsp;</td><td class="glocal-blk">x</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td class="glocal-blk">y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-left glocal-blk">&nbsp;+</td><td></td><td>y</td><td>&nbsp;*&nbsp;</td><td class="glocal-border">sy</td><td class="glocal-left glocal-blk">&nbsp;+&nbsp;</td><td></td><td class="glocal-blk">y</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk glocal-border">0.0</td><td class="glocal-blk">&nbsp;+</td></tr>
    <tr><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">0.0</td><td>&nbsp;</td><td></td><td class="glocal-blk">1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">0.0</td><td>&nbsp;&nbsp;</td><td></td><td>1</td><td class="glocal-blk">&nbsp;*&nbsp;</td><td class="glocal-blk">1.0</td><td>&nbsp;</td></tr>
  </table>
</div>

which simplified is

简化之后是：

```
newX = x * sx;
newY = y * sy;
```

Which is the same as our <a href="webgl-2d-scale.html">scaling sample</a>.

结果和我们的 <a href="./WebGL-2D-Scale.html" target="_blank">缩放例子</a> 一模一样。

Now I'm sure you might still be thinking "So what? What's the point?" That seems like a lot of work just to do the same thing we were already doing.

现在我坚信你大概还在想 “这又怎样？意义在哪里？”。那看起来很像我们之前已经做过的很多事情。

This is where the magic comes in. It turns out we can multiply matrices together and apply all the transformations at once. Let's assume we have a function, `matrixMultiply`, that takes two matrices, multiplies them and returns the result.

接下来就是见证奇迹的时刻。数学上已经证明了我们可以将多个矩阵一起相乘并且一次性执行所有的变换。假设我们有一个函数，`matrixMultiply`，它将两个矩阵作为参数，将它们相乘并将结果作为返回值。

To make things clearer let's make functions to build matrices for translation, rotation and scale.

为了让事情变得更加简单，我们定义一些函数来构建用于平移、旋转和缩放的矩阵。

```
  function makeTranslation(tx, ty) {
    return [
      1, 0, 0,
      0, 1, 0,
      tx, ty, 1
    ];
  }

  function makeRotation(angleInRadians) {
    var c = Math.cos(angleInRadians);
    var s = Math.sin(angleInRadians);
    return [
      c,-s, 0,
      s, c, 0,
      0, 0, 1
    ];
  }

  function makeScale(sx, sy) {
    return [
      sx, 0, 0,
      0, sy, 0,
      0, 0, 1
    ];
  }
```

Now let's change our shader. The old shader looked like this

现在我们来修改一下我们的着色器。之前的着色器看起来像这样：

```
  <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;

    uniform vec2 u_resolution;
    uniform vec2 u_translation;
    uniform vec2 u_rotation;
    uniform vec2 u_scale;

    void main() {
      // Scale the positon
      vec2 scaledPosition = a_position * u_scale;

      // Rotate the position
      vec2 rotatedPosition = vec2(
          scaledPosition.x * u_rotation.y + scaledPosition.y * u_rotation.x,
          scaledPosition.y * u_rotation.y - scaledPosition.x * u_rotation.x);

      // Add in the translation.
      vec2 position = rotatedPosition + u_translation;
      ...
    }
  </script>
```

Our new shader will be much simpler.

现在我们新的着色器会简单很多。

```
  <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;

    uniform vec2 u_resolution;
    uniform mat3 u_matrix;

    void main() {
      // Multiply the position by the matrix.
      vec2 position = (u_matrix * vec3(a_position, 1)).xy;
      ...
    }
  </script>
```

And here's how we use it

下面是我们如何使用新的着色器：

```
  // Draw the scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Compute the matrices
    var translationMatrix = makeTranslation(translation[0], translation[1]);
    var rotationMatrix = makeRotation(angleInRadians);
    var scaleMatrix = makeScale(scale[0], scale[1]);

    // Multiply the matrices.
    var matrix = matrixMultiply(scaleMatrix, rotationMatrix);
    matrix = matrixMultiply(matrix, translationMatrix);

    // Set the matrix.
    gl.uniformMatrix3fv(matrixLocation, false, matrix);

    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 18);
  }
```

Here's a sample using our new code. The sliders are the same, translation, rotation and scale. But the way they get used in the shader is much simpler.

下面是使用我们的新代码的例子。平移、旋转、缩放的滑动器还是和之前的一样。但它们被用于着色器中过程变得更加简单。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform.html" target="_blank">点击这里在新窗口查看例子</a>

Still, you might be asking, so what? That doesn't seem like much of a benefit. But, now if we want to change the order we don't have to write a new shader. We can just change the math.

这时候你可能还会问，所以呢？那看起来好像没有太多方便的地方。但是，如果我们现在想要改变变换的执行顺序，我们就不需要去写一个新的着色器，我们只要在计算过程中改变就可以了。

```
  ...
  // Multiply the matrices.
  var matrix = matrixMultiply(translationMatrix, rotationMatrix);
  matrix = matrixMultiply(matrix, scaleMatrix);
  ...
```

Here's that version.

下面是新版本的示例。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-trs.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-trs.html" target="_blank">点击这里在新窗口查看例子</a>

Being able to apply matrices like this is especially important for hierarchical animation like arms on a body, moons on a planet around a sun, or branches on a tree. For a simple example of hierarchical animation lets draw draw our 'F' 5 times but each time lets start with the matrix from the previous 'F'.

能够这样来执行矩阵变换对层阶式动画来说尤其重要，如身体上的手臂运动，月球在绕着太阳旋转的地球上的公转，或者树枝在树上。举一个简单的层阶式动画的例子，我们绘制 5 次 “F”，但每一次绘制都从上一个 “F” 使用的变换矩阵开始进行变换。

```
  // Draw the scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Compute the matrices
    var translationMatrix = makeTranslation(translation[0], translation[1]);
    var rotationMatrix = makeRotation(angleInRadians);
    var scaleMatrix = makeScale(scale[0], scale[1]);

    // Starting Matrix.
    var matrix = makeIdentity();

    for (var i = 0; i < 5; ++i) {
      // Multiply the matrices.
      matrix = matrixMultiply(matrix, scaleMatrix);
      matrix = matrixMultiply(matrix, rotationMatrix);
      matrix = matrixMultiply(matrix, translationMatrix);

      // Set the matrix.
      gl.uniformMatrix3fv(matrixLocation, false, matrix);

      // Draw the geometry.
      gl.drawArrays(gl.TRIANGLES, 0, 18);
    }
  }
```

To do this we introduced the function, `makeIdentity`, that makes an identity matrix. An identity matrix is a matrix that effectively represents 1.0 so that if you multiply by the identity nothing happens. Just like

为了完成这个事情，我们引入 `makeIdentity` 函数，这个函数返回一个单位矩阵。单位矩阵是一个实际上表示 1.0 的矩阵，如果你用一个矩阵乘以单位矩阵，不会发生任何事情。就像：

```
  X * 1 = X
```

so too

同样的：

```
  matrixX * identity = matrixX
```

Here's the code to make an identity matrix.

下面是构造出单位矩阵的代码。

```
  function makeIdentity() {
    return [
      1, 0, 0,
      0, 1, 0,
      0, 0, 1
    ];
  }
```

Here's the 5 Fs.

5 个 “F” 的例子如下：

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-hierarchical.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-hierarchical.html" target="_blank">点击这里在新窗口查看例子</a>

Let's see one more example. In every sample so far our 'F' rotates around its top left corner. This is because the math we are using always rotates around the origin and the top left corner of our 'F' is at the origin, (0, 0).

让我们再看一个例子。在之前我们所有的例子中， “F” 的旋转都是绕着它的左上角进行。这是因为在计算中我们都是选择绕着原点旋转而 “F” 的原点就是它的左上角 （0，0）。

But now, because we can do matrix math and we can choose the order that transforms are applied we can move the origin before the rest of the transforms are applied.

但现在，因为我们现在可以使用矩阵计算并且我们可以选择图形变换的顺序，我们可以在执行其他的变换之前移动原点的位置。

```
  // make a matrix that will move the origin of the 'F' to its center.
  var moveOriginMatrix = makeTranslation(-50, -75);
  ...

  // Multiply the matrices.
  var matrix = matrixMultiply(moveOriginMatrix, scaleMatrix);
  matrix = matrixMultiply(matrix, rotationMatrix);
  matrix = matrixMultiply(matrix, translationMatrix);
```

Here's that sample. Notice the F rotates and scales around the center.

下面就是上诉代码的例子。注意 “F” 的旋转和缩放都是围绕它的中心进行。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-center-f.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-center-f.html" target="_blank">点击这里在新窗口查看例子</a>

Using that technique you can rotate or scale from any point. Now you know how Photoshop or Flash let you move the rotation point.

使用这个技术你可以从任意点进行旋转或缩放。现在你知道 Photoshop 或者 Flash 是如何让你实现绕某点进行旋转的了。

Let's go even more crazy. If you go back to the first article on <a href="webgl-fundamentals.html">WebGL fundamentals</a> you might remember we have code in the shader to convert from pixels to clipspace that looks like this.

让我们进一步去深入研究。如果你回去看一下第一篇教程 <a href="../fundamentals/WebGL-Fundamentals.html" target="_blank">WebGL 基础</a> 你大概还记得我们在着色器中编写了从像素空间转换到裁剪空间的代码，如下：

```
  ...
  // convert the rectangle from pixels to 0.0 to 1.0
  // 将矩形的像素区间变换成 0.0 到 1.0
  vec2 zeroToOne = position / u_resolution;

  // convert from 0->1 to 0->2
  // 0->1 区间变换成 0->2区间
  vec2 zeroToTwo = zeroToOne * 2.0;

  // convert from 0->2 to -1->+1 (clipspace)
  // 0->2 区间变换成 -1->+1 （裁剪空间）
  vec2 clipSpace = zeroToTwo - 1.0;

  gl_Position = vec4(clipSpace * vec2(1, -1), 0, 1);
```

If you look at each of those steps in turn, the first step, "convert from pixels to 0.0 to 1.0", is really a scale operation. The second is also a scale operation. The next is a translation and the very last scales Y by -1. We can actually do that all in the matrix we pass into the shader. We could make 2 scale matrices, one to scale by 1.0/resolution, another to scale by 2.0, a 3rd to translate by -1.0,-1.0 and a 4th to scale Y by -1 then multiply them all together but instead, because the math is simple, we'll just make a function that makes a 'projection' matrix for a given resolution directly.

如果你依次看这些转换步骤中的每一步，第一步，“将像素区间变换到从 0.0 到 1.0”，实际上是一个缩放操作。第二步同样也是一个缩放操作。下一步是一个平移操作，并且最后的 Y 轴方向的缩放因子是 -1。我们实际上可以通过给着色器传递一个矩阵参数执行上面所有的操作。我们可以构建两个缩放矩阵，一个缩放因子为 1.0/分辨率，另一个缩放因子为 2.0，以及第三个矩阵为平移矩阵，平移因子为 -1.0 和 -1.0，第四个矩阵是 Y 轴方向的缩放因子为 -1 的缩放矩阵，最后将这四个矩阵相乘，但是，因为数学计算是简单的，我们只需要编写一个函数，通过给定的分辨率直接计算出相应的 “投影” 矩阵。

```
  function make2DProjection(width, height) {
    // Note: This matrix flips the Y axis so that 0 is at the top.
    return [
      2 / width, 0, 0,
      0, -2 / height, 0,
      -1, 1, 1
    ];
  }
```

Now we can simplify the shader even more. Here's the entire new vertex shader.

现在我们可以更进一步简化着色器。下面是整个新的顶点着色器。

```
  <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;

    uniform mat3 u_matrix;

    void main() {
      // Multiply the position by the matrix.
      gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);
    }
  </script>
```

And in JavaScript we need to multiply by the projection matrix

然后在 JavaScript 代码中我们需要与投影矩阵相乘。

```
  // Draw the scene.
  function drawScene() {
    ...
    // Compute the matrices
    var projectionMatrix = make2DProjection(
        canvas.clientWidth, canvas.clientHeight);
    ...

    // Multiply the matrices.
    var matrix = matrixMultiply(scaleMatrix, rotationMatrix);
    matrix = matrixMultiply(matrix, translationMatrix);
    matrix = matrixMultiply(matrix, projectionMatrix);
    ...
  }
```

We also removed the code that set the resolution. With this last step we've gone from a rather complicated shader with 6-7 steps to a very simple shader with only 1 step all due to the magic of matrix math.

我们也移除了设置分辨率的代码。全靠矩阵数学的魔力，原先我们要花 6-7 步操作的复杂的着色器变成了现在只需要 1 步就能完成操作的简单的着色器。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-with-projection.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-matrix-transform-with-projection.html" target="_blank">点击这里在新窗口查看例子</a>

I hope these posts have helped demystify matrix math. If you really want to become an expert
in matrix math [check out this amazing videos](https://www.youtube.com/watch?v=kjBOesZCoqc&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab).

我希望这些教程可以让矩阵数学不再那么神秘。如果你想成为矩阵数学的专家，你可以去看一下这个 [有趣的视频](https://www.youtube.com/watch?v=kjBOesZCoqc&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)。

<a href="webgl-3d-orthographic.html">I'll move on to 3D next</a>. In 3D matrix math follows the same principles and usage. I started with 2D to hopefully keep it simple to understand.

<a href="#">下一篇教程我们开始进入 3D 的内容</a>。在 3D 中，矩阵数学也遵从同样的原理和一样的用途。我从 2D 开始讲起就是希望能让大家更容易去理解这些内容。

> ### What are `clientWidth` and `clientHeight`?
>
> Up until this point whenever I referred to the canvas's dimensions I used `canvas.width` and `canvas.height` but above when I called `make2DProjection` I instead used `canvas.clientWidth` and `canvas.clientHeight`. Why?
>
> Projection matrixes are concerned with how to take clipspace (-1 to +1 in each dimension) and convert it back to pixels. But, in the browser, there are 2 types of pixels we are dealing with. One is the number of pixels in the canvas itself. So for example a canvas defined like this.
>
> ```
> <canvas width="400" height="300"></canvas>
> ```
>
> or one defined like this
>
> ```
> var canvas = document.createElement("canvas");
> canvas.width = 400;
> canvas.height = 300;
> ```
>
> both contain an image 400 pixels wide by 300 pixels tall. But, that size is separate from what size the browser actually displays that 400x300 pixel canvas. CSS defines what size the canvas is displayed. For example if we made a canvas like this.
>
> ```
> <style>
> canvas {
>   width: 100%;
>   height: 100%;
> }
> </style>
> ...
> <canvas width="400" height="300"></canvas>
> ```
>
> The canvas will be displayed whatever size its container is. That's likely not 400x300.
>
> Here are two examples that set the canvas's CSS display size to 100% so the canvas is stretched out to fill the page. The first one uses `canvas.width` and `canvas.height`. Open it in a new window and resize the window. Notice how the 'F' doesn't have the correct aspect. It gets distorted.
>
> <iframe src="http://webglfundamentals.org/webgl/webgl-canvas-width-height.html" width="500" height="150"></iframe>
>
> <a href="http://webglfundamentals.org/webgl/webgl-canvas-width-height.html" target="_blank">点击这里在新窗口查看例子</a>
>
> In this second example we use `canvas.clientWidth` and `canvas.clientHeight`. `canvas.clientWidth` and `canvas.clientHeight` report the size the canvas is actually being displayed by the browser so in this case, even though the canvas still only has 400x300 pixels since we're defining our aspect ratio based on the size the canvas is being displayed the F always looks correct.
>
> <iframe src="http://webglfundamentals.org/webgl/webgl-canvas-clientwidth-clientheight.html" width="500" height="150"></iframe>
>
> <a href="http://webglfundamentals.org/webgl/webgl-canvas-clientwidth-clientheight.html" target="_blank">点击这里在新窗口查看例子</a>
>
> Most apps that allow their canvases to be resized try to make the `canvas.width` and `canvas.height` match the `canvas.clientWidth` and `canvas.clientHeight` because they want there to be one pixel in the canvas for each pixel displayed by the browser. But, as we've seen above, that's not the only option. That means, in almost all cases, it's more technically correct to compute a projection matrix's aspect ratio using `canvas.clientHeight` and `canvas.clientWidth`.
>
> ### 什么是 `clientWidth` 和 `clientHeight`？
>
> 在这之前，每当我引用 canvas 的空间尺寸的时候我都调用 `canvas.width` 和 `canvas.height`，但在上文中我调用 `make2DProjection` 函数的时候我使用的却是 `canvas.clientWidth` 和 `canvas.clientHeight`，这是为什么呢？ 
>
> 投影矩阵专注于如何获取一个裁剪空间（在每个维度上都是 -1 到 +1）以及如何将其转换回到像素空间。但是在浏览器中，我们需要处理两种像素类型。一种是 canvas 自身的像素大小。举个例子，一个 canvas 的定义如下：
>
> ```
> <canvas width="400" height="300"></canvas>
> ```
>
> 或者像下面那样:
>
> ```
> var canvas = document.createElement("canvas");
> canvas.width = 400;
> canvas.height = 300;
> ```
>
> 两种方法都创建了一个 400 像素宽、300 像素高的图像。但是这个尺寸跟浏览器实际上如何现实 400 x 300 像素大小的 canvas 元素是独立开来的。CSS 定义 canvas 显示的尺寸。举个例子，如果我们像下面那样创建一个 canvas：
>
> ```
> <style>
> canvas {
>   width: 100%;
>   height: 100%;
> }
> </style>
> ...
> <canvas width="400" height="300"></canvas>
> ```
>
> 这个 canvas 元素无论怎么样都会显示出来。但它看起来不像是 400 x 300 的大小。
>
> 下面有两个设置 canvas 的 CSS 显示属性（宽和高）的值为 100% 使得 canvas 自动拉伸去填满页面的例子。第一个例子使用的是 `canvas.width` 和 `canvas.height`。在一个新窗口打开这个例子然后再调节窗口的大小，会看到 “F” 是如何地没有保持正常的显示。它变得不正常。
>
> <iframe src="http://webglfundamentals.org/webgl/webgl-canvas-width-height.html" width="500" height="150"></iframe>
>
> <a href="http://webglfundamentals.org/webgl/webgl-canvas-width-height.html" target="_blank">点击这里在新窗口查看例子</a>
>
> 在第二个例子中我们使用的是 `canvas.clientWidth` 和 `canvas.clientHeight`。`canvas.clientWidth` 和 `canvas.clientHeight` 得到的是 canvas 实际上在浏览器中显示的大小，所以，在这个例子中，尽管这个 canvas 依然还是只有 400 x 300 像素，但我们已经定义了我们基于 canvas 大小的长宽比，“F” 在 canvas 中的显示看起来总是正确的。
>
> <iframe src="http://webglfundamentals.org/webgl/webgl-canvas-clientwidth-clientheight.html" width="500" height="150"></iframe>
>
> <a href="http://webglfundamentals.org/webgl/webgl-canvas-clientwidth-clientheight.html" target="_blank">点击这里在新窗口查看例子</a>
>
> 大多数应用都允许它们的 canvas 可以被调整大小并尝试让 `canvas.width` 跟 `canvas.height` 能和 `canvas.clientWidth` 跟 `canvas.clientHeight` 一致，因为他们想使得 canvas 中显示的一个像素和在浏览器中显示的一个像素一样。但如我们上文所说，这个并不是唯一的解决方案。这表示，在大多数情况下，使用 `canvas.clientWidth` 和 `canvas.clientHeight` 来计算投影矩阵的长宽比从技术上来讲是正确的。
