# WebGL 2D 平移（Translation）

在开始进入 3D 之前我们先在 2D 停留一段时间。请各位耐心听我说，这篇文章或许对一些同学来说极其浅显，但我还是会用一些篇幅由点到面地加以叙述。

这篇文章是由 <a href="/fundamentals/webgl-fundamentals.html" target="_blank">WebGL Fundamentals</a> 开始的一系列延续的内容。如果你还没有阅读过这一部分，我建议你至少先阅读第一部分，然后再回来这里。

平移（Translation）的本质是一些复杂的数学计算，平移的基本意义是“去移动”一些东西。我认为将一个句子从英语翻译（Translation）成日语也符合定义，但在这个情况下，我们只讨论移动（Translation）几何体。使用我们之前在<a href="/fundamentals/webgl-fundamentals.html" target="_blank">第一篇教程中</a>给出的示例代码你可以通过改变传递给 `setRectangle` 的参数很快地完成矩形的平移。这里给出一个基于<a href="/fundamentals/webgl-fundamentals.html" target="_blank">之前例子</a>的代码示例。

<!-- more -->
```
  // 首先让我们使用一些变量来存储矩形的宽度、高度和平移量
  var translation = [0, 0];
  var width = 100;
  var height = 30;

  // 然后让我们定义一个函数来重绘所有内容。
  // 在我们更新矩形的平移设置后我们可以调用这个函数进行重绘操作。

  // 绘制场景。
  function drawScene() {
    // 清除 canvas 的内容。
    gl.clear(gl.COLOR_BUFFER_BIT);

    // 设置一个矩形
    setRectangle(gl, translation[0], translation[1], width, height);

    // 绘制矩形。
    gl.drawArrays(gl.TRIANGLES, 0, 6);
  }
```

在下面的例子中，我将一对滑动器（slider）与平移的变量绑定在一起，它们会更新 `translation[0]` 和 `translation[1]` 并且每次它们发生改变时都会调用 `drawScene` 函数。通过拖动滑动器去平移矩形。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-rectangle-translate.html" frameborder="0" style="border: 1px solid; width: 100%; height: 200px;"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-rectangle-translate.html" target="_blank">点击这里在新窗口查看例子</a>

到目前为止，一切都好。但试想一下，假如现在我们想要对更加复杂的形状做同样的事情。

假设我们想要去绘制一个如下图所示的由 6 个三角形组成的字母“F”。

<img src="http://webglfundamentals.org/webgl/resources/polygon-f.svg" width="200" height="270">

那么，接下来我们得将 `setRectangle` 函数改成像下面的代码那样。

```
  // 用表示字母“F”的顶点数组填充 buffer
  function setGeometry(gl, x, y) {
    var width = 100;
    var height = 150;
    var thickness = 30;
    gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array([
            // 左边的一列
            x, y,
            x + thickness, y,
            x, y + height,
            x, y + height,
            x + thickness, y,
            x + thickness, y + height,

            // 上边的一横
            x + thickness, y,
            x + width, y,
            x + thickness, y + thickness,
            x + thickness, y + thickness,
            x + width, y,
            x + width, y + thickness,

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

你可以满怀希望的看到这段代码并不能很好的进行拓展。如果我们想要绘制一些有着成百上千条线段的非常复杂的几何体，我们必须写上一些很复杂的代码。除此之外，每一次绘制 JavaScript 都必须更新所有的顶点数据。

这里有一个相对简单的方法。只需要上传几何体然后在着色器中进行平移。

下面是新的着色器代码

```
<script id="2d-vertex-shader" type="x-shader/x-vertex">
  attribute vec2 a_position;

  uniform vec2 u_resolution;
  uniform vec2 u_translation;

  void main() {
    // 增加平移变量
    vec2 position = a_position + u_translation;

    // 将矩阵大小从像素单位转换到区间 0.0 到 1.0 之间
    vec2 zeroToOne = position / u_resolution;
    ...
  }
</script>
```

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

注意 `setGeometry` 函数只被调用一次。它不再在 `drawScene` 里面。

示例如下。再一次，通过拖动滑动器对几何形体进行平移。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate-better.html" frameborder="0" style="border: 1px solid; width: 100%; height: 200px;"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate-better.html" target="_blank">点击这里在新窗口查看例子</a>

现在当我们进行绘制的时候，WebGL 几乎帮我们完成所有的事情。我们所需要的事情只有设置平移变量然后请求绘制。即使我们的几何体有数以万计的顶点，主要的代码也会照常运行。

如果你想要，你可以比较一下<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-translate.html" target="_blank">上面提到的比较复杂的更新所有顶点的 JavaScript 版本</a>和现在的版本。

我希望这个例子不会太平淡无奇。在 <a href="webgl-2d-rotation.html">下一篇文章中我们将开始进入旋转（Rotation）</a>。
