# WebGL How It Works

# WebGL 工作原理


This is a continuation from WebGL Fundamentals。Before we continue I think we need to discuss at a basic level what WebGL and your GPU actually do。There are basically 2 parts to this GPU thing。The first part processes vertices (or streams of data) into clipspace vertices。The second part draws pixels based on the first part.

上一节我们主要讲解了 [WebGL 的基础][1]。在开始之前，我们来了解一下WebGL 和你的 GPU 底层的工作原理。GPU 主要做了两件事。第一个是转换顶点(或者说 数据流)到裁剪平面中。第二个就是基于前一部分来渲染图像。


When you call

当你调用:
```
gl.drawArrays(gl.TRIANGLE, 0, 9);
```
The 9 there means "process 9 vertices" so here are 9 vertices being processed。

后面的 9 代表着 "处理9个点"。所以，这里就有9个点被处理了。

![GPU process][2]


On the left is the data you provide。The vertex shader is a function you write in GLSL。It gets called once for each vertex。You do some math and set the special variable gl_Position with a clipspace value for the current vertex。The GPU takes that value and stores it internally.

左边是你提供的数据，vertex shader 是你用 [GLSL][3] 写出来的函数。他会在每个顶点被处理时调用。该点的值经过相应的数学运算，转换成为裁剪坐标的值，并且赋值给特殊的变量 `gl_Position`。GPU 会获得该值并保存。

Assuming you're drawing TRIANGLES，every time this first part generates 3 vertices the GPU uses them to make a triangle。It figures out which pixels the 3 points of the triangle correspond to，and then rasterizes the triangle which is a fancy word for “draws it with pixels”。For each pixel it will call your fragment shader asking you what color to make that pixel。Your fragment shader has to set a special variable gl_FragColor with the color it wants for that pixel.

当你在画三角形时，前一个部分每次会渲染出3个点，GPU 就可以利用这3个点去画一个三角形。GPU 会找到这3个点在图上对应着的像素点，然后渲染出一个三角形。接着，对于每个像素点，GPU 会调用你的 fragment shader，给相应的点加上颜色。 这个 fragment shader 实际就是一特殊的变量 `gl_FragColor`。该变量存储了每个像素的颜色值。


That’s all very interesting but as you can see in our examples to up this point the fragment shader has very little info per pixel。Fortunately we can pass it more info。We define “varyings” for each value we want to pass from the vertex shader to the fragment shader.

不过，在我们的例子中，fragment shader 并没有区分每一个点的颜色值。当然，办法是有的。我们可以定义 "varyings" 将每一个颜色值通过 vertex shader 赋给 fragment shader。

As a simple example，let's just pass the clipspace coordinates we computed directly from the vertex shader to the fragment shader.

看一个简单的例子，我们将从 vertex shader 计算出的裁剪坐标值传递给 fragment shader。

We'll draw with a simple triangle。Continuing from our previous example let's change our F to a triangle.

然后，画一个简单的三角形。我们修改[前面的例子][4]中的代码，将画 `F` 改为画一个三角形。
```
// 将三角形的像素值存储在 buffer 中
function setGeometry(gl) {
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
             0，-100,
           150， 125,
          -175， 100]),
      gl.STATIC_DRAW);
}
```
And we have to only draw 3 vertices.

我们只需要画三个点:
```
// 画场景
function drawScene() {
  ...
  // 画几何图形
  gl.drawArrays(gl.TRIANGLES，0，3);
}
```
Then in our vertex shader we declare a varying to pass data to the fragment shader.

在 vertex shader 中，我们声明一个 `varying` 将数据传递给 fragment shader。
```
varying vec4 v_color;
...
void main() {
  // 使用 matrix 乘以该位置
  gl_Position = vec4((u_matrix * vec3(a_position，1)).xy，0，1);
 
  // 从 裁剪空间值 转换为 颜色空间值
  // 裁剪空间值的范围为 -1.0 到 +1.0
  // 颜色空间的值从 0.0 到 1.0
  v_color = gl_Position * 0.5 + 0.5;
}
```
And then we declare the same varying in the fragment shader.

然后，我们在 fragment shader 中，声明一个一样的 `varying`。
```
precision mediump float;
 
varying vec4 v_color;
 
void main() {
  gl_FragColor = v_color;
}
```

WebGL will connect the varying in the vertex shader to the varying of the same name and type in the fragment shader.

WebGL 会自动关联在 vertex shader 和 fragment shader 存在的同名 varying。


Here's the working version.

具体效果,可以[查看网页][5].

![iamge][6]

Move，scale and rotate the rectangle。Notice that since the colors are computed from clipspace they don't move with the rectangle。They are relative to the background.

上面例子中，我们可以移动，缩放并且旋转该三角形。注意，因为这个颜色是直接根据裁剪空间来的，而不是根据三角形上的点来的。所以，他们不会随三角形一起移动，而是固定在背景中。

Now think about it。We only compute 3 vertices。Our vertex shader only gets called 3 times therefore it's only computing 3 colors yet our triangle is many colors。This is why it's called a varying.

想一想，我们只计算了3个点。所以，vertex shader 也只会调用3次，如果这样的话，我们也只能得出3种颜色，但是，我们的三角形却有很多颜色。这就是为什么我们需要一个 varying。

In the example above we start out with the 3 vertices

在上面的例子中，我们设置了3个点:

![vertices][7]

Our vertex shader applies a matrix to translate，rotate，scale and convert to clipspace。The defaults for translation，rotation and scale are translation = 200，150，rotation = 0，scale = 1,1 so that's really only translation。Given our backbuffer is 400x300 our vertex shader applies the matrix and then computes the following 3 clipspace vertices.

我们的 vertex shader 使用一个模型去移动,旋转,放缩 并且转化为 裁剪坐标的值。上述变化的默认值分别是：

 - 平移为 200,150
 - 旋转为 0
 - 放缩为 1,1

所以，只有平移有点不同。前面给出的 backbuffer 是 400×300，我们的vertex shader 会将其传递给模型，然后计算出下面3个裁剪坐标。
 
![gl_position][8]


It also converts those to colorspace and writes them to the varying v_color that we declared.

它同样会转换上述值到颜色空间，并且赋值给我们刚才声明的 varying v_color 变量中。

![v_color][9]


Those 3 values written to v_color are then interpolated and passed to the fragment shader for each pixel.

这三个值被写入 v_color 之后，会添加并且传递给 fragment shader 去渲染每个点的颜色.v_color 会插入在 v0,v1和v2 之间。

整个渲染过程,可以[查看具体网页][10].

![v_color][11]

We can also pass in more data to the vertex shader which we can then pass on to the fragment shader。So for example let's draw a rectangle，that consists of 2 triangles，in 2 colors。To do this we'll add another attribute to the vertex shader so we can pass it more data and we'll pass that data directly to the fragment shader.

我们可以传递更多的数据给 vertex shader，同样，也可以传递给 fragment shader。 如果我们想画一个矩形，就需要 2 个三角形，2个不同的颜色。在 vertex shader 中，我们还需要另外一个 attribute 去传递更多的数据。那么 fragment shader 也会处理更多的数据。

```
attribute vec2 a_position;
attribute vec4 a_color;
...
varying vec4 v_color;
 
void main() {
   ...
  // 把 attribute 中的颜色值复制到 varying 中
  v_color = a_color;
}
```
We now have to supply colors for WebGL to use.

我们现在向 WebGL 提供需要用到的颜色。

```
  // 找到 vertex data 绑定的位置
  var positionLocation = gl.getAttribLocation(program，"a_position");
  var colorLocation = gl.getAttribLocation(program，"a_color");
  ...
  // 创建一个 buffer 去加载颜色值
  var buffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER，buffer);
  gl.enableVertexAttribArray(colorLocation);
  gl.vertexAttribPointer(colorLocation，4，gl.FLOAT，false，0，0);
 
  // 设置颜色值
  setColors(gl);
  ...
 
// 给 buffer 填充2个三角形需要用到的颜色值
function setColors(gl) {
  // 随机选取2种颜色
  var r1 = Math.random();
  var b1 = Math.random();
  var g1 = Math.random();
 
  var r2 = Math.random();
  var b2 = Math.random();
  var g2 = Math.random();
 
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array(
        [ r1,b1,g1,1,
          r1,b1,g1,1,
          r1,b1,g1,1,
          r2,b2,g2,1,
          r2,b2,g2,1,
          r2,b2,g2,1]),
      gl.STATIC_DRAW);
}
```
[最终的结果][12].

![two_triangle][13]


Notice that we have 2 solid color triangles。Yet we're passing the values in a varying so they are being varied or interpolated across the triangle。It's just that we used the same color on each of the 3 vertices of each triangle。If we make each color different we'll see the interpolation.

注意，上面我们使用的是两个固定的颜色值。但是，我们是将值赋给 varying，所以，三角形中的颜色值是可以变化的。我们在上面给每个三角形的3个点设置的是相同的颜色值，如果我们设置不同的值，将会看到不一样的图。
```
// 给 buffer 填充2个三角形需要用到的颜色值
function setColors(gl) {
  // 给每个顶点设置不同的颜色
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array(
        [ Math.random(),Math.random(),Math.random(),1,
          Math.random(),Math.random(),Math.random(),1,
          Math.random(),Math.random(),Math.random(),1,
          Math.random(),Math.random(),Math.random(),1,
          Math.random(),Math.random(),Math.random(),1,
          Math.random(),Math.random(),Math.random(),1]),
      gl.STATIC_DRAW);
}
```
现在，我们能看见[不一样的地方][14]。

![varying_color][15]


Not very exciting I suppose but it does demonstrate using more than one attribute and passing data from a vertex shader to a fragment shader。If you check out the image processing examples you'll see they also use an extra attribute to pass in texture coordinates.

看起来和第一个例子差不多，但是我们了解了使用更多的 attribute 将数据由 vertex shader 传递给 fragment shader。如果你看了[图片处理的例子][16]的话，你会发现那里用了一个额外的 attribute 去传递纹理坐标。

## buffer 和 attribute 做了什么？

Buffers are the way of getting vertex and other per vertex data onto the GPU. gl.createBuffer creates a buffer. gl.bindBuffer sets that buffer as the buffer to be worked on. gl.bufferData copies data into the buffer.

Buffers 用来获取顶点和其他点的数据，并传输给 CPU。 `gl.createBuffer` 创建一个 buffer。 `gl.bindBuffer` 绑定需要处理的 buffer。`gl.bufferData` 将数据拷贝到指定的 buffer。

Once the data is in the buffer we need to tell WebGL how to get data out of it and provide it to the vertex shader's attributes.

一旦数据在 buffer 中准备好了，我们就需要告诉 WebGL 如何将数据提取出来并且传递给 vertex shader 属性.

To do this, first we ask WebGL what locations it assigned to the attributes. For example in the code above we have

为了完成上述过程，首先，我们先要了解 WebGL 把哪些 locations 赋值给了 attributes。比如，在上面的代码中:

```
// 解析 vertex 数据的流向
var positionLocation = gl.getAttribLocation(program,"a_position");
var colorLocation = gl.getAttribLocation(program,"a_color");
```

Once we know the location of the attribute we then issue 2 commands。

一旦我们知道了 attribute 的相关 location，就需要调用2个命令。

```
gl.enableVertexAttribArray(location);
```
This command tells WebGL we want to supply data from a buffer.

这个命令告诉 WebGL，我们通过 buffer 提供的数据。

```
gl.vertexAttribPointer(
    location,
    numComponents,
    typeOfData,
    normalizeFlag,
    strideToNextPieceOfData,
    offsetIntoBuffer);
```

And this command tells WebGL to get data from the buffer that was last bound with gl.bindBuffer, how many components per vertex (1 - 4), what the type of data is (BYTE, FLOAT, INT, UNSIGNED_SHORT, etc...), the stride which means how many bytes to skip to get from one piece of data to the next piece of data, and an offset for how far into the buffer our data is.

这个命令让 WebGL 从刚才通过 `gl.bindBuffer` 绑定的 buffer 中提取数据。该 buffer 包含了每个顶点的组成部分(1 - 4)，具体数据的类型是什么 (BYTE, FLOAT, INT, UNSIGNED_SHORT, etc...)，每个有效数据之间的步长是多少，真实数据在 buffer 中的偏移量是多少。

Number of components is always 1 to 4.

每个顶点的组成部分一般都是 1 到 4。

If you are using 1 buffer per type of data then both stride and offset can always be 0. 0 for stride means "use a stride that matches the type and size". 0 for offset means start at the beginning of the buffer. Setting them to values other than 0 is more complicated and though it has some benefits in terms of performance it's not worth the complication unless you are trying to push WebGL to its absolute limits.

如果你使用 1 buffer 的单一类型的数据的话，那么步长和偏移量总是 0。步长为 0 意味着 “每个步长包括了数据的类型和大小”。偏移量为 0 意味着数据从 buffer 的起始位开始. 将他们设置为其他值而不是 0 来说, 会更复杂，尽管这样做在性能方面有些好处。不过，相对于复杂度来说，这并不值得，除非你是想让 WebGL 受它绝对的限制。

I hope that clears up buffers and attributes.

我希望上面解决了 buffers 和 attributes 怎样工作的问题。

Next let's go over shaders and GLSL.

下面，让我们接着学习 [shaders and GLSL][17]。


----------
**补充**

## normalizeFlag 在 vertexAttribPointer 中代表什么？

The normalize flag is for all the non floating point types. If you pass in false then values will be interpreted as the type they are. BYTE goes from -128 to 127, UNSIGNED_BYTE goes from 0 to 255, SHORT goes from -32768 to 32767 etc...

normalize flag 主要影响所有的非浮点类型。 如果你将它设置为 false， 那么类型的值还是保持不变。BYTE 的大小还是从 -128 到 127，UNSIGNED_BYTE 的大小从 0 到 255, SHORT 的大小从 -32768 到 32767 等等。

If you set the normalize flag to true then the values of a BYTE (-128 to 127) represent the values -1.0 to +1.0, UNSIGNED_BYTE (0 to 255) become 0.0 to +1.0. A normalized SHORT also goes from -1.0 to +1.0 it just has more resolution than a BYTE.

如果 normalize flag 设置为 true, 那么 BYTE (-128 到 127) 的大小变为 -1.0 到 +1.0，UNSIGNED_BYTE (0 到 255) 的大小变为 0.0 到 +1.0。 SHORT 的大小和 BYTE 一样也是 -1.0 到 +1.0, 不过他比 BYTE 的分辨率更高.

The most common use for normalized data is for colors. Most of the time colors only go from 0.0 to 1.0. Using a full float each for red, green, blue and alpha would use 16 bytes per vertex per color. If you have complicated geometry that can add up to a lot of bytes. Instead you could convert your colors to UNSIGNED_BYTEs where 0 represents 0.0 and 255 represents 1.0. Now you'd only need 4 bytes per color per vertex, a 75% savings.

归一化数据最常用在颜色值上。颜色值基本上都是从 0.0 到 1.0。 使用浮点数去表达红色，绿色，蓝色和透明度，每个顶点的颜色会花掉 16 bytes 的大小。如果你有更复杂的几何图形的话,那累积下来就很大了。如果，你能用 UNSIGNED_BYTEs 去表达你的颜色值， 比如 0 代表 0.0, 255 代表 1.0。那么，你仅仅只需要 4 bytes 去表达每个点的颜色值，这节省了 75% 的大小。

Let's change our code to do this. When we tell WebGL how to extract our colors we'd use

我们来改写一下代码。当我们告诉 WebGL 怎样去提取颜色时，我们需要使用

```
gl.vertexAttribPointer(colorLocation, 4, gl.UNSIGNED_BYTE, true, 0, 0);
```

And when we fill out our buffer with colors we'd use

然后, 当我们将颜色值填充到 buffer 里，我们需要使用

```
// 填充 2 个三角颜色值到 buffer 里
function setColors(gl) {
  // 选取 2 个随机颜色
  var r1 = Math.random() * 256; // 0 to 255.99999
  var b1 = Math.random() * 256; // these values
  var g1 = Math.random() * 256; // will be truncated
  var r2 = Math.random() * 256; // when stored in the
  var b2 = Math.random() * 256; // Uint8Array
  var g2 = Math.random() * 256;
 
  gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array(   // Uint8Array
        [ r1, b1, g1, 255,
          r1, b1, g1, 255,
          r1, b1, g1, 255,
          r2, b2, g2, 255,
          r2, b2, g2, 255,
          r2, b2, g2, 255]),
      gl.STATIC_DRAW);
}
```

Here's that sample.
下面是个[实例][18]。

![unsigned_colors][19]

有问题? [在 stack overflow 上提问][20]。
Issue/Bug? [在 github 上创建 issue][21]。


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [2]: http://webglfundamentals.org/webgl/lessons/resources/vertex-shader-anim.gif
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html
  [4]: http://webglfundamentals.org/webgl/lessons/webgl-2d-matrices.html
  [5]: http://webglfundamentals.org/webgl/webgl-2d-triangle-with-position-for-color.html
  [6]: http://static.zybuluo.com/jimmythr/b8vlmbiy0lznx53xb83m0xu8/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-25%2019.25.05.png
  [7]: http://static.zybuluo.com/jimmythr/adw0jtqkxsnz4uvpa267qjlr/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-24%2010.17.12.png
  [8]: http://static.zybuluo.com/jimmythr/sauoxvxpqladbt4jz8nvz17g/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-24%2010.17.56.png
  [9]: http://static.zybuluo.com/jimmythr/pbbjikfwyyt4n6phle3ffyvr/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-24%2010.18.34.png
  [10]: http://webglfundamentals.org/webgl/lessons/resources/fragment-shader-anim.html
  [11]: http://static.zybuluo.com/jimmythr/b79tktap4oxyq449uytkjblu/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-25%2019.28.18.png
  [12]: http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-2-colors.html
  [13]: http://static.zybuluo.com/jimmythr/thbg545hxy5sfvy16ay1080j/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-25%2019.29.32.png
  [14]: http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-random-colors.html
  [15]: http://static.zybuluo.com/jimmythr/54juuw1l3lw5myhc9t258fgt/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-25%2019.30.25.png
  [16]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
  [17]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html
  [18]: http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-2-byte-colors.html
  [19]: http://static.zybuluo.com/jimmythr/a9gcqxzknnp7fnh54ht8qpa5/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-25%2023.33.00.png
  [20]: http://stackoverflow.com/questions/tagged/webgl
  [21]: http://github.com/greggman/webgl-fundamentals/issues