## WebGL 3D - Textures

## WebGL 3D - 纹理

This post is a continuation of a series of posts about WebGL. The first [started with fundamentals][1] and the previous was about [animation][2].

这篇文章是 WebGL 系列文章的续篇。第一篇是[从基础原理开始][1]，前一篇是[动画][2]

How do we apply textures in WebGL? You could probably derive how by reading [the articles on image processing][3] but it will probably be easier to understand if we go over it in more detail.

我们在 WebGL 中如何处理纹理？你可以阅读[关于图像处理的文章][3]了解到，但如果我们更详细的重温可能会更容易。

The first thing we need to do is adjust our shaders to use textures. Here are the changes to the vertex shader. We need to pass in texture coordinates. In this case we just pass them straight through to the fragment shader.

首先，使用纹理我们就需要对着色器进行调整。以下是对顶点着色器（vertex shader）的 调整。我们需要使用纹理坐标。在这里示例中我们直接将它们传递给片元着色器（fragment shader）。

```
	attribute vec4 a_position;
	attribute vec2 a_texcoord;
	 
	uniform mat4 u_matrix;
	 
	varying vec2 v_texcoord;
	 
	void main() {
	  // Multiply the position by the matrix.
	  // position 乘以 matrix。
	  gl_Position = u_matrix * a_position;
	 
	  // Pass the texcoord to the fragment shader.
	  // 传递 texcoord 给片元着色器。
	  v_texcoord = a_texcoord;
	}
```
In the fragment shader we declare a uniform sampler2D which lets us reference a texture. We use the texture coordinates passed from the vertex shader and we call texture2D to look up a color from that texture.

在片元着色器中我们声明一个 uniform sampler2D 变量引用纹理。使用顶点着色器传入的纹理坐标和 texture2D 方法查找纹理。

```
	precision mediump float;
	 
	// Passed in from the vertex shader.
	// 从顶点着色器中传入。
	varying vec2 v_texcoord;
	 
	// The texture.
	// 纹理。
	uniform sampler2D u_texture;
	 
	void main() {
	   gl_FragColor = texture2D(u_texture, v_texcoord);
	}
```
We need to setup the texture coordinates

我们需要建立纹理坐标。

```
	// look up where the vertex data needs to go.
	// 查找顶点数据的位置。
	var positionLocation = gl.getAttribLocation(program, "a_position");
	var texcoordLocation = gl.getAttribLocation(program, "a_texcoords");
	 
	...
	 
	// Create a buffer for texcoords.
	// 纹理坐标创建缓冲区。
	var buffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
	gl.enableVertexAttribArray(texcoordLocation);
	 
	// We'll supply texcoords as floats.
	// 提供浮点型的纹理坐标。
	gl.vertexAttribPointer(texcoordLocation, 2, gl.FLOAT, false, 0, 0);
	 
	// Set Texcoords.
	// 设置纹理坐标；
	setTexcoords(gl);
```
And you can see the coordinates we're using which are mapping the entire texture to each quad on our 'F'.

你可以看到我们使用的坐标将整个纹理映射到‘F‘的每一个矩形上。

```
	// Fill the buffer with texture coordinates for the F.
	// 将 F 用纹理坐标填充缓冲区。
	function setTexcoords(gl) {
	  gl.bufferData(
	      gl.ARRAY_BUFFER,
	      new Float32Array([
	        // left column front
	        // 左边竖线
	        0, 0,
	        0, 1,
	        1, 0,
	        0, 1,
	        1, 1,
	        1, 0,
	 
	        // top rung front
	        // 顶部横线
	        0, 0,
	        0, 1,
	        1, 0,
	        0, 1,
	        1, 1,
	        1, 0,
	 ...
	       ]),
	       gl.STATIC_DRAW);
```
We also need a texture. We could make one from scratch but in this case let's load an image since that's probably the most common way.

我们需要一个纹理。可以从头开始做一个，但最常见的方法可能是加载一个图片。

Here's the image we're going to use

这是我们将要使用的图片

![textrue][4]

What an exciting image! Actually an image with an 'F' on it has a clear direction so it's easy to tell if it's turned or flipped etc when we use it as a texture.

多么令人兴奋的图片！实际上图片中的‘F’已经有一个明确的方向了，当我们使用它作为一个纹理时，很容易判断它是否旋转或翻转。

The thing about loading an image is it happens asynchronously. We request the image to be loaded but it takes a while for the browser to download it. There are generally 2 solutions to this. We could make the code wait until the texture has downloaded and only then start drawing. The other solution is to make up some texture to use until the image is downloaded. That way we can start rendering immediately. Then, once the image has been downloaded we copy the image to the texture. We'll use that method below.

图片是异步加载的。我们请求加载图像，浏览器需要一段时间去加载。一般有两种解决方法。一种方法是等待纹理加载完再开始绘制。另一种方法是图像加载完再补充纹理，这样我们可以立即开始渲染。一旦图像加载完，就将图像复制到纹理中。我们将使用以下方法。

```
	// Create a texture.
	//创建纹理。
	var texture = gl.createTexture();
	gl.bindTexture(gl.TEXTURE_2D, texture);
	 
	// Fill the texture with a 1x1 blue pixel.
	// 用 1x1 的蓝色像素填充到纹理中。
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE,
	              new Uint8Array([0, 0, 255, 255]));
	 
	// Asynchronously load an image
	// 异步加载图像
	var image = new Image();
	image.src = "resources/f-texture.png";
	image.addEventListener('load', function() {
	  // Now that the image has loaded make copy it to the texture.
	  // 图像加载完后复制到纹理中。
	  gl.bindTexture(gl.TEXTURE_2D, texture);
	  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
	  gl.generateMipmap(gl.TEXTURE_2D);
	});
```
And here it is

效果如下

<iframe class="webgl_example"  style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures.html"></iframe>

[click here to open in a separate window][5] 


[点击这里在新窗口打开][5]

What if we wanted to just use a part of the texture across the front of the 'F'? Textures are referenced with "texture coordinates" and texture coordinates go from 0.0 to 1.0 from left to right across the texture and 0.0 to 1.0 from bottom to top up the texture.

如果我们只想在‘F’的正面使用纹理的一部分呢？纹理是被‘纹理坐标’引用的，在纹理中纹理坐标从左到右的范围是 [0.0，1.0]，从下到上的范围是 [0.0，1.0]。

![svg][6]

So if we match up our vertices to the texture we can figure out what texture coordinates to use.

因此，如果我们将顶点匹配到纹理中，我们可以计算出需要使用的纹理坐标。

![svg1][7]

Here are the texture coordinates for the front.

这是正面的纹理坐标

```
	    // left column front
	    // 左边竖线
	    0.22, 0.19,
	    0.22, 0.79,
	    0.34, 0.19,
	    0.22, 0.79,
	    0.34, 0.79,
	    0.34, 0.19,
	 
	    // top rung front
	    // 顶部横线
	    0.34, 0.19,
	    0.34, 0.31,
	    0.62, 0.19,
	    0.34, 0.31,
	    0.62, 0.31,
	    0.62, 0.19,
	 
	    // middle rung front
	    // 中间横线
	    0.34, 0.43,
	    0.34, 0.55,
	    0.49, 0.43,
	    0.34, 0.55,
	    0.49, 0.55,
	    0.49, 0.43,
```

And here it is.

效果如下。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-texture-coords-mapped.html"></iframe> 

[click here to open in a separate window][8]

[点击这里在新窗口打开][8]

Not a very exciting display but hopefully it demonstrates how to use texture coordinates. If you're making geometry in code (cubes, spheres, etc) it's usually pretty easy to compute whatever texture coordinates you want. On the other hand if you're getting 3d models from 3d modeling software like Blender, Maya, 3D Studio Max, then your artists (or you) will adjust texture coordinates in those packages.

不是一个令人非常兴奋的展示，但它如期演示了如何使用纹理坐标。如果你在代码中使用了几何体（立方体，球体等），它们通常很容易就能计算出任何你想要的纹理坐标。另一方面，如果你从 3d 建模软件如 Blender，Maya，3D Studio Max 中获取 3d 模型，你的设计师（或者你）会在这些软件包中校准纹理坐标。

So what happens if we use texture coordinates outside the 0.0 to 1.0 range. By default WebGL repeats the texture. 0.0 to 1.0 is one 'copy' of the texture. 1.0 to 2.0 is another copy. even -4.0 to -3.0 is yet another copy. Let's display a plane using these texture coordinates.

如果我们使用 [0.0，1.0] 之外的纹理坐标会发生什么。WebGL 默认是重复纹理的。[0.0， 1.0] 是一个被‘复制’的纹理。[1.0，2.0] 是一个副本。即使是 [－4.0，－3.0] 也是另一个副本。让我们使用这些纹理坐标展示平面吧。

```
 -3, -1,
  2, -1,
 -3,  4,
 -3,  4,
  2, -1,
  2,  4,
```
and here it is

效果如下

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-repeat-clamp.html"></iframe> 

[click here to open in a separate window][9]

[点击这里在新窗口打开][9]

You can tell WebGL to not repeat the texture in a certain direction by using CLAMP_TO_EDGE. For example

你可以利用 CLAMP_TO_EDGE 告诉 WebGL 在特定的方向不重复纹理。例如

```
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
```
Click the buttons in the sample above to see the difference.

点击上面示例中的按钮，查看差异。

You might have noticed a call to `gl.generateMipmap` back when we loaded the texture. What is that for?

你可能注意到了，当我们加载完纹理之后调用了`gl.generateMipmap`。这有什么作用？

Imagine we had this 16x16 pixel texture.

假设我们有这个 16x16 像素的纹理。


![16x16][10]

Now imagine we tried to draw that texture on a polygon 2x2 pixels big on the screen. What colors should we make those 4 pixels? There are 256 pixels to choose from. In Photoshop if you scaled a 16x16 pixel image to 2x2 it would average the 8x8 pixels in each corner to make the 4 pixels in a 2x2 image. Unfortunately reading 64 pixels and averaging them all together would be way too slow for a GPU. In fact imagine if you had a 2048x2084 pixel texture and you tried to draw it 2x2 pixels. To do what Photoshop does for each of the 4 pixels in the 2x2 result it would have to average 1024x1024 pixel or 1 million pixels times 4. That's way way too much to do and still be fast.

现在假设我们尝试在屏幕中将它绘制到 2x2 像素的多边形中。我们应该给这 4 个像素什么颜色？有256个像素的颜色可以选择。在 Photoshop 中，如果你将一个 16x16 的图像缩小至 2x2，它会将每个角落的 8x8 像素平均分配到一个 2x2 图像的 4 像素中。不幸的是，一个 GPU 读取 64 个像素并平均分配，速度太慢。实际上，假设你有一个 2048x2048 像素的纹理，你试图将它绘制到 2x2 像素中，它必须平均分配 1024x1024 像素或 100 万像素乘以 4，以达到 Photoshop 分配给 2x2 的 4 像素的效果。这样的方法很多并且都很快速。

So what the GPU does is it uses a mipmap. A mipmap is a collection of progressively smaller images, each one 1/4th the size of the previous one. The mipmap for the 16x16 texture above would look something like this.

GPU 使用 mipmap 实现。mipmap 是一个图像渐小的集合，每一个 mip 的大小是前一个的 1/4。对于上述的 16x16 的纹理在 mipmap 中是这个样子。


![mipmap][11]

Generally each smaller level is just a bilinear interpolation of the previous level and that's what gl.generateMipmap does. It looks at the biggest level and generates all the smaller levels for you. Of course you can supply the smaller levels yourself if you want.

一般来说，每一个较小的只是前一个的一个双线性插值，这就是 gl.generateMipmap 所做的事情。它看起来是在最大 mip 的基础上，生成所有较小的 mip。当然你可以提供更小的 mip。

Now if you try to draw that 16x16 pixel texture only 2x2 pixels on the screen WebGL can select the mip that's 2x2 which has already been averaged from the previous mips.

现在如果你在屏幕中的 2x2 像素上绘制 16x16 像素的纹理，WebGL 可以从已经平均分配好的 mips 中选择 2x2 的 mip。

You can choose what WebGL does by setting the texture filtering for each texture. There are 6 modes

你可以为每一个纹理设置纹理过滤器从而选择 WebGL 的行为。一共有 6 种模式。

*	NEAREST = choose 1 pixel from the biggest mip
*	NEAREST = 从最大的 mip 中选择 1 个像素
*	LINEAR = choose 4 pixels from the biggest mip and blend them
*	LINEAR = 从最大的 mip 中选择 4 个像素并混淆它们
*	NEAREST_MIPMAP_NEAREST = choose the best mip, then pick one pixel from that mip
*	NEAREST_MIPMAP_NEAREST = 在最恰当的 mip中选择 1 个像素
*	LINEAR_MIPMAP_NEAREST = choose the best mip, then blend 4 pixels from that mip
*	LINEAR_MIPMAP_NEAREST = 在最恰当的 mip中选择 4 个像素并混淆
*	NEAREST_MIPMAP_LINEAR = choose the best 2 mips, choose 1 pixel from each, blend them
*	NEAREST_MIPMAP_LINEAR = 从最恰当的 2 个 mip中，各自选择 1 个像素，混淆它们
*	LINEAR_MIPMAP_LINEAR = choose the best 2 mips. choose 4 pixels from each, blend them
*	LINEAR_MIPMAP_LINEAR = 从最恰当的 2 个的 mip中，各自选择 4 个像素，混淆它们

You can see the importance of mips in these 2 examples. The first one shows that if you use `NEAREST` or `LINEAR` and only pick from the largest image then you'll get a lot of flickering because as things move, for each pixel it draws it has to pick a single pixel from the largest image. That changes depending on the size and position and so sometimes it will pick one pixel, other times a different one and so it flickers.

你可以在这两个示例中看到 mips 的重要性。第一个展示了如果使用`NEAREST`或者`LINEAR`，WebGL 只会选择最大的图像，你会看到很多的闪烁，因为随着模型的移动，绘制的每一个像素都必须从最大的图像中选择 1 个像素。这种变化取决于大小和位置的变化，有时它会选择一个像素，其他时间选择另外的像素，因此它会闪烁。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-mips.html"></iframe> 

[click here to open in a separate window][12] 

[点击这里在新窗口打开][12]

Notice how much the ones on the left and middle flicker where as the ones on the right flicker less. The ones on the right also have blended colors since they are using the mips. The smaller you draw the texture the further apart WebGL is going to pick pixels. That's why for example the bottom middle one, even though it's using LINEAR and blending 4 pixels it flickers because those 4 pixels are from different corners of the 16x16 image depending on which 4 are picked you'll get a different color. The one on the bottom right though stays a consistent color because it's using the 2nd to the smallest mip.

注意左边和中间的闪烁比右边的多。而右边使用的 mips 也有混合颜色值。WebGL 对于越远的部分会选择越小像素的纹理填充。为什么示例中的中下部分，即使是使用 LINEAR 并混淆了 4 像素，仍然闪烁，这是因为这 4 个像素来自 16x16 图像中不同的角落，颜色取决于选择了哪 4 个像素。右下的保持一致的颜色，是因为它使用了第二个最小的 mip。

The second example shows polygons that go deep into the screen.

第二个示例展示了在屏幕深处的多边形。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-mips-tri-linear.html"></iframe> 

[click here to open in a separate window][13]

[点击这里在新窗口打开][13]

The 6 beams going into the screen are using the 6 filtering modes listed above. The top left beam is using `NEAREST` and you can see it's clearly very blocky. The top middle is using `LINEAR` and it's not much better. The top right is using `NEAREST_MIPMAP_NEAREST`. Click on the image to switch to a texture where every mip is a different color and you'll easily see where it chooses to use a specific mip. The bottom left is using `LINEAR_MIPMAP_NEAREST` meaning it picks the best mip and then blends 4 pixels within that mip. You can still see a clear area where it switches from one mip to the next mip. The bottom middle is using `NEAREST_MIPMAP_LINEAR` meaning picking the best 2 mips, picking one pixel from each and blending them. If you look close you can see how it's still blocky, especially in the horizontal direction. The bottom right is using `LINEAR_MIPMAP_LINEAR` which is picking the best 2 mips, picking 4 pixels from each, and blends all 8 pixels.

进入屏幕的 6 个光束分别使用上面列出的 6 种过滤模式。左上的光使用了`NEAREST`，你可以看到非常清晰的块状。中上的光使用了`LINEAR`，它就看起来没那么清晰。右上的光使用了`NEAREST_MIPMAP_NEAREST`。点击图像切换纹理，每一个 map 的颜色都不同，你很容易的看到 WebGL 使用了哪个 mip。左下的光使用了`LINEAR_MIPMAP_NEAREST`，它会从最恰当的 mip 中选择 4 个像素并混淆。你任然可以清楚的看到从一个 mip 切换到下一个 mip 的区域。中下的光使用了`NEAREST_MIPMAP_LINEAR`，它会从最恰当的 2 个 mip 中各自选择 1 个像素，混淆它们。如果仔细看，你可以看到，尤其是在水平方向，它仍然是块状的，。右下的光使用了`LINEAR_MIPMAP_LINEAR`，它选择 2 个最恰当的 mip，从各自中选择 4 个像素，并混淆这 8 个像素。

![different colored mips][14]

different colored mips

不同颜色的 mips

You might be thinking why would you ever pick anything other than `LINEAR_MIPMAP_LINEAR` which is arguably the best one. There are many reasons. One is that `LINEAR_MIPMAP_LINEAR` is the slowest. Reading 8 pixels is slower than reading 1 pixel. On modern GPU hardware it's probably not an issue if you are only using 1 texture at a time but modern games might use 2 to 4 textures at once. 4 textures * 8 pixels per texture = needing to read 32 pixels for every pixel drawn. That's going to be slow. Another reason is if you're trying to achieve a certain effect. For example if you want something to have that pixelated retro look maybe you want to use `NEAREST`. Mips also take memory. In fact they take 33% more memory. That can be a lot of memory especially for a very large texture like you might use on a title screen of a game. If you are never going to draw something smaller than the largest mip why waste memory for those mips. Instead just use `NEAREST` or `LINEAR` as they only ever use the first mip.

你会想为什么选择除`LINEAR_MIPMAP_LINEAR`之外的选项无疑是最好的。这有很多的原因。其中一个是`LINEAR_MIPMAP_LINEAR`的响应速度是最慢的。读取 8 个像素比读取 1 个像素慢。如果你一次只使用1个纹理，对于现代GPU的硬件可能不是问题，但是现代游戏中可能会同时使用 2 到 4 个纹理。4 个纹理 * 8 个像素 = 每画一个像素需读取 32 个像素。这响应速度将会变慢。另一个原因是在你试图达到一定的效果时。例如，如果你希望图像看起来复古，你可能会使用`NEAREST`。Mips 是占用内存的。实际上它们会占用 33% 以上的内存。假如你的游戏在一个极小的屏幕上使用一个很大的纹理，会占用更多的内存。如果你绘制的图像比最大的 mip 都大，那为什么要浪费内存来存储这些 mips 呢。`NEAREST`或者`LINEAR`只使用第一张 mip。

To set filtering you call `gl.texParameter` like this

你可以这样调用`gl.texParameteri`设置过滤器。

```
	gl.texParameter(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
	gl.texParameter(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
```
`TEXTURE_MIN_FILTER` is the setting used when the size you are drawing is smaller than the largest mip. `TEXTURE_MAG_FILTER` is the setting used when the size you are drawing is larger than the largest mip. For `TEXTURE_MAG_FILTER` only `NEAREST` and `LINEAR` are valid settings.

`TEXTURE_MIN_FILTER`是用于图片尺寸小于最大的 mip 的。`TEXTURE_MAG_FILTER`是用于图片尺寸大于最大的 mip 的。`TEXTURE_MAG_FILTER`只有`NEAREST`和`LINEAR`两个有效值。

Let's say we wanted to apply this texture.

我们想应用这个纹理。

![keyboard][15]

Here it is.

效果如下。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-bad-npot.html"></iframe> 

[click here to open in a separate window][16]

[点击这里在新窗口打开][16]

Why doesn't the keyboard texture show up? That's beacuse WebGL has a kind of severe restriction on textures that are not a power of 2 in both dimensions. Powers of 2 are 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, etc. The 'F' texture was 256x256. 256 is a power of 2. The keyboard texture is 320x240. Neither of those are a power of 2 so trying to display the texture fails. In the shader when `texture2D` is called and when the texture referenced is not setup correctly WebGL will use color (0, 0, 0, 1) which is black. If you open up the JavaScript console or Web Console, depending on the browser you might see errors pointing out the problem like this

为什么键盘纹理没有显示出来？这是因为 WebGL 对于纹理的限制很严格，纹理的每个方向上尺寸都须为 2 的指数，而这个纹理每个方向上都不符合。2 的倍数有 1，2，4，8，16，32，64，128，256，512，1024，2048，等等。‘F’的纹理尺寸是 256x256。256 是 2 的指数。键盘纹理尺寸是 320x240。两者均不是 2 的指数，所以显示纹理失败。当调用`texture2D`，引用的纹理没有被正确设置时，WebGL 会使用黑色（0，0，0，1）代替。如果你打开 JavaScript 控制台或者 WebGL 控制台，这取决于你使用的浏览器，你可能会看到这样的错误信息

```
	WebGL: INVALID_OPERATION: generateMipmap: level 0 not power of 2
	   or not all the same size
	WebGL: drawArrays: texture bound to texture unit 0 is not renderable.
	   It maybe non-power-of-2 and have incompatible texture filtering or
	   is not 'texture complete'.
```
To fix it we need to set the wrap mode to `CLAMP_TO_EDGE` and turn off mip mapping by setting filtering to `LINEAR` or `NEAREST`.

为了解决这个问题，我们需要设置环绕模式为 `CLAMP_TO_EDGE`，并且关闭过滤器设置的`LINEAR`或者`NEAREST`的 mip 映射。

Let's update our image loading code to handle this. First we need a function that will tell us if a value is a power of 2.

让我们更新图片加载的代码。首先我们需要一个函数判断是否为 2 的指数。

```
	function isPowerOf2(value) {
	  return (value & (value - 1)) == 0;
	}
```
I'm not going to go into the binary math on why this works. Since it does work though, we can use it as follows.

这里我不打算深究二进制数学。既然它有效，我们可以如下使用它。

```
	// Asynchronously load an image
	// 异步加载图像
	var image = new Image();
	image.src = "resources/keyboard.jpg";
	image.addEventListener('load', function() {
	  // Now that the image has loaded make copy it to the texture.
	  // 现在图像已经加载完，将它复制到纹理中。
	  gl.bindTexture(gl.TEXTURE_2D, texture);
	  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
	 
	  // Check if the image is a power of 2 in both dimensions.
	  // 检查图像在每个方向上是否均为 2 的指数。
	  if (isPowerOf2(image.width) && isPowerOf2(image.height)) {
	     // Yes, it's a power of 2. Generate mips.
	     // 是 2 的指数，则生成 mips。
	     gl.generateMipmap(gl.TEXTURE_2D);
	  } else {
	     // No, it's not a power of 2. Turn of mips and set wrapping to clamp to edge
	     // 不是 2 的指数，则设置 wrapping 包裹 mips 的边缘。
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
	  }
	}
```
And here's that

效果如下

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-good-npot.html"></iframe>

[click here to open in a separate window][17]

[点击这里在新窗口打开][17]

A common question is "How do I apply a different image to each face of a cube?". For example let's say we had these 6 images.

一个常见的问题是“我如何在立方体的每一面中应用不同的图像？”。例如，我们有这 6 个图像。

![images of cube][18]

3 answers come to mind

有 3 种方法实现

1）  make a complicated shader that references 6 textures and pass in some extra per vertex info into the vertex shader that gets passed to the fragment shader to decide which texture to use. DON'T DO THIS! A little thought would make it clear that you'd end up having to write tons of different shaders if you wanted to do the same thing for different shapes with more sides etc.

1）使用一个复杂的着色器；这个着色器引用了 6 个纹理，并向顶点着色器（vertex shader）传入每个顶点的更多信息，让片元着色器（fragment shader）决定使用哪个纹理。不要这样做！稍微思考一下就会明白，如果你想在更多面的立体中应用同样的方法，你最后必须写大量的着色器。

2） draw 6 planes instead of a cube. This is a common solution. It's not bad but it also only really works for small shapes like a cube. If you had a sphere with 1000 quads and you wanted to put a different texture on each quad you'd have to draw 1000 planes and that would be slow.

2）绘制 6 个平面而不是一个立方体。这是常见的解决方法。这个方法并不差，但它只有在像立方体这样的小形状上才真正起效。如果你有一个拥有 1000 个矩形的球体，你想为每一个矩形设置不同的纹理，你必须绘制 1000 个平面，这将是漫长的过程。

3） The, dare I say, best solution is to put all of the images in 1 texture and use texture coordinates to map a different part of the texture to each face of the cube. This is the technique that pretty much all high performance apps (read games) use. So for example we'd put all the images in one texture possibly like this.

3）最好的解决办法是将所有的图像放入 1 个纹理中，使用纹理坐标为立方体的每一面映射不同的纹理部分。这几乎是所有高性能应用程序（游戏）都使用的技术。因此，我们会如下存放所有图像到一个纹理中。

![all the images in one texture ][19]

and then use a different set of texture coordinates for each face of the cube.

然后为立方体的每一面设置不同的纹理坐标。

```
	    // select the bottom left image
	    // 选择左下的图像
	    0   , 0  ,
	    0   , 0.5,
	    0.25, 0  ,
	    0   , 0.5,
	    0.25, 0.5,
	    0.25, 0  ,
	    // select the bottom middle image
	    // 选择中下的图像
	    0.25, 0  ,
	    0.5 , 0  ,
	    0.25, 0.5,
	    0.25, 0.5,
	    0.5 , 0  ,
	    0.5 , 0.5,
	    // select to bottom right image
	    // 选择右下的图像
	    0.5 , 0  ,
	    0.5 , 0.5,
	    0.75, 0  ,
	    0.5 , 0.5,
	    0.75, 0.5,
	    0.75, 0  ,
	    // select the top left image
	    // 选择左上的图像
	    0   , 0.5,
	    0.25, 0.5,
	    0   , 1  ,
	    0   , 1  ,
	    0.25, 0.5,
	    0.25, 1  ,
	    // select the top middle image
	    // 选择中上的图像
	    0.25, 0.5,
	    0.25, 1  ,
	    0.5 , 0.5,
	    0.25, 1  ,
	    0.5 , 1  ,
	    0.5 , 0.5,
	    // select the top right image
	    // 选择右上的图像
	    0.5 , 0.5,
	    0.75, 0.5,
	    0.5 , 1  ,
	    0.5 , 1  ,
	    0.75, 0.5,
	    0.75, 1  ,
```
And we get

我们得到如下效果

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-texture-atlas.html"></iframe> 

[click here to open in a separate window][20]

[点击这里在新窗口打开][20]

This style of applying multiple images using 1 texture is often called a texture atlas. It's best because there's just 1 texture to load, the shader stays simple as it only has to reference 1 texture, and it only requires 1 draw call to draw the shape instead of 1 draw call per texture as it might if we split it into planes.

这种使用一个纹理应用多个图片的方法称为纹理集。这是最好的方法，因为只需要加载一个纹理，着色器保持着简单，它只须引用了一个纹理，只调用一次绘制方法绘制图形，如果我们将它分解为多个平面就需要为每一个纹理调用一次绘制方法。

Next up [lets start simplifying with less code more fun][21].

下一步，[一起简化代码，获得更多乐趣][21]

-----------
## UVs vs. Texture Coordinates
## UVs vs. Texture Coordinates
Texture coordinates are often shortened to texture coords, texcoords or UVs (pronounced Ew-Vees). I have no idea where the term UVs came from except that vertex positions often use `x, y, z, w` so for texture coordinates they decided to use `s, t, u, v` to try to make it clear which of the 2 types you're refering to. Given that though you'd think they'd be called Es-Tees and in fact if you look at the texture wrap settings they are called `TEXTURE_WRAP_S` and `TEXTURE_WRAP_T` but for some reason as long as I've been working in graphics people have called them Ew-Vees. 
So now you know if someone says UVs they're talking about texture coordinates.

Texture coordinates 往往会缩写为texture coords，texcoords 或者 UVs (pronounced Ew-Vees)。我不知道 UVs 这一术语从何而来，但是顶点位置通常使用 x，y，z，w 表示，而纹理坐标使用s，t，u，v 表示，试图区分你是指哪个坐标。尽管你觉得他们应该被称为 Es-Tees，但实际上，当你查看纹理的 wrap 设置，会发现它们被称为 `TEXTURE_WRAP_S`和`TEXTURE_WRAP_T`，而由于某种原因，在制图学上人们称它们为 Ew-Vees。所以你现在清楚了，如果有人说 UVs，他们指的是纹理坐标。

[1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
[2]: http://webglfundamentals.org/webgl/lessons/webgl-animation.html
[3]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
[4]: http://webglfundamentals.org/webgl/resources/f-texture.png
[5]: http://webglfundamentals.org/webgl/webgl-3d-textures.html
[6]: http://webglfundamentals.org/webgl/lessons/resources/texture-coordinates-diagram.svg
[7]: http://webglfundamentals.org/webgl/lessons/resources/f-texture-coordinates-diagram.svg
[8]: http://webglfundamentals.org/webgl/webgl-3d-textures-texture-coords-mapped.html
[9]: http://webglfundamentals.org/webgl/webgl-3d-textures-repeat-clamp.html
[10]: http://webglfundamentals.org/webgl/lessons/resources/mip-low-res-enlarged.png
[11]: http://webglfundamentals.org/webgl/lessons/resources/mipmap-low-res-enlarged.png
[12]: http://webglfundamentals.org/webgl/webgl-3d-textures-mips.html
[13]: http://webglfundamentals.org/webgl/webgl-3d-textures-mips-tri-linear.html
[14]: http://webglfundamentals.org/webgl/lessons/resources/different-colored-mips.png
[15]: http://webglfundamentals.org/webgl/resources/keyboard.jpg
[16]: http://webglfundamentals.org/webgl/webgl-3d-textures-bad-npot.html
[17]: http://webglfundamentals.org/webgl/webgl-3d-textures-good-npot.html
[18]: http://webglfundamentals.org/webgl/lessons/resources/noodles-06.jpg
[19]: http://webglfundamentals.org/webgl/resources/noodles.jpg
[20]: http://webglfundamentals.org/webgl/webgl-3d-textures-texture-atlas.html
[21]: http://webglfundamentals.org/webgl/lessons/webgl-less-code-more-fun.html