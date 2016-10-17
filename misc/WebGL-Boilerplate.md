# WebGL Boilerplate
# WebGL 示例
This is a continuation from WebGL Fundamentals. WebGL sometimes appears complicated to learn because most lessons go over everything all at once. I'll try to avoid that where possible and break it down into smaller pieces.

这篇是 <a href="/fundamentals/webgl-fundamentals.html" target="_blank">WebGL Fundamentals</a> 的延续。有时候 WebGL 学习起来似乎很复杂，因为大部分的教程需要将所有知识点重温一遍。我会尽量避免这样，并在可能的情况下，分解为小的片段。

One of things that makes WebGL seem complicated is that you have these 2 tiny functions, a vertex shader and a fragment shader. Those two functions usually run on your GPU which is where all the speed comes from. That's also why they are written in a custom language, a language that matches what a GPU can do. Those 2 functions need to be compiled and linked. That process is, 99% of the time, the same in every WebGL program.

让 WebGL 看似复杂的其中一件事是你有两个小函数，顶点着色器（vertex shader）和片元着色器（fragment shader）。这两个函数通常运行在 GPU 中，GPU 是所有速度的来源。这也是为什么它们用一种 GPU 能够运行的自定义语言编译的原因。这两个函数需要编译和链接。
这一过程，99% 的时间，都是在同一个 WebGL 程序中进行的。

Here's the boilerplate code for compiling a shader.

这是编辑一个着色器的示例代码。

```
/**
 * Creates and compiles a shader.
 *创建并编译一个着色器
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl 为WebGL 上下文。
 * @param {string} shaderSource The GLSL source code for the shader.
 * shaderSource 为GLSL 中着色器源码。
 * @param {number} shaderType The type of shader, VERTEX_SHADER or
 *     FRAGMENT_SHADER.
 * shaderType 为着色器类型，VERTEX_SHADER 或者 FRAGMENT_SHADER。
 * @return {!WebGLShader} The shader.
 * 返回着色器。
 */
function compileShader(gl, shaderSource, shaderType) {
  // Create the shader object
  // 创建着色器对象
  var shader = gl.createShader(shaderType);
 
  // Set the shader source code.
  // 设置着色器源码
  gl.shaderSource(shader, shaderSource);
 
  // Compile the shader
  // 编译着色器
  gl.compileShader(shader);
 
  // Check if it compiled
  // 检查着色器是否编译成功
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (!success) {
    // Something went wrong during compilation; get the error
    // 编译过程出错；抛出异常
    throw "could not compile shader:" + gl.getShaderInfoLog(shader);
  }
 
  return shader;
}
```

And the boilerplate code for linking 2 shaders into a program.

接下来的示例代码是将两个着色器连接到一个程序中

```
/**
 * Creates a program from 2 shaders.
 * 为两个着色器创建一个程序。
 *
 * @param {!WebGLRenderingContext) gl The WebGL context.
 * gl 为 WebGL 上下文。
 * @param {!WebGLShader} vertexShader A vertex shader.
 * vertexShader 为顶点着色器。
 * @param {!WebGLShader} fragmentShader A fragment shader.
  * fragmentShader 为片段着色器。
 * @return {!WebGLProgram} A program.
 * 返回一个程序。
 */
function createProgram(gl, vertexShader, fragmentShader) {
  // create a program.
  // 创建一个程序。
  var program = gl.createProgram();
 
  // attach the shaders.
  // 绑定着色器。
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
 
  // link the program.
  // 连接程序。
  gl.linkProgram(program);
 
  // Check if it linked.
  // 检查程序是否连接成功。
  var success = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (!success) {
      // something went wrong with the link
      // 连接出错
      throw ("program filed to link:" + gl.getProgramInfoLog (program));
  }
 
  return program;
};
```

Of course how you decide to handle errors might be different. Throwing exceptions might not be the best way to handle things. Still, those few lines of code are pretty much the same in nearly every WebGL program.

当然你决定如何错误的方式可能有所不同。抛出异常可能不是最好的处理方式。尽管如此，这几行代码几乎在每一个 WebGL 程序中都是相同的。

I like to store my shaders in non javascript <script> tags. It makes them easy to edit so I use code like this.

我喜欢将我的着色器存放在非 javascript 标签中。这使得它们容易编辑，所以我这样使用。

```
/**
 * Creates a shader from the content of a script tag.
 * 创建一个上下文来自 script 标签的着色器。
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl 为 WebGL 上下文。
 * @param {string} scriptId The id of the script tag.
 * scriptId 为 script 标签的id。
 * @param {string} opt_shaderType. The type of shader to create.
 * opt_shaderType 为需要创建的着色器的类型。
 *     If not passed in will use the type attribute from the
 *     script tag.
 * 如果没有传递这一参数，将会使用 script 标签中的 type 属性值。
 * @return {!WebGLShader} A shader.
 * 返回一个着色器。
 */
function createShaderFromScript(gl, scriptId, opt_shaderType) {
  // look up the script tag by id.
  // 查找 script 标签的id。
  var shaderScript = document.getElementById(scriptId);
  if (!shaderScript) {
    throw("*** Error: unknown script element" + scriptId);
  }
 
  // extract the contents of the script tag.
  // 获取 script 标签中的内容。
  var shaderSource = shaderScript.text;
 
  // If we didn't pass in a type, use the 'type' from
  // the script tag.
  // 如果没有传递类型参数，就使用 script 标签的‘type’的值。
  if (!opt_shaderType) {
    if (shaderScript.type == "x-shader/x-vertex") {
      opt_shaderType = gl.VERTEX_SHADER;
    } else if (shaderScript.type == "x-shader/x-fragment") {
      opt_shaderType = gl.FRAGMENT_SHADER;
    } else if (!opt_shaderType) {
      throw("*** Error: shader type not set");
    }
  }
 
  return compileShader(gl, shaderSource, opt_shaderType);
};
```

Now to compile a shader I can just do.

现在我可以编译着色器了。

```
var shader = compileShaderFromScript(gl, "someScriptTagId");
```

I'll usually go one step further and make a function to compile two shaders from script tags, attach them to a program and link them.

我通常会更进一步地利用一个函数编辑两个来自 script 标签的着色器，把他们连接到一个程序，并链接他们。

```
/**
 * Creates a program from 2 script tags.
 * 从两个 script 标签中创建一个程勋。
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl 为 WebGL 的上下文。
 * @param {string} vertexShaderId The id of the vertex shader script tag.
 * vertexShaderId 为 定义顶点着色器的 script 标签的 id。
 * @param {string} fragmentShaderId The id of the fragment shader script tag.
  * fragmentShaderId 为 定义片元着色器的 script 标签的 id。
 * @return {!WebGLProgram} A program
 * 返回一个程序。
 */
function createProgramFromScripts(
    gl, vertexShaderId, fragmentShaderId) {
  var vertexShader = createShaderFromScriptTag(gl, vertexShaderId);
  var fragmentShader = createShaderFromScriptTag(gl, fragmentShaderId);
  return createProgram(gl, vertexShader, fragmentShader);
}
```

That's most of my minimum set of WebGL boilerplate code. You can find that code here. If you want something slightly more organized check out TWGL.js.

这是我设置 WebGL 最起码的示例代码。<a href="https://github.com/greggman/webgl-fundamentals/blob/master/webgl/resources/webgl-utils.js" target="_blank">你可以在这里找到这些代码</a>。
如果你想要更有条理的请查看 <a href="http://twgljs.org/" target="_blank">TWGL.js</a>。

The rest of what makes WebGL look complicated is setting up all the inputs to your shaders. See how it works.

让 WebGL 看起来复杂的其余部分是着色器的所有输入设置。请看 <a href="/fundamentals/WebGL-How-It-Works.html" target="_blank">how it works</a>。

I'd also suggest you read up on less code more fun and check out TWGL.

我也建议你详细研究 [less code more fun][4] 并且查看 <a href="http://twgljs.org/" target="_blank">TWGL</a>。

