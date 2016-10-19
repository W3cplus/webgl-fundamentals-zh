# 使用 WebGL 的 DrawImage

This article is a continuation of WebGL orthographic 3D. If you haven't read that I suggest you start there. You should also be aware of how textures and texture coordinates work please read WebGL 3D textures.

这篇文章是接着 [WebGL orthographic 3D][1] 讲解。如果你还没有阅读它，建议[你从那里开始][2]。你可以从 [WebGL 3D textures][3] 了解到纹理和纹理坐标是怎么工作的。

To implement most games in 2D requires just a single function to draw an image. Sure some 2d games do fancy thing with lines etc but if you only have a way to draw a 2D image on the screen you can pretty much make most 2d games.

通常，你只需要一个函数去绘制图像，就可以做出大部分的 2D 游戏。当然，某些 2d 游戏可以利用线条来做出很棒的效果，不过，如果你只知道如果在屏幕上绘制 2D 图像的话，你同样可以写出很多 2d 游戏。

The Canvas 2D api has very flexible function for drawing image called drawImage. It has 3 versions

Canvas 2D api 有一个很灵活的绘制图片的函数，叫做 `drawImage`。它有 3 个版本
```
ctx.drawImage(image, dstX, dstY);
ctx.drawImage(image, dstX, dstY, dstWidth, dstHeight);
ctx.drawImage(image, srcX, srcY, srcWidth, srcHeight,
                     dstX, dstY, dstWidth, dstHeight);
```
Given everything you've learned so far how would you implement this in WebGL? Your first solution might be to generate vertices like some of the first articles on this site did. Sending vertices to the GPU is generally a slow operation (although there are cases where it will be faster).

现在利用你所学过的知识，你现在如何在在 WebGL 中绘制图像呢？首先想到的应该就是，生成很多点进行绘制，和本系列的第一篇文章里介绍的一样。不过，把这些点交给 GPU 绘制，性能并不好 （虽然有办法让它变快）。

This is where the whole point of WebGL comes into play. It's all about creatively writing a shader and then creatively using that shader to solve your problem.

当然，这是 WebGL 的核心内容。通过创建一个着色器，然后利用这些着色器去姐姐你的问题。

Let's start with the first version

让我们先看第一个版本：
```
ctx.drawImage(image, x, y);
```
It draws an image at location x, y the same size as the image. To make a similar WebGL based funciton we could upload vertices that for x, y, x + width, y, x, y + height, and x + width, y + height then as we draw different images at different locations we'd generate different sets of vertices.

它将原始的图片的大小，画在了 x，y 的位置。为了实现一个相似的 WebGL 版的函数，我们可以上传很多点，代表着 x，y， x + width，y，x，y + height，和 x + width，y + height。当我们在不同点绘制不同的图像时，就可以生成相应的顶点集合。

A far more common way though is just to use a unit quad. We upload a single square 1 unit big. We then use matrix math to scale and translate that unit quad so that it ends up being at the desired place.

通常的做法是，使用单元格。我们上传一个单位大小的单元格。然后使用矩阵，将该单元格通过放缩，移动到我们期望的位置。

Here's the code.

请看代码。

First we need a simple vertex shader

首先，我们需要一个简单地顶点着色器
```
attribute vec4 a_position;
attribute vec2 a_texcoord;
 
uniform mat4 u_matrix;
 
varying vec2 v_texcoord;
 
void main() {
   gl_Position = u_matrix * a_position;
   v_texcoord = a_texcoord;
}
```
And a simple fragment shader

还需要一个简单的片元着色器
```
precision mediump float;
 
varying vec2 v_texcoord;
 
uniform sampler2D texture;
 
void main() {
   gl_FragColor = texture2D(texture, v_texcoord);
}
```
And now the function

接下来，就是上述的转换函数
```
// 纹理并不像图片一样有自带的宽高，
// 所以，我们需要传入匹配的宽高
function drawImage(tex, texWidth, texHeight, dstX, dstY) {
  gl.bindTexture(gl.TEXTURE_2D, tex);
 
  // 该矩阵会将像素值转换为裁剪坐标的值
  var projectionMatrix = make2DProjection(canvas.width, canvas.height, 1);
 
   // 该矩阵会放缩我们的单元格
  // 1 个单位代表着 texWidth，texHeight 单位
  var scaleMatrix = makeScale(texWidth, texHeight, 1);
 
  // 该矩阵会移动我们的单元格到 dstX，dstY
  var translationMatrix = makeTranslation(dstX, dstY, 0);
 
  // 将他们积乘起来
  var matrix = matrixMultiply(scaleMatrix, translationMatrix);
  matrix = matrixMultiply(matrix, projectionMatrix);
 
  // 传入该矩阵
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
 
  // 绘制该单元格（2 个三角形，6 个顶点）
  gl.drawArrays(gl.TRIANGLES, 0, 6);
}
```
Let's load some images into textures

接着，将图片传给 `textures`。
```
// 创建一个纹理信息 { width: w, height: h, texture: tex }
// 该纹理起始值为 1x1 像素 
// 当图片加载完成时，起始值就会更新
function loadImageAndCreateTextureInfo(url) {
  var tex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, tex);
 
  // 然我们假设所有的图片大小没有 2 的幂
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
 
  var textureInfo = {
    width: 1,   // 知道图片加载完成，我们才知道图片的大小
    height: 1,
    texture: tex,
  };
  var img = new Image();
  img.addEventListener('load', function() {
    textureInfo.width = img.width;
    textureInfo.height = img.height;
 
    gl.bindTexture(gl.TEXTURE_2D, textureInfo.texture);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, img);
  });
 
  return textureInfo;
}
 
var textureInfos = [
  loadImageAndCreateTextureInfo('resources/star.jpg'),
  loadImageAndCreateTextureInfo('resources/leaves.jpg'),
  loadImageAndCreateTextureInfo('resources/keyboard.jpg'),
];
```
And lets draw them at random places

我们将这些图片随机画在幕布上。
```
var drawInfos = [];
var numToDraw = 9;
var speed = 60;
for (var ii = 0; ii < numToDraw; ++ii) {
  var drawInfo = {
    x: Math.random() * gl.canvas.width,
    y: Math.random() * gl.canvas.height,
    dx: Math.random() > 0.5 ? -1 : 1,
    dy: Math.random() > 0.5 ? -1 : 1,
    textureInfo: textureInfos[Math.random() * textureInfos.length | 0],
  };
  drawInfos.push(drawInfo);
}
 
function update(deltaTime) {
  drawInfos.forEach(function(drawInfo) {
    drawInfo.x += drawInfo.dx * speed * deltaTime;
    drawInfo.y += drawInfo.dy * speed * deltaTime;
    if (drawInfo.x < 0) {
      drawInfo.dx = 1;
    }
    if (drawInfo.x >= gl.canvas.width) {
      drawInfo.dx = -1;
    }
    if (drawInfo.y < 0) {
      drawInfo.dy = 1;
    }
    if (drawInfo.y >= gl.canvas.height) {
      drawInfo.dy = -1;
    }
  });
}
 
function draw() {
  gl.clear(gl.COLOR_BUFFER_BIT);
 
  drawInfos.forEach(function(drawInfo) {
    drawImage(
      drawInfo.textureInfo.texture,
      drawInfo.textureInfo.width,
      drawInfo.textureInfo.height,
      drawInfo.x,
      drawInfo.y);
  });
}
 
var then = 0;
function render(time) {
  var now = time * 0.001;
  var deltaTime = Math.min(0.1, now - then);
  then = now;
 
  update(time);
  draw();
 
  requestAnimationFrame(render);
}
requestAnimationFrame(render);
```
You can see that running here

你可以看到运行情况

![s_random_photo][4]

[查看网页][5]

Handling version 2 of the original canvas drawImage function

看一下第 2 个版本的 canvas `drawImage` 函数。
```
ctx.drawImage(image, dstX, dstY, dstWidth, dstHeight);
```
Is really no different. We just use dstWidth and dstHeight instead of texWidth and texHeight.

这并没有什么不一样，我们只要使用 dstWidth 和 dstHeight 来代替 texWidth 和 texHeight。
```
function drawImage(tex, texWidth, texHeight, dstX, dstY, dstWidth, dstHeight) {
  if (dstWidth === undefined) {
    dstWidth = texWidth;
  }
 
  if (dstHeight === undefined) {
    dstHeight = texHeight;
  }
 
  gl.bindTexture(gl.TEXTURE_2D, tex);

  // 该矩阵会将像素值转换为裁剪坐标的值
  var projectionMatrix = make2DProjection(canvas.width, canvas.height, 1);
 
   // 该矩阵会放缩我们的单元格
  // 1 个单位代表着 dstWidth，dstHeight 单位
  var scaleMatrix = makeScale(dstWidth, dstHeight, 1);
 
  // 该矩阵会移动我们的单元格到 dstX，dstY
  var translationMatrix = makeTranslation(dstX, dstY, 0);
 
  // 将他们积乘起来
  var matrix = matrixMultiply(scaleMatrix, translationMatrix);
  matrix = matrixMultiply(matrix, projectionMatrix);
 
  // 传入该矩阵
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
 
  // 绘制该单元格（2 个三角形，6 个顶点）
  gl.drawArrays(gl.TRIANGLES, 0, 6);
}
```
I've updated the code to use different sizes

并且，上面的代码可以使用不同的尺寸。

![ss_random_photo.gif-374.6kB][6]

[查看网页][7]

So that was easy. But what about the 3rd version of canvas drawImage?

开起来挺简单的。那第 3 个版本的 canvas `drawImage` 是什么样的？
```
ctx.drawImage(image, srcX, srcY, srcWidth, srcHeight,
                     dstX, dstY, dstWidth, dstHeight);
```

In order to select part of the texture we need to manipulate the texure coordinates. How texture coordinates work was covered in the article about textures. In that article we manually created texture coordinates which is a very common way to do this but we can also create them on the fly and just like we're manipulating our positions using a matrix we can similarly manipluate texture coordinates using another matrix.

为了能过获取纹理中的指定部分，我们需要处理一下纹理坐标。[这篇文章][8]已经讲解了纹理坐标的工作原理。在那片文章中，我们手动创建了纹理坐标，这看起来并不难。不过，这里我们可以动态的创建他们，就像上面我们使用矩阵处理顶点一样，我们可以使用另外一个矩阵来处理纹理坐标。

Let's add a texture matrix to the vertex shader and multiply the texture coordinates by this texture matrix.

接着，在顶点着色器中添加一个纹理矩阵，然后将纹理坐标和该矩阵相乘。

```
attribute vec4 a_position;
attribute vec2 a_texcoord;
 
uniform mat4 u_matrix;
uniform mat4 u_textureMatrix;
 
varying vec2 v_texcoord;
 
void main() {
   gl_Position = u_matrix * a_position;
   v_texcoord = (u_textureMatrix * vec4(a_texcoord, 0, 1)).xy;
}
```
Now we need to look up the location of the texture matrix

现在，我们需要获得纹理矩阵的位置。
```
var matrixLocation = gl.getUniformLocation(program, "u_matrix");
var textureMatrixLocation = gl.getUniformLocation(program, "u_textureMatrix");
```
And inside drawImage we need to set it so it will select the part of the texture we want. We know the texture coordinates are also effectively a unit quad so it's very similar to what we've already done for the positions.

并且，在 `drawImage` 函数里，我们需要给该矩阵赋值，这样，它才能找到纹理中我们想要的那一个部分。实际上，纹理坐标也是一个有效的单元格，它的处理方式和我们处理上述 `positions` 类似。
```
function drawImage(
    tex, texWidth, texHeight,
    srcX, srcY, srcWidth, srcHeight,
    dstX, dstY, dstWidth, dstHeight) {
  if (dstX === undefined) {
    dstX = srcX;
  }
  if (dstY === undefined) {
    dstY = srcY;
  }
  if (srcWidth === undefined) {
    srcWidth = texWidth;
  }
  if (srcHeight === undefined) {
    srcHeight = texHeight;
  }
  if (dstWidth === undefined) {
    dstWidth = srcWidth;
  }
  if (dstHeight === undefined) {
    dstHeight = srcHeight;
  }
 
  gl.bindTexture(gl.TEXTURE_2D, tex);

  // 该矩阵会将像素值转换为裁剪坐标的值
  var projectionMatrix = make2DProjection(canvas.width, canvas.height, 1);
 
   // 该矩阵会放缩我们的单元格
  // 1 个单位代表着 dstWidth，dstHeight 单位
  var scaleMatrix = makeScale(dstWidth, dstHeight, 1);
 
  // 该矩阵会移动我们的单元格到 dstX，dstY
  var translationMatrix = makeTranslation(dstX, dstY, 0);
 
  // 将他们积乘起来
  var matrix = matrixMultiply(scaleMatrix, translationMatrix);
  matrix = matrixMultiply(matrix, projectionMatrix);
 
  // 传入该矩阵
  gl.uniformMatrix4fv(matrixLocation, false, matrix);
 
  // 因为纹理坐标值的范围是从 0 到 1
  // 并且，它是以一个单元格为最小单位
  // 所以，我们可能通过缩放单元格的大小，来选择我们想要的纹理区域
  var texScaleMatrix = makeScale(srcWidth / texWidth, srcHeight / texHeight, 1);
  var texTranslationMatrix = makeTranslation(srcX / texWidth, srcY / texHeight, 0);
 
  // 将他们积乘起来
  var texMatrix = matrixMultiply(texScaleMatrix, texTranslationMatrix);
 
  // 传入该矩阵
  gl.uniformMatrix4fv(textureMatrixLocation, false, texMatrix);
 
  // 绘制该单元格（2 个三角形，6 个顶点）
  gl.drawArrays(gl.TRIANGLES, 0, 6);
}
```
I also updated the code to pick parts of the textures. Here's the result

这里，我将上述代码更新了，这样就可以选择纹理的部分内容。

![scale_random_photo.gif-589.1kB][9]

[查看网页][10]

Unlike the canvas 2D api our WebGL version handles cases the canvas 2D drawImage does not.

不像 canvas 2D api 一样，我们的 WebGL 版的可以处理 canvas 不能处理的情况。

For one we can pass in a negative width or height for either source or dest. A negative srcWidth will select pixels to the left of srcX. A negative dstWidth will draw to the left of dstX. In the canvas 2D api these are errors at best or undefined behavior at worst.

例如，我们可以给 `source` 或 `dest` 传入负的宽或高。负的 `srcWidth` 会相对于 `srcX` 的左边来获取像素内容。负的 `dstWidth` 会相当于 `dstX` 的左边来绘制像素。在 canvas 2D api 里，如果传入负值的haunted，会抛出错误或者发生不可描述的行为。

![rotate_random_photo.gif-662.5kB][11]

[查看网页][12]

Another is since we're using a matrix we can do any matrix math we want.

另外，因为我们使用的是矩阵，所有我们可以做[任何的矩阵运算][13]。

For example we could rotate the texture coordinates around the center of the texture.

例如，我们可以让着纹理中心旋转纹理坐标

Changing the texture matrix code to this

改变纹理的矩阵代码如下：
```
// 其实就像 2d 投影的矩阵，只是它是在纹理空间而非裁剪空间中
var texProjectionMatrix = makeScale(1 / texWidth, 1 / texHeight, 1);

// 因为，我们使用了一个投影矩阵，将坐标值转化为像素值
// 所以，放缩和平移是直接使用的像素单位
var texMatrix = m4.scaling(1 / texWidth, 1 / texHeight, 1);
 
  // 我们需要选择一个基准点去旋转
  // 我们会将其移动他中间，接着旋转，然后往回移动
  var texMatrix = m4.translate(texMatrix, texWidth * 0.5, texHeight * 0.5, 0);
  var texMatrix = m4.zRotate(texMatrix, srcRotation);
  var texMatrix = m4.translate(texMatrix, texWidth * -0.5, texHeight * -0.5, 0);
 
  // 因为在像素空间中
  // 所以，缩放和移动使用的是像素单位
  var texMatrix = m4.translate(texMatrix, srcX, srcY, 0);
  var texMatrix = m4.scale(texMatrix, srcWidth, srcHeight, 1);

  // 设置纹理矩阵
  gl.uniformMatrix4fv(textureMatrixLocation, false, texMatrix);
```

![rotate_random_photo.gif-662.5kB][14]

[查看网页][15]

you can see one problem which is that because of the rotation sometimes we see past the edge of the texture. As it's set to CLAMP_TO_EDGE the edge just gets repeated.

不过，这里有个问题，当图片在旋转时，我们能够看见残留的纹理边缘。因为它设置的是 `CLAMP_TO_EDGE`，所以，边缘会出现重复的现象。

We could fix that by discarding any pixels outside of the 0 to 1 range inside the shader. discard exits the shader immediately without writing a pixel.

我们可以通过，在着色器中，移除不在 [0,1] 返回之间的像素值，去修复该问题。`discard` 会立即终止着色器，从而不会再绘制像素。
```
precision mediump float;
 
varying vec2 v_texcoord;
 
uniform sampler2D texture;
 
void main() {
   if (v_texcoord.x < 0.0 ||
       v_texcoord.y < 0.0 ||
       v_texcoord.x > 1.0 ||
       v_texcoord.y > 1.0) {
     discard;
   }
   gl_FragColor = texture2D(texture, v_texcoord);
}
```
And now the corners are gone

现在，那些边缘模糊角便消失了。

![rotate_dual_random_photo.gif-641.1kB][16]


[查看网页][17]

or maybe you'd like to use a solid color when the texture coordinates are outside the texture

或许你想使用纯色，当纹理坐标超出了当前的纹理区域。
```
precision mediump float;
 
varying vec2 v_texcoord;
 
uniform sampler2D texture;
 
void main() {
   if (v_texcoord.x < 0.0 ||
       v_texcoord.y < 0.0 ||
       v_texcoord.x > 1.0 ||
       v_texcoord.y > 1.0) {
     gl_FragColor = vec4(0, 0, 1, 1); // blue
     return;
   }
   gl_FragColor = texture2D(texture, v_texcoord);
}
```

![blue_random_photo.gif-552kB][18]

The sky's really the limit. It's all up to your creative use of shaders.

当然，创造性的使用着色器是没有限制的。

Next up we'll implement canvas 2d's matrix stack.

接下来，[我们将实践 canvas 2d's 的矩阵栈][19]

## 细小的优化

I'm not recommending this optimization. Rather I want to point out more creative thinking since WebGL is all about creative use of the features it provides.

我其实并不推荐这个优化。不过，我想说的是如何创造性的思维，因为 WebGL 的本质其实就是你如果创造性的使用它提供的功能。

You might have noticed we're using a unit quad for our positions and those positions of a unit quad exactly match our texture coordinates. As such we can use the positions as the texture coordinates.

你可能已经注意到，我们使用单元格来表达我们的位置，这些单元格的位置，实际上是和我们的纹理坐标相匹配的。因为这样，我们就可以像使用纹理坐标一样，使用这里位置信息。

```
attribute vec4 a_position;
// attribute vec2 a_texcoord;
 
uniform mat4 u_matrix;
uniform mat4 u_textureMatrix;
 
varying vec2 v_texcoord;
 
void main() {
   gl_Position = u_matrix * a_position;
   v_texcoord = (u_textureMatrix * a_position).xy;
}
```
We can now remove the code that setup the texture coordinates and it will work just the same as before.

我们现在可以将用来建立纹理坐标的代码删除，并且它还是可以像以前一样正常工作。

![blue_rotate_random_photo.gif-432kB][20]

[查看网页][21]


  [1]: http://webglfundamentals.org/webgl/lessons/webgl-3d-orthographic.html
  [2]: http://webglfundamentals.org/webgl/lessons/webgl-3d-orthographic.html
  [3]: http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html
  [4]: http://static.zybuluo.com/jimmythr/9t89y4icxy7lihfpkrn3n8kf/s_random_photo.gif
  [5]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-01.html
  [6]: http://static.zybuluo.com/jimmythr/txeep3jcwbvj9d2zo8zv5n6l/ss_random_photo.gif
  [7]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-02.html
  [8]: http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html
  [9]: http://static.zybuluo.com/jimmythr/k4bjawwpuwc0pvjpauf0c58r/scale_random_photo.gif
  [10]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-03.html
  [11]: http://static.zybuluo.com/jimmythr/ce6ycw1z1jib7hl04g234wct/rotate_random_photo.gif
  [12]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-04.html
  [13]: http://webglfundamentals.org/webgl/lessons/webgl-2d-matrices.html
  [14]: http://static.zybuluo.com/jimmythr/h9ca3mtg63o4i6omchhbkoty/rotate_random_photo.gif
  [15]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-05.html
  [16]: http://static.zybuluo.com/jimmythr/gkloel26jibijnj3v0e2yd8x/rotate_dual_random_photo.gif
  [17]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-07.html
  [18]: http://static.zybuluo.com/jimmythr/jb51jlsh8l0b8syhgrfnbeww/blue_random_photo.gif
  [19]: http://webglfundamentals.org/webgl/lessons/webgl-2d-matrix-stack.html
  [20]: http://static.zybuluo.com/jimmythr/f4tzaawfml3ykfxkbzgyk4wp/blue_rotate_random_photo.gif
  [21]: http://webglfundamentals.org/webgl/webgl-2d-drawimage-08.html