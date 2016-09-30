# WebGL - Orthographic 3D

# WebGL - 正交3D

This post is a continuation of a series of posts about WebGL. The first [started with fundamentals][1] and the previous was [about 2D matrices][2]. If you haven't read those please view them first.

在这里，我们将继续学习WebGL。本文假定你已经对[WebGL基础][3]，[二维矩阵（2D matrices）][4]有所了解。如果你还未阅读过这些章节，请先移步到相应章节。

In the last post we went over how 2D matrices worked. We talked about how translation, rotation, scaling, and even projecting from pixels into clip space can all be done by 1 matrix and some magic matrix math. To do 3D is only a small step from there.

上一篇文章中，我们学习了二维矩阵（2D matrices）的工作方式。如平移（translation）、旋转（rotation）、缩放（scale）、像素空间到剪切空间的映射（projection from pixels into clip space），这些操作都能通过矩阵运算，然后用1个矩阵表示。现在，我们只需在此基础上稍加改动，便能实现3D效果。

In our previous 2D examples we had 2D points (x, y) that we multiplied by a 3x3 matrix. To do 3D we need 3D points (x, y, z) and a 4x4 matrix.

在之前2D的例子中，我们用二维坐标(x,y)乘以一个3X3矩阵 。为了实现3D效果，我们这次需要三维坐标(x,y,z)和一个4X4矩阵。

Let's take our last example and change it to 3D. We'll use an F again but this time a 3D 'F'.

The first thing we need to do is change the vertex shader to handle 3D. Here's the old shader.

And here's the new one

It got even simpler!

Then we need to provide 3D data.

在这里我们使用上一个例子，然后将其转换为3D。依然是一个“F”，但这次是3D的“F”。

首先，需要让顶点着色器（vertex shader）具有处理3D的能力。
旧版本顶点着色器：
```html
<script id="2d-vertex-shader" type="x-shader/x-vertex">
attribute vec2 a_position;
 
uniform mat3 u_matrix;
 
void main() {
  // Multiply the position by the matrix.
  gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);
}
</script>
```

新版本顶点着色器：
```html
<script id="3d-vertex-shader" type="x-shader/x-vertex">
attribute vec4 a_position;
 
uniform mat4 u_matrix;
 
void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;
}
</script>
```

甚至变得更加简洁了！

提供3D坐标数据：
```js
...
 
  gl.vertexAttribPointer(positionLocation, 3, gl.FLOAT, false, 0, 0);
 
  ...
 
// Fill the buffer with the values that define a letter 'F'.
function setGeometry(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
          // left column
            0,   0,  0,
           30,   0,  0,
            0, 150,  0,
            0, 150,  0,
           30,   0,  0,
           30, 150,  0,
 
          // top rung
           30,   0,  0,
          100,   0,  0,
           30,  30,  0,
           30,  30,  0,
          100,   0,  0,
          100,  30,  0,
 
          // middle rung
           30,  60,  0,
           67,  60,  0,
           30,  90,  0,
           30,  90,  0,
           67,  60,  0,
           67,  90,  0]),
      gl.STATIC_DRAW);
}
```

Next we need to change all the matrix functions from 2D to 3D

Here are the 2D (before) versions of makeTranslation, makeRotation and makeScale

接下来，需要将2D矩阵函数转换为3D矩阵函数

2D版本的``makeTranslation``、``makeRotation``、``makeScale``：
```js
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

And here are the updated 3D versions.

更新后的3D版本：
```js
function makeTranslation(tx, ty, tz) {
  return [
     1,  0,  0,  0,
     0,  1,  0,  0,
     0,  0,  1,  0,
     tx, ty, tz, 1
  ];
}
 
function makeXRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);
 
  return [
    1, 0, 0, 0,
    0, c, s, 0,
    0, -s, c, 0,
    0, 0, 0, 1
  ];
};
 
function makeYRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);
 
  return [
    c, 0, -s, 0,
    0, 1, 0, 0,
    s, 0, c, 0,
    0, 0, 0, 1
  ];
};
 
function makeZRotation(angleInRadians) {
  var c = Math.cos(angleInRadians);
  var s = Math.sin(angleInRadians);
 
  return [
     c, s, 0, 0,
    -s, c, 0, 0,
     0, 0, 1, 0,
     0, 0, 0, 1,
  ];
}
 
function makeScale(sx, sy, sz) {
  return [
    sx, 0,  0,  0,
    0, sy,  0,  0,
    0,  0, sz,  0,
    0,  0,  0,  1,
  ];
}
```

Notice we now have 3 rotation functions. We only needed one in 2D as we were effectively only rotating around the Z axis. Now though to do 3D we also want to be able to rotate around the X axis and Y axis as well. You can see from looking at them they are all very similar. If we were to work them out you'd see them simplify just like before

注意，在这里有3个旋转函数。由于在2D中，图形仅仅围绕Z轴旋转，所以只需要一个函数。而在3D中，还存在围绕X轴和Y轴旋转的情况。但是，通过观察不难发现，3个函数非常相似，而且可以用之前的方法很轻易地算出。

绕Z轴旋转：
$$newX = x * c + y * s;$$
$$newY = x * -s + y * c;$$

绕Y轴旋转：
$$newX = x * c + Z * s;$$
$$newZ = x * -s + Z * c;$$

绕X轴旋转：
$$newY = Y * c + Z * s;$$
$$newZ = Y * -s + Z * c;$$

三种旋转：
<iframe class="external_diagram" src="http://webglfundamentals.org/webgl/lessons/resources/axis-diagram.html" style="width: 540px; height: 240px;"></iframe>


We also need to update the projection function. Here's the old one

我们同样需要更新空间映射函数。
原来的版本：
```js
function make2DProjection(width, height) {
  // Note: This matrix flips the Y axis so 0 is at the top.
  return [
    2 / width, 0, 0,
    0, -2 / height, 0,
    -1, 1, 1
  ];
}
```

which converted from pixels to clip space. For our first attempt at expanding it to 3D let's try

上面的函数将坐标从像素空间映射到了裁剪空间。
我们尝试将它扩展为3D版本：
```js
function make2DProjection(width, height, depth) {
  // Note: This matrix flips the Y axis so 0 is at the top.
  return [
     2 / width, 0, 0, 0,
     0, -2 / height, 0, 0,
     0, 0, 2 / depth, 0,
    -1, 1, 0, 1,
  ];
}
```

Just like we needed to convert from pixels to clip space for X and Y, for Z we need to do the same thing. In this case I'm making the Z axis pixel units as well. I'll pass in some value similar to width for the depth so our space will be 0 to width pixels wide, 0 to height pixels tall, but for depth it will be -depth / 2 to +depth / 2.

同X，Y坐标一样，我们可以用相同的方法将Z坐标从像素空间映射到裁剪空间。与width(宽度)相似，可以将depth(深度)值传入函数，则像素空间的宽度范围为0~width，高度范围0~height，深度范围-depth/2 ~ +depth/2。

Finally we need to to update the code that computes the matrix.

最后，需要修改映射矩阵：
```js
// Compute the matrices
  var projectionMatrix =
      make2DProjection(canvas.clientWidth, canvas.clientHeight, 400);
  var translationMatrix =
      makeTranslation(translation[0], translation[1], translation[2]);
  var rotationXMatrix = makeXRotation(rotation[0]);
  var rotationYMatrix = makeYRotation(rotation[1]);
  var rotationZMatrix = makeZRotation(rotation[2]);
  var scaleMatrix = makeScale(scale[0], scale[1], scale[2]);

  // Multiply the matrices.
  var matrix = matrixMultiply(scaleMatrix, rotationZMatrix);
  matrix = matrixMultiply(matrix, rotationYMatrix);
  matrix = matrixMultiply(matrix, rotationXMatrix);
  matrix = matrixMultiply(matrix, translationMatrix);
  matrix = matrixMultiply(matrix, projectionMatrix);

  // Set the matrix.
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
```

这是样例：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step1.html"></iframe>
[点击这里在新窗口打开][5]

The first problem we have is that our geometry is a flat F which makes it hard to see any 3D. To fix that let's expand the geometry to 3D. Our current F is made of 3 rectangles, 2 triangles each. To make it 3D will require a total of 16 rectangles. That's quite a few to list out here. 16 rectangles with 2 triangles per rectangle and 3 vertices per triangle is 96 vertices. If you want to see all of them view the source of the sample.

We have to draw more vertices so

首要问题，由于图形是扁平的“F”，所以很难看到任何3D效果。为了解决这一问题，我们需要将图形转换为3D。现在的“F”由3个矩形组成，每个矩形包含2个三角形。为了实现3D的“F”，需要16个矩形，每个矩形包含2个三角形，每个三角形有3个顶点，共计96个顶点。如果你想查看所有的顶点，可以浏览样例的源码。

可以像这样绘制更多的顶点：
```js
    // Draw the geometry.
    gl.drawArrays(gl.TRIANGLES, 0, 16 * 6);
```

这里是上面的样例：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step2.html"></iframe>
[点击这里在新窗口中打开][6]

Moving the sliders it's pretty hard to tell that it's 3D. Let's try coloring each rectangle a different color. To do this we will add another attribute to our vertex shader and a varying to pass it from the vertex shader to the fragment shader.

Here's the new vertex shader

尽管能不断移动滑块，但还是很难分辨出这是3D图形，所以我们尝试给每个矩形添加不同颜色。为了达到这一目的，需要给顶点着色器添加另外的attribute，然后通过varing传递给片元着色器。

新的顶点着色器：
```html
<script id="3d-vertex-shader" type="x-shader/x-vertex">
attribute vec4 a_position;
attribute vec4 a_color;

uniform mat4 u_matrix;

varying vec4 v_color;

void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;

  // Pass the color to the fragment shader.
  v_color = a_color;
}
</script>
```

And we need to use that color in the fragment shader

然后在片元着色器（fragment shader）中使用varing：
```html
<script id="3d-vertex-shader" type="x-shader/x-fragment">
precision mediump float;

// Passed in from the vertex shader.
varying vec4 v_color;

void main() {
   gl_FragColor = v_color;
}
</script>
```

We need to lookup the location to supply the colors, then setup another buffer and attribute to give it the colors.

接下来，需要在相应位置应用颜色，所以再创建一个缓冲区（buffer）和attribute用来接收颜色：
```js
...
  var colorLocation = gl.getAttribLocation(program, "a_color");

  ...
  // Create a buffer for colors.
  var buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
  gl.enableVertexAttribArray(colorLocation);

  // We'll supply RGB as bytes.
  gl.vertexAttribPointer(colorLocation, 3, gl.UNSIGNED_BYTE, true, 0, 0);

  // Set Colors.
  setColors(gl);

  ...
// Fill the buffer with colors for the 'F'.

function setColors(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array([
          // left column front
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,
        200,  70, 120,

          // top rung front
        200,  70, 120,
        200,  70, 120,
        ...
        ...
      gl.STATIC_DRAW);
}
```

得到结果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step3.html"></iframe>
[点击这里在新窗口中打开][7]

Uh oh, what's that mess? Well, it turns out all the various parts of that 3D 'F', front, back, sides, etc get drawn in the order they appear in in our geometry. That doesn't give us quite the desired results as sometimes the ones in the back get drawn after the ones in the front.

呃。。结果看起来很糟糕。仿佛这个3D“F”的所有部分，如正面、背面、侧面、等等都按照在图形中出现的顺序被绘制了出来。有的时候图形背面后于正面被绘制，这并不是我们想要的结果。

Triangles in WebGL have the concept of front facing and back facing. A front facing triangle has its vertices go in a clockwise direction. A back facing triangle has its vertices go in a counter clockwise direction

在WebGL中三角形分为背面（Back Face）和正面（Front Face）。一个正面三角形，其顶点排列是顺时针方向（Clockwise）。背面三角形，其顶点排列则是逆时针方向（Counter Clockwise）。

![正反三角形SVG][8]

WebGL has the ability to draw only forward facing or back facing triangles. We can turn that feature on with

WebGL有能力选择只绘制正面三角形或者背面三角形。我们可以像下面这样开启这个功能：
```js
 gl.enable(gl.CULL_FACE);
```

which we do just once, right at the start of our program. With that feature turned on, WebGL defaults to "culling" back facing triangles. "Culling" in this case is a fancy word for "not drawing".

只需在程序开始的时候开启一次即可。该功能开启后，WebGL会默认对三角形进行“背面剔除（Culling）”，也就是剔除背面三角形。

Note that as far as WebGL is concerned, whether or not a triangle is considered to be going clockwise or counter clockwise depends on the vertices of that triangle in clip space. In other words, WebGL figures out whether a triangle is front or back AFTER you've applied math to the vertices in the vertex shader. That means for example a clockwise triangle that is scaled in X by -1 becomes a counter clockwise triangle or a clockwise triangle rotated 180 degrees becomes a couter clockwise triangle. Because we had CULL_FACE disabled we can see both clockwise(front) and counter clockwise(back) triangles. Now that we've turned it on, any time a front facing triangle flips around either because of scaling or rotation or for whatever reason, WebGL won't draw it. That's a good thing since as your turn something around in 3D you generally want whichever triangles are facing you to be considered front facing.

注意，在WebGL中，判断一个三角形是顺时针还是逆时针的依据是：该三角形在裁剪空间的顶点顺序。也就是说，WebGL会在对顶点着色器中的顶点应用数学运算后断定一个三角形的朝向是正面或是背面。比如，一个顶点顺序为顺时针的三角形，在X方向上应用值为-1的缩放，或者旋转180°后都会变成一个逆时针的三角形。因为**CULL_FACE**功能还未开启，所以我们可以同时看到顺时针（正面）三角形和逆时针（背面）三角形。该功能开启后，当一个正面三角形被应用缩放、旋转，或其它原因而左右翻转的时候，WebGL都不会绘制它。这是个不错的功能，因为当翻转3D图形的时候始终希望呈现面向我们的三角形。

With CULL_FACE turned on this is what we get

开启**CULL_FACE**后，得到下面的效果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step4.html"></iframe>
[点击这里在新窗口中打开][9]

Hey! Where did all the triangles go? It turns out, many of them are facing the wrong way. Rotate it and you'll see them appear when you look at the other side. Fortunately it's easy to fix. We just look at which ones are backward and exchange 2 of their vertices. For example if one backward triangle is

嘿！！这些三角形跑哪去了？很多三角形的朝向貌似都是错误的。旋转一下它们就会在另一面出现。还好，这个问题非常容易解决。我们需要找到那些应朝向正面的三角形，然后改变其中两个顶点的顺序即可。如果一个应朝向正面的三角形是下面这样：
```js
1,   2,   3,
40,  50,  60,
700, 800, 900,
```

we just flip the last 2 vertices to make it forward.

只需交换最后两个顶点的位置即可使其朝向正面：
```js
1,   2,   3,
700, 800, 900,
40,  50,  60,
```

Going through and fixing all the backward triangles gets us to this

在修正了所有应朝向正面的三角形顶点顺序后，我们得到这样的结果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step5.html"></iframe>
[点击这里在新窗口中打开][10]

That's closer but there's still one more problem. Even with all the triangles facing in the correct direction and with the back facing ones being culled we still have places where triangles that should be in the back are being drawn over triangles that should be in front.

这样的效果接近完美，但在这里还存在一个问题。尽管所有三角形的朝向正确，并且都被背面剔除了，还是有一些应处于后面的三角形被绘制在理应处于前面的三角形之上。

Enter the DEPTH BUFFER.

让我们来看一看深度缓冲区（DEPTH BUFFER）。

A depth buffer, sometimes called a Z-Buffer, is a rectangle of depth pixels, one depth pixel for each color pixel used to make the image. As WebGL draws each color pixel it can also draw a depth pixel. It does this based on the values we return from the vertex shader for Z. Just like we had to convert to clip space for X and Y, Z is also in clip space or (-1 to +1). That value is then converted into a depth space value (0 to +1). Before WebGL draws a color pixel it will check the corresponding depth pixel. If the depth value for the pixel it's about to draw is greater than the value of the corresponding depth pixel then WebGL does not draw the new color pixel. Otherwise it draws both the new color pixel with the color from your fragment shader AND it draws the depth pixel with the new depth value. This means, pixels that are behind other pixels won't get drawn.

深度缓冲区有时也被称作Z-Buffer，它由一连串的深度像素组成。每次的成像都会用到每个颜色像素对应的深度像素。WebGL绘制每个颜色像素的同时，都会绘制其深度像素。深度像素的绘制是基于顶点着色器中顶点的Z坐标。就像X坐标和Y坐标一样，Z坐标同样处于裁剪空间或者说在（-1 ~ +1）中。在此之后Z又会被转换为深度空间值（0 ~ +1）。WebGL在绘制颜色像素前，会检查该像素对应的深度像素。如果将要绘制的深度值大于当前像素的深度值，那么新的颜色像素将不会被绘制。否则，不仅会从片元着色器中取出颜色来绘制新的颜色像素，还会以新的深度值来绘制深度像素。也就是说，藏在其它像素之后的像素都不会被绘制。

We can turn on this feature nearly as simply as we turned on culling with

我们可以像开启“背面剔除”一样轻松的地开启这个功能：
```js
gl.enable(gl.DEPTH_TEST);
```

We also need to clear the depth buffer back to 1.0 before we start drawing.

在开始绘制图形前，还需要将深度缓冲区重置为1.0：
```js
// Draw the scene.
  function drawScene() {
    // Clear the canvas AND the depth buffer.
    gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
    ...
```

And now we get

现在我们得到下面的效果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-step6.html"></iframe>
[点击这里在新窗口中打开][11]

which is 3D!

这就是3D！！！

In the next post I'll go over how to make it have perspective.

下一篇文章，我们将讨论如何[添加透视效果][12]


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-2d-matrices.html
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [4]: http://webglfundamentals.org/webgl/lessons/webgl-2d-matrices.html
  [5]: http://webglfundamentals.org/webgl/webgl-3d-step1.html
  [6]: http://webglfundamentals.org/webgl/webgl-3d-step2.html
  [7]: http://webglfundamentals.org/webgl/webgl-3d-step3.html
  [8]: http://oc3wui92y.bkt.clouddn.com/SVG/webgl/triangle-winding.svg
  [9]: http://webglfundamentals.org/webgl/webgl-3d-step4.html
  [10]: http://webglfundamentals.org/webgl/webgl-3d-step5.html
  [11]: http://webglfundamentals.org/webgl/webgl-3d-step6.html
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html