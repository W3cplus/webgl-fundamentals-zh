## WebGL Resizing the Canvas.
## WebGL 画布大小
  
Here's what you need to know to change the size of the canvas.
这里你需要知道改变画布的大小的方法。

Every canvas has 2 sizes. The size of its drawingbuffer. This is how many pixels are in the canvas. The second size is the size the canvas is displayed. CSS determines the size the canvas is displayed.

每一个 canvas 都有2个大小。第一个大小是 drawingbuffer。即在画布上有多少个像素。第二个大小是 canvas 显示的大小。CSS 决定了画布显示的大小。

You can set the size of the canvas's drawingbuffer in 2 ways. One using HTML

你有两种方式可以设置 canvas 的 drawingbuffer 的大小。其中一种是使用 HTML

```
<canvas id="c" width="400" height="300"></canvas>
```

The other using JavaScript

另一种是使用 JavaScript

```
<canvas id="c" ></canvas>
```

JavaScript

JavaScript

```
var canvas = document.getElementById("c");
canvas.width = 400;
canvas.height = 300;
```

As for setting a canvas's display size if you don't have any CSS that affects the canvas's display size the display size will be the same size as its drawingbuffer. So in the 2 examples above the canvas's drawingbuffer is 400x300 and its display size is also 400x300.

至于设置画布的显示大小，如果你没有使用任何 CSS 改变 canvas 的显示大小，它的显示大小将会和 drawingbuffer 的大小一样。因此在以上两个示例中 canvas 的 drawingbuffer 大小是 400x300，显示大小也是 400x300。

Here's an example of a canvas whose drawingbuffer is 10x15 pixels that is displayed 400x300 pixels on the page

这个示例中的 canvas 的 drawingbuffer 是 10x15 像素，在页面上的显示大小是 400x300 像素。

```
<canvas id="c" width="10" height="15" style="width: 400px; height: 300px;"></canvas>
```

or for example like this

或者像这个示例

```
<style>
#c {
  width: 400px;
  height: 300px;
}
</style>
<canvas id="c" width="10" height="15"></canvas>
```

If we draw a single pixel wide rotating line into that canvas we'll see something like this

如果我们在画布上画一个单像素宽的旋转线，我们会看到以下效果

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-10x15-canvas-400x300-css.html"></iframe>

[click here to open in a separate window][1]

[点击这里在新窗口中打开][1]

Why is it so blurry? Because the browser takes our 10x15 pixel canvas and stretches it to 400x300 pixels and generally it filters it when it stretches it.

为什么它如此模糊？因为浏览器将我们 10x15 像素的画布拉伸到 400x300 像素，通常当浏览器拉伸画布时使图像失真。

So, what do we do if, for example, we want the canvas to fill the window? Well, first we can get the browser to stretch the canvas to fill the window with CSS. Example

所以，如果我们做些什么，例如，我们希望画布能够填满窗口？嗯，首先我们可以通过 CSS 让浏览器延伸画布以填满窗口。示例

```
<html>
  <head>
    <style>
      /* remove the border */
      /* 移除边框 */
      body {
        border: 0;
        background-color: white;
      }
      /* make the canvas the size of the viewport */
      /* 设置 canvas 大小为视口大小 */
      canvas {
        width: 100vw;
        height: 100vh;
        display: block;
      }
    <style>
  </head>
  <body>
    <canvas id="c"></canvas>
  </body>
</html>
```

Now we just need to make the drawingbuffer match whatever size the browser has stretched the canvas. We can do that using `clientWidth` and `clientHeigh`t which are properties every element in HTML has that let JavaScript check what size that element is being displayed.

现在我们只需要让 drawingbuffer 匹配任意尺寸的拉伸画布的浏览器。我们可以使用每一个 HTML 元素都有的属性 `clientWidth` 和 `clientHeight`，让 JavaScript 检查元素的显示大小。

Most WebGL apps are animated so let's call this function just before we render so it will always adjust the canvas to our desired size just before drawing.

大多数 WebGL 应用是有动画的，因此让我们在渲染之前调用这个函数，这样它将总是在绘画之前调整画布到我们所需的尺寸。

```
function drawScene() {
   resize(gl.canvas);
 
   ...
```

And here's that

效果如下

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-resize-canvas.html"></iframe>

[click here to open in a separate window][2]

[点击这里在新的窗口中打开][2]

Hey, something is wrong? Why is the line not covering the entire area?

嘿，有什么错了？为什么没有覆盖整个区域？

The reason is when we resize the canvas we also need to call `gl.viewport` to set the viewport. `gl.viewport` tells WebGL how to convert from clipspace (-1 to +1) back to pixels and where to do it within the canvas. When you first create the WebGL context WebGL will set the viewport to match the size of the canvas but after that it's up to you to set it. If you change the size of the canvas you need to tell WebGL a new viewport setting.

这是因为我们调整画布的同时需要调用 `gl.viewport` 来设置视口。`gl.viewport` 告诉 WebGL 在画布中如何从裁剪空间（-1 到 +1）转换回像素空间。当你首次创建 WebGL 上下文，WebGL 将设置匹配画布尺寸的视口，但之后它的尺寸就取决于你的设置。如果你改变了画布大小，你需要告诉 WebGL 新的视口设置。

Let's change the code to handle this. On top of that, since the WebGL context has a reference to the canvas let's pass that into resize.

让我们一起改变代码来解决这个问题吧。最重要的是，因为 WebGL 上下文有对画布的引用，我们可以通过这一引用调整尺寸。

```
function drawScene() {
   resize(gl.canvas);
 
   gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
   ...
```

Now it's working.

现在它起作用了。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-resize-canvas-viewport.html"></iframe>

[click here to open in a separate window][3]

[点击这里在新的窗口中打开][3]

Open that in a separate window, size the window, notice it always fills the window.

在新的窗口中打开，调整窗口，注意它总是填满窗口。

I can hear you asking, why doesn't WebGL set the viewport for us automatically when we change the size of the canvas? The reason is it doesn't know how or why you are using the viewport. You could be rendering to a framebuffer or doing something else that requires a different viewport size. WebGL has no way of knowing your intent so it can't automatically set the viewport for you.

我能够听见你的疑问，当我们改变画布大小时为什么 WebGL 不自动为我们设置视口？这是因为它不知道你如何或为什么使用视口。你可能渲染到帧缓存区，或者做其它需要不同的视口大小的事情。WebGL 无从得知你的意图，所以它不能为你自动的设置视口大小。

If you look at many WebGL programs they handle resizing or setting the size of the canvas in many different ways. If you're curious [here are some of the reasons I think the way described above is preferable][4].

你是否看到很多 WebGL 程序以许多不同的方式调整大小或者设置画布大小。如果你对这好奇，[我认为用这些原因描述以上现象会更好][4]

**补充知识**

----------

## How do I handle Retina or HD-DPI displays?

## 我如何处理 Retina 或者 HD-DPI 的显示器？

When you specify a size in CSS or Canvas by pixels those are called CSS pixels which may or may not be actual pixels. Most current smartphones have what's called a high-definition DPI display (HD-DPI) or as Apple calls it a "Retina Display". For text and most CSS styling the browser can automatically render HD-DPI graphics but for WebGL, since you're drawing the graphics it's up to you to render at a higher resolution if you want your graphics to be "HD-DPI" quality. 
To do that we can look at the window.devicePixelRatio value. This value tells us how many real pixels equals 1 CSS pixel. We can change our resize function to handle that like this.

当你在 CSS 或者 Canvas 中用像素指定大小，这些被称为 CSS pixels，可能是也可能不是实际的像素。现在大多数智能手机都是高清分辨率（HD-DPI）或者苹果称之为“Retina Display”。对于浏览器中的文本和大多数 CSS 样式都能够自动进行 HD-DPI 图形渲染，但对于 WebGL，如果你希望图形是“HD-DPI”质量的，这取决于你在一个更高的分辨率中绘制图形。为此我们可以查看 window.devicePixelRatio 的值。这个值告诉我们多少 real pixels 等于 1 CSS pixel。我们可以改变 resize 方法来处理这问题。

```
function resize(gl) {
  var realToCSSPixels = window.devicePixelRatio || 1;

  // Lookup the size the browser is displaying the canvas in CSS pixels
  // 查找浏览器中展示的 canvas 的 CSS pixels
  // and compute a size needed to make our drawingbuffer match it in
  // device pixels.
  // 计算 drawingbuffer 匹配 device pixels 的尺寸。
  var displayWidth  = Math.floor(gl.canvas.clientWidth  * realToCSSPixels);
  var displayHeight = Math.floor(gl.canvas.clientHeight * realToCSSPixels);

  // Check if the canvas is not the same size.
  // 检查 canvas 尺寸是否不一样。
  if (gl.canvas.width  != displayWidth ||
      gl.canvas.height != displayHeight) {

    // Make the canvas the same size
    // 是画布大小相同
    gl.canvas.width  = displayWidth;
    gl.canvas.height = displayHeight;
  }
}
```
If you have an HD-DPI display, for example if you view this page on your smartphone you should notice the line below is thinner than the one above which didn't adjust for HD-DPI displays

如果你有 HD-DPI 显示器，例如，你在智能手机上查看这个页面，你会注意到下面的线比上面没有进行 HD-DPI 显示调整的线更细。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-resize-canvas-hd-dpi.html"></iframe>

[click here to open in a separate window][5]

[点击这里在新的窗口中打开][5]

Whether you really want to adjust for HD-DPI is up to you. On iPhone4 or iPhone5 window.devicePixelRatio is 2 which means you'll be drawing 4 times as many pixels. I believe on an iPhone6Plus that value is 3 which means you'd be drawing 9 times as many pixels. That can really slow down your program. In fact it's a common optimization in games to actually render less pixels than are displayed and let the GPU scale them up. It really depends on what your needs are. If you're drawing a graph for printing you might want to support HD-DPI. If you're making a game you might not or you might want to give the user the option to turn support on or off if their system is not fast enough to draw so many pixels.

是否进行 HD-DPI 调整取决于你，在 iPhone4 或者 iPhone5 中 window.devicePixelRatio 为 2，意味着你将画出4倍的像素。我相信在 iPhone6 Plus 的值是 3，这意味着你将画9倍的像素。这会使你的程序变得迟钝。实际上渲染比需要显示的更少的像素然后让 GPU 放大，是在游戏中常见的优化。这真的取决于你的需求。如果你如果你在画一个印刷的图你可能会希望支持 HD-DPI。如果你是在做游戏，当用户的系统没有足够快的速度去绘制更多的像素时，你可能希望让用户选择是否开启支持 HD-DPI 的选项。

[1]: http://webglfundamentals.org/webgl/webgl-10x15-canvas-400x300-css.html
[2]: http://webglfundamentals.org/webgl/webgl-resize-canvas.html
[3]: http://webglfundamentals.org/webgl/webgl-resize-canvas-viewport.html
[4]: http://webglfundamentals.org/webgl/lessons/webgl-anti-patterns.html
[5]: http://webglfundamentals.org/webgl/webgl-resize-canvas-hd-dpi.html