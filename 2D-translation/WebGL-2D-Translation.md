# WebGL 2D 平移（Translation）

Before we move on to 3D let's stick with 2D for a little while longer. Bear with me please. This article might seem exceedingly obvious to some but I'll build up to a point in a few articles.

在开始进入 3D 之前我们先在 2D 停留一段时间。请各位耐心听我说，这篇文章或许对一些同学来说极其浅显，但我还是会用一些篇幅由点到面地加以叙述。

This article is a continuation of a series starting with <a href="/fundamentals/webgl-fundamentals.html">WebGL Fundamentals</a>. If you haven't read them I suggest you read at least the first one, then come back here.

这篇文章是由 <a href="/fundamentals/webgl-fundamentals.html">WebGL Fundamentals</a> 开始的一系列延续的内容。如果你还没有阅读过这一部分，我建议你至少先阅读第一部分，然后再回来这里。

Translation is some fancy math name that basically means "to move" something. I suppose moving a sentence from English to Japanese fits as well but in this case we're talking about moving geometry. Using the sample code we ended up with in <a href="/fundamentals/webgl-fundamentals.html">the first post</a> you could easily translate our rectangle just by changing the values passed to setRectangle right? Here's a sample based on our <a href="/fundamentals/webgl-fundamentals.html">previous sample</a>.

平移（Translation）的本质是一些复杂的数学计算，平移的基本意义是“去移动”一些东西。我认为将一个句子从英语翻译（Translation）成日语也符合定义，但在这个情况下，我们只讨论移动（Translation）几何体。使用我们之前在<a href="/fundamentals/webgl-fundamentals.html">第一篇教程中</a>给出的示例代码你可以通过改变传递给 `setRectangle` 的参数很快地完成矩形的平移。这里给出一个基于<a href="/fundamentals/webgl-fundamentals.html">之前例子</a>的代码示例。

<!-- more -->
```
  // First let's make some variables
  // to hold the translation, width and height of the rectangle
  // 首先让我们使用一些变量来存储矩形的宽度、高度和平移量
  var translation = [0, 0];
  var width = 100;
  var height = 30;

  // Then let's make a function to
  // re-draw everything. We can call this
  // function after we update the translation.
  // 然后让我们定义一个函数来重绘所有内容。
  // 在我们更新矩形的平移设置后我们可以调用这个函数进行重绘操作。

  // Draw a the scene.
  // 绘制场景。
  function drawScene() {
    // Clear the canvas.
    // 清除 canvas 的内容。
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Setup a rectangle
    // 设置一个矩形
    setRectangle(gl, translation[0], translation[1], width, height);

    // Draw the rectangle.
    // 绘制矩形。
    gl.drawArrays(gl.TRIANGLES, 0, 6);
  }
```

In the example below I've attached a couple of sliders that will update `translation[0]` and `translation[1]` and call `drawScene` anytime they change. Drag the sliders to translate the rectangle.

在下面的例子中，我将一对滑动器（slider）与平移的变量绑定在一起，它们会更新 `translation[0]` 和 `translation[1]` 并且每次它们发生改变时都会调用 `drawScene` 函数。通过拖动滑动器去平移矩形。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-rectangle-translate.html" frameborder="0" style="border: 1px solid; width: 100%; height: 200px;"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-rectangle-translate.html" target="_blank">点击这里在新窗口查看例子</a>

So far so good. But now imagine we wanted to do the same thing with a more complicated shape.

到目前为止，一切都好。但试想一下，假如现在我们想要对更加复杂的形状做同样的事情。

Let's say we wanted to draw an 'F' that consists of 6 triangles like this.

假设我们想要去绘制一个如下图所示的由 6 个三角形组成的字母“F”。

<img src="http://webglfundamentals.org/webgl/resources/polygon-f.svg" width="200" height="270">

Well, following our current code we'd have to change `setRectangle` to something more like this.

那么，接下来我们得将 `setRectangle` 函数改成像下面的代码那样。

```
  // Fill the buffer with the values that define a letter 'F'.
  // 用表示字母“F”的顶点数组填充 buffer
  function setGeometry(gl, x, y) {
    var width = 100;
    var height = 150;
    var thickness = 30;
    gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array([
            // left column
            // 左边的一列
            x, y,
            x + thickness, y,
            x, y + height,
            x, y + height,
            x + thickness, y,
            x + thickness, y + height,

            // top rung
            // 上边的一横
            x + thickness, y,
            x + width, y,
            x + thickness, y + thickness,
            x + thickness, y + thickness,
            x + width, y,
            x + width, y + thickness,

            // middle rung
            // 中间的一横
            x + thickness, y + thickness * 2,
            x + width * 2 / 3, y + thickness * 2,
            x + thickness, y + thickness * 3,
            x + thickness, y + thickness * 3,
            x + width * 2 / 3, y + thickness * 2,
            x + width * 2 / 3, y + thickness * 3]),
        gl.STATIC_DRAW);
  }
```

You can hopefully see that's not going to scale well. If we want to draw some very complex geometry with hundreds or thousands of lines we'd have to write some pretty complex code. On top of that, every time we draw JavaScript has to update all the points.

你可以满怀希望的看到这段代码并不能很好的进行拓展。如果我们想要绘制一些有着成百上千条线段的非常复杂的几何体，我们必须写上一些很复杂的代码。除此之外，每一次绘制 JavaScript 都必须更新所有的顶点数据。

There's a simpler way. Just upload the geometry and do the translation in the shader.

这里有一个相对简单的方法。只需要上传几何体然后在着色器中进行平移。

Here's the new shader

下面是新的着色器代码

```
<script id="2d-vertex-shader" type="x-shader/x-vertex">
  attribute vec2 a_position;

  uniform vec2 u_resolution;
  uniform vec2 u_translation;

  void main() {
    // Add in the translation.
    // 增加平移变量
    vec2 position = a_position + u_translation;

    // convert the rectangle from pixels to 0.0 to 1.0
    // 将矩阵大小从像素单位转换到区间 0.0 到 1.0 之间
    vec2 zeroToOne = position / u_resolution;
    ...
  }
</script>
```

and we'll restructure the code a little. For one we only need to set the geometry once.

我们将稍微重构一下代码。使得我们只需要设置一次几何体。

```
// Fill the buffer with the values that define a letter 'F'.
function setGeometry(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
          // left column
          0, 0,
          30, 0,
          0, 150,
          0, 150,
          30, 0,
          30, 150,

          // top rung
          30, 0,
          100, 0,
          30, 30,
          30, 30,
          100, 0,
          100, 30,

          // middle rung
          30, 60,
          67, 60,
          30, 90,
          30, 90,
          67, 60,
          67, 90]),
      gl.STATIC_DRAW);
}
```

Then we just need to update `u_translation` before we draw with the translation that we desire.

然后我们只需要在我们绘制之前更新 `u_translation` 变量就能得到我们想要的平移效果。

```
  ...
  var translationLocation = gl.getUniformLocation(
             program, "u_translation");
  ...
  // Set Geometry.
  setGeometry(gl);
  ..
  // Draw scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Set the translation.
    gl.uniform2fv(translationLocation, translation);

    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 18);
  }
```

Notice `setGeometry` is called only once. It is no longer inside `drawScene`.

注意 `setGeometry` 函数只被调用一次。它不再在 `drawScene` 里面。

And here's that example. Again, drag the sliders to update the translation.

示例如下。再一次，通过拖动滑动器对几何形体进行平移。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate-better.html" frameborder="0" style="border: 1px solid; width: 100%; height: 200px;"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate-better.html" target="_blank">点击这里在新窗口查看例子</a>

Now when we draw, WebGL is doing practically everything. All we are doing is setting a translation and asking it to draw. Even if our geometry had tens of thousands of points the main code would stay the same.

现在当我们进行绘制的时候，WebGL 几乎帮我们完成所有的事情。我们所需要的事情只有设置平移变量然后请求绘制。即使我们的几何体有数以万计的顶点，主要的代码也会照常运行。

If you want you can compare <a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate.html" target="_blank">the version that uses the complex JavaScript above to update all the points</a>.

如果你想要，你可以比较一下<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate.html" target="_blank">上面提到的比较复杂的更新所有顶点的 JavaScript 版本</a>和现在的版本。

I hope this example was not too obvious. In the <a href="webgl-2d-rotation.html">next article we'll move on to rotation</a>.

我希望这个例子不会太平淡无奇。在 <a href="webgl-2d-rotation.html">下一篇文章中我们将开始进入旋转（Rotation）</a>。
