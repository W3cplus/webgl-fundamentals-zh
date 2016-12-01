# WebGL Text - Using a Glyph Texture

# WebGL 文本 - 使用字形纹理

This post is a continuation of many articles about WebGL. The last one was about using textures for rendering text in WebGL. If you haven't read it you might want to check that out before continuing.

该文章是接着其他文章继续讲解 WebGL。这是最后一篇关于[在WebGL 中使用纹理去渲染文本][1]。如果你还没有看过前面的文章，建议先看看，然后再接着往下看。

In the last article we went over how to use a texture to draw text in your WebGL scene. That technique is very common and it's great for things like in multi-player games where you want to put a name over an avatar. As that name rarely changes it's perfect.

在上一篇文章中，我们简单复习了[如何使用纹理在你的 WebGL 场景中绘制文本][2]。那个技术非常常见，并且对于某些场景来说特别适用，比如，在多人游戏中，你想将名字放在头像的上方。如果名字不是经常改变的话，这样做应该没啥毛病。

Let's say you want to render a lot of text that changes often like a UI. Given the last example in the previous article an obvious solution is to make a texture for each letter. Let's change the last sample to do that.

如果说，你想去渲染很多容易改变的文字，感觉就像 UI 一样。在[上一篇文章][3]的例子中，显而易见的办法是给每一个字母创建一个 `texture`。接下来，我们来修改一下那个例子。

```
var names = [
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
];
 
// create text textures, one for each letter
var textTextures = [
  "a",    // 0
  "b",    // 1
  "c",    // 2
  "d",    // 3
  "e",    // 4
  "f",    // 5
  "g",    // 6
  "h",    // 7
  "i",    // 8
  "j",    // 9
  "k",    // 10
  "l",    // 11
  "m",    // 12,
  "n",    // 13,
  "o",    // 14,
  "p",    // 14,
  "q",    // 14,
  "r",    // 14,
  "s",    // 14,
  "t",    // 14,
  "u",    // 14,
  "v",    // 14,
  "w",    // 14,
  "x",    // 14,
  "y",    // 14,
  "z",    // 14,
].map(function(name) {
  var textCanvas = makeTextCanvas(name, 10, 26);
```

Then instead of rendering one quad for each name we'll render one quad for each letter in each name.

然后，我们不去给每个 `name` 渲染一个单元格，而是相对于名字中的每个字母渲染一个单元格。

```
// 开始渲染文本
// 因为每个字母使用同样的实行和同样的程序
// 我们只要执行一次下列程序即可
gl.useProgram(textProgramInfo.program);
setBuffersAndAttributes(gl, textProgramInfo.attribSetters, textBufferInfo);
 
textPositions.forEach(function(pos, ndx) {
  var name = names[ndx];
 
  // 对于每个字母
  for (var ii = 0; ii < name.length; ++ii) {
    var letter = name.charCodeAt(ii);
    var letterNdx = letter - "a".charCodeAt(0);
 
    // 选择某个字母的纹理
    var tex = textTextures[letterNdx];
 
    // 将 F 字母的位置传递给纹理
    // 因为 pos 是在视野空间的，这意味着，它是一个从眼睛到某个位置的矢量表示。
    // 所以，沿着该矢量的反方向（朝着眼睛）移动一定距离
    var fromEye = m4.normalize(pos);
    var amountToMoveTowardEye = 150;  // because the F is 150 units long
    var viewX = pos[0] - fromEye[0] * amountToMoveTowardEye;
    var viewY = pos[1] - fromEye[1] * amountToMoveTowardEye;
    var viewZ = pos[2] - fromEye[2] * amountToMoveTowardEye;
    var desiredTextScale = -1 / gl.canvas.height;  // 1x1 pixels
    var scale = viewZ * desiredTextScale;
 
    var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
    // 将 F 字母缩放到适合大小
    textMatrix = m4.scale(textMatrix, tex.width * scale, tex.height * scale, 1);
    +textMatrix = m4.translate(textMatrix, ii, 0, 0);
 
    // 设置 texture uniform
    m4.copy(textMatrix, textUniforms.u_matrix);
    textUniforms.u_texture = tex.texture;
    webglUtils.setUniforms(textProgramInfo, textUniforms);
 
    // 绘制文本
    gl.drawElements(gl.TRIANGLES, textBufferInfo.numElements, gl.UNSIGNED_SHORT, 0);
  }
});
```
And you can see it works

下面它就可以正常工作了

![webgl_text][4]

[查看网页][5]

Unfortunately it's SLOW. The example below doesn't show it but we're individually drawing 73 quads. We're computing 73 matrices and 292 matrix multiplies. A typical UI might easily have 1000 letters showing. That's way way too much work to get a reasonable framerate.

不幸的是它灰常的慢。从下面的例子中看不出，我们实际上独立渲染了 73 个单元格。我们需要计算 73 个矩阵和 292 个矩阵乘法。一个经典的 UI 一般都需要差不多 1000 个字母显示。不过，如果以上面那种方法来做的话，需要做很多很多的优化，才能得到合理的帧率。

So to fix that the way this is usually done is to make a texture atlas that contains all the letters. We went over what a texture atlas is when we talked about texturing the 6 faces of a cube.

所以，为了解决这个痛点，通常的做法是使用一个能包含所有字母的纹理地图。我们在[设置立方体的 6 个面的纹理][6]时讲过纹理地图。

Searching the web I found this simple open source font texture atlas 

通过搜索，我发现了一个简单的[开源字体纹理地图][7]。

```
var fontInfo = {
  letterHeight: 8,
  spaceWidth: 8,
  spacing: -1,
  textureWidth: 64,
  textureHeight: 40,
  glyphInfos: {
    'a': { x:  0, y:  0, width: 8, },
    'b': { x:  8, y:  0, width: 8, },
    'c': { x: 16, y:  0, width: 8, },
    'd': { x: 24, y:  0, width: 8, },
    'e': { x: 32, y:  0, width: 8, },
    'f': { x: 40, y:  0, width: 8, },
    'g': { x: 48, y:  0, width: 8, },
    'h': { x: 56, y:  0, width: 8, },
    'i': { x:  0, y:  8, width: 8, },
    'j': { x:  8, y:  8, width: 8, },
    'k': { x: 16, y:  8, width: 8, },
    'l': { x: 24, y:  8, width: 8, },
    'm': { x: 32, y:  8, width: 8, },
    'n': { x: 40, y:  8, width: 8, },
    'o': { x: 48, y:  8, width: 8, },
    'p': { x: 56, y:  8, width: 8, },
    'q': { x:  0, y: 16, width: 8, },
    'r': { x:  8, y: 16, width: 8, },
    's': { x: 16, y: 16, width: 8, },
    't': { x: 24, y: 16, width: 8, },
    'u': { x: 32, y: 16, width: 8, },
    'v': { x: 40, y: 16, width: 8, },
    'w': { x: 48, y: 16, width: 8, },
    'x': { x: 56, y: 16, width: 8, },
    'y': { x:  0, y: 24, width: 8, },
    'z': { x:  8, y: 24, width: 8, },
    '0': { x: 16, y: 24, width: 8, },
    '1': { x: 24, y: 24, width: 8, },
    '2': { x: 32, y: 24, width: 8, },
    '3': { x: 40, y: 24, width: 8, },
    '4': { x: 48, y: 24, width: 8, },
    '5': { x: 56, y: 24, width: 8, },
    '6': { x:  0, y: 32, width: 8, },
    '7': { x:  8, y: 32, width: 8, },
    '8': { x: 16, y: 32, width: 8, },
    '9': { x: 24, y: 32, width: 8, },
    '-': { x: 32, y: 32, width: 8, },
    '*': { x: 40, y: 32, width: 8, },
    '!': { x: 48, y: 32, width: 8, },
    '?': { x: 56, y: 32, width: 8, },
  },
};
```
And we'll load the image just like we loaded textures before

并且，我们将加载图片就像之前我们加载纹理一样。

```
// 创建一个纹理
var glyphTex = gl.createTexture();
gl.bindTexture(gl.TEXTURE_2D, glyphTex);
// 将 1x1 的蓝色像素填充在纹理中
gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE,
              new Uint8Array([0, 0, 255, 255]));
// 异步加载图片
var image = new Image();
image.src = "resources/8x8-font.png";
image.addEventListener('load', function() {
  // 现在，指定的图片已经加载，然后将它复制给纹理
  gl.bindTexture(gl.TEXTURE_2D, glyphTex);
  gl.pixelStorei(gl.UNPACK_PREMULTIPLY_ALPHA_WEBGL, true);
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
});
```
Now that we have a texture with glyphs in it we need to use it. To do that we'll build quad vertices on the fly for each glyph. Those vertices will use texture coordinates to select a particlar glyph

现在，我们已经有了一个包含字形的纹理，接着就可以使用它了。我们需要动态创建一个单元格纹理给每个字形内容。那些顶点将会使用纹理坐标去选择某个特定的字形。

Given a string let's build the vertices

使用一个字符串去创建节点。
```
function makeVerticesForString(fontInfo, s) {
  var len = s.length;
  var numVertices = len * 6;
  var positions = new Float32Array(numVertices * 2);
  var texcoords = new Float32Array(numVertices * 2);
  var offset = 0;
  var x = 0;
  var maxX = fontInfo.textureWidth;
  var maxY = fontInfo.textureHeight;
  for (var ii = 0; ii < len; ++ii) {
    var letter = s[ii];
    var glyphInfo = fontInfo.glyphInfos[letter];
    if (glyphInfo) {
      var x2 = x + glyphInfo.width;
      var u1 = glyphInfo.x / maxX;
      var v1 = (glyphInfo.y + fontInfo.letterHeight - 1) / maxY;
      var u2 = (glyphInfo.x + glyphInfo.width - 1) / maxX;
      var v2 = glyphInfo.y / maxY;
 
      // 每个字母配置 6 顶点
      positions[offset + 0] = x;
      positions[offset + 1] = 0;
      texcoords[offset + 0] = u1;
      texcoords[offset + 1] = v1;
 
      positions[offset + 2] = x2;
      positions[offset + 3] = 0;
      texcoords[offset + 2] = u2;
      texcoords[offset + 3] = v1;
 
      positions[offset + 4] = x;
      positions[offset + 5] = fontInfo.letterHeight;
      texcoords[offset + 4] = u1;
      texcoords[offset + 5] = v2;
 
      positions[offset + 6] = x;
      positions[offset + 7] = fontInfo.letterHeight;
      texcoords[offset + 6] = u1;
      texcoords[offset + 7] = v2;
 
      positions[offset + 8] = x2;
      positions[offset + 9] = 0;
      texcoords[offset + 8] = u2;
      texcoords[offset + 9] = v1;
 
      positions[offset + 10] = x2;
      positions[offset + 11] = fontInfo.letterHeight;
      texcoords[offset + 10] = u2;
      texcoords[offset + 11] = v2;
 
      x += glyphInfo.width + fontInfo.spacing;
      offset += 12;
    } else {
      // 不需要的字母就可以跳过
      x += fontInfo.spaceWidth;
    }
  }
 
  // 返回正在使用的 TypedArrays 部分内容
  return {
    arrays: {
      position: new Float32Array(positions.buffer, 0, offset),
      texcoord: new Float32Array(texcoords.buffer, 0, offset),
    },
    numVertices: offset / 2,
  };
}
```
To use it we'll manually create a bufferInfo. (See previous article if you don't remember what a bufferInfo is).

我们需要手动创建一个 `bufferInfo` 方便去使用上面的返回的结果。（如果你不清楚 bufferInfo 是什么，可以参考[前文][8]）

```
// 手动创建一个 bufferInfo
var textBufferInfo = {
  attribs: {
    a_position: { buffer: gl.createBuffer(), numComponents: 2, },
    a_texcoord: { buffer: gl.createBuffer(), numComponents: 2, },
  },
  numElements: 0,
};
```
And then to render text we'll update the buffers. We'll also make the text dynamic

接着，我们需要更新 buffers 去渲染文本。同样我们将让文本动态的变化
```
textPositions.forEach(function(pos, ndx) {
 
  var name = names[ndx];
  var s = name + ":" + pos[0].toFixed(0) + "," + pos[1].toFixed(0) + "," + pos[2].toFixed(0);
  var vertices = makeVerticesForString(fontInfo, s);
 
  // 更新 buffers
  textBufferInfo.attribs.a_position.numComponents = 2;
  gl.bindBuffer(gl.ARRAY_BUFFER, textBufferInfo.attribs.a_position.buffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices.arrays.position, gl.DYNAMIC_DRAW);
  gl.bindBuffer(gl.ARRAY_BUFFER, textBufferInfo.attribs.a_texcoord.buffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices.arrays.texcoord, gl.DYNAMIC_DRAW);
 
  // 将 F 的位置传给文本
 
  // 因为 pos 是在视野空间的，这意味着，它是一个从眼睛到某个位置的矢量表示。
  // 所以，沿着该矢量的反方向（朝着眼睛）移动一定距离
  
  var fromEye = m4.normalize(pos);
  var amountToMoveTowardEye = 150;  // because the F is 150 units long
  var viewX = pos[0] - fromEye[0] * amountToMoveTowardEye;
  var viewY = pos[1] - fromEye[1] * amountToMoveTowardEye;
  var viewZ = pos[2] - fromEye[2] * amountToMoveTowardEye;
  var desiredTextScale = -1 / gl.canvas.height * 2;  // 1x1 pixels
  var scale = viewZ * desiredTextScale;
 
  var textMatrix = m4.translate(projectionMatrix, viewX, viewY, viewZ);
  textMatrix = m4.scale(textMatrix, scale, scale, 1);
 
  m4.copy(textMatrix, textUniforms.u_matrix);
  webglUtils.setUniforms(textProgramInfo, textUniforms);
 
  // 渲染文本
  gl.drawArrays(gl.TRIANGLES, 0, vertices.numVertices);
});
```
And here's that

结果是:

![dynamic_text][9]

[查看网页][10]

That's the basic technique of using a texture atlas of glyphs. There are a few obvious things to add or ways to improve it.

这只是使用字形纹理地图的基本点。这还有很多工作可以去优化它。

Reuse the same arrays.

 - 重用同样的数组
    Currently makeVerticesForString allocates new Float32Arrays each time it's called. That's probably going to eventually cause garbage collection hiccups. Re-using the same arrays would probably be better. You'd enlarge the array if it's not large enough and keep that size around
    现在每次调用 `makeVerticesForString`，会分配新的 Float32Arrays。这最终可能会使垃圾回收装置过载。重用同样的数组可能会减轻上述的情况。如果这个数组不够大，你可以适当的扩大该数组的大小



Add support for carriage return

 - 支持换行符
    Check for \n and go down a line when generating vertices. This would make it easy to make paragraphs of text.
    当想要生成新的节点时，可以使用 `\n` 并且可以实现换行。这对于生成文本段落来说非常方便。

Add support for all kinds of other formatting.

 - 支持其他文本格式'
    If you wanted to center the text or justify it you could add all that.
    如果你想让文本居中或者任意调整它，这都可以加上。

Add support for vertex colors.

 - 支持文本颜色值
    Then you could color the text different colors per letter. Of course you'd have to decide how to specify when to change colors.
    接着，你可以针对于每个字母使用不用颜色做优化。当然，你也可以决定什么时候去改变颜色。

Consider generating the glyph texture atlas at runtime using a 2D canvas

 - 考虑使用 2D canvas 动态生成字形纹理地图

The other big issue which I'm not going to cover is that textures have a limited size but fonts are effectively unlimited. If you want to support all of Unicode so that you can handle Chinese and Japanese and Arabic and all the other languages, well, as of 2015 there are over 110,000 glyphs in Unicode! You can't fit all of those in textures. There just isn't enough room.

另外，还有一些其他的问题，这里，我并不打算提出来，主要是纹理有大小的限制但字体却多种多样。如果你想支持所有 Unicode 字体，这样，你就可以使用中文，日文，阿拉伯文以及其他的语言，当然，自 2015 起，Unicode 字体集里面已经有超过 110,000 的字形！不过，你不能用纹理来实现所有语言，因为并没有足够的空间可以使用。

The way the OS and browsers handle this when they're GPU accelerated is by using a glyph texture cache. Like above they might put textures in a texture atlas but they probably make the area for each glpyh a fixed size. They keep the most recently used glyphs in the texture. If they need to draw a glyph that's not in the texture they replace the least recently used one with the new one they need. Of course if that glyph they are about to replace is still being referenced by a quad yet to be drawn then they need to draw with what they have before replacing the glyph.


操作系统和浏览器在GPU加速时，处理这种情况的方式是使用字形纹理缓存。像上面，他们可能会将多个纹理放在一个纹理地图中，但是他们可能会给每个地图设置一个固定大小。他们会将最近使用的字形缓存在纹理中。如果他们需要绘制一个新的字形，并且该字形并不在当前纹理中，则会将很少使用的某个字形替换为新的字形。当然，如果他们将要替换的字形，仍然被一个单元格引用，而且将要被回执，那么他们需要在替换之前绘制它。

Another thing you can do, though I'm not recommending it, is combine this technique with the previous technique. You can render glyphs directly into another texture.

并且，你还可以使用另外一种方式来进行优化，尽管我不是很推荐它，就是将这项技术和[前一项技术][11]结合起来。你可以将字形直接渲染到另一个纹理。

Yet one more way to draw text in WebGL is to actually use 3D text. The 'F' in all the samples above is a 3D letter. You'd make one for each letter. 3D letters are common for titles and movie logos but not much else.

还有一种在 WebGL 中绘制文本的方式就是使用 3D 文本。在所有例子中的 F 字母实际上就是一个 3D 字母。你可以给每一个字母设置 3D 属性。3D 字母对于标题和电影图标来说非常适合，不过，也不是很多。

I hope that's covered text in WebGL.

我希望这已经讲清楚了 WebGL 中的文本。


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html
  [4]: http://static.zybuluo.com/jimmythr/j5uqyudjbra9y113rl1sc6rh/%7B5702B36C-4A8D-43B4-A21D-684946A9246E%7D.png
  [5]: http://webglfundamentals.org/webgl/webgl-text-glyphs.html
  [6]: http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html#texture-atlas
  [7]: http://opengameart.org/content/8x8-font-chomps-wacky-worlds-beta
  [8]: http://webglfundamentals.org/webgl/lessons/webgl-drawing-multiple-things
  [9]: http://static.zybuluo.com/jimmythr/08gwb1uhrqkgki0jq7a2fcu4/dynamic.png
  [10]: http://webglfundamentals.org/webgl/webgl-text-glyphs-texture-atlas.html
  [11]: http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html