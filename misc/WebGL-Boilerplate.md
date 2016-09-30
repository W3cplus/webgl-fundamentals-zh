# WebGL Boilerplate
# WebGL ʾ��
This is a continuation from WebGL Fundamentals. WebGL sometimes appears complicated to learn because most lessons go over everything all at once. I'll try to avoid that where possible and break it down into smaller pieces.

��ƪ��[WebGL Fundamentals][1] ����������ʱ�� WebGL ѧϰ�����ƺ��ܸ��ӣ���Ϊ�󲿷ֵĽ̳���Ҫ������֪ʶ������һ�顣�һᾡ���������������ڿ��ܵ�����£��ֽ�ΪС��Ƭ�Ρ�

One of things that makes WebGL seem complicated is that you have these 2 tiny functions, a vertex shader and a fragment shader. Those two functions usually run on your GPU which is where all the speed comes from. That's also why they are written in a custom language, a language that matches what a GPU can do. Those 2 functions need to be compiled and linked. That process is, 99% of the time, the same in every WebGL program.

�� WebGL ���Ƹ��ӵ�����һ��������������С������������ɫ����vertex shader����ƬԪ��ɫ����fragment shader��������������ͨ�������� GPU �У�GPU �������ٶȵ���Դ����Ҳ��Ϊʲô������һ�� GPU �ܹ����е��Զ������Ա����ԭ��������������Ҫ��������ӡ�
��һ���̣�99% ��ʱ�䣬������ͬһ�� WebGL �����н��еġ�

Here's the boilerplate code for compiling a shader.

���Ǳ༭һ����ɫ����ʾ�����롣

```
/**
 * Creates and compiles a shader.
 *����������һ����ɫ��
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl ΪWebGL �����ġ�
 * @param {string} shaderSource The GLSL source code for the shader.
 * shaderSource ΪGLSL ����ɫ��Դ�롣
 * @param {number} shaderType The type of shader, VERTEX_SHADER or
 *     FRAGMENT_SHADER.
 * shaderType Ϊ��ɫ�����ͣ�VERTEX_SHADER ���� FRAGMENT_SHADER��
 * @return {!WebGLShader} The shader.
 * ������ɫ����
 */
function compileShader(gl, shaderSource, shaderType) {
  // Create the shader object
  // ������ɫ������
  var shader = gl.createShader(shaderType);
 
  // Set the shader source code.
  // ������ɫ��Դ��
  gl.shaderSource(shader, shaderSource);
 
  // Compile the shader
  // ������ɫ��
  gl.compileShader(shader);
 
  // Check if it compiled
  // �����ɫ���Ƿ����ɹ�
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (!success) {
    // Something went wrong during compilation; get the error
    // ������̳����׳��쳣
    throw "could not compile shader:" + gl.getShaderInfoLog(shader);
  }
 
  return shader;
}
```

And the boilerplate code for linking 2 shaders into a program.

��������ʾ�������ǽ�������ɫ�����ӵ�һ��������

```
/**
 * Creates a program from 2 shaders.
 * Ϊ������ɫ������һ������
 *
 * @param {!WebGLRenderingContext) gl The WebGL context.
 * gl Ϊ WebGL �����ġ�
 * @param {!WebGLShader} vertexShader A vertex shader.
 * vertexShader Ϊ������ɫ����
 * @param {!WebGLShader} fragmentShader A fragment shader.
  * fragmentShader ΪƬ����ɫ����
 * @return {!WebGLProgram} A program.
 * ����һ������
 */
function createProgram(gl, vertexShader, fragmentShader) {
  // create a program.
  // ����һ������
  var program = gl.createProgram();
 
  // attach the shaders.
  // ����ɫ����
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
 
  // link the program.
  // ���ӳ���
  gl.linkProgram(program);
 
  // Check if it linked.
  // �������Ƿ����ӳɹ���
  var success = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (!success) {
      // something went wrong with the link
      // ���ӳ���
      throw ("program filed to link:" + gl.getProgramInfoLog (program));
  }
 
  return program;
};
```

Of course how you decide to handle errors might be different. Throwing exceptions might not be the best way to handle things. Still, those few lines of code are pretty much the same in nearly every WebGL program.

��Ȼ�������δ���ķ�ʽ����������ͬ���׳��쳣���ܲ�����õĴ���ʽ��������ˣ��⼸�д��뼸����ÿһ�� WebGL �����ж�����ͬ�ġ�

I like to store my shaders in non javascript <script> tags. It makes them easy to edit so I use code like this.

��ϲ�����ҵ���ɫ������ڷ� javascript ��ǩ�С���ʹ���������ױ༭������������ʹ�á�

```
/**
 * Creates a shader from the content of a script tag.
 * ����һ������������ script ��ǩ����ɫ����
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl Ϊ WebGL �����ġ�
 * @param {string} scriptId The id of the script tag.
 * scriptId Ϊ script ��ǩ��id��
 * @param {string} opt_shaderType. The type of shader to create.
 * opt_shaderType Ϊ��Ҫ��������ɫ�������͡�
 *     If not passed in will use the type attribute from the
 *     script tag.
 * ���û�д�����һ����������ʹ�� script ��ǩ�е� type ����ֵ��
 * @return {!WebGLShader} A shader.
 * ����һ����ɫ����
 */
function createShaderFromScript(gl, scriptId, opt_shaderType) {
  // look up the script tag by id.
  // ���� script ��ǩ��id��
  var shaderScript = document.getElementById(scriptId);
  if (!shaderScript) {
    throw("*** Error: unknown script element" + scriptId);
  }
 
  // extract the contents of the script tag.
  // ��ȡ script ��ǩ�е����ݡ�
  var shaderSource = shaderScript.text;
 
  // If we didn't pass in a type, use the 'type' from
  // the script tag.
  // ���û�д������Ͳ�������ʹ�� script ��ǩ�ġ�type����ֵ��
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

�����ҿ��Ա�����ɫ���ˡ�

```
var shader = compileShaderFromScript(gl, "someScriptTagId");
```

I'll usually go one step further and make a function to compile two shaders from script tags, attach them to a program and link them.

��ͨ�������һ��������һ�������༭�������� script ��ǩ����ɫ�������������ӵ�һ�����򣬲��������ǡ�

```
/**
 * Creates a program from 2 script tags.
 * ������ script ��ǩ�д���һ����ѫ��
 *
 * @param {!WebGLRenderingContext} gl The WebGL Context.
 * gl Ϊ WebGL �������ġ�
 * @param {string} vertexShaderId The id of the vertex shader script tag.
 * vertexShaderId Ϊ ���嶥����ɫ���� script ��ǩ�� id��
 * @param {string} fragmentShaderId The id of the fragment shader script tag.
  * fragmentShaderId Ϊ ����ƬԪ��ɫ���� script ��ǩ�� id��
 * @return {!WebGLProgram} A program
 * ����һ������
 */
function createProgramFromScripts(
    gl, vertexShaderId, fragmentShaderId) {
  var vertexShader = createShaderFromScriptTag(gl, vertexShaderId);
  var fragmentShader = createShaderFromScriptTag(gl, fragmentShaderId);
  return createProgram(gl, vertexShader, fragmentShader);
}
```

That's most of my minimum set of WebGL boilerplate code. You can find that code here. If you want something slightly more organized check out TWGL.js.

���������� WebGL �������ʾ�����롣[������������ҵ���Щ����][2]��
�������Ҫ�����������鿴 [TWGL.js][3]��

The rest of what makes WebGL look complicated is setting up all the inputs to your shaders. See how it works.

�� WebGL ���������ӵ����ಿ������ɫ���������������á��뿴 [how it works][4]��

I'd also suggest you read up on less code more fun and check out TWGL.

��Ҳ��������ϸ�о� [less code more fun][4] ���Ҳ鿴 [TEGL][3]��

  [1]: </fundamentals/WebGL-Fundamentals.html>
  [2]: <https://github.com/greggman/webgl-fundamentals/blob/master/webgl/resources/webgl-utils.js>
  [3]: <http://twgljs.org/>
  [4]: </fundamentals/WebGL-How-It-Works.html>
