# WebGL Shaders and GLSL

# WebGL 着色器和 GLSL

This is a continuation from WebGL Fundamentals. If you haven't read about how WebGL works you might want to read this first.

这篇文章是 [WebGL 基础][1]的续集。如果你还没有阅读过 WebGL 怎样工作的，你可以参考[这里][2]。

We've talked about shaders and GLSL but haven't really given them any specific details. I think I was hoping it would be clear by example but let's try to make it clearer just in case.

我们已经了解了着色器和 GLSL，但还没对其进行详细的说明。我希望通过例子能够让大家清楚的了解它，不过，以防万一，我们还是更加仔细的去了解它。

As mentioned in how it works WebGL requires 2 shaders every time you draw something. A vertex shader and a fragment shader. Each shader is a function. A vertex shader and fragment shader are linked together into a shader program (or just program). A typical WebGL app will have many shader programs.

在 [WebGL 的工作原理][3] 中，我们提到过，当你每次绘制图像时，WebGL 需要两个着色器。一个是顶点着色器，一个是片元着色器。顶点着色器和片元着色器在一个着色器程序中（或者称为 程序 ）。一个典型的 WebGL app 会包含很多着色器程序。

## 顶点着色器
A Vertex Shader's job is to generate clipspace coordinates. It always takes the form

顶点着色器的工作就是输出裁剪坐标。它的格式总是：
```
void main() {
   gl_Position = doMathToMakeClipspaceCoordinates
}
```
Your shader is called once per vertex. Each time it's called you are required to set the the special global variable, gl_Position to some clipspace coordinates.

在计算每个顶点时，都会调用你的着色器。每次它被调用时，你需要设置特殊的全局变量，gl_Position 给某些裁剪坐标系。

Vertex shaders need data. They can get that data in 3 ways.
顶点着色器需要有数据。它通常有 3 种方式可以得到数据。

 1. [Attributes][4] (从 buffers 中拉取的数据)
 2. [Uniforms][5] (在每一次绘制所有顶点时，保持不变的值)
 3. [Textures][6] (从像素/纹理中获得的数据)

### Attributes

The most common way is through buffers and attributes. How it works covered buffers and attributes. You create buffers,

最常用的方式是通过 buffers 和 attributes。 [WebGL 的工作原理][7]已经讲解了如何使用 buffers 和 attributes。首先创建 buffers，
```
var buf = gl.createBuffer();
```
put data in those buffers

然后将数据存放在 buffers 中
```
gl.bindBuffer(gl.ARRAY_BUFFER, buf);
gl.bufferData(gl.ARRAY_BUFFER, someData, gl.STATIC_DRAW);
```
Then, given a shader program you made you look up the location of its attributes,

接着, 假设有一个你已经写好的着色器程序，该需要去解析 attributes 的位置，
```
var positionLoc = gl.getAttribLocation(someShaderProgram, "a_position");
```
then tell WebGL how to pull data out of those buffers and into the attribute

然后，告诉 WebGL 如何将数据从 buffers 中提取出来，并赋给 attribute

```
// 从 buffer 中提取数据到 attribute
gl.enableVertexAttribArray(positionLoc);
 
var numComponents = 3;  // (x, y, z)
var type = gl.FLOAT;
var normalize = false;  // 设为默认值
var offset = 0;         // 从 buffer 的开始位置起
var stride = 0;         // 距下个顶点的步长
                        // 0 = 参考类型和节点数设置正确的步长值
 
gl.vertexAttribPointer(positionLoc, numComponents, type, false, stride, offset);
```
In WebGL fundamentals we showed that we can do no math in the shader and just pass the data directly through.

在 [WebGL 基础][8]里，我们可以直接传递数据而不用在着色器中做任何数学运算。
```
attribute vec4 a_position;
 
void main() {
   gl_Position = a_position;
}
```
If we put clipspace vertices into our buffers it will work.

如果我们在 buffers 中设置裁剪系的顶点坐标，上面才能正常运行。

Attributes can use float, vec2, vec3, vec4, mat2, mat3, and mat4 as types.

Attributes 的类型可以取: float, vec2, vec3, vec4, mat2, mat3, 和 mat4。

### Uniforms
For a vertex shader uniforms are values passed to the vertex shader that stay the same for all vertices in a draw call. As a very simple example we could add an offset to the vertex shader above

对于顶点着色器来说，uniforms 会传递给顶点着色器并且在一次绘制时，相对于所有的顶点保持不变。看一个简单的例子，我们给上面的顶点着色器加上一个偏移量
```
attribute vec4 a_position;
uniform vec4 u_offset;
 
void main() {
   gl_Position = a_position + u_offset;
}
```
And now we could offset every vertex by a certain amount. First we'd look up the location of the uniform

现在，我们可以将每个顶点加以一定的偏移量。首先，我们需要解析 uniform 的位置。
```
var offsetLoc = gl.getUniformLocation(someProgram, "u_offset");
```
And then before drawing we'd set the uniform

然后，在绘制之前，我们需要设置 uniform 的值
```
gl.uniform4fv(offsetLoc, [1, 0, 0, 0]);  // 向右偏移半个屏幕
```
Uniforms can be many types. For each type you have to call the corresponding function to set it.

Uniform 有很多类型。对于不同的类型，你需要调用相应的函数去设置他。

```
gl.uniform1f (floatUniformLoc, v);                 // 浮点类型
gl.uniform1fv(floatUniformLoc, [v]);               // 浮点数或浮点数组
gl.uniform2f (vec2UniformLoc,  v0, v1);            // vec2 类型
gl.uniform2fv(vec2UniformLoc,  [v0, v1]);          // vec2 或 vec2数组
gl.uniform3f (vec3UniformLoc,  v0, v1, v2);        // vec3
gl.uniform3fv(vec3UniformLoc,  [v0, v1, v2]);      // vec3 或 vec3数组
gl.uniform4f (vec4UniformLoc,  v0, v1, v2, v4);    // vec4
gl.uniform4fv(vec4UniformLoc,  [v0, v1, v2, v4]);  // vec4 或 vec4数组
 
gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ])  // mat2 或 mat2数组
gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ])  // mat3 或 mat3数组
gl.uniformMatrix4fv(mat4UniformLoc, false, [ 16x element array ])  // mat4 或 mat4数组
 
gl.uniform1i (intUniformLoc,   v);                 // int
gl.uniform1iv(intUniformLoc, [v]);                 // int 或 int数组
gl.uniform2i (ivec2UniformLoc, v0, v1);            // ivec2
gl.uniform2iv(ivec2UniformLoc, [v0, v1]);          // ivec2 或 ivec2数组
gl.uniform3i (ivec3UniformLoc, v0, v1, v2);        // ivec3
gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);      // ivec3 或 ivec3数组
gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);    // ivec4
gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]);  // ivec4 或 ivec4数组
 
gl.uniform1i (sampler2DUniformLoc,   v);           // sampler2D (纹理)
gl.uniform1iv(sampler2DUniformLoc, [v]);           // sampler2D 或 sampler2D数组
 
gl.uniform1i (samplerCubeUniformLoc,   v);         // samplerCube (纹理)
gl.uniform1iv(samplerCubeUniformLoc, [v]);         // samplerCube 或 samplerCube数组
```
There's also types bool, bvec2, bvec3, and bvec4. They use either the gl.uniform?f? or gl.uniform?i? functions.

另外，还有类型 book，bvec2，bvec3，和 bvec4。它们既可以使用 `gl.uniform?f?` 或者 `gl.uniform?i?` 方法。

Note that for an array you can set all the uniforms of the array at once. For example

注意，对于数组而言，你可以一次性设置所有的 unifomrs。比如：
```
// 在着色器中
uniform vec2 u_someVec2[3];
 
// 在 JavaScript 代码初始化时
var someVec2Loc = gl.getUniformLocation(someProgram, "u_someVec2");
 
// 在渲染时
gl.uniform2fv(someVec2Loc, [1, 2, 3, 4, 5, 6]);  // 设置完整的 u_someVec3 数组
```
But if you want to set individual elements of the array you must look up the location of each element individually.

但如果你想分别设置数组中的元素值，那么你需要找到每个元素的位置。

```
// 在 JavaScript 代码初始化时
var someVec2Element0Loc = gl.getUniformLocation(someProgram, "u_someVec2[0]");
var someVec2Element1Loc = gl.getUniformLocation(someProgram, "u_someVec2[1]");
var someVec2Element2Loc = gl.getUniformLocation(someProgram, "u_someVec2[2]");
 
// 在渲染时
gl.uniform2fv(someVec2Element0Loc, [1, 2]);  // set element 0
gl.uniform2fv(someVec2Element1Loc, [3, 4]);  // set element 1
gl.uniform2fv(someVec2Element2Loc, [5, 6]);  // set element 2
```
Similarly if you create a struct

如果你创建了一个结构，情况类似：
```
struct SomeStruct {
  bool active;
  vec2 someVec2;
};
uniform SomeStruct u_someThing;
```
you have to look up each field individually

你需要解析每个字段

```
var someThingActiveLoc = gl.getUniformLocation(someProgram, "u_someThing.active");
var someThingSomeVec2Loc = gl.getUniformLocation(someProgram, "u_someThing.someVec2");
```

### Textures
请参考[片元着色器中的纹理][9]

## 片元着色器
A Fragment Shader's job is to provide a color for the current pixel being rasterized. It always takes the form

一个片元着色器的工作是提供当前正渲染像素点的颜色。它的形式总为
```
precision mediump float;
 
void main() {
   gl_FragColor = doMathToMakeAColor;
}
```

Your fragment shader is called once per pixel. Each time it's called you are required to set the special global variable, gl_FragColor to some color.

在计算每个顶点时，都会调用你的着色器。每次它被调用时，你需要设置特殊的全局变量，gl_FragColor 去保存某些颜色。

Fragment shaders need data. They can get data in 3 ways

片元着色器需要获得数据。它通常有 3 中方式可以获得：

 1. [Uniforms][10] (在每一次绘制所有顶点时，保持不变的值)
 2. [Textures][11] (从像素/纹理中获得的数据)
 3. [Varyings][12] (从顶点着色器中获得数据并插入)

### Uniforms
请参考: [顶点着色器中的 Uniforms][13]

### Textures
Getting a value from a texture in a shader we create a sampler2D uniform and use the GLSL function texture2D to extract a value from it.

我们创建一个 sampler2D uniform 从着色器中的纹理提取数据，并且使用 GLSL 的 texture2D 函数再从 uniform 从提取数据。
```
precision mediump float;
 
uniform sampler2D u_texture;
 
void main() {
   vec2 texcoord = vec2(0.5, 0.5)  // 得到纹理的中间值
   gl_FragColor = texture2D(u_texture, texcoord);
}
```
What data comes out of the texture is dependent on many settings. At a minimum we need to create and put data in the texture, for example

从纹理中提取哪些数据依赖很多设置项。但最少，我们应该创建并且将数据存放在纹理中，例如：
```
var tex = gl.createTexture();
gl.bindTexture(gl.TEXTURE_2D, tex);
var level = 0;
var width = 2;
var height = 1;
var data = new Uint8Array([255, 0, 0, 255, 0, 255, 0, 255]);
gl.texImage2D(gl.TEXTURE_2D, level, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, data);
```
Then look up the uniform location in the shader program
然后，在着色器程序中找到 uniform 的位置。
```
var someSamplerLoc = gl.getUniformLocation(someProgram, "u_texture");
```
WebGL then requires you to bind it to a texture unit

接着, WebGL 需要你手动绑定一个纹理单位
```
var unit = 5;  // 选择一个纹理单位
gl.activeTexture(gl.TEXTURE0 + unit);
gl.bindTexture(gl.TEXTURE_2D, tex);
```
And tell the shader which unit you bound the texture to

然后，告诉着色器你给纹理绑定的是哪一个单位。
```
gl.uniform1i(someSamplerLoc, unit);
```

### Varyings
A varying is a way to pass a value from a vertex shader to a fragment shader which we covered in how it works.

varying 可以将顶点着色器的值传递给片元着色器，我们在 [WebGL 的工作原理][14]里面已经讲解过。

To use a varying we need to declare matching varyings in both a vertex and fragment shader. We set the varying in the vertex shader with some value per vertex. When WebGL draws pixels it will interpolate between those values and pass them to the corresponding varying in the fragment shader

使用varying时，需要注意，我们要在顶点着色器和片元着色器中声明相同的 varyings。在顶点着色器中，我们通过 varying 给每个顶点设置值。当 WebGL 在绘制像素时，它将会这些值插入并且传递给片元着色器的相应 varying 中。

顶点着色器
```
attribute vec4 a_position;
 
uniform vec4 u_offset;
 
varying vec4 v_positionWithOffset;
 
void main() {
  gl_Position = a_position + u_offset;
  v_positionWithOffset = a_position + u_offset;
}
```
片元着色器
```
precision mediump float;
 
varying vec4 v_positionWithOffset;
 
void main() {
  // 将裁剪空间坐标(-1 <-> 1)转化为颜色空间值(0 -> 1)
  vec4 color = v_positionWithOffset * 0.5 + 0.5
  gl_FragColor = color;
}
```

The example above is a mostly nonsense example. It doesn't generally make sense to directly copy the clipspace values to the fragment shader and use them as colors. Nevertheless it will work and produce colors.

上面是一个没有多大意义的例子。它并没有直接复制裁剪坐标的值给片元着色器并且当做颜色值使用。但他却可以正常工作并且输出颜色。

## GLSL
GLSL stands for Graphics Library Shader Language. It's the language shaders are written in. It has some special semi unique features that are certainly not common in JavaScript. It's designed to do the math that is commonly needed to compute things for rasterizing graphics. So for example it has built in types like vec2, vec3, and vec4 which represent 2 values, 3 values, and 4 values respectively. Similarly it has mat2, mat3 and mat4 which represent 2x2, 3x3, and 4x4 matrices. You can do things like multiply a vec by a scalar.

GLSL 全称是 Graphics Library Shader Language。 着色器就是用这种语言写的。他有一些特殊的半独特的功能，这些功能在 JavaScript 里面是不常见的。它设计的初衷就是做一些数学运算，将计算的结果用来栅格化图像。所以，例如，他有一些内置的类型，像 vec2，vec3，和 vec4，分别代表着 2 个值，3个值，和4个值。另外，他还有 mat2，mat3 和 mat4 分别代表着 2x2, 3x3 和 4x4 的矩阵。当然，你还可以做一些数学运算，比如使用一个常量乘以一个向量 (vector)。
```
vec4 a = vec4(1, 2, 3, 4);
vec4 b = a * 2.0;
// b 现在是 vec4(2, 4, 6, 8);
```
Similarly it can do matrix multiplication and vector to matrix multiplication

另外，你还可以做矩阵之间的乘法，和 向量与矩阵的乘法。
```
mat4 a = ???
mat4 b = ???
mat4 c = a * b;
 
vec4 v = ???
vec4 y = c * v;
```
It also has various selectors for the parts of a vec. For a vec4

一个 vec 有不同的选择器。对于 vec4 而言：

 - v.x 等同于 v.s 等同于 v.r 等同于 v\[0]
 - v.y 等同于 v.t 等同于 v.g 等同于 v\[1]
 - v.z 等同于 v.p 等同于 v.b 等同于 v\[2]
 - v.w 等同于 v.q 等同于 v.a 等同于 v\[3]

我们可以调配 (swizzle) vec 的组件。换句话说，我们可以交换或者重复某一个组件。
```
v.yyyy
```
等同于
```
vec4(v.y, v.y, v.y, v.y)
```
又例如：
```
v.bgra
```
等同于
```
vec4(v.b, v.g, v.r, v.a)
```
when constructing a vec or a mat you can supply multiple parts at once. So for example

当在组成里一个 vec 或者 mat 时，你可以提供多个部分。比如：
```
vec4(v.rgb, 1)
```
等同于
```
vec4(v.r, v.g, v.b, 1)
```
One thing you'll likely get caught up on is that GLSL is very type strict.

有一件事你需要注意一下，GLSL 是强类型语言。
```
float f = 1; // ERROR 1 是整数。你不能将整数赋值给浮点数。
```

正确的格式如下：

```
float f = 1.0;      // 使用浮点数
float f = float(1)  // 将整数转换为浮点数
```
The example above of vec4(v.rgb, 1) doesn't complain about the 1 because vec4 is casting the things inside just like float(1).

上面例子中的 vec4(v.rgb, 1) 里面也使用了整数 1 但并没有报错，主要是因为 vec4() 在内部将参数进行了转化，像 float(1)。

GLSL has a bunch of built in functions. Many of them operate on multiple components at once. So for example

GLSL 有很多内置的函数。其中有一大部分会在不同的执行部分时，被调用一次。比如：
```
T sin(T angle)
```
Means T can be float, vec2, vec3 or vec4. If you pass in vec4 you get vec4 back which the sine of each of the components. In other words if v is a vec4 then

T可以是 float，vec2，vec3 或者 vec4。如果你传入的是 vec4 类型，那么返回的也是 vec4, 只是每个部分都会取 sine 函数的值。如果 v 是 vec4 类型，那么

```
vec4 s = sin(v);
```
等同于
```
vec4 s = vec4(sin(v.x), sin(v.y), sin(v.z), sin(v.w));
```
Sometimes one argument is a float and the rest is T. That means that float will be applied to all the components. For example if v1 and v2 are vec4 and f is a float then

有时，一个参数是 float 而其余的参数是 T。那么就以为着，float 这个参数会在所有的组件中使用。例如，v1 和 v2 是 vec4 类型，而 f 是 float 类型，则有
```
vec4 m = mix(v1, v2, f);
```
等同于
```
vec4 m = vec4(
  mix(v1.x, v2.x, f),
  mix(v1.y, v2.y, f),
  mix(v1.z, v2.z, f),
  mix(v1.w, v2.w, f));
```
You can see a list of all the GLSL functions on the last page of the WebGL Reference Card. If you like really dry and verbose stuff you can try the GLSL spec.

你可以查询所有的 GLSL 函数在 [WebGL 参考卡片][15]最后一页上。如果你要真正的干货，你可以参考 [GLSL 详解][16]。

## 综合讲解
That's the point of this entire series of posts. WebGL is all about creating various shaders, supplying the data to those shaders and then calling gl.drawArrays or gl.drawElements to have WebGL process the vertices by calling the current vertex shader for each vertex and then render pixels by calling the the current fragment shader for each pixel.

这是整个系列的关键点。WebGL 做的事情就是创建各种着色器，然后将数据交给这些着色器，接着调用 `gl.drawArrays` 或者 `gl.drawElements` ，通过顶点着色器处理顶点，然后在渲染像素时，通过片元着色器处理每个点的颜色值。

Actually creating the shaders requires several lines of code. Since those lines are the same in most WebGL programs and since once written you can pretty much ignore them how to compile GLSL shaders and link them into a shader program is covered here.

实际上，创建着色器需要多行代码。由于这些代码在大多数 WebGL 程序中都是差不多的，并且一旦你写好了，差不多就可以不用去管[怎样编译 GLSL 的着色器和怎样将他们与着色器程序联系起来][17]。

If you're just starting from here you can go in 2 directions. If you are interested in image procesing I'll show you how to do some 2D image processing. If you are interesting in learning about translation, rotation and scale then start here.

如果你是从这里开始学习的，那么接下来你可以选择两个方向学习。如果你对图像处理感兴趣，你可以学习[怎么做 2D 的图像处理][18]。如果你对图形的变换刚兴趣，比如，移动，旋转和缩放，那么你可以参考[这里][19]。

有问题? [在 stack overflow 上提问][20]。
Issue/Bug? [在 github 上创建 issue][21]。
  

  [1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html
  [4]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#attributes
  [5]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#uniforms
  [6]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#textures-in-vertex-shaders
  [7]: http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html
  [8]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
  [9]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#textures-in-fragment-shaders
  [10]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#uniforms
  [11]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#textures-in-vertex-shaders
  [12]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#varyings
  [13]: http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#uniforms
  [14]: http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html
  [15]: https://www.khronos.org/files/webgl/webgl-reference-card-1_0.pdf
  [16]: https://www.khronos.org/files/opengles_shading_language.pdf
  [17]: http://webglfundamentals.org/webgl/lessons/webgl-boilerplate.html
  [18]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
  [19]: http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html
  [20]: http://stackoverflow.com/questions/tagged/webgl
  [21]: http://github.com/greggman/webgl-fundamentals/issues