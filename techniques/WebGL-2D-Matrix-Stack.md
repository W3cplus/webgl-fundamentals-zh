# WebGL 2D - Matrix Stack

# 使用 WebGL 实现矩阵栈


This article is a continuation of WebGL 2D DrawImage. If you haven't read that I suggest you start there.

上一篇文章讲解了 [WebGL 2D DrawImage][1]。如果你还没阅读它，我建议你先去查阅一下。

In that last article we implemented the WebGL equivilent of Canvas 2D's drawImage function including its ability to specify both the source rectangle and the destination rectangle.

在上一篇文章中，我们使用 WebGL 去实现了 Canvas 2D 的 `drawImage` 方法。并且，可以去设置截取图片的大小和显示图片的大小。

What we haven't done yet is let us rotate and/or scale it from any arbitrary point. We could do that by adding more arguments, at a minimum we'd to specify a center point, a rotation and an x and y scale. Fortunately there's a more generic and useful way. The way the Canvas 2D API does that is with a matrix stack. The matrix stack functions of the Canvas 2D API are save, restore, translate, rotate, and scale.

我们还没有完成的，就是将图片按照任意一个点进行旋转，缩放。当然，我们可以添加更多的参数来实现这样的功能，比如，至少需要一个旋转中心点，旋转角度和 x，y 的缩放比。不过，幸运的是，这有一个更普遍，有用的方式。Canvas 2D API 使用的是一个矩阵栈，来实现上述功能。该矩阵栈具有的基本函数是 `save`，`restore`，`translate`，`rotate` 和 `scale`。

A matrix stack is pretty simple to implement. We make a stack of matrices. We make functions to multiply the top matrix of the stack using by either a translation, rotation, or scale matrix using the functions we created eariler.

实现一个矩阵栈并不是很难。我们可以创建一个栈用来存放矩阵，使用的时候，就用函数去和栈顶的矩阵相乘。平移，旋转或缩放矩阵就是我们之前创建的。

Here's the implementation

下面是实现细节。

First the constructor and the save and restore functions

首先，有构造函数，`save` 函数，和 `restore` 函数。

```
function MatrixStack() {
  this.stack = [];
 
  // 因为该栈是空的，所以需要设置一个初始的栈在里面
  this.restore();
}
 
// 弹出栈顶的元素，用来恢复到之前保存的矩阵
MatrixStack.prototype.restore = function() {
  this.stack.pop();
  // 不能让栈为空
  if (this.stack.length < 1) {
    this.stack[0] = m4.identity();
  }
};
 
// 放置一个当前矩阵的副本到栈里
MatrixStack.prototype.save = function() {
  this.stack.push(this.getCurrentMatrix());
};
```
We also need functions for getting and setting the top matrix

我们需要函数去获取和设置顶部的矩阵
```
// 得到当前栈顶矩阵的副本
MatrixStack.prototype.getCurrentMatrix = function() {
  return this.stack[this.stack.length - 1].slice();
};
 
// 让我们设置当前矩阵
MatrixStack.prototype.setCurrentMatrix = function(m) {
  return this.stack[this.stack.length - 1] = m;
};
```
Finally we need to implement translate, rotate, and scale using our previous matrix functions.

最后，我们需要使用之前的矩阵函数去实现 `translate`，`rotate` 和 `scale` 函数。

```
// 平移矩阵
MatrixStack.prototype.translate = function(x, y, z) {
  var m = this.getCurrentMatrix();
  this.setCurrentMatrix(m4.translate(m, x, y, z));
};
 
// 绕 Z 轴旋转当前矩阵
MatrixStack.prototype.rotateZ = function(angleInRadians) {
  var m = this.getCurrentMatrix();
  this.setCurrentMatrix(m4.zRotate(m, angleInRadians));
};
 
// 缩放矩阵
MatrixStack.prototype.scale = function(x, y, z) {
  var m = this.getCurrentMatrix();
  this.setCurrentMatrix(m4.scale(m, x, y, z));
};
```
Note we're using the 3d matrix math functions. We could just use 0 for z on translation and 1 for z on scale but I find that I'm so used to using the 2d functions from Canvas 2d that I often forget to specify z an then the code breaks so let's make z optional

这里需要提醒一点，我们使用的是 3D 数学矩阵函数，可以使用 0 来作为 Z 轴的默认平移值，1 作为 Z 轴的默认缩放值。但是，我发现，我已经习惯使用 Canvas 2D 的平面函数，导致经常忘了设置 Z 的参数，所以，为了防止运行错误，我们可以让 Z 轴参数变为可选参数。

```
// 平移矩阵
MatrixStack.prototype.translate = function(x, y, z) {
  if (z === undefined) {
    z = 0;
  }
  var m = this.getCurrentMatrix();
  this.setCurrentMatrix(m4.translate(m, x, y, z));
};
 
...
 
// 缩放矩阵
MatrixStack.prototype.scale = function(x, y, z) {
  if (z === undefined) {
    z = 1;
  }
  var m = this.getCurrentMatrix();
  this.setCurrentMatrix(m4.scale(m, x, y, z));
};
```
Using our drawImage from the previous lesson we had these lines

从[前一篇教程][2]中的 `drawImage`，我们可以找到下列代码
```
// 该矩阵会从像素值转换到裁剪空间
var matrix = m4.orthographic(0, gl.canvas.width, gl.canvas.height, 0, -1, 1);
 
// 该矩阵会移动我们的单元格到 dstX，dstY
matrix = m4.translate(matrix, dstX, dstY, 0);
 
// 该矩阵会缩放我们 1 个单位的单元格
// 1 个单位代表着 texWidth，texHeight 单位
matrix = m4.scale(matrix, dstWidth, dstHeight, 1);
```
We just need to create a matrix stack

我们需要创建一个矩阵栈

```
var matrixStack = new MatrixStack();
```
and multiply in the top matrix from our stack in

然后，乘以栈顶的矩阵
```
// 该矩阵会从像素值转换到裁剪空间
var matrix = m4.orthographic(0, gl.canvas.width, gl.canvas.height, 0, -1, 1);
 
// 该矩阵会将原点移动到另外一个当前矩阵栈
matrix = m4.multiply(matrix, matrixStack.getCurrentMatrix());
 
// 该矩阵会移动我们的单元格到 dstX，dstY
matrix = m4.translate(matrix, dstX, dstY, 0);
 
// 该矩阵会缩放 1 个单位的单元格
// 1 个单位代表着 texWidth，texHeight 单位
matrix = m4.scale(matrix, dstWidth, dstHeight, 1);
```
And now we can use it the same way we'd use it with the Canvas 2D API.

现在，我们可以像 Canvas 2D API 一样去使用它了。

If you're not aware of how to use the matrix stack you can think of it as moving and orientating the origin. So for example by default in a 2D canvas the origin (0,0) is at the top left corner.

如果你不知道怎样去使用矩阵栈，你可以认为它就是移动和旋转原点。例如，在默认情况下，canvas 2D 的原点是 (0,0) 在左上角。

For example if we move the origin to the center of the canvas then drawing an image at 0,0 will draw it starting at the center of the canvas

假设，我们现在移动该原点到 canvas 的中心，然后在 0，0 位置绘制图片，那么该图片会从 canvas 的中心开始绘制。

Let's take our previous example and just draw a single image

我们用[前一篇文章][3]的例子，然后画一张图。
```
var textureInfo = loadImageAndCreateTextureInfo('resources/star.jpg');
 
function draw(time) {
  gl.clear(gl.COLOR_BUFFER_BIT);
 
  matrixStack.save();
  matrixStack.translate(gl.canvas.width / 2, gl.canvas.height / 2);
  matrixStack.rotateZ(time);
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
  matrixStack.restore();
}
```
And here it is.

结果是：

![定点旋转][4]


[查看网页][5]

you can see even though we're passing 0, 0 to drawImage since we use matrixStack.translate to move the origin to the center of the canvas the image is drawn and rotates around that center.

你可以看见，即使我们传递了 0，0 给 `drawImage` 函数，但后面使用了 `matrixStack.translate` 去移动原点到 canvas 的中心，所以，图片都会在中心进行绘制和旋转。

Let's move the center of rotation to center of the image

现在，让我们将旋转中心移动到图片的中心去。

```
matrixStack.translate(gl.canvas.width / 2, gl.canvas.height / 2);
matrixStack.rotateZ(time);
matrixStack.translate(textureInfo.width / -2, textureInfo.height / -2);
```
And now it rotates around the center of the image in the center of the canvas

现在，图片在 canvas 中心绕着其自身的中心旋转。

![中心旋转][6]

[查看网页][7]
Let's draw the same image at each corner rotating on different corners

接下来，让我们在图片的四个角绘制一张图片。

```
matrixStack.translate(gl.canvas.width / 2, gl.canvas.height / 2);
matrixStack.rotateZ(time);
 
matrixStack.save();
{
  matrixStack.translate(textureInfo.width / -2, textureInfo.height / -2);
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
}
matrixStack.restore();
 
matrixStack.save();
{
  // We're at the center of the center image so go to the top/left corner
  matrixStack.translate(textureInfo.width / -2, textureInfo.height / -2);
  matrixStack.rotateZ(Math.sin(time * 2.2));
  matrixStack.scale(0.2, 0.2);
  // Now we want the bottom/right corner of the image we're about to draw
  matrixStack.translate(-textureInfo.width, -textureInfo.height);
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
}
matrixStack.restore();
 
matrixStack.save();
{
  // We're at the center of the center image so go to the top/right corner
  matrixStack.translate(textureInfo.width / 2, textureInfo.height / -2);
  matrixStack.rotateZ(Math.sin(time * 2.3));
  matrixStack.scale(0.2, 0.2);
  // Now we want the bottom/right corner of the image we're about to draw
  matrixStack.translate(0, -textureInfo.height);
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
}
matrixStack.restore();
 
matrixStack.save();
{
  // We're at the center of the center image so go to the bottom/left corner
  matrixStack.translate(textureInfo.width / -2, textureInfo.height / 2);
  matrixStack.rotateZ(Math.sin(time * 2.4));
  matrixStack.scale(0.2, 0.2);
  // Now we want the top/right corner of the image we're about to draw
  matrixStack.translate(-textureInfo.width, 0);
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
}
matrixStack.restore();
 
matrixStack.save();
{
  // We're at the center of the center image so go to the bottom/right corner
  matrixStack.translate(textureInfo.width / 2, textureInfo.height / 2);
  matrixStack.rotateZ(Math.sin(time * 2.5));
  matrixStack.scale(0.2, 0.2);
  // Now we want the top/left corner of the image we're about to draw
  matrixStack.translate(0, 0);  // 0,0 means this line is not really doing anything
 
  drawImage(
    textureInfo.texture,
    textureInfo.width,
    textureInfo.height,
    0, 0);
 
}
matrixStack.restore();
```
And here's that

结果是：

![带子图片旋转][8]

[查看网页][9]

If you think of the various matrix stack functions, translate, rotateZ, and scale as moving the origin then the way I think of setting the center of rotation is where would I have to move the origin so that when I call drawImage a certain part of the image is at the previous origin?

如果你认为不同的矩阵栈函数，比如 `translate`，`rotateZ`，和 `scale` 是用来移动原点的，那么相似的，我也可以认为设置了旋转中心也是移动了原点，那么当我调用 `drawImage` 时，图片部分是相对于前面的原点吗？

In other words let's say on a 400x300 canvas I call matrixStack.translate(220, 150). At that point the origin is at 220, 150 and all drawing will be relative that point. If we call drawImage with 0, 0 this is where the image will be drawn.

话句话说，当我使用 400x300 的 canvas 画布，然后调用 `matrixStack.translate(220, 150)`。此时原点的位置在 200，150，并且接下来的都会相对于该点进行绘制。如果我们传入 0，0 参数去调用 `drawImage` ，这是图片相对于现在坐标的进行绘制。

![axis绘制坐标][10]

Lets say we want the center of rotation to be the bottom right. In that case where would be have to move the origin so that when we call drawImage the point we want to be the center of rotation is at the current origin? For the bottom right of the texture that would be -textureWidth, -textureHeight so now when we call drawImage with 0, 0 the texture would be drawn here and it's bottom right corner as at the previous origin.

现在，我们想让旋转中心变为右下角。在这种情况下，原点的位置应该移到哪以至于，当我们调用 `drawImage` 时，我们想要的旋转中心点就是当前原点呢？ 当前纹理的右下角的坐标是 `-textureWidth`，`-textureHeight`，所以，现在当我们调用 `drawImage`，使用 0，0参数时，当前纹理就会从这开始绘制并且该位置就是右下角，同样也是前一个坐标的原点。

![drawImage][11]

At any point whatever we did before that on the matrix stack it doesn't matter. We did a bunch of stuff to move or scale or rotate the origin but just before we call drawImage wherever the origin happens to be at the moment is irrelavent. It's the new origin so we just have to decide where to move that origin relative where the texture would be drawn if we had nothing before it on the stack.

无论我们对任意点做过怎样的处理，在被用于矩阵栈之前，这些处理不重要。我们已经对原点做了一系列操作，比如平移，缩放或者旋转，但在我们调用 `drawImage` 之前，原点现在的位置和处理操作是相关的。我们决定将当前原点移去哪，是相对于纹理绘制的位置，如果我们已经对原点的位置处理完了，那么最终的点就是新的起始点。

You might notice a matrix stack is very similar to a scene graph that we covered before. A scene graph had a tree of nodes and as we walked the tree we multiplied each node by its parent's node. A matrix stack is effectively antoher version that same process.

你可能注意到，该矩阵栈和我们[以前说的场景图表][12]很类似。一个场景图表包含一个节点树，我们可以遍历这颗树，然后用其父节点去和每个节点相乘。矩阵栈实际就是另外一种同样效果的高效替代版本


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-2d-drawimage.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-2d-drawimage.html
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-2d-drawimage.html
  [4]: http://static.zybuluo.com/jimmythr/oiqizkh79clclsepsprxcz00/%E5%AE%9A%E7%82%B9%E6%97%8B%E8%BD%AC.gif
  [5]: http://webglfundamentals.org/webgl/webgl-2d-matrixstack-01.html
  [6]: http://static.zybuluo.com/jimmythr/k01zrn7m9stbzo9b8ueu3ch4/%E4%B8%AD%E5%BF%83%E6%97%8B%E8%BD%AC.gif
  [7]: http://webglfundamentals.org/webgl/webgl-2d-matrixstack-02.html
  [8]: http://static.zybuluo.com/jimmythr/9kr0i9zjfxk1w1pobfusabk3/%E5%B8%A6%E5%AD%90%E5%9B%BE%E7%89%87%E6%97%8B%E8%BD%AC.gif
  [9]: http://webglfundamentals.org/webgl/webgl-2d-matrixstack-03.html
  [10]: http://static.zybuluo.com/jimmythr/fvaovg0dx1oejnwjtw08dk8j/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-21%2009.30.40.png
  [11]: http://static.zybuluo.com/jimmythr/y6vhf3046mor0u5njho230uh/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-21%2009.30.52.png
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-scene-graph.html