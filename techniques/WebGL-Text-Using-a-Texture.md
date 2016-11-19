# webgl Text 文本

## WebGL Text - 纹理
该文章是很多关于 WebGL 的教程的一篇。上一篇，我们讲解了[如何使用 Canvas 2D 在 WebGL 幕布上来渲染文本][1]。如果你还没有阅读它，你可以在继续下面内容前先阅读以下。
在上一篇文章，我们基本了解了如果使用 2D Canvas 去在 WebGL 场景中绘制文本。那种办法可以正常工作，并且也很简单，但它也有一个限制，这些文本不能被其它 3D 对象捕获。现在，解决办法就是直接使用 WebGL 直接绘制文本。
最简单的办法就是在 WebGL 中创建文本纹理。实现的效果，你可以参照在 Photoshop 中或者其他绘图程序中，绘制一个图片并带上一些文字。

![绘图软件][2]
然后，做一些平面几何处理并展示它。这实际就是我在一些游戏中绘制文本的办法。例如在 Locoroco 中，只有 270 个字符串。它可以翻译为 17 中语言。我们有一个 Excel 表用来存放所有的语言版本和一个脚本，用来启动 Photoshop 并且产生纹理，每个语言中都有一个这样的脚本。
当然，你也可以在运行时生成这些纹理。因为，WebGL 在运行在浏览器环境中的，所以，我们可以依靠 Canvas 2D API 去生成我们的纹理。
参考前一篇文章中的例子，我们添加一个函数用来填充在 2D canvas 里填充文本。
```
var textCtx = document.createElement("canvas").getContext("2d");
 
// 将文本放在 canvas 的中间
function makeTextCanvas(text, width, height) {
  textCtx.canvas.width  = width;
  textCtx.canvas.height = height;
  textCtx.font = "20px monospace";
  textCtx.textAlign = "center";
  textCtx.textBaseline = "middle";
  textCtx.fillStyle = "black";
  textCtx.clearRect(0, 0, textCtx.canvas.width, textCtx.canvas.height);
  textCtx.fillText(text, width / 2, height / 2);
  return textCtx.canvas;
}
```
现在，我们需要在 WebGL中，绘制两个不同的物体，'F' 和我们的文本。我将使用[一些辅助函数在前一篇文章提到过的][3]。如果那没有说清楚什么是 `programInfo`，`bufferInfo`等等，这里我们将具体的说明一下。
让我们创建 'F' 字幕和一个单元格。
```
// 创建 'F' 数据
var fBufferInfo = primitives.create3DFBufferInfo(gl);
// 创建一个文本单元格
var textBufferInfo = primitives.createPlaneBufferInfo(gl, 1, 1, 1, 1, m4.xRotation(Math.PI / 2));
```
一个单元格就是一个单位大小的正方形小格。该是原点为中心。`createPlaneBufferInfo` 在 xy 平面创造了一个面。我们传入一个矩阵去旋转并且得到一个 xy 平面的单元格。
接下来，创建两个着色器

```
// 创建 GLSL 程序
var fProgramInfo = createProgramInfo(gl, ["3d-vertex-shader", "3d-fragment-shader"]);
var textProgramInfo = createProgramInfo(gl, ["text-vertex-shader", "text-fragment-shader"]);
```
然后，创建我们的文本纹理
```
// 创建文本纹理
var textCanvas = makeTextCanvas("Hello!", 100, 26);
var textWidth  = textCanvas.width;
var textHeight = textCanvas.height;
var textTex = gl.createTexture();
gl.bindTexture(gl.TEXTURE_2D, textTex);
gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, textCanvas);
// make sure we can render it even if it's not a power of 2
gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
```
给 'F' 字母和文本创建 `uniforms`

```
var fUniforms = {
  u_matrix: m4.identity(),
};
 
var textUniforms = {
  u_matrix: m4.identity(),
  u_texture: textTex,
};
```
现在，当我们给 F 计算矩阵时，我们会保存 F 的视图矩阵。
```
var fViewMatrix = m4.translate(viewMatrix,
    translation[0] + xx * spread, translation[1] + yy * spread, translation[2]);
fViewMatrix = m4.xRotate(fViewMatrix, rotation[0]);
fViewMatrix = m4.yRotate(fViewMatrix, rotation[1] + yy * xx * 0.2);
fViewMatrix = m4.zRotate(fViewMatrix, rotation[2] + now + (yy * 3 + xx) * 0.1);
fViewMatrix = m4.scale(fViewMatrix, scale[0], scale[1], scale[2]);
fViewMatrix = m4.translate(fViewMatrix, -50, -75, 0);
```
像下面一样，开始绘制 F
```
gl.useProgram(fProgramInfo.program);
 
webglUtils.setBuffersAndAttributes(gl, fProgramInfo, fBufferInfo);
 
fUniforms.u_matrix = m4.multiply(projectionMatrix, fViewMatrix);
 
webglUtils.setUniforms(fProgramInfo, fUniforms);
 
// 画几何图形
gl.drawElements(gl.TRIANGLES, fBufferInfo.numElements, gl.UNSIGNED_SHORT, 0);
```
对于文本而言，我们只需要以 F 为参考原点的位置。同样，我们需要去缩放我们的单元格去匹配纹理的面积。最后，我们需要通过投影矩阵进行放大。

```
// 使用 'F' 视窗的位置，给文本作为参考
var textMatrix = m4.translate(projectionMatrix,
    fViewMatrix[12], fViewMatrix[13], fViewMatrix[14]);
// 将单元格缩放到合适的大小
textMatrix = m4.scale(textMatrix, textWidth, textHeight, 1);
```
然后，渲染文本
```
// 开始绘制文本
gl.useProgram(textProgramInfo.program);
 
webglUtils.setBuffersAndAttributes(gl, textProgramInfo, textBufferInfo);
 
m4.copy(textMatrix, textUniforms.u_matrix);
webglUtils.setUniforms(textProgramInfo, textUniforms);
 
// 触发绘制文本
gl.drawElements(gl.TRIANGLES, textBufferInfo.numElements, gl.UNSIGNED_SHORT, 0);
```
结果如下：

![rotate旋转文本][4]

[查看网页][5]
你会注意到，有时，我们文本的一部分会覆盖到我们的 F 上。那是因为，我们实际上绘制的是一个单元格。canvas 的默认颜色为透明黑色 (0,0,0,0)，并且，绘制单元格时，默认也是该颜色。我们可以混合像素来解决该问题。
```
gl.enable(gl.BLEND);
gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
```
通过上面的设置，可以使 WebGL 直接会取源像素（片元着色器中的颜色）然后将其和目标像素结合（canvas 中的颜色值）通过上面的 `blendFunc` 函数。我们将 `blend` 函数参数进行设置，`SRC_ALPHA` 作为源像素，`ONE_MINUS_SRC_ALPHA` 作为目标像素。
```
result = dest * (1 - src_alpha) + src * src_alpha
```
例如，目标颜色是绿色 `0,1,0,1`，然后，源颜色是 `1,0,0,1` 我们可以得到：

```
src = [1, 0, 0, 1]
dst = [0, 1, 0, 1]
src_alpha = src[3]  // 这里是 1
result = dst * (1 - src_alpha) + src * src_alpha
 
// 等同于
result = dst * 0 + src * 1
 
// 等同于
result = src
```
对于纹理部分的透明黑色 0,0,0,o0

```
src = [0, 0, 0, 0]
dst = [0, 1, 0, 1]
src_alpha = src[3]  // this is 0
result = dst * (1 - src_alpha) + src * src_alpha
 
// 等同于
result = dst * 1 + src * 0
 
// 等同于
result = dst
```
下面是开启混合模式的结果。

![blend混合模式][6]

[查看网页][7]
这看起来好一点了，但并不是最好的。如果你看仔细一点的话，会发现这个问题

![blend_issue_混合模式问题][8]
这怎么回事？我们现在正在绘制一个 F 然后才是文本，接着是下一个 F 然后是下一个的文本，依次重复。我们依旧使用的是[深度缓存][9]，所以，当我们在绘制 F 的文本时，及时混合模式将某些像素放在背景颜色当中，但深度缓存依旧会更新。当我们绘制下一个 F 时，如果该 F 的部分在前一个文本的像素后面的话，该部分是不会被渲染的。
我们刚才讨论的就是在 GPU 3D 渲染中，最难的问题之一。
**透明并不是完美的。**
最常用的解决透明渲染的办法是先将所有不透明的内容绘制上去，再将所有透明的元素通过在深度缓存中 z 的距离，在深度缓存更新完成后进行绘制。
首先，让我们先将绘制不透明内容（Fs）和透明内容（文本）分开。我们先申明一个变量去保存文本的位置。
```
var textPositions = [];
```
在循环绘制 F 中，我们将这些绘制缓存起来
```
var fViewMatrix = m4.translate(viewMatrix,
    translation[0] + xx * spread, translation[1] + yy * spread, translation[2]);
fViewMatrix = m4.xRotate(fViewMatrix, rotation[0]);
fViewMatrix = m4.yRotate(fViewMatrix, rotation[1] + yy * xx * 0.2);
fViewMatrix = m4.zRotate(fViewMatrix, rotation[2] + now + (yy * 3 + xx) * 0.1);
fViewMatrix = m4.scale(fViewMatrix, scale[0], scale[1], scale[2]);
fViewMatrix = m4.translate(fViewMatrix, -50, -75, 0);
// 保存 每个 f 的位置
textPositions.push([fViewMatrix[12], fViewMatrix[13], fViewMatrix[14]]);
```
在我们开始绘制 'F' 之前，我们需要禁掉混合模式并且写入深度缓存中。
```
gl.disable(gl.BLEND);
gl.depthMask(true);
```
对于绘制文本，我们会打开混合模式并且关闭写入深度缓存。
```
gl.enable(gl.BLEND);
gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
gl.depthMask(false);
```
然后通过我们上述保存的位置绘制文本
```
// 开始绘制文本
gl.useProgram(textProgramInfo.program);
 
webglUtils.setBuffersAndAttributes(gl, textProgramInfo, textBufferInfo);
 
textPositions.forEach(function(pos) {
  // 绘制文本 
  // 使用 'F' 的视图位置来绘制文本
  var textMatrix = m4.translate(projectionMatrix, pos[0], pos[1], pos[2]);
  // 放缩 F 到合适的大小
  textMatrix = m4.scale(textMatrix, textWidth, textHeight, 1);
 
  m4.copy(textMatrix, textUniforms.u_matrix);
  webglUtils.setUniforms(textProgramInfo, textUniforms);
 
  // 绘制文本
  gl.drawElements(gl.TRIANGLES, textBufferInfo.numElements, gl.UNSIGNED_SHORT, 0);
});
```
注意，我们已经将程序和属性的设置移到循环外，因为我们每次循环都只是画一样的内容，所以没必要每次循环都进行设置。
现在，它的工作样式为

![now_工作样式][10]

[查看网页][11]
注意，这里我们没有像上面提到的将设置放在循环外。因为，在这种情况下，我们需要绘制不透明的文本，而绘制后的结果并不能有什么明显的区分，所以，如果我们区分放置后，我也会在其他文章里提及。
另外一个问题是，该文本会穿过本身的 ‘F’ 字母。这里并没有什么很特别的解决办法。如果你已经写了一个 MMO（多人在线游戏） 并且想让每个玩家的文本总是出现在头部并且不会被遮挡。你可以简单的将文本沿着 Y 轴方向移动几个单位，直到它总是现在玩家的前上方。
你也可以将其向着照相机的方向移动。为了找点乐子，我们可以简单的做一下。因为 ‘pos’ 是在视野空间中，这意味着它也是相对于眼睛的（眼睛的位置在 0,0,0 的视野空间位置）。

```
// 因为 pos 是在视野空间中，这意味着它也是相对于眼睛的所能看到的空间的某个位置的
// 所以，沿着矢量方向，将文本沿着眼睛的方向移动一定的距离
var fromEye = m4.normalize(pos);
var amountToMoveTowardEye = 150;  // because the F is 150 units long
var viewX = pos[0] - fromEye[0] * amountToMoveTowardEye;
var viewY = pos[1] - fromEye[1] * amountToMoveTowardEye;
var viewZ = pos[2] - fromEye[2] * amountToMoveTowardEye;
var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
 
var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
// 将 F 字母缩放为合适大小
textMatrix = m4.scale(textMatrix, textWidth, textHeight, 1);
```
结果为：

![move_眼睛方向][12]

[查看网页][13]
可能你还会注意到，在文本的边缘会存在一些问题。

![edge_边缘][14]
该问题的原因主要是因为 Cavnas 2D API 只会产生自左乘的 alpha 值。当我们上传 cavnas 的内容给 纹理，WebGL总是会进行非自左乘的值，但是 Canvas 并不能做的很完美，因为自左乘的 alpha 是有损的。
为了解决这个问题，我们需要告诉 WebGL 使用`自左乘`模式

```
gl.pixelStorei(gl.UNPACK_PREMULTIPLY_ALPHA_WEBGL, true);
```
设置之后，WebGL 会提供自左乘的 alpha 值给 `gl.texImage2D` 和 `gl.texSubImage2D`。 如果传递给 `gl.texImage2D` 的数据已经是自左乘的，并符合 Canvas 2D 的数据要求，那么 WebGL 会直接传递它，而不会做其它处理。
我们还需要改变混合设置的函数
```
// gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
gl.blendFunc(gl.ONE, gl.ONE_MINUS_SRC_ALPHA);
```
上面被注释掉的使用 `src` 的颜色值乘以 alpha。这就是 `SRC_ALPHA` 代表的意思。但是，现在我们的纹理数据已经乘以它的 alpha。那就是自左乘的意思。所以，我们不需要 GPU 去做乘法。将 WebGL 设置为 `ONE` 意味着会乘以 1。

![ONE_测试][15]

[查看网页][16]
现在白色边缘就已经消失了。
如果你想将文本设置为固定大小，并且能够正确的区分，那怎么办？OK，如果你记得前文 [透视][17] 中提到过的透视矩阵，该会将我们的对象缩放 1/ -Z，让其看起来更小。所以，我们能够将文本缩放 -Z 倍为理想大小。

```
...
// 因为 pos 是在视野空间中，这意味着它也是相对于眼睛的所能看到的空间的某个位置的
// 所以，沿着矢量方向，将文本沿着眼睛的方向移动一定的距离
var fromEye = normalize(pos);
var amountToMoveTowardEye = 150;  // because the F is 150 units long
var viewX = pos[0] - fromEye[0] * amountToMoveTowardEye;
var viewY = pos[1] - fromEye[1] * amountToMoveTowardEye;
var viewZ = pos[2] - fromEye[2] * amountToMoveTowardEye;
var desiredTextScale = -1 / gl.canvas.height;  // 1x1 pixels
var scale = viewZ * desiredTextScale;
 
var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
// 将 F 字母缩放为合适大小
textMatrix = m4.scale(textMatrix, textWidth * scale, textHeight * scale, 1);
...
```

![scale_font文字缩放][18]

[查看网页][19]
如果你想在每个 F 字母上绘制不同的文本，你应该对于每个 F 字母创建新的纹理并且仅仅更新对应 F 的文本 `uniforms`。

```
// 给每个 F 字母创建新的纹理
var textTextures = [
  "anna",   // 0
  "colin",  // 1
  "james",  // 2
  "danny",  // 3
  "kalin",  // 4
  "hiro",   // 5
  "eddie",  // 6
  "shu",    // 7
  "brian",  // 8
  "tami",   // 9
  "rick",   // 10
  "gene",   // 11
  "natalie",// 12,
  "evan",   // 13,
  "sakura", // 14,
  "kai",    // 15,
].map(function(name) {
  var textCanvas = makeTextCanvas(name, 100, 26);
  var textWidth  = textCanvas.width;
  var textHeight = textCanvas.height;
  var textTex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, textTex);
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, textCanvas);
  // 即使 TEXTURE_2D 不是 2 的幂，我们也要确保能渲染它
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  return {
    texture: textTex,
    width: textWidth,
    height: textHeight,
  };
});
```
然后，在渲染时间内，选择一个纹理

```
textPositions.forEach(function(pos, ndx) {
 
  +// select a texture
  +var tex = textTextures[ndx];
 
  // scale the F to the size we need it.
  var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
  // scale the F to the size we need it.
  *textMatrix = m4.scale(textMatrix, tex.width * scale, tex.height * scale, 1);
```
然后，在绘制之间给纹理设置 uniform
```
  *textUniforms.u_texture = tex.texture;
```

![纹理_setting][20]
[查看网页][21]
前面我们使用了黑色在 canvas 中绘制文本。如果我们使用白色的话，这应该会更有效。然后，我们可以拓展文本的颜色，使其能变为任意的颜色。
首先，我们将修改文本着色器去乘以一个颜色。

```
varying vec2 v_texcoord;
 
uniform sampler2D u_texture;
uniform vec4 u_color;
 
void main() {
   gl_FragColor = texture2D(u_texture, v_texcoord) * u_color;
}
```
然后，当我们在 canvas 中绘制文本时，设置颜色为白色
```
textCtx.fillStyle = "white";
```
接着，我们来弄一点其他的颜色
```
// colors, 1 表示是对于每个 F 应用
var colors = [
  [0.0, 0.0, 0.0, 1], // 0
  [1.0, 0.0, 0.0, 1], // 1
  [0.0, 1.0, 0.0, 1], // 2
  [1.0, 1.0, 0.0, 1], // 3
  [0.0, 0.0, 1.0, 1], // 4
  [1.0, 0.0, 1.0, 1], // 5
  [0.0, 1.0, 1.0, 1], // 6
  [0.5, 0.5, 0.5, 1], // 7
  [0.5, 0.0, 0.0, 1], // 8
  [0.0, 0.0, 0.0, 1], // 9
  [0.5, 5.0, 0.0, 1], // 10
  [0.0, 5.0, 0.0, 1], // 11
  [0.5, 0.0, 5.0, 1], // 12,
  [0.0, 0.0, 5.0, 1], // 13,
  [0.5, 5.0, 5.0, 1], // 14,
  [0.0, 5.0, 5.0, 1], // 15,
];
```
在绘制的时候，我们选择一个颜色值
```
// 设置颜色 uniform
textUniforms.u_color = colors[ndx];
```
颜色为：

![different_颜色][22]

[查看网页][23]
这项技术实际上是大部分浏览器启用 GPU 加速时使用到的技术。浏览器会和 HTML 内容以及你所提供的样式一起生成纹理。并且，只要 HTML 内容不改变，他们可以再次重新渲染纹理当你滑动屏幕时。当然，如果你更新频率很高的话，那么这个技术的结果可能会有些慢，因为重新生成纹理并且重新提交他们到 GPU 相对来说是一个比较慢的操作。
在下一篇文章中，我们会探讨[另外一项技术][24]。它可能会更合适高频率更新的情况。
## Scaling Text without pixelation

## 非像素化放缩文本
你可能注意到在前面例子中，我们使用了固定大小的文本，当它靠近相机时，会变得像像素化（模糊）。那我们应该怎样解决呢？
老实说，在 3D 里面取缩放 2D 的文本不是很常见。在很多游戏或者 3D 编辑器中，你可以发现，不管离相机有多远或者多近，文本都是一样的大小。实际上，文本是使用 2D 绘制而非 3D 绘制，所以，即使某人或者某物在其他物体后面，比如你的队友在一堵墙后面，你依旧可以看到这个文本。
如果你确实想缩放 2D 文本在 3D 环境里。我确实不知道其他简单的办法。这是我一下能够想到的了。

 - 对于不同的分辨率，使用不同大小的纹理字体。接着，你就可以使用高清的纹理，当字体变得更大。这叫做 LOD（使用不同的分辨率层）
 - 另外一种方法可以使用具体每一帧文本精确的大小去渲染纹理。不过，那是相当的慢。
 - 还有一种办法是，使用几何图形来渲染 2D 文本。话句话说就是，不是直接使用纹理来渲染，而是将文本使用很多很多的三角形来渲染。这理论上是可以的，但它也有些问题，对于尺寸比较小的文本而言，它并不会渲染的很棒，而却对很大的文本来说，你会看见这些三角形。
 - 最后一种办法是使用一个特别的着色器，它能够[渲染曲线][25]。那听起来很棒，但是这已经超出我们现在所要阐述的内容。


   [1]: http://webglfundamentals.org/webgl/lessons/webgl-text-canvas2d.html
  [2]: http://webglfundamentals.org/webgl/lessons/resources/my-awesme-text.png
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-drawing-multiple-things.html
  [4]: http://static.zybuluo.com/jimmythr/vce4nrk5xnmbdrwh5hofm68p/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-05%2013.25.43.png
  [5]: http://webglfundamentals.org/webgl/webgl-text-texture.html
  [6]: http://static.zybuluo.com/jimmythr/ljhki466pc2qa2typeupllmo/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-05%2014.02.55.png
  [7]: http://webglfundamentals.org/webgl/webgl-text-texture-enable-blend.html
  [8]: http://webglfundamentals.org/webgl/lessons/resources/text-zbuffer-issue.png
  [9]: http://webglfundamentals.org/webgl/lessons/webgl-3d-orthographic.html
  [10]: http://static.zybuluo.com/jimmythr/z6va7eog7qwa878cdmyl1oh1/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-10%2023.41.58.png
  [11]: http://webglfundamentals.org/webgl/webgl-text-texture-separate-opaque-from-transparent.html
  [12]: http://static.zybuluo.com/jimmythr/mmzklwvdq4vixhah37ndltff/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%2014.41.55.png
  [13]: http://webglfundamentals.org/webgl/webgl-text-texture-moved-toward-view.html
  [14]: http://webglfundamentals.org/webgl/lessons/resources/text-gray-outline.png
  [15]: http://static.zybuluo.com/jimmythr/eiqusgjgxfe4esw0fnox08af/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%2018.51.08.png
  [16]: http://webglfundamentals.org/webgl/webgl-text-texture-premultiplied-alpha.html
  [17]: http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html
  [18]: http://static.zybuluo.com/jimmythr/vj2ft78vbad9a0hnx72ihvbs/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%2019.12.34.png
  [19]: http://webglfundamentals.org/webgl/webgl-text-texture-consistent-scale.html
  [20]: http://static.zybuluo.com/jimmythr/uzrq5jj4sob0g7fejvvg8i4g/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-11%2019.42.14.png
  [21]: http://webglfundamentals.org/webgl/webgl-text-texture-different-text.html
  [22]: http://static.zybuluo.com/jimmythr/dqbx8z81s9vw6mbq7a13p5y2/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-11-13%2022.20.51.png
  [23]: http://webglfundamentals.org/webgl/webgl-text-texture-different-colors.html
  [24]: http://webglfundamentals.org/webgl/lessons/webgl-text-glyphs.html
  [25]: http://research.microsoft.com/en-us/um/people/cloop/loopblinn05.pdf