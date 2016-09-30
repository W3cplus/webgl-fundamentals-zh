# webGL 基础


WebGL is often thought of as a 3D API。People think "I'll use WebGL and magic I'll get cool 3d"。In reality WebGL is just a rasterization engine。It draws points，lines，and triangles based on code you supply。Getting WebGL to do anything else is up to you to provide code to use points，lines，and triangles to accomplish your task.

WebGL 一直被认为提供了 3D 图像的API。人们总会想 “用了 WebGL 我就能神奇地得到酷炫的三维效果了”。但，事实上，WebGL 就是一个栅格化引擎。他可以根据你的代码画点，线和三角形。WebGL 做的任何事情都是取决于你写的代码，你可以画点，线和三角形去完成你想要的效果。


WebGL runs on the GPU on your computer。As such you need to provide the code that runs on that GPU。You provide that code in the form of pairs of functions。Those 2 functions are called a顶点着色器and a 片元着色器 and they are each written in a very strictly typed C/C++ like language called GLSL。(GL Shader Language)。Paired together they are called a program.

WebGL 会在你电脑的 GPU 上运行。所以，你需要提供一些能够在 GPU 上运行的代码。这些代码通常是成对出现的函数。这两个函数可以叫做顶点着色器（vertex shader）和片元着色器（fragment shader）。它们是用一种类似强类型 C/C++ 语言写的，叫做 [GLSL][1]。（GL Sahder Language）。当它们成对出现时，可以被称为程序。


A顶点着色器's job is to compute vertex positions。Based on the positions the function outputs WebGL can then rasterize various kinds of primitives including points，lines，or triangles。When rasterizing these primitives it calls a second user supplied function called a 片元着色器。A 片元着色器's job is to compute a color for each pixel of the primitive currently being drawn.

顶点着色器 主要任务就是计算点的位置。找到点后，根据你提供的函数，就可以画出各种原始图形，比如，点，线，三角形。当在渲染图形的时，接着就调用你提供的另外一个函数--片元着色器，他来决定图形上每个像素点，到底是什么颜色。

Nearly all of the entire WebGL API is about setting up state for these pairs of functions to run. For each thing you want to draw you setup a bunch of state then execute a pair of functions by calling gl.drawArrays or gl.drawElements which executes your shaders on the GPU.

几乎所有的 WebGL API 都是执行上述成对函数。 如果你想画某些东西，需要建立一系列的状态，然后通过调用 `gl.drawArrays` 或者 `gl.drawElements`去执行那些成对的函数，它们会在 GPU 上执行你写好的着色器。

Any data you want those functions to have access to must be provided to the GPU. There are 4 ways a shader can receive data.

如果你想提供一些具体的数据给函数渲染，那么你首先得让 GPU 看懂。你可以通过四种方式，将你的数据发送给着色器：

 1. 属性（Attribute）和Buffers
Buffers 这个类型在 js 中已经出现很久了，他就是一个数组，用来存放二进制数据。而，GPU 也主要接收的数据就是 Buffer。通常，每一个 Buffer 包含的数据信息有，位置，法线，纹理坐标，颜色 等等。当然，你可以放任何你想放的数据在里面。
属性则是用来定义如何获取你的 data ，buffers 具体内容和提供给顶点着色器。比如，你想把 3 个 32bit 的浮点数据作为位置(x，y，z)，放在 Buffer 里面。那么，你就得提供特定的属性，告诉哪个 Buffer 里面有这些数据？哪种数据类型应该提取(比如，3个浮点类型的数据)？ 数据在 Buffer 的具体位置在哪？现在的位置距离下个位置的间隔是多少？
Buffers 并不是随机获取的。另外，顶点着色器会根据提供的数据执行特定的次数。每次下一个值从指定的 Buffer 中提取，指定给某个属性并执行。

 2. Uniforms
Uniforms 是一个有效的全局变量。我们会在着色器程序执行前，先把他给定义了。
 3. 纹理（Textures）
纹理是一串数组，你可以在你的着色器程序中的任何位置访问他。我们最经常做的事情，就是获取图片数据。不过，纹理不仅仅只是数据，还可以包含除颜色以外的其他数据。
 4. Varyings
顶点着色器常常利用 Varyings 变量将数据传给片元着色器。在顶点着色器设置的 varyings 变量值，通常取决于被渲染的基本图形是啥，比如有，点，线，三角形。varyings 会在片元着色器 执行时，插入进去。


## WebGL Hello World

WebGL only cares about 2 things. Clipspace coordinates and colors. Your job as a programmer using WebGL is to provide WebGL with those 2 things. You provide your 2 "shaders" to do this. A Vertex shader which provides the clipspace coordinates and a fragment shader that provides the color.

WebGL 只关心两个东西。裁剪坐标系和颜色。而我们程序员也只需要向 WebGL 提供者两样东西--两个“着色器”。顶点着色器提供裁剪坐标，片元着色器提供了对应的颜色。

Clipspace coordinates always go from -1 to +1 no matter what size your canvas is. Here is a simple WebGL example that shows WebGL in its simplest form.

裁剪坐标系取值范围永远是 -1 到 1，不管你的canvas有多大。下列是一个精简版的 WebGL 例子:

我们先看一下顶点着色器
```
// 从 buffer 中接受数据的属性
attribute vec4 a_position;
 
// 所有的 shader 都需要有一个 main 函数
void main() {
 
  // gl_Position 在 vertex 中是一个特殊的变量
  // 设置具体的值
  gl_Position = a_position;
}
```
我们用 js 脚本翻译一下上面的 GLSL 代码:
```
// *** 伪代码!! ***
 
var positionBuffer = [
  0，0，0，0，
  0，0.5，0，0，
  0.7，0，0，0，
];
var attributes = {};
var gl_Position;
 
drawArrays(...，offset，count) {
  var stride = 4;
  var size = 4;
  for (var i = 0; i < count; ++i) {
     // 从 positionBuffer 获取 4个为一个单位的坐标值给a_position
     attributes.a_position = positionBuffer.slice((offset + i) * stride，size);
     runVertexShader();
     ...
     doSomethingWith_gl_Position();
}
```
实际的代码可能比这要复杂一点，因为 `positionBuffer` 需要被转换为二进制数据。所以，从 buffer 里面获取数据的计算可能会有点不同。不过，这应该能够说清楚顶点着色器到底是怎么执行的。

接着，来看一下片元着色器。
```
// 片元着色器没有默认精度，所以我们需要手动指定一个。中等精度就ok。设置为 "mediump".
// 实际上就是 "medium precision"
precision mediump float;
 
void main() {
  // gl_FragColor 是一个变量
  gl_FragColor = vec4(1，0，0.5，1); // 返回某一种紫色
}
```

上面，我们设置的 `gl_FragColor` 变量值是 `1，0，0.5，1`。1 代表红色，0 代表绿色，0.5 代表蓝色，1代表透明度（alpha）。WebGL 的颜色值总是从 0 到 1。

Now that we have written the 2 shader functions lets get started with WebGL

现在，我们已经写好了两个着色器函数，接着，让我们开始看看 WebGL。

首先，我们需要一个 canvas
```
<canvas id="c" width="400" height="300"></canvas>
```
然后在 JavaScript 里面获取它
```
var canvas = document.getElementById("c");
```
接着，创建一个 WebGLRenderingContext
```
 var gl = canvas.getContext("webgl");
 if (!gl) {
    // webgl 不存在
```
现在，我们需要编译这些 shaders 来让他们在 GPU 上运行。所以，为了编译，我们需要将这些 shaders 变为字符串。比如，通过字符串拼接，AJAX 拉取，使用多行的字符串模板，或者 按照下列的写法，将 shader 使用非 JavaScript 的标签进行包裹。
```
<script id="2d-vertex-shader" type="notjs">

  void main() {
    gl_Position = a_position;
  }
 
</script>
 
<script id="2d-fragment-shader" type="notjs">
  precision mediump float;
  void main() {
    gl_FragColor = vec4(1，0，0.5，1); // return redish-purple
  }
</script>
```
In fact, most 3D engines generate GLSL shaders on the fly using various types of templates, concatenation, etc. For the samples on this site though none of them are complex enough to need to generate GLSL at runtime.

事实上，很多 3D 引擎会使用不同的方式来动态生成 GLSL 着色器，比如模板，拼接等。不过，在本教程中，没有哪个例子复杂到需要动态生成 GLSL。
 
下一步，我们就需要写一个函数。用它去创建着色器，提交 GLSL 代码，并且编译 着色器。下面函数，我没有写什么注释，因为从命名上来看，他已经很清楚了。

```
function createShader(gl，type，source) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader，source);
  gl.compileShader(shader);
  var success = gl.getShaderParameter(shader，gl.COMPILE_STATUS);
  if (success) {
    return shader;
  }
 
  console.log(gl.getShaderInfoLog(shader));
  gl.deleteShader(shader);
}
```
现在，我们需要使用该函数去创建两个着色器。
```
var vertexShaderSource = document.getElementById("2d-vertex-shader").text;
var fragmentShaderSource = document.getElementById("2d-fragment-shader").text;
 
var vertexShader = createShader(gl，gl.VERTEX_SHADER，vertexShaderSource);
var fragmentShader = createShader(gl，gl.FRAGMENT_SHADER，fragmentShaderSource);
```
接着，我们需要将两个着色器和实际的程序结合起来。
```
function createProgram(gl，vertexShader，fragmentShader) {
  var program = gl.createProgram();
  gl.attachShader(program，vertexShader);
  gl.attachShader(program，fragmentShader);
  gl.linkProgram(program);
  var sucesss = gl.getProgramParameter(program，gl.LINK_STATUS);
  if (success) {
    return program;
  }
 
  console.log(gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
}
```
然后，调用
```
var program = createProgram(gl，vertexShader，fragmentShader);
```
现在，我们已经在 GPU 上，写好的 GLSL 代码。接下来我们要做的事情就是使用 WebGL 提供相关的数据给 GLSL。在下面的这个例子中，我们只向 GLSL 程序提供一个 `a_position` 的属性。首先，我们需要提供位置属性，给我们刚刚创建的程序.
```
var positionAttributeLocation = gl.getAttribLocation(program，"a_position");
```
我们需要在初始化时提供位置属性(和统一位置)，而不是在渲染时才提供。

接着，我们需要创建一个 buffer 来给属性提供值。
```
var positionBuffer = gl.createBuffer();
```
WebGL 能让我们在全局绑定点上，使用 WebGL 的大部分资源。你可以认为，绑定点在 WebGL 中，就是一个个全局变量。首先，你绑定一个资源给一个点。然后，涉及到该资源的所有方法都绑定到这个点上了。所以，我们需要先绑定一个位置 Buffer。
```
gl.bindBuffer(gl.ARRAY_BUFFER,positionBuffer);
```
现在，我们需要把数据放到 buffer 中。然后，通过这些数据找到具体的绑定点。
```
// 三个平面点的位置
var positions = [
  0,0,
  0,0.5,
  0.7,0,
];
gl.bufferData(gl.ARRAY_BUFFER，new Float32Array(positions)，gl.STATIC_DRAW);
```
There's a lot going on here. The first thing is we have positions which is a JavaScript array. WebGL on other hand needs strongly typed data so the part new Float32Array(positions) creates a new array of 32bit floating point numbers and copies the values from positions. gl.bufferData then copies that data to the positionBuffer on the GPU. It's using the position buffer because we bound it to the ARRAY_BUFFER bind point above.

上面的代码看起来有点复杂。首先，我们使用 JavaScript 的 Array 结构，去定义 `positions`。但，WebGL 需要的是强类型的数据，所以，`new Float32Array(positions)` 创建了一个 32位 浮点数的数组，并拷贝 `poistions` 里的数据。然后，`gl.bufferData` 将拷贝的数据给了  `positionBuffer`。我们上面已经将 `positions` 和 `ARRAY_BUFFER` 绑定在一起，所以，`gl.bufferData` 可以使用 position buffer。

The last argument, gl.STATIC_DRAW is a hint to WebGL about how we'll use the data. WebGL can try to use that hint to optimize certain things. gl.STATIC_DRAW tells WebGL we are not likely to change this data much.

最后一个参数 `gl.STATIC_DRAW` 告诉 WebGL ，我们这个数据是作什么的。WebGL 能够根据这个参数去做一些优化。`gl.STATIC_DRAW` 告诉 WebGL 按照常量的方式，来处理这次传入的数据。

Now that we've put data in the a buffer we need to tell the attribute how to get data out of it. First off we need to turn the attribute on

我们现在已经把数据放在 Buffer 里，但我们还需要告诉属性，怎么将数据从 Buffer 中取出来。首先，需要启用属性（attribute）。
```
gl.enableVertexAttribArray(positionAttributeLocation);
```
然后，我们写一下提取数据的程序。
```
var size = 2;          // 每次迭代获取2个数据
var type = gl.FLOAT;   // 数据的类型是32位的浮点数
var normalize = false; // 不需要格式化数据
var stride = 0;        // 数据与数据之间的间隔是0。间隔计算就是 size * sizeof(type)
var offset = 0;        // 有效数据在 buffer 中开始的位置
gl.vertexAttribPointer(
    positionAttributeLocation，size，type，normalize，stride，offset)
```
A hidden part of gl.vertexAttribPointer is that it binds the current ARRAY_BUFFER to the attribute. In other words now this attribute is bound to positionBuffer. That means we're free to bind something else to the ARRAY_BUFFER bind point. The attribute will continue to use positionBuffer.

gl.vertexAttribPointer 在内部将 `ARRAY_BUFFER` 绑定给了属性值。换句话就是，这个属性（attribute）和 `positionBuffer` 绑定在一起。这样，我们也不需要额外给 `ARRAY_BUFFER` 再绑定什么了。这个属性会直接使用 `positionBuffer` 里的数据。

注意，我们在上面写的 GLSL 顶点着色器。`a_position` 属性是一个 vec4 类型。
```
attribute vec4 a_position;
```
vec4 is a 4 float value. In JavaScript you could think of it something like a_position = {x: 0, y: 0, z: 0, w: 0}. Above we set size = 2. Attributes default to 0, 0, 0, 1 so this attribute will get its first 2 values (x and y) from our buffer. The z, and w will be the default 0 and 1 respectively.

`vec4` 是一个 4 单位的浮点值。在 JavaScript 里，我们可以认为他的形式如同 `a_position = {x: 0，y: 0，z: 0，w: 1}`。上面，我们将 `size = 2`。因为，属性值默认为 0，0，0，1 ，所以该属性会从我们的 buffer 中，得到 `a_position` 的前两个值（x 和 y）。而 z 和 w 默认为 0 和 1。

Before we draw we should resize the canvas to match it's display size. Canvases just like Images have 2 sizes. The number of pixels actually in them and separately the size they are displayed. CSS determines the size the canvas is displayed. You should always set the size you want a canvas with CSS since it is far far more flexible than any other method.

在绘制之前，我们应该改变 canvas 的大小去满足需要显示的大小。Canvases 和 图片一样有长，宽两个大小值。CSS 可以设置 canvas 显示的大小。**你应该使用 CSS 将 canvas 设为你想要的大小**，因为 CSS 相比于其他方法而言更灵活。

To make the number of pixels in the canvas match the size it's displayed I'm using a helper function you can read about here.

让 canvas 里的像素量能刚好合适他展示的大小。我使用一个[辅助函数][2]。


In nearly all of these samples the canvas size is 400x300 pixels if the sample is run in its own window but stretches to fill the available space if it's in side an iframe like it is on this page. By letting CSS determine the size and then adjusting to match we easily handle both of these cases.

几乎所有例子中 canvas 的大小都是 400x300,即使该例子需要在他自己的 window 里面运行而且需要拉伸去填充可用的空间，即使像该页面一样，他是在一个 iframe 中。我们可以通过 CSS 设置 canvas 的大小，然后调整到合适的范围。

```
webglUtils.resizeCanvasToDisplaySize(gl.canvas);
```

We need to tell WebGL how to convert from the clip space values we'll be setting gl_Position to back into pixels, often called screen space. To do this we call gl.viewport and pass it the current size of the canvas.

另外，我们需要告诉 WebGL，怎样将 `gl_Position` 设置的裁剪空间(三维坐标)的值，转换为平面空间(二维坐标)的值。这里，我们使用 `gl.viewport` 并且获得 canvas 的大小。
```
gl.viewport(0，0，gl.canvas.width，gl.canvas.height);
```
This tells WebGL the -1 +1 clip space maps to 0 -> gl.canvas.width for x and 0 -> gl.canvas.height for y.

这里告诉 WebGL 将 -1 +1 的裁剪坐标分别进行映射， 0 -> gl.canvas.width 为 x，0 -> gl.canvas.height 为 y。

We clear the canvas. 0, 0, 0, 0 are r, g, b, alpha so in this case we're making the canvas transparent.

我们先清空 canvas。因为 0，0，0，0 分别是 r，g，b，alpha。所以，我们实际上将 canvas 变透明了。

```
// 清空 canvas
gl.clearColor(0, 0, 0, 0);
gl.clear(gl.COLOR_BUFFER_BIT);
```
Finally we need to tell WebGL which shader program to execute.

最后，我们需要告诉 WebGL 去调用哪一个 shader。
```
// 告诉 WebGL 去调用我们的程序（成对的着色器）
gl.useProgram(program);
```

After all that we can finally ask WebGL to execute our GLSL program.

之后，我们就可以让 WebGL 去执行我们的 GLSL 程序了。
```
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 3;
gl.drawArrays(primitiveType，offset，count);
```

Because the count is 3 this will execute our vertex shader 3 times. The first time a_position.x and a_position.y in our vertex shader attribute will be set to the first 2 values from the positionBuffer. The 2nd time a_position.xy will be set to the 2nd two values. The last time it will be set to the last 2 values.

因为 `count` 的值是 3，代表着 WebGL 会执行顶点着色器 3 次。第一次，`a_position.x` 和 `a_position.y` 被设置为上面提供的 `positionBuffer` 的前两个值。第二次，则是接下来的两个值。第三次，就是最后两个值。

Because we set primitiveType to gl.TRIANGLES, each time our vertex shader is run 3 times WebGL will draw a triangle based on the 3 values we set gl_Position to. No matter what size our canvas is those values are in clip space coordinates that go from -1 to 1 in each direction.

因为我们一开始设置的 `primitiveType` 是 `gl.TRIANGLES`，所以，每次我们的顶点着色器会被调用 3 次，然后，WebGL 会根据我们提供给 `gl_Position` 的三个值画一个三角形。不管 canvas 有多大，只要映射到裁剪空间的值在 -1 和 1 之间，就没问题。

因为顶点着色器只是简单的将 positionBuffer 的值赋给了 `gl_Position`。所以，这个三角形会很直接绘制到裁剪的坐标系上。
```
  0，0，
  0，0.5，
  0.7，0，
```

Converting from clip space to screen space WebGL is going to draw a triangle at. If the canvas size happned to be 400x300 we'd get something like this

当坐标值从裁剪坐标转换到平面坐标中，WebGL 会在其中画一个三角形。如果 canvas 的大小是 400x300，我们会得到下面的结果。
```
 clip space      screen space
   0，0       ->   200，150
   0，0.5     ->   200，225
 0.7，0       ->   340，150
```

WebGL will now render that triangle. For every pixel it is about to draw WebGL will call our fragment shader. Our fragment shader just sets gl_FragColor to 1, 0, 0.5, 1. Since the Canvas is an 8bit per channel canvas that means WebGL is going to write the values [255, 0, 127, 255] into the canvas.

WebGL 接着开始渲染三角形。它会调用片元着色器去渲染每一个小苏的颜色。我们片元着色器设置的值为 ` 1，0，0.5，1`。因为，canvas 是 8bit 通道，WebGL 最后渲染的颜色值就是 `[255，0，127，255]`.

看个具体的例子:

![triangle][3]

[查看具体的网页][4]

In the case above you can see our vertex shader is doing nothing but passing on our position data directly. Since the position data is already in clipspace there is no work to do. If you want 3D it's up to you to supply shaders that convert from 3D to clipspace because WebGL is only a rasterization API.

上面的例子中，因为位置数据已经在裁剪坐标中，不需要做额外的转换，所以，顶点着色器只是帮我们传递了位置数据而已。如果你想要3D的效果，那么你需要提供一个着色器，可以将 3D 坐标转换为 裁剪空间的位置，而这和 WebGL 没有太大的关系，因为 WebGL 只是一个栅格化的 API。

You might be wondering why does the triangle start in the middle and go to toward the top right. Clip space in x goes from -1 to +1. That means 0 is in the center and positive values will be to the right of that. 

你可能会疑惑，为什么这个三角形从中点开始然后朝向右上方。我们来分析一下。x 轴的范围是 -1 到 1。零点在中心并且向右值增大。

对于纵坐标而言，-1 就是底部，1 就是顶部。零点在中心，向上值增大。

For 2D stuff you would probably rather work in pixels than clipspace so let's change the shader so we can supply the position in pixels and have it convert to clipspace for us. Here's the new vertex shader

对于 2D 这种情况，你完全可以使用像素单位来代替裁剪单位。这样，我们就需要换一种方式写 着色器。然后，我们就可以直接提供像素单位，并且将像素单位转换为裁剪单位。
下面是一种新的顶点着色器：
```
<script id="2d-vertex-shader" type="notjs">
 
  // 移除 attribute vec4 a_position;
  attribute vec2 a_position;
 
  uniform vec2 u_resolution;
 
  void main() {
    // 将像素单位映射到 0 和 1 之间
    vec2 zeroToOne = a_position / u_resolution;
 
    // 将0到1的映射，变为0到2的映射
    vec2 zeroToTwo = zeroToOne * 2.0;
 
    // 将0到2的映射，重新变为-1到1的映射(原来的裁剪坐标)
    vec2 clipSpace = zeroToTwo - 1.0;
 
    gl_Position = vec4(clipSpace，0，1);
  }
 
</script>
```

Some things to notice about the changes. We changed a_position to a vec2 since we're only using x and y anyway. A vec2 is similar to a vec4 but only has x and y.

上面，我们将 `a_position` 改为 `vec2` 类型。vec2 和 vec4 间的区别在于，vec2 只有 x，y 两个坐标。

Next we added a uniform called u_resolution. To set that we need to look up its location.

接着，我们添加一个 `uniform` --> `u_resolution`。不过，我们需要解析它的位置。
```
var resolutionUniformLocation = gl.getUniformLocation(program，"u_resolution");
```

The rest should be clear from the comments. By setting u_resolution to the resolution of our canvas the shader will now take the positions we put in positionBuffer supplied in pixels coordinates and convert them to clip space.

剩下的部分，我们从注释中应该能大致看懂。通过 `u_resolution` 获取到 canvas 的分辨率之后，着色器就会直接从我们提供的 `positionBuffer` 里面获取的像素坐标，并把它们转换位裁剪坐标。

现在，我们就可以直接使用像素值了。接下来，我们来一个矩形。该矩形有两个三角形，六个点组成。
```
var positions = [
  10，20，
  80，20，
  10，30，
  10，30，
  80，20，
  80，30，
];
gl.bufferData(gl.ARRAY_BUFFER，new Float32Array(positions)，gl.STATIC_DRAW);
```

And after we set which program to use we can set the value for the uniform we created. Use program is like gl.bindBuffer above in that it sets the current program. After that all the gl.uniformXXX functions set uniforms on the current program.

当我们设置了需要调用的程序之后，就需要给 uniform 赋值了。和 `gl.bindBuffer` 方法类似，我们使用 `gl.uniformXXX` 方法给当前程序中 uniform 赋值。
```
gl.useProgram(program);
 
// 设置分辨率
gl.uniform2f(resolutionUniformLocation，gl.canvas.width，gl.canvas.height);
```
为了画两个三角形，我们需要调用顶点着色器6次，所以，下面的 count 需要设置为 6。
```
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 6;
gl.drawArrays(primitiveType，offset，count);
```
结果如图:

![rectangle][5]

[查看具体的网页][6]

Again you might notice the rectangle is near the bottom of that area. WebGL considers the bottom left corner to be 0,0. To get it to be the more traditional top left corner used for 2d graphics APIs we can just flip the clip space y coordinate.

另外，你可能会注意到，矩形现在位于画布的地步。WebGL 会将 左下角作为 0，0 点。为了让它和传统 2d 图像 APIs参考的左上方原点统一，我们将 y 坐标值坐一次反转。

```
gl_Position = vec4(clipSpace * vec2(1, -1), 0, 1);
```

And now our rectangle is where we expect it.

现在，矩形和我们平常见到的一样了。

![top_left_corner][7]

[查看具体的网页][8]

Let's make the code that defines a rectangle into a function so we can call it for different sized rectangles. While we're at it we'll make the color settable.

我们可以将画矩形的代码写为函数，这样，我们就能很快的画出不同大小的矩形。不过，在我们做之前，需要确定一下颜色。

```
<script id="2d-fragment-shader" type="notjs">
  precision mediump float;
 
  uniform vec4 u_color;
 
  void main() {
    gl_FragColor = u_color;
  }
</script>
```
下面，我们来实践一下，画出50个不同颜色，不同位置的矩形。
```
  var colorLocation = gl.getUniformLocation(program，"u_color");
  ...
 
  // 设置矩形的随机位置
  for (var ii = 0; ii < 50; ++ii) {
    // Setup a random rectangle
    setRectangle(
        gl，randomInt(300)，randomInt(300)，randomInt(300)，randomInt(300));
 
    // 设置随机的颜色
    gl.uniform4f(colorLocation，Math.random()，Math.random()，Math.random()，1);
 
    // 画矩形
    gl.drawArrays(gl.TRIANGLES，0，6);
  }
}
 
// Returns a random integer from 0 to range - 1.
// 返回 0到 （range-1）之间的整数
function randomInt(range) {
  return Math.floor(Math.random() * range);
}
 
// 封装好的画矩形的函数
 
function setRectangle(gl，x，y，width，height) {
  var x1 = x;
  var x2 = x + width;
  var y1 = y;
  var y2 = y + height;
 
  // 注意: gl.bufferData(gl.ARRAY_BUFFER，...) 都会影响到，
  // 任何绑定在 `ARRAY_BUFFER` 上的 buffer。但现在，我们就只有一个 buffer
  // 如果，我们有多个 buffer 需要绑定的话，那么这些 buffer 需要提前绑定到 `ARRAY_BUFFER` 上
 
 
  gl.bufferData(gl.ARRAY_BUFFER，new Float32Array([
     x1，y1，
     x2，y1，
     x1，y2，
     x1，y2，
     x2，y1，
     x2，y2])，gl.STATIC_DRAW);
}
```
下面是用代码画出来的矩形图:

![random_rectangle][9]

[查看具体的网页][10]

WebGL 看起来确实提供了挺简单的 API。简单吗？ 因为我们这里仅仅使用 2 user 提供的函数，顶点着色器 和 fragmetn shader。然后使用它们来画三角，线 或者点。如果你想画比较复杂的3D，那么就需要更复杂的 shaders。WebGL API 只是一个光栅化并且原理也挺简单(坐标经过光栅化，转化为逐像素的数据)。

上面，我们已经讲解了怎样将数据给一个 attribute 和2个 uniforms。这是比较简单的，当然，也存在多个 attribute 和 uniforms 的情况。在文章的开头，我们提到过 `varyings` 和 `textures`。这两个概念会在后面的章节里面进行阐述。

在继续讲解之前，我想说一点，像上面 setRectangle 那样，动态的更新 buffer 里面的数据并不常见。 我使用那个例子的，主要考虑到他本身比较简单，而且他使用的是像素坐标，并且做的数学运算也不多。当然，关于 WebGL 不止这一点点内容。你可以继续学习在 [WebGL 中的定位，方向和缩放的基本方式][11]。

如果，你对 WebGL，GLSL 和 shaders 不了解，或者不知道 GPU 是怎么运行的，你可以参考 [WebGL 工作原理][12]。

另外，你还需要简单的读一读[在很多例子中使用的模板代码][13]。简单的浏览一下[怎么去画不同的东西][14]，或许这能够给你一些启发，关于大部分 WebGL 应用是如何运行的。因为，我们很多例子仅仅只是画一个东西，并且，也没把大致的结构写出来。

接下来，你有两个学习方向。如果你对图片处理感兴趣，这里有如何对[图片进行处理][15]的相关内容。如果你更想学习平移，旋转，缩放等技能的话，[这更适合你][16]。

**补充知识**

----------

## type="nojs" 标签的意义

`<script>`标签最大的用途就是包裹 JavaScript 代码。当`<script>` 没有设置 type，或者设置 `type="javascript"` 或 `type="text/javascript"`，浏览器就会将里面的内容当做 JavaScript 代码来解析。如果，type 设置为其他的内容话，那么浏览器就会跳过该 `<script>` 标签。所以，我们设置的 `type="notjs"` 或者 `type="foobar"` 浏览器并不会关心里面的内容。

这让我们很容易对着色器进行编辑。另外，还有其他着色器的写法，比如字符串的拼接:
```
 var shaderSource =
    "void main() {\n" +
    "  gl_FragColor = vec4(1，0，0，1);\n" +
    "}";
```

或者，我们可以使用 ajax 动态加载着色器，不过，这有点慢并且不同步。

另外一种可选的方案，可以使用多行模板字面量(这是 es6 新提出的特性)
```
  var shaderSource = `
    void main() {
      gl_FragColor = vec4(1，0，0，1);
    }
  `;
```

多行模板字面量在所有的浏览器中，都已经得到支持。不过，低版本的就不支持了，所以，如果你需要向下兼容的话，那么就不能使用多行模板字面量或者使用转译器进行代替。

有问题? [在 stack overflow 上提问][17]。
Issue/Bug? [在 github 上创建 issue][18]。


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-resizing-the-canvas.html
  [3]: http://static.zybuluo.com/jimmythr/ojqdq6nck0hj9nwjmtg9yknn/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-21%2021.45.39.png
  [4]: http://webglfundamentals.org/webgl/webgl-fundamentals.html
  [5]: http://static.zybuluo.com/jimmythr/55um2fx97pm80c0kfoaxcs9p/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-22%2022.20.56.png
  [6]: http://webglfundamentals.org/webgl/webgl-2d-rectangle-top-left.html
  [7]: http://static.zybuluo.com/jimmythr/r0ndu8iohe37o219mxm4cvgj/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-30%2020.57.26.png
  [8]: http://webglfundamentals.org/webgl/webgl-2d-rectangle-top-left.html
  [9]: http://static.zybuluo.com/jimmythr/k4vb543kjesvsj162x3m8iv9/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-22%2022.38.33.png
  [10]: http://webglfundamentals.org/webgl/webgl-2d-rectangles.html
  [11]: http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html
  [13]: http://webglfundamentals.org/webgl/lessons/webgl-boilerplate.html
  [14]: http://webglfundamentals.org/webgl/lessons/webgl-drawing-multiple-things.html
  [15]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
  [16]: http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html
  [17]: http://stackoverflow.com/questions/tagged/webgl
  [18]: https://github.com/greggman/webgl-fundamentals/issues