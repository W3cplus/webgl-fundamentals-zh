# WebGL 文本 - HTML


This article is a continuation of previous WebGL articles. If you haven't read them I suggest you start there and work your way back.

该文章是接着前一篇文章继续讲解。如果你还没阅读[上一篇][1]，我建议从那开始。

A common question is "how to I draw text in WebGL". The first thing to ask yourself is what's your purpose in drawing the text. You're in a browser, the browser displays text. So your first answer should be to use HTML to display text.

先问一个常见的问题 “怎样在 WebGL 中绘制文本呢？”。首先，你需要问你自己，为什么要绘制文本。现在运行的环境是浏览器，并且，是由浏览器来显示文本的。所以，你第一个回答应该是使用 HTML 来展示文本。

Let's do the easiest example first: You just want to draw some text over your WebGL. We might call this a text overlay. Basically this is text that stays in the same position.

我们先试一试最简单的办法：如果你仅仅是想在你的 WebGL 上写一些文字，那么我们可以把这个叫做文字浮层。本质上来说，文本还是在原来的位置。

The simple way is to make an HTML element or elements and use CSS to make them overlap.

该方式就是用一个或多个的 HTML 元素，然后使用 CSS 去使他们浮动。

For example: First make a container and put both a canvas and some HTML to be overlaid inside the container.

例如：首先，用一个容器，然后将 canvas 和 HTML 元素平铺在其中。
```
<div class="container">
  <canvas id="canvas"></canvas>
  <div id="overlay">
    <div>Time: <span id="time"></span></div>
    <div>Angle: <span id="angle"></span></div>
  </div>
</div>
```
Next setup the CSS so that the canvas and the HTML overlap

然后，开始写 CSS，让 canvas 和 HTML 重叠。
```
.container {
    position: relative;
}
#overlay {
    position: absolute;
    left: 10px;
    top: 10px;
}
```
Now look up those elements at init time and create or lookup the areas you want to change.

现在，在初始化时解析这些元素，然后创建或解析你想要改变的区域。
```
// 解析我们想要改变的元素
var timeElement = document.getElementById("time");
var angleElement = document.getElementById("angle");
 
// 为了节省浏览器加载时间，可以创建文本节点
var timeNode = document.createTextNode("");
var angleNode = document.createTextNode("");
 
// 然后，将这些节点添加到指定位置
timeElement.appendChild(timeNode);
angleElement.appendChild(angleNode);
```
Finally update the nodes when rendering

最后，在渲染的时候，更新指定节点
```
function drawScene() {
    ...
 
    // 将旋转单位从弧度变为角度
    var angle = radToDeg(rotation[1]);
 
    // 只支持 0 - 360
    angle = angle % 360;
 
    // 设置指定节点
    angleNode.nodeValue = angle.toFixed(0);  // no decimal place
    timeNode.nodeValue = clock.toFixed(2);   // 2 decimal places
```
And here's that example

下面是实例

![F_顺时针][2]

[查看网页][3]

Notice how I put spans inside the divs specifically for the parts I wanted to change. I'm making the assumption here that that's faster than just using the divs with no spans and saying something like

注意我是具体怎样将 `spans` 放到我想改变的部分的。我实际做了假设，这种方式比那种直接使用 div 和如下格式的方式更快
```
timeNode.value = "Time " + clock.toFixed(2);
```

Also I'm using text nodes by calling node = document.createTextNode() and later node.nodeValue = someMsg. I could also use someElement.innerHTML = someHTML. That would be more flexible because you could insert arbitrary HTML strings though it might be slightly slower since the browser has to create and destroy nodes each time you set it. Which is better is up to you.

同样，我通过调用 `node = document.createTextNode()` 来使用文本节点，然后使用 `node.nodeValue = someMsg`。我也能使用 `someElement.innerHTML = someHTML`。这种方式会更灵活，因为你可以插入任意的 HTML 字符串，尽管它可能会更慢，因为当你赋值时，浏览器需要去创建和销毁相应的节点。具体使用哪种，主要看你喜欢哪种。

The important point to take way from the overlay technique is that WebGL runs in a browser. Remember to use the browser's features when appropriate. Lots of OpenGL programmers are used to having to render every part of their app 100% themselves from scratch but because WebGL runs in a browser it already has tons of features. Use them. This has lots of benefits. For example you can use CSS styling to easily give that overlay an interesting style.

使用浮层技术的关键点是因为 WebGL 是运行在浏览器当中。要因地制宜的使用浏览器合适的特性。很多 OpenGL 程序员已经习惯使用 WebGL 去渲染整个应用，这确实让人有点不快。不过，因为 WebGL 是运行在浏览器中的，所以，它具有很多 OpenGL 所没有的特性。合理的使用它们可以带来很多好处。比如，你可以使用 CSS 去设置一个吸引人的罩层。

For example here's the same example but adding some style. The background is rounded, the letters have a glow around them. There's a red border. You get all that essentially for free by using HTML.

例如，这里和上个例子一样，但是添加了新的样式。该背景带上圆角和红色边框，并且字母，数字有发光的特效。你可以轻易使用 HTML 就可以完成核心的内容。

![模糊字体_顺时针][4]

[查看网页][5]

The next most common thing to want to do is position some text relative to something you're rendering. We can do that in HTML as well.

接下来，另外一个很普遍的特性，就是相对于你渲染的内容，确定文本的位置。

In this case we'll again make a container with the canvas and another container for our moving HTML

这样的情况下，我们同样会使用一个容器去存放 canvas 和另外一个子容器。然后该自容器方便我们移动 HTML 内容。
```
<div class="container">
  <canvas id="canvas" width="400" height="300"></canvas>
  <div id="divcontainer"></div>
</div>
```
And we'll setup the CSS

然后，设置 CSS 样式
```
.container {
    position: relative;
    overflow: none;
    width: 400px;
    height: 300px;
}
 
#divcontainer {
    position: absolute;
    left: 0px;
    top: 0px;
    width: 100%;
    height: 100%;
    z-index: 10;
    overflow: hidden;
 
}
 
.floating-div {
    position: absolute;
}
```
The position: absolute; part makes the #divcontainer be positioned in absolute terms relative to the first parent with another position: relative or position: absolute style. In this case that's the container that both the canvas and the #divcontainer are in.

`position: absolute;` 部分会让 `#divcontainer` 相对于第一个使用 `position:relative` 或者 `position:absolute` 的父元素进行绝对定位。在这中情况下，canvas 和 #divcontainer 就在该容器中了。

The left: 0px; top: 0px makes the #divcontainer align with everything. The z-index: 10 makes it float over the canvas. And the overflow: hidden makes its children get clipped.

`left: 0px; top: 0px` 则是让 `#divcontainer` 和左上角对齐。`z-index: 10` 让其浮动在 canvas 上。并且 `overflow: hidden` 能让超出的子元素自动被裁剪隐藏。

Finally .floating-div will be used for the positionable div we create.

最后 `.floating-div` 用来给我们创建的 div 进行定位。

So now we need to look up the divcontainer, create a div and append it.

现在，我们需要找到 `divcontainer`，然后创建一个 `div` 并添加到里面。
```
// 找到 divcontainer
var divContainerElement = document.getElementById("divcontainer");
 
// 创建一个 div 元素
var div = document.createElement("div");
 
// 添加 className
div.className = "floating-div";
 
// 给该元素创建一个文本节点
var textNode = document.createTextNode("");
div.appendChild(textNode);
 
// 将 div 添加到 divcontainer
divContainerElement.appendChild(div);
```
Now we can position the div by setting its style.

下载我们可能通过设置它的样式来进行定位。
```
div.style.left = Math.floor(x) + "px";
div.style.top  = Math.floor(y) + "px";
textNode.nodeValue = clock.toFixed(2);
```
Here's an example where we're just bounding the div around.
这里有一个例子，我们是 div 在容器内弹跳。

![bound_弹跳][6]

[查看网页][7]

So the next step is we want to place it relative to something in the 3D scene. How do we do that? We do it exactly how we asked the GPU to do it when we covered perspective projection.

所以，下一步我们想将它放置于 3D 场景里。那应该怎么做呢？ 当我们完成一个透视的项目时，我们实际上是让 GPU 去实现这样的效果。

Up through that example we learned how to use matrices, how to multiply them, and how to apply a projection matrix to convert them to clipspace. We pass all of that to our shader and it multiplies vertices in local space and converts them to clipspace. We can do all the math ourselves in JavaScript as well. Then we can multiply clipspace (-1 to +1) into pixels and use that to position the div.

通过上面的例子，我们学习了如何合适矩阵，如何去将他们积乘，如何提供一个项目矩阵去将它们转换为裁剪空间。我们将它们传递给着色器，然后它们在本地空间内去处理顶点并转换为裁剪空间。同样，我们自己在 JavaScript 中完成所有的数学计算。然后，我们可以放大裁剪坐标的值变为像素值，接着就可以使用相关内容去定位上面的 div 了。

```
gl.drawArrays(...);
 
// 我们通过计算一个矩阵，然后画一个 3D 的 F
 
// 在 'F' 的本地空间内选择一点
//             X  Y  Z  W
var point = [100, 0, 0, 1];  // this is the front top right corner
 
// 计算一个 F 的裁剪空间位置, 我们需要一个矩阵来作辅助
var clipspace = m4.transformVector(matrix, point);
 
// 用 X 和 Y 分别除以 W，就像 GPU 做的一样
clipspace[0] /= clipspace[3];
clipspace[1] /= clipspace[3];
 
// 将裁剪空间的值转为像素值
var pixelX = (clipspace[0] *  0.5 + 0.5) * gl.canvas.width;
var pixelY = (clipspace[1] * -0.5 + 0.5) * gl.canvas.height;
 
// 设置该 div 的具体位置
div.style.left = Math.floor(pixelX) + "px";
div.style.top  = Math.floor(pixelY) + "px";
textNode.nodeValue = clock.toFixed(2);
```
And voila, the top left corner of our div is perfectly aligned with the top right front corner of the F.

看，现在 `div` 的左上角已经完美的固定在 F 字母的前右上角。

![fixed_前右上角][8]

[查看网页][9]

Of course if you want more text make more divs.

当然，如果你想要跟多的 div 也行

![more_fixed_div_固定][10]

[查看网页][11]

You can look at the source of that last example to see the details. One important point is I'm just guessing that creating, appending and removing HTML elements from the DOM is slow so the example above creates them and keeps them around. It hides any unused ones rather than removing them from the DOM. You'd have to profile to know if that's faster. That was just the method I chose.

你可以通过看最后一个例子的源码了解更多细节。一个很重要的点是，我只是假设从 DOM 中创建，添加和移除 HTML 元素非常慢。所以，在上面的例子中，我创建它们之后进行了缓存。我将不再使用的节点隐藏而不是将它们从 DOM 中删除。你需要具体测试才能知道，这是否更快。这仅仅只是我选择的办法。

Hopefully it's clear how to use HTML for text. Next we'll cover using Canvas 2D for text.

希望上面已经说清楚了怎样使用 HTML 文本。下一节，[我们将讲解使用 Canvas 2D 文本][12]。


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html
  [2]: http://static.zybuluo.com/jimmythr/lyv0d1thfeuej5ey66ovkrl2/F_%E9%A1%BA%E6%97%B6%E9%92%88.gif
  [3]: http://webglfundamentals.org/webgl/webgl-text-html-overlay.html
  [4]: http://static.zybuluo.com/jimmythr/xy58leqmsea9450w7meq54ns/%E6%A8%A1%E7%B3%8A%E5%AD%97%E4%BD%93_%E9%A1%BA%E6%97%B6%E9%92%88.gif
  [5]: http://webglfundamentals.org/webgl/webgl-text-html-overlay-styled.html
  [6]: http://static.zybuluo.com/jimmythr/1vht2pfp09w377lacwwx2wmh/bound_%E5%BC%B9%E8%B7%B3.gif
  [7]: http://webglfundamentals.org/webgl/webgl-text-html-bouncing-div.html
  [8]: http://static.zybuluo.com/jimmythr/9r6tdo68zqwe230io17fwzu2/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-26%2016.35.57.png
  [9]: http://webglfundamentals.org/webgl/webgl-text-html-div.html
  [10]: http://static.zybuluo.com/jimmythr/wvylxls6jxl3y6dvj3i6k2eh/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-10-26%2016.36.04.png
  [11]: http://webglfundamentals.org/webgl/webgl-text-html-divs.html
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-text-canvas2d.html
