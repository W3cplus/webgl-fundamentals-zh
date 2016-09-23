# WebGL Fundamentals

webGL 一直被人们和 Unity3D 联系起来. 认为 webGL 不是用来写 3D 的 web 网页吗？ 额... 没错. 但画不画 3D 是你自己来决定的和 webGL 并没太大的关系, 它只是个栅格化引擎. 什么是栅格化？ 实际上就是坐标, 你通过 code 来定位点,线,三角 都是基于这一个坐标系来的. 所以, webGL 并不是提供什么高端 API,让你一下就能做出高逼格的 3D 动画, 记住,他只是个引擎, 3D 还是 2D 是由你来决定的.

webGL 的高性能, 主要是因为他可以调用 GPU 进行动画的加速和优化. 当然, 这也是 css3 动画的一大特性. 如果, 你想写 webGL, 那么最重要的两个概念: vertex shader(顶点着色器) 和 fragment shader(片段着色器) , 是你必须要搞清楚的. 他俩是借鉴的是一种非常严格的类 C/C++ 语言--[GLSL][1] (GL Shader Language). 他们两者组合在一起,就是我们通常写的 webGL 程序. 

vertex shader 主要任务就是计算点的位置. 找到点后, 根据你提供的函数, 就可以画出各种原始图形,比如, 点,线,三角等. 当在渲染图形的时, 接着就调用你提供的另外一个函数--fragment shader, 他来决定图形上每个像素点,到底是什么颜色.

几乎所有的 webGL API 都是组合上面两个成对的函数(vertex && fragment) 然后,画出一个图像当前的状态. 你可以通过调用 `gl.drawArrays` 或者 `gl.drawElements`, 然后, webGL 就会基于 GPU, 执行你事先设置的一些列图像的状态. 

如果你想提供一些具体的数据给函数渲染, 那么你首先得让 GPU 看懂. 比如, 位置, 颜色,角度等等. 这里, 你可以通过四种方式,将你的数据发送给 shader:

1. 属性和Buffers
Buffers 这个类型在 js 中已经出现很久了, 他就是一个数组,用来存放二进制数据. 而, GPU 也主要接收的数据就是 Buffer. 通常, 每一个 Buffer 包含的数据信息有, 位置, 法线, 纹理坐标, 颜色 等等. 当然, 你可以放任何你想放的数据在里面, 只要 GPU 看得懂(不过, 一般情况这些已经 enough).
属性则是用来定义如何获取你的 data , buffers 具体内容和提供给 vertex shader. 比如, 你想把 3 个 32bit 的浮点数据作为位置(x,y,z), 放在 Buffer 里面. 那么,你就得提供特定的属性,告诉哪个 Buffer 里面有这些数据？哪种数据类型应该提取(比如,3个浮点类型的数据)？ 数据在 Buffer 的具体位置在哪？现在的位置距离下个位置的间隔是多少？
Buffers 并不是随机获取的. 另外, vertex shaders 会根据提供的数据执行特定的次数. 每次下一个值从指定的 Buffer 中提取,指定给某个属性并执行.

2. Uniforms
Uniforms 是一个有效的全局变量. 我们会在 shader 程序执行前, 先把他给定义了. 
3. Textures
Textures(纹理) 是一串数组, 你可以在你的 shader 程序中的任何位置访问他. 我们最经常做的事情,就是获取图片数据. 不过, textures 不仅仅只是数据, 还可以包含除颜色以外的其他数据.
4. Varyings
vertex shader 常常利用 Varyings 变量将数据传给 fragment shader. 在 vertex shader 设置的 varyings 变量值, 通常取决于被渲染的基本图形是啥, 比如有, 点,线,三角形. varyings 会在 fragment shader 执行时, 插入进去.

上面的概念很干... 我们用幅图具体表示一下.

(FROM [叶斋][2])

![webGL][3]

webGL 具体的概念内容,就是上面所叙述的.

## WebGL Hello World

WebGL 只关心两个东西. 裁剪坐标系(通常理解为 xyz 坐标系)和颜色. 而我们程序员也只需要向 WebGL 提供者两样东西. 实际上就是上文提到的 vertex shader 和 fragment shader. vertex shader 代表裁剪坐标系, fragment shader 代表颜色.

裁剪坐标系取值范围永远是 -1 到 1, 不管你的canvas有多大. 下列是一个精简版的 WebGL 例子:

我们先看一下 vertex shader
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
  0, 0, 0, 0,
  0, 0.5, 0, 0,
  0.7, 0, 0, 0,
];
var attributes = {};
var gl_Position;
 
drawArrays(..., offset, count) {
  var stride = 4;
  var size = 4;
  for (var i = 0; i < count; ++i) {
     // 从 positionBuffer 获取 4个为一个单位的坐标值给a_position
     attributes.a_position = positionBuffer.slice((offset + i) * stride, size);
     runVertexShader();
     ...
     doSomethingWith_gl_Position();
}
```
实际的代码可能比这要复杂一点, 因为 `positionBuffer` 需要被转换为二进制数据. 所以, 从 buffer 里面获取数据的计算可能会有点不同. 不过, 这应该能够说清楚 vertex 到底是怎么执行的.

接着, 来看一下 fragment shader.
```
// fragment shaders 没有默认精度, 所以我们需要手动指定一个. 中等精度就ok. 设置为 "mediump".
// 实际上就是 "medium precision"
precision mediump float;
 
void main() {
  // gl_FragColor 是一个变量
  gl_FragColor = vec4(1, 0, 0.5, 1); // 返回某一种紫色
}
```

上面, 我们设置的gl_FragColor变量值是(1, 0, 0.5, 1). 这实际上就是 rgba 的表达形式. 相信同学们在使用 background 设置背景色时已经见过了. 只是范围不同.

 - 第一位: red
 - 第二位: green
 - 第三位: blue
 - 第四位: apha, 即透明度

上面的范围都是从 0-1.

现在, 我们已经了解了两个 shader 的流程. 接下来,我们正式使用前端语言来表达一下吧.

首先, 我们需要一个 canvas
```
<canvas id="c" width="400" height="300"></canvas>
```
然后在 js 里面获取它
```
var canvas = document.getElementById("c");
```
接着,创建一个 WebGLRenderingContext
```
 var gl = canvas.getContext("webgl");
 if (!gl) {
    // webgl 不存在
```
现在, 我们需要编译这些 shaders 来让他们在 GPU 上运行. 所以, 为了编译, 我们需要将这些 shaders 变为字符串. 比如, 通过字符串拼接, AJAX 拉取,使用多行的字符串模板, 或者 按照下列的写法, 将 shader 使用非 JavaScript 的标签进行包裹.
```
<script id="2d-vertex-shader" type="notjs">

  void main() {
    gl_Position = a_position;
  }
 
</script>
 
<script id="2d-fragment-shader" type="notjs">
  precision mediump float;
  void main() {
    gl_FragColor = vec4(1, 0, 0.5, 1); // return redish-purple
  }
</script>
```
实际上, 大多数的3D引擎会使用各种方式,比如,模板,拼接 动态的生成 GLSL shaders. 不过, 在本教程中, 没有哪个 demo 复杂到需要动态生成 GLSL
 
下一步,我们就需要写一个函数. 用它去创建 shader, 提交 GLSL 代码, 并且编译 shader. 下面函数,我没有写什么注释, 因为从命名上来看, 他已经很清楚了.

```
function createShader(gl, type, source) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (success) {
    return shader;
  }
 
  console.log(gl.getShaderInfoLog(shader));
  gl.deleteShader(shader);
}
```
现在,我们需要使用该函数去创建两个 shader.
```
var vertexShaderSource = document.getElementById("2d-vertex-shader").text;
var fragmentShaderSource = document.getElementById("2d-fragment-shader").text;
 
var vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
var fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);
```
接着,我们需要将两个 shader 和实际的程序结合起来.
```
function createProgram(gl, vertexShader, fragmentShader) {
  var program = gl.createProgram();
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  var sucesss = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (success) {
    return program;
  }
 
  console.log(gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
}
```
然后, 调用他.
```
var program = createProgram(gl, vertexShader, fragmentShader);
```
现在,我们已经在 GPU 上,写好的 GLSL 代码. 接下来我们要做的事情就是使用 WebGL 提供相关的数据给 GLSL. 在下面的这个例子中, 我们只向 GLSL 程序提供一个 `a_position` 的属性. 首先, 我们需要提供位置属性, 给我们刚刚创建的程序.
```
var positionAttributeLocation = gl.getAttribLocation(program, "a_position");
```
我们需要在初始化时提供位置属性(和统一位置), 而不是在渲染时才提供.

接着,我们需要创建一个 buffer 来给属性提供值.
```
var positionBuffer = gl.createBuffer();
```
WebGL 能让我们在全局绑定点上, 使用 WebGL 的大部分资源. 你可以认为, 绑定点在 WebGL 中,就是一个个全局变量. 首先, 你绑定一个资源给一个点. 然后, 涉及到该资源的所有方法都绑定到这个点上了. 所以, 我们需要先绑定一个位置 Buffer.
```
gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
```
现在, 我们需要把数据放到 buffer 中. 然后,通过这些数据找到具体的绑定点.
```
// 三个平面点的位置
var positions = [
  0, 0,
  0, 0.5,
  0.7, 0,
];
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
```
上面的代码看起来有点复杂. 首先, 我们使用 JavaScript 的 Array 结构, 去定义 `positions`. 我们都知道, js 是一门弱类型的语言, 但 WebGL 需要的是强类型的数据, 所以, 我们需要 `Float32Array(positions)` 去转换 js 原生的 Array 为特定的 32bit 的浮点数. 此时, 在 GPU 上, ` gl.bufferData` 将位置信息给了 `positionBuffer`. 上面, 我们已经将 `positionBuffer` 绑定给了 `ARRAY_BUFFER`.

最后一个参数 `gl.STATIC_DRAW` 的作用, 是告诉 WebGL , 我们这个数据是什么行为. 有没有经常变动, 还是, 一个常量呢？ WebGL 能够根据这个参数去做一些优化. `gl.STATIC_DRAW` 的意思告诉, WebGL 按照常量的方式,来处理这次传入的数据.

我们现在已经把数据放在 Buffer 里, 但我们还需要告诉属性,怎么将数据从 Buffer 中取出来. 我们先绑定一个属性.
```
gl.enableVertexAttribArray(positionAttributeLocation);
```
然后, 我们写一下提取数据的 程序.
```
var size = 2;          // 每次迭代获取2个数据
var type = gl.FLOAT;   // 数据的类型是32位的浮点数
var normalize = false; // 不需要格式化数据
var stride = 0;        // 数据与数据之间的间隔是0. 间隔计算就是 size * sizeof(type)
var offset = 0;        // 有效数据在 buffer 中开始的位置
gl.vertexAttribPointer(
    positionAttributeLocation, size, type, normalize, stride, offset)
```
gl.vertexAttribPointer 这一步将 `ARRAY_BUFFER` 绑定给了属性值. 同理, 属性值也就可以从 `positionBuffer` 里获取数据了.另外, 我们也不需要额外给 `ARRAY_BUFFER` 再绑定啥了, 这个属性会直接使用 `positionBuffer` 里的数据.

注意,我们在上面写的 GLSL vertex shader. `a_position` 属性是一个 vec4 类型.
```
attribute vec4 a_position;
```
`vec4` 代表的是四个浮点数组合的对象. 在 JavaScript 里, 我们可以认为 `a_position = {x: 0, y: 0, z: 0, w: 1}`. 而默认值 vec4 的默认值实际上是 `0, 0, 0, 1`. 上面,我们将 `size = 2`. 那么 `a_position` 会从 buffer 中获取前两个数据(x,y), 而,剩下的 z 和 w 则会设为默认值 0 ,1.

另外, 我们需要告诉 WebGL, 怎样将 `gl_Position` 设置的裁剪空间(三维坐标)的值, 转换为平面空间(二维坐标)的值. 这里, 我们需要使用 `gl.viewport` 来帮助我们完成这件事.
```
gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
```
这一句就是用来设置 WebGL viewport 的大小. 他的实际格式为:
```
gl.viewport(x, y, width, height);
```
实际上, WebGL 的默认 viewport 大小就是 canvas 的大小. 如果,我们没有显示的改变 canvas 的大小, 就没必要使用 viewport API.

最后, 我们需要告诉 WebGL 去执行哪一个 shader.
```
gl.useProgram(program);
```
然后, 让 WebGL 去执行我们整个的 GLSL 程序.
```
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 3;
gl.drawArrays(primitiveType, offset, count);
```
注意 drawArrays 的第三个参数 `count` 该使用指定渲染次数的. 第一次, `a_position.x` 和 `a_position.y` 被设置为上面提供的 `positionBuffer` 的前两个值. 第二次, 则是接下来的两个值.第三次, 就是最后两个值. 
```
// 这是上文提供的 positions
var positions = [
  0, 0,  // 第一次
  0, 0.5, // 第二次
  0.7, 0, // 第三次
];
```
因为我们一开始设置的 `primitiveType` 是 `gl.TRIANGLES`,WebGL 会根据我们提供给 `gl_Position` 的三个值画一个三角形. 不管 canvas 有多大, 只要给的值在 -1 和 1 之间,就没问题.

上面, vertex shader 只是简单的将 positionBuffer 的值赋给了 `gl_Position`. 所以, 这个三角形简单地绘制到裁剪的坐标系上.
```
  0, 0,
  0, 0.5,
  0.7, 0,
```
经过转换, WebGL 最终绘制在平面坐标系中的三角形坐标是:
```
// canvas 的大小为: 
// width: 400px
// height: 150px
 clip space      screen space
   0, 0       ->   200, 150
   0, 0.5     ->   200, 225
 0.7, 0       ->   340, 150
```
现在我们已经画好三角形了. 接着 WebGL 会调用 fragment shader 去渲染每一帧的颜色. 我们 fragment shader 设置的值为 ` 1, 0, 0.5, 1`. 因为, canvas 是 8bit 通道(即, 255色值). 最终, WebGL 渲染的颜色值就是 `[255, 0, 127, 255]`.

看个具体的例子:

![triangle][4]

[查看具体的网页][5]

上面的例子中, vertex shader 只是帮我们传递了位置数据,并没有做什么复杂的工作. 如果你想要3D的效果, 那么你需要提供一个 shader, 可以将 3D 转换为 裁剪空间的位置. 

看了上面的变换, 你可能在想 WebGL 的坐标系到底是怎么建立的. 首先, 在裁剪空间里, x 轴的范围是 -1 到 1. 零点在中心并且向右值增大.

对于纵坐标而言, -1 就是底部, 1 就是顶部. 零点在中心,向上值增大.

具体的图就是:

![axis][6]

对于 2D 这种情况, 你完全可以使用像素单位来代替裁剪单位. 这样, 我们就需要换一种方式写 `vertex shader`. 然后, 我们就可以直接提供像素单位了. 不过, 后面还是需要将像素单位转换为裁剪单位.
下面是, 另外一种 `vertex shader` 写法:
```
<script id="2d-vertex-shader" type="notjs">
 
  // 移除 attribute vec4 a_position;
  attribute vec2 a_position;
 
  uniform vec2 u_resolution;
 
  void main() {
    // 将像素单位映射到 0 和 1 之间
    vec2 zeroToOne = a_position / u_resolution;
 
    // 将0到1的映射,变为0到2的映射
    vec2 zeroToTwo = zeroToOne * 2.0;
 
    // 将0到2的映射,重新变为-1到1的映射(原来的裁剪坐标)
    vec2 clipSpace = zeroToTwo - 1.0;
 
    gl_Position = vec4(clipSpace, 0, 1);
  }
 
</script>
```
上面, 我们将 `a_position` 改为 `vec2` 类型. vec2 和 vec4 间的区别在于, vec2 只有 x,y 两个坐标.

接着, 我们添加一个 `uniform` --> `u_resolution`. 该变量主要就是获得屏幕的分辨率.
```
var resolutionUniformLocation = gl.getUniformLocation(program, "u_resolution");
```
通过 `u_resolution` 获取到 canvas 的分辨率之后, shader 就会直接从我们提供的 `positionBuffer` 里面获取的像素坐标, 并把它们转换位裁剪坐标.

现在, 我们就可以直接使用像素值了. 接下来, 我们来一个矩形. 该矩形有两个三角形, 六个点组成.
```
var positions = [
  10, 20,
  80, 20,
  10, 30,
  10, 30,
  80, 20,
  80, 30,
];
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
```
当我们设置了需要调用的程序之后,我们就需要给 uniform 赋值了. 和 `gl.bindBuffer` 方法类似,我们使用 `gl.uniformXXX` 方法给当前程序中 uniform 赋值.
```
gl.useProgram(program);
 
// 设置分辨率
gl.uniform2f(resolutionUniformLocation, gl.canvas.width, gl.canvas.height);
```
为了画两个三角形, 我们需要调用 vertex shader 6次,所以, 下面的 count 需要设置为 6.
```
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 6;
gl.drawArrays(primitiveType, offset, count);
```
结果如图:

![rectangle][7]

[查看具体的网页][8]

我们可以将画矩形的代码写为函数, 这样,我们就能很快的画出不同大小的矩形. 接着, 我们设置一下渲染的颜色.

```
<script id="2d-fragment-shader" type="notjs">
  precision mediump float;
 
  uniform vec4 u_color;
 
  void main() {
    gl_FragColor = u_color;
  }
</script>
```

下面, 我们来实践一下, 画出50个不同颜色,不同位置的矩形.
```
  var colorLocation = gl.getUniformLocation(program, "u_color");
  ...
 
  // 设置矩形的随机位置
  for (var ii = 0; ii < 50; ++ii) {
    // Setup a random rectangle
    setRectangle(
        gl, randomInt(300), randomInt(300), randomInt(300), randomInt(300));
 
    // 设置随机的颜色
    gl.uniform4f(colorLocation, Math.random(), Math.random(), Math.random(), 1);
 
    // 画矩形
    gl.drawArrays(gl.TRIANGLES, 0, 6);
  }
}
 
// Returns a random integer from 0 to range - 1.
// 返回 0到 （range-1）之间的整数
function randomInt(range) {
  return Math.floor(Math.random() * range);
}
 
// 封装好的画矩形的函数
 
function setRectangle(gl, x, y, width, height) {
  var x1 = x;
  var x2 = x + width;
  var y1 = y;
  var y2 = y + height;
 
  // 注意: gl.bufferData(gl.ARRAY_BUFFER, ...) 都会影响到,
  // 任何绑定在 `ARRAY_BUFFER` 上的 buffer. 但现在, 我们就只有一个 buffer
  // 如果,我们有多个 buffer 需要绑定的话, 那么这些 buffer 需要提前绑定到 `ARRAY_BUFFER` 上
 
 
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
     x1, y1,
     x2, y1,
     x1, y2,
     x1, y2,
     x2, y1,
     x2, y2]), gl.STATIC_DRAW);
}
```
下面是用代码画出来的矩形图:

![random_rectangle][9]

[查看具体的网页][10]

WebGL 看起来确实提供了挺简单的 API. 简单吗？ 因为我们这里仅仅使用 2 user 提供的函数, vertex shader 和 fragmetn shader. 然后使用它们来画三角, 线 或者点. 如果你想画比较复杂的3D, 那么就需要更复杂的 shaders. WebGL API 只是一个光栅化并且原理也挺简单(坐标经过光栅化，转化为逐像素的数据).

上面, 我们已经讲解了怎样将数据给一个 attribute 和2个 uniforms. 这是比较简单的, 当然, 也存在多个 attribute 和 uniforms 的情况. 在文章的开头, 我们提到过 `varyings` 和 `textures`. 这两个概念会在后面的章节里面进行阐述.

在继续讲解之前, 我想说一点, 像上面 setRectangle 那样,动态的更新 buffer 里面的数据并不常见.  我使用那个例子的, 主要考虑到他本身比较简单, 而且他使用的是像素坐标, 并且做的数学运算也不多. 当然, 关于 WebGL 不止这一点点内容. 你可以继续学习在 [WebGL 中的定位, 方向和缩放的基本方式][11].

如果, 你对 WebGL, GLSL 和 shaders 不了解, 或者不知道 GPU 是怎么运行的, 你可以参考 [WebGL 工作原理][12].

另外, 你还需要简单的读一读[在很多例子中使用的模板代码][13]. 简单的浏览一下[怎么去画不同的东西][14], 或许这能够给你一些启发,关于大部分 WebGL 应用是如何运行的. 因为, 我们很多例子仅仅只是画一个东西, 并且, 也没把大致的结构写出来.

接下来, 你有两个学习方向. 如果你对图片处理感兴趣, 这里有如何对[图片进行处理][15]的相关内容. 如果你更想学习平移, 旋转, 缩放等技能的话, [这更适合你][16].

**补充知识**

----------

## type="nojs" 标签的意义

`<script>`标签最大的用途就是包裹 JavaScript 代码. 当`<script>` 没有设置 type, 或者设置 `type="javascript"` 或 `type="text/javascript"`, 浏览器就会将里面的内容当做 JavaScript 代码来解析. 如果, type 设置为其他的内容话, 那么浏览器就会跳过该 `<script>` 标签. 所以, 我们设置的 `type="notjs"` 或者 `type="foobar"` 浏览器并不会关心里面的内容.

这让我们很容易对 shaders 进行编辑. 其他 shader 的写法,比如字符串的拼接:
```
 var shaderSource =
    "void main() {\n" +
    "  gl_FragColor = vec4(1,0,0,1);\n" +
    "}";
```

或者,我们可以使用 ajax 动态加载 shaders, 不过,这有点慢并且不同步.

另外一种可选的方案,可以使用多行模板字面量(这是 es6 新提出的特性)
```
  var shaderSource = `
    void main() {
      gl_FragColor = vec4(1,0,0,1);
    }
  `;
```

多行模板字面量在所有的浏览器中,都已经得到支持. 不过,低版本的就不支持了, 所以, 如果你需要向下兼容的话, 那么就不能使用多行模板字面量或者使用转译器进行代替.

有问题? [在 stack overflow 上提问][17].
Issue/Bug? [在 github 上创建 issue][18].


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html
  [2]: http://taobaofed.org/blog/2015/12/21/webgl-handbook/
  [3]: https://gw.alicdn.com/tps/TB1H9xALXXXXXaYXXXXXXXXXXXX-567-463.png
  [4]: http://static.zybuluo.com/jimmythr/ojqdq6nck0hj9nwjmtg9yknn/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-21%2021.45.39.png
  [5]: http://webglfundamentals.org/webgl/webgl-fundamentals.html
  [6]: http://static.zybuluo.com/jimmythr/i8j4qbqrpw2kg5ys5cmrm6qq/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-21%2022.10.49.png
  [7]: http://static.zybuluo.com/jimmythr/55um2fx97pm80c0kfoaxcs9p/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-22%2022.20.56.png
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