# WebGL 3D Perspective

# WebGL 3D 透视

This post is a continuation of a series of posts about WebGL. The first started with fundamentals and the previous was about 3D Basics. If you haven't read those please view them first.

在这里，我们将继续学习WebGL。本文假定你已经对[WebGL基础][1]，[3D基础][2]有所了解。如果你还未阅读过这些章节，请先移步到相应章节。

In the last post we went over how to do 3D but that 3D didn't have any perspective. It was using what's called an "orthographic" view which has its uses but it's generally not what people want when they say "3D".

上一篇文章中，我们学习了如何实现3D，不过该“3D”不具有任何透视效果（perspective）。这种“3D”使用的是“正交（orthographic）”视图，正交视图其实也有其用武之地，但并不是用来呈现通常意义的“3D”的。

Instead we need to add perspective. Just what is perspective? It's basically the feature that things that are further away appear smaller.

为了实现通常意义的“3D”，需要为其添加透视效果。那么问题来了，什么是透视？越远的物体看起来会越小，这种效果就是透视效果。
![透视示例][3]

Looking at the example above we see that things further away are drawn smaller. Given our current sample one easy way to make it so that things that are further away appear smaller would be to divide the clip space X and Y by Z.

从上面的示例中可以看到，那些越远的物体会被绘制得越小。为了在目前样例中实现“越远的物体越小”，我们可以粗暴地将裁剪空间中顶点的X，Y坐标分别除以Z坐标。

Think of it this way: If you have a line from (10, 15) to (20,15) it's 10 units long. In our current sample it would be drawn 10 pixels long. But if we divide by Z then for example if Z is 1

可以这样想：有一个从（10，15）到（20，15）长度为10单位的直线L。在目前的样例中，L的长度将会是10像素。如果除以值为1的Z坐标：
$$10 / 1 = 10$$
$$20 / 1 = 20$$
$$abs(10-20) = 10$$

it would be 10 pixels long, If Z is 2 it would be

L的长度会是10像素，如果除以值为2的Z坐标：
$$10 / 2 = 5$$
$$20 / 2 = 10$$
$$abs(5-10) = 5$$

5 pixels long. At Z = 3 it would be

L的长度会是5像素。如果$Z=3$：
$$10 / 3 = 3.333$$
$$20 / 3 = 6.666$$
$$abs(3.333-6.666) = 3.333$$

You can see that as Z increases, as it gets further away, we'll end up drawing it smaller. If we divide in clip space we might get better results because Z will be a smaller number (-1 to +1). If we add a fudgeFactor to multiply Z before we divide we can adjust how much smaller things get for a given distance.

可以看到，随着Z的增长，物体越来越远，我们会将其绘制得越来越小。如果将除法运算放在裁剪空间中进行，结果会更加尽如人意，因为的Z坐标会是一个比较小的数值（-1 ~ +1）。还可以添加一个*fudgeFactor*（附加系数），然后乘以Z，如此一来，在做除法运算之前，就可以根据给定的距离调整缩小程度。

Let's try it. First let's change the vertex shader to divide by Z after we've multiplied it by our "fudgeFactor".

来试一试吧。首先，修改顶点着色器，在Z坐标乘以“fudgeFactor”之后，顶点的X，Y坐标分别除以Z：
```html
<script id="2d-vertex-shader" type="x-shader/x-vertex">
...
uniform float u_fudgeFactor;
...
void main() {
  // Multiply the position by the matrix.
  vec4 position = u_matrix * a_position;
 
  // Adjust the z to divide by
  float zToDivideBy = 1.0 + position.z * u_fudgeFactor;
 
  // Divide x and y by z.
  gl_Position = vec4(position.xy / zToDivideBy, position.zw);
}
</script>
```

Note, because Z in clip space goes from -1 to +1 I added 1 to get zToDivideBy to go from 0 to +2 * fudgeFactor

注意，由于裁剪空间中的Z坐标范围为-1 ~ +1，我们需要在此基础上加1，使得*zToDivideBy*在0 ~ 2 * *fudgeFactor*之间。

We also need to update the code to let us set the fudgeFactor.

我们需要设置*fudgeFactor*，所以再次修改代码：
```js
...
  var fudgeLocation = gl.getUniformLocation(program, "u_fudgeFactor");
 
  ...
  var fudgeFactor = 1;
  ...
  function drawScene() {
    ...
    // Set the fudgeFactor
    gl.uniform1f(fudgeLocation, fudgeFactor);
 
    // Draw the geometry.
    gl.drawArrays(gl.TRIANGLES, 0, 16 * 6);
```

And here's the result.

得到结果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-perspective.html"></iframe>
[点击这里在新窗口中打开][4]

If it's not clear drag the "fudgeFactor" slider from 1.0 to 0.0 to see what things used to look like before we added our divide by Z code.

如果效果看起来不是很明显，你可以将“fudgeFactor”滑动条从1.0拖动至0.0来观察在添加除法因子之前的效果。
![正交3D vs 透视3D][5]

It turns out WebGL takes the x,y,z,w value we assign to gl_Position in our vertex shader and divides it by w automatically.

在顶点着色器中，``gl_Position``接收具有x，y，z，w值的4维向量，之后，WebGL会自动将w作为除法因子进行除法运算。

We can prove this very easily by changing the shader and instead of doing the division ourselves, put zToDivideBy in gl_Position.w.

不用手动进行除法运算，只需将``zToDivideBy``放在``gl_Position.w``的位置即可：
```html
<script id="2d-vertex-shader" type="x-shader/x-vertex">
...
uniform float u_fudgeFactor;
...
void main() {
  // Multiply the position by the matrix.
  vec4 position = u_matrix * a_position;
 
  // Adjust the z to divide by
  float zToDivideBy = 1.0 + position.z * u_fudgeFactor;
 
  // Divide x, y and z by zToDivideBy
  gl_Position = vec4(position.xyz,  zToDivideBy);
}
</script>
```

and see how it's exactly the same.
效果与之前一样：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-perspective-w.html"></iframe>
[点击这里在新窗口中打开][6]

Why is the fact that WebGL automatically divides by W useful? Because now, using more matrix magic, we can just use yet another matrix to copy z to w.

A Matrix like this

WebGL会自动将W作为除法因子进行除法运算，这项特性很有用，因为现在我们可以使用矩阵来表达最终的结果。创建一个矩阵，实现将z的值复制到w。

上述矩阵：
1, 0, 0, 0,
0, 1, 0, 0,
0, 0, 1, 1,
0, 0, 0, 0,

will copy z to w. You can look at each of those columns as

which when simplified is

该矩阵将z的值复制到w。可以将每列看作：
x_out = x_in * 1 +
        y_in * 0 +
        z_in * 0 +
        w_in * 0 ;

y_out = x_in * 0 +
        y_in * 1 +
        z_in * 0 +
        w_in * 0 ;

z_out = x_in * 0 +
        y_in * 0 +
        z_in * 1 +
        w_in * 0 ;

w_out = x_in * 0 +
        y_in * 0 +
        z_in * 1 +
        w_in * 0 ;
        
化简后得：
x_out = x_in;
y_out = y_in;
z_out = z_in;
w_out = z_in;

We can add the plus 1 we had before with this matrix since we know w_in is always 1.0.

由于w_in的初始值始终为1.0，所以我们需要修改矩阵右下角的值为1：
1, 0, 0, 0,
0, 1, 0, 0,
0, 0, 1, 1,
0, 0, 0, 1,

that will change the W calculation to

W的值将会变成：
w_out = x_in * 0 +
        y_in * 0 +
        z_in * 1 +
        w_in * 1 ;

and since we know w_in = 1.0 then that's really

由于w_in = 1.0，所以最终表达式为：
w_out = z_in + 1;

Finally we can work our fudgeFactor back in if the matrix is this

which means

最后将“fudgeFactor”带入到矩阵中：
1, 0, 0, 0,
0, 1, 0, 0,
0, 0, 1, fudgeFactor,
0, 0, 0, 1,

与其等价的表达式为：
w_out = x_in * 0 +
        y_in * 0 +
        z_in * fudgeFactor +
        w_in * 1 ;


and simplified that's

化简后可得：
w_out = z_in * fudgeFactor + 1;


So, let's modify the program again to just use matrices.

First let's put the vertex shader back. It's simple again

接下来，让我们尝试将上述矩阵应用到程序中。

首先，还原顶点着色器。代码又变得简洁了：
```html
<script id="2d-vertex-shader" type="x-shader/x-vertex">
uniform mat4 u_matrix;
 
void main() {
  // Multiply the position by the matrix.
  gl_Position = u_matrix * a_position;
  ...
}
</script>
```

Next let's make a function to make our Z → W matrix.

然后，创建一个用于生成Z → W矩阵的函数：
```js
function makeZToWMatrix(fudgeFactor) {
  return [
    1, 0, 0, 0,
    0, 1, 0, 0,
    0, 0, 1, fudgeFactor,
    0, 0, 0, 1,
  ];
}
```

and we'll change the code to use it.

在程序中应用该函数：
```js
...
    // Compute the matrices
    var zToWMatrix =
        makeZToWMatrix(fudgeFactor);
 
    ...
 
    // Multiply the matrices.
    var matrix = matrixMultiply(scaleMatrix, rotationZMatrix);
    matrix = matrixMultiply(matrix, rotationYMatrix);
    matrix = matrixMultiply(matrix, rotationXMatrix);
    matrix = matrixMultiply(matrix, translationMatrix);
    matrix = matrixMultiply(matrix, projectionMatrix);
    matrix = matrixMultiply(matrix, zToWMatrix);
 
    ...

```

and note, again, it's exactly the same.

效果与之前一样：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-perspective-w-matrix.html"></iframe>
[点击这里在新窗口中打开][7]


All that was basically just to show you that dividing by Z gives us perspective and that WebGL conveniently does this divide by Z for us.

到目前为止，本文基本上就是在告诉你两件事：1.将Z坐标相关值当作除法因子便可以实现透视效果，2.WebGL会自动为我们进行除法运算。

But there are still some problems. For example if you set Z to around -100 you'll see something like the animation below

不过，现在的程序还存在一些问题。如果Z的值被设置为-100左右，你将会看到像下面动画中出现的状况：
![意外状况][8]

What's going on? Why is the F disappearing early? Just like WebGL clips X and Y or +1 to -1 it also clips Z. What we're seeing here is where Z < -1.

发生了什么？“F”为什么会这么早消失？与X，Y坐标一样，WebGL同样会裁剪Z坐标。我们所看到的状况就是在裁剪空间内当Z < -1时的情况。

I could go into detail about the math to fix it but you can derive it the same way we did 2D projection. We need to take Z, add some amount and scale some amount and we can make any range we want get remapped to the -1 to +1.

可以通过数学方法修正这个问题，不过，也可以使用[类似在2D映射中的方式][9]去解决。通过对Z先后做加法和乘法运算，我们就可以将任何范围内的数值映射到-1~+1之间。

The cool thing is all of these steps can be done in 1 matrix. Even better, rather than a fudgeFactor we'll decide on a fieldOfView and compute the right values to make that happen.

上面的操作同样可以被合并到之前的矩阵中，是不是很酷？让我们来改进矩阵函数，使得其可以通过设置``fieldOfView``从而计算出``fudgeFactor``，而不是直接设置``fudgeFactor``。

下面是改进后的矩阵函数：
```js
function makePerspective(fieldOfViewInRadians, aspect, near, far) {
  var f = Math.tan(Math.PI * 0.5 - 0.5 * fieldOfViewInRadians);
  var rangeInv = 1.0 / (near - far);
 
  return [
    f / aspect, 0, 0, 0,
    0, f, 0, 0,
    0, 0, (near + far) * rangeInv, -1,
    0, 0, near * far * rangeInv * 2, 0
  ];
};
```

This matrix will do all our conversions for us. It will adjust the units so they are in clip space, it will do the math so that we can choose a field of view by angle and it will let us choose our Z-clipping space. It assumes there's an eye or camera at the origin (0, 0, 0) and given a zNear and a fieldOfView it computes what it would take so that stuff at zNear ends up at Z = -1 and stuff at zNear that is half of fieldOfView above or below the center ends up with Y = -1 and Y = 1 respectively. It computes what to use for X by just multiplying by the aspect passed in. We'd normally set this to the width / height of the display area. Finally, it figures out how much to scale things in Z so that stuff at zFar ends up at Z = 1.

Here's a diagram of the matrix in action.

这个函数生成的矩阵会为我们处理好一切事情。比如，将坐标映射到裁剪空间，做一些数学运算使得我们可以调整视口角度和Z轴方向的裁剪范围。该矩阵假定，眼睛（或者说相机）处于原点（0，0，0）处，``zNear``以Z=-1为界限，其Y轴方向上的长度为``fieldOfView``的一半并且其中心点位于Y轴范围（+1~-1）的中央。至于X轴方向的数据，会对参数`` aspect``进行乘法运算得出。我们通常会将``aspect``设置为展示区域的宽高比``width / height``。最后，它会算出Z轴方向上的缩放比例，使得zFar不超过``Z = 1``。

下面是上述矩阵的行为图解：
<iframe src="blob:http%3A//webglfundamentals.org/19783b84-9d71-4821-a83a-b68a310c6156"></iframe>
[点击这里在新窗口中打开][10]

That shape that looks like a 4 sided cone the cubes are spinning in is called a "frustum". The matrix takes the space inside the frustum and converts that to clip space. zNear defines where things will get clipped in the front and zFar defines where things get clipped in the back. Set zNear to 23 and you'll see the front of the spinning cubes get clipped. Set zFar to 24 and you'll see the back of the cubes get clipped.

这些立方体所处的4面锥体叫做“截头锥体”。上述矩阵会将截头锥体内的空间转换为裁剪空间，``zNear``之前和``zFar``之后的物体都会被裁剪。让我们试一试，将``zNear``设置为23，你会看到立方体靠前部分会被裁剪。如果将``zFar``设置为24，立方体靠后部分也会被裁剪。

There's just one problem left. This matrix assumes there's a viewer at 0,0,0 and it assumes it's looking in the negative Z direction and that positive Y is up. Our matrices up to this point have done things in a different way. To make this work we need to put our objects in front of the view.

到目前为止，我们的程序还剩下最后一个问题。上述矩阵其实假定观察者处于（0，0，0），视线的方向是Z轴负半轴方向，并且Y轴的负半轴是朝上的。上述假设是不是很奇怪。为了修正这个问题，我们需要将所有物体放置在视线前方。

We could do that by moving our F. We were drawing at (45, 150, 0). Let's move it to (-150, 0, -360)

也就是说，我们可以将绘制在（45，150，0）的“F”移动至（-150，0，-360）。

Now, to use it we just need to replace our old call to make2DProjection with a call to makePerspective

现在，只需要将原先的``make2DProjection``替换为``makePerspective``：
```js
var aspect = canvas.clientWidth / canvas.clientHeight;
    var projectionMatrix =
        makePerspective(fieldOfViewRadians, aspect, 1, 2000);
    var translationMatrix =
        makeTranslation(translation[0], translation[1], translation[2]);
    var rotationXMatrix = makeXRotation(rotation[0]);
    var rotationYMatrix = makeYRotation(rotation[1]);
    var rotationZMatrix = makeZRotation(rotation[2]);
    var scaleMatrix = makeScale(scale[0], scale[1], scale[2]);
```

And here it is.

终极效果：
<iframe class="webgl_example" style=" " src="http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-perspective-matrix.html"></iframe>
[点击这里在新窗口中打开][11]

We're back to just a matrix multiply and we're getting both a field of view and we're able to choose our Z space. We're not done but this article is getting too long. Next up, cameras.

一路下来，又回到了1个矩阵的时代，该矩阵同时具有调整Z轴方向的空间大小和视口区域的能力。不过，还没有结束，由于文章已经过长，剩下的知识会被放在下一章节中详细讨论。下一篇文章，我们将讨论[WebGL 3D 相机][12]




  
  


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-3d-orthographic.html
  [3]: http://oc3wui92y.bkt.clouddn.com/SVG/webgl/perspective-example.svg
  [4]: http://webglfundamentals.org/webgl/webgl-3d-perspective.html
  [5]: http://oc3wui92y.bkt.clouddn.com/image/webgl/orthographic-vs-perspective.png
  [6]: http://webglfundamentals.org/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-perspective-w.html
  [7]: http://webglfundamentals.org/webgl/webgl-3d-perspective-w-matrix.html
  [8]: http://oc3wui92y.bkt.clouddn.com/image/webgl/z-clipping.gif
  [9]: http://stackoverflow.com/a/28301213/128511
  [10]: http://webglfundamentals.org/webgl/frustum-diagram.html
  [11]: http://webglfundamentals.org/webgl/webgl-3d-perspective-matrix.html
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-3d-camera.html