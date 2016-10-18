# WebGL 2D 缩放（Scale）

This post is a continuation of a series of posts about WebGL. The first <a href="webgl-fundamentals.html">started with fundamentals</a> and the previous was <a href="webgl-2d-rotation.html">about rotating geometry</a>.

这篇教程是一系列关于 WebGL 的教程中的一篇。第一篇教程是 <a href="../fundamentals/WebGL-Fundamentals.html" target="_blank"> WebGL 基础</a>，上一篇教程是 <a href="./WebGL-2D-Rotation.html" target="_blank">关于旋转几何体</a>。

Scaling is just as <a href="webgl-2d-translation.html">easy as translation</a>.
<!--more-->
We multiply the position by our desired scale. Here are the changes from our <a href="webgl-2d-rotation.html">previous sample</a>.

缩放就像 <a href="./WebGL-2D-Translation.html" target="_blank">平移</a> 一样简单。我们只要用我们想要的比例去乘以几何体的坐标。下面的代码是从我们 <a href="./WebGL-2D-Rotation.html" target="_blank">之前的例子</a>修改而来的。

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

and we add the JavaScript needed to set the scale when we draw.

然后我们添加必要的 JavasScript 代码在我们进行绘制的时候去设置缩放。

```
  ...
  var scaleLocation = gl.getUniformLocation(program, "u_scale");
  ...
  var scale = [1, 1];
  ...
  // Draw the scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Set the translation.
    gl.uniform2fv(translationLocation, translation);

    // Set the rotation.
    gl.uniform2fv(rotationLocation, rotation);

    // Set the scale.
    gl.uniform2fv(scaleLocation, scale);

    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 18);
  }
```

And now we have scale. Drag the sliders.

现在我们有了缩放。同样是通过拖动滑动器进行缩放。

<iframe src="http://webglfundamentals.org/webgl/webgl-2d-geometry-scale.html" width="100%" height="500"></iframe>

<a href="http://webglfundamentals.org/webgl/webgl-2d-geometry-scale.html" target="_blank">点击这里在新窗口查看例子</a>

One thing to notice is that scaling by a negative value flips our geometry.

有一件需要注意的事情是设置一个负数的缩放比例会使得我们的几何体翻转。

I hope these last 3 posts were helpful in understanding <a href="webgl-2d-translation.html">translation</a>, <a href="webgl-2d-rotation.html">rotation</a> and scale. Next we'll go over <a href="webgl-2d-matrices.html">the magic that is matrices</a> that combines all 3 of these into a much simpler and often more useful form.

我希望这 3 篇文章可以帮助你去理解 <a href="./WebGL-2D-Translation.html" target="_blank">平移</a>、<a href="./WebGL-2D-Rotation.html" target="_blank">旋转</a>、缩放。接下来我们将详细的讲述 <a href="./WebGL-2D-Matrices.html" target="_blank">具有魔力的矩阵</a>，矩阵可以更加简单的把这三种变换组合在一起，而且通常还会是更加有用的形式。

> ## Why an 'F'?
>
> The first time I saw someone use an 'F' was on a texture. The 'F' itself is not important. What is important is that you can tell its orientation from any direction. If we used a heart ❤ or a triangle △ for example we couldn't tell if it was flipped horizontally. A circle ○ would be even worse. A colored rectangle would arguably work with different colors on each corner but then you'd have to remember which corner was which. An F's orientation is instantly recognizable.
>
> ![](http://webglfundamentals.org/webgl/resources/f-orientation.svg)
>
> Any shape that you can tell the orientation of would work, I've just used 'F' ever since I was 'F'irst introduced to the idea.
>
> ## 为什么是 “F”？
>
> 我第一次看到某人使用 “F” 是在一个纹理上面。“F” 本身不是那么重要。重要的是你从任意方向都能知道它的指向。如果我们使用心形 ❤ 或者一个三角形 △ 作为例子，如果它们发生水平翻转那我们则无法得知他们的指向是否发生改变。使用圆 ⚪ 更加糟糕。一个为每个角都涂上不同颜色的矩形可以认为是胜任这个工作的，但我们得记住哪个角对应哪个颜色。一个 “F” 的指向可以很直观的识别出来。
>
> ![](http://webglfundamentals.org/webgl/resources/f-orientation.svg)
>
> 任何你可以一看就知道其指向的图形都能胜任这份工作。当我第一次了解到这个概念的时候，我开始使用 “F”。
