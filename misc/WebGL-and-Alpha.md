## WebGL and Alpha

## WebGL 和  Alpha

I've noticed some OpenGL developers having issues with how WebGL treats alpha in the backbuffer (ie, the canvas), so I thought it might be good to go over some of the differences between WebGL and OpenGL related to alpha.

我留意到一些 OpenGL 开发者对于 WebGL 处理双缓存中的 alpha 通道有所疑问，所以我认为重温一下 alpha 通道在 WebGL 和OpenGL 之间的差异何有必要。

The biggest difference between OpenGL and WebGL is that OpenGL renders to a backbuffer that is not composited with anything, or effectively not composited with anything by the OS's window manager, so it doesn't matter what your alpha is.

OpenGL 和 WebGL 最大的差异是，OpenGL 渲染一个双缓存不需要复合任何东西，或有效地不与 OS 的窗口管理器的任何东西复合，所以 alpha  通道是多少不重要。

WebGL is composited by the browser with the web page and the default is to use pre-multiplied alpha the same as .png `<img>` tags with transparency and 2D canvas tags.

而 WebGL 是需要与浏览器页面结合的，默认会使用 pre-multiplied alpha ，就像`<img>`标签中.png格式和 2D 画布标签的 alpha 通道一样。

WebGL has several ways to make this more like OpenGL.

WebGL 有多种方式使 alpha 通道的效果更接近 OpenGL。

**#1) Tell WebGL you want it composited with non-premultiplied alpha**

**#1）通知 WebGL 不使用 pre-multiplied alpha**

```
	gl = canvas.getContext("webgl", {
	  premultipliedAlpha: false  // Ask for non-premultiplied alpha   // 请求不使用 pre-multiplied alpha
	});
```

The default is true.

默认值是 true。

Of course the result will still be composited over the page with whatever background color ends up being under the canvas (the canvas's background color, the canvas's container background color, the page's background color, the stuff behind the canvas if the canvas has a z-index > 0, etc....) in other words, the color CSS defines for that area of the webpage.

当然尽管背景色在画布之下，结果还是与页面结合了（包括画布背景色，画布容器背景色，页面背景色，如果画布的 z-index 属性 > 0，还包括画布背后的东西的背景色，等等。。。。。。），这个区域用 CSS 定义的颜色。

A really good way to find if you have any alpha problems is to set the canvas's background to a bright color like red. You'll immediately see what is happening.

如果你有任何关于 alpha 通道的问题，有一个很好的办法，就是将画布的背景色设置为像红色一样的明亮的颜色。

```
<canvas style="background: red;"><canvas>
```

You could also set it to black which will hide any alpha issues you have.

你也可以将它设置为黑色，这可以将 alpha 通道的问题隐藏起来。

**#2) Tell WebGL you don't want alpha in the backbuffer**

**#2）通知 WebGL 双缓存不需要 alpha **

```
gl = canvas.getContext("webgl", { alpha: false }};
```

This will make it act more like OpenGL since the backbuffer will only have RGB. This is probably the best option because a good browser could see that you have no alpha and actually optimize the way WebGL is composited. Of course that also means it actually won't have alpha in the backbuffer so if you are using alpha in the backbuffer for some purpose that might not work for you. Few apps that I know of use alpha in the backbuffer. I think arguably this should have been the default.

双缓存只有 RGB 值，这让 WebGL 的行为更加接近 OpenGL。这可能是最好的选择，因为好的浏览器能够明白你没有设置 alpha 通道，并优化 WebGL 的复合。当然这也意味着在双缓存中是没有 alpha 通道的。如果你出于某些原因，在双缓存中使用了 alpha　通道，可能不会起作用。我知道几个在双缓存使用了透明度的应用程序。

**#3) Clear alpha at the end of your rendering**

**#3）在渲染的结束时清除 alpha**

```
	..
	renderScene();
	..
	// Set the backbuffer's alpha to 1.0
	// 双缓存中的 alpha 通道设置为 1.0
	gl.clearColor(1, 1, 1, 1);
	gl.colorMask(false, false, false, true);
	gl.clear(gl.COLOR_BUFFER_BIT);
```

Clearing is generally very fast as there is a special case for it in most hardware. I did this in most of my demos. If I was smart I'd switch to method #2 above. Maybe I'll do that right after I post this. It seems like most WebGL libraries should default to this method. Those few developers that are actually using alpha for compositing effects can ask for it. The rest will just get the best perf and the least surprises.

清除一般情况下都是很快的，因为大多数的硬盘中都会为它留了一个位置。我的大多数 demo 中都是用这一方法的。如果我足够聪明，我很使用上面第二种的方法。也许我贴了这篇文章之后我会使用那样做。一些开发者可以使用这一方法有效地复合 alpha 通道。剩下的就是获得最佳的性能和小小的惊喜。

**#4) Clear the alpha once then don't render to it anymore**

**#4）不渲染时立即清除 alpha 通道**

```
	// At init time. Clear the back buffer.
	// 在初始时，清除缓存区。
	gl.clearColor(1,1,1,1);
	gl.clear(gl.COLOR_BUFFER_BIT);
	 
	// Turn off rendering to alpha
	// 关闭 alpha 通道的渲染
	gl.colorMask(true, true, true, false);
```

Of course if you are rendering to your own framebuffers you may need to turn rendering to alpha back on and then turn it off again when you switch to rendering to the canvas.

当然如果你正在渲染帧缓存，你可能需要重新打开 alpha 通道的渲染，渲染画布时再关闭。

**#5) Handling Images**

**#5）处理图像**

My default if you are loading images with alpha into WebGL. WebGL will provide the values as they are in the PNG file with color values not premultiplied. This is generally what I'm used to for OpenGL programs because it's lossless whereas pre-multiplied is lossy.

我默认你是将 alpha 通道加载进了 WebGL。WebGL 会提供它们在 PNG 文件中没有预乘的颜色值。通常在 OpenGL 中我会使用这一方法，因为这是无损的而 pre-multiplied 是有损的。

```
1, 0.5, 0.5, 0  // RGBA  
```

Is a possible value un-premultiplied whereas pre-multiplied it's an impossible value because `a = 0` which means `r`,`g`, and `b` have to be zero.

这是一个 un-premultiplied 值而不可能 pre-multiplied ，因为 `a = 0`意味着`r`，`g`和`b`都必须为 0。

You can have WebGL pre-multiply the alpha if you want. You do this by setting `UNPACK_PREMULTIPLY_ALPHA_WEBGL` to true like this

你可以预乘 alpha　通道。像这样设置`UNPACK_PREMULTIPLY_ALPHA_WEBGL`为 true

```
gl.pixelStorei(gl.UNPACK_PREMULTIPLY_ALPHA_WEBGL, true);
```

The default is un-premultiplied.

Be aware that most if not all Canvas 2D implementations work with pre-multiplied alpha. That means when you transfer them to WebGL and `UNPACK_PREMULTIPLY_ALPHA_WEBGL` is false WebGL will convert them back to un-premultipiled.

默认是 un-premultiplied 的。

要知道不是所有的 Canvas 2D 都实现了pre-multiplied alpha 通道的。这意味着当你将它们传递到 WebGL，并且`UNPACK_PREMULTIPLY_ALPHA_WEBGL` 的值为 false 时，WebGL 会将它们转换回 un-premultipiled。

**#6) Using a blending equation that works with pre-multiplied alpha.**

**#6）使用 pre-multiplied alpha 混合方法**

Almost all OpenGL apps I've writing or worked on use

几乎所有的 OpenGL 应用程序我都使用这一方法。

```
gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
```

That works for non pre-multiplied alpha textures.

这用于非 pre-multiplied alpha 纹理。

If you actually want to work with pre-multiplied alpha textures then you probably want

如果你想使用 pre-multiplied alpha 的纹理，你可能需要

```
gl.blendFunc(gl.ONE, gl.ONE_MINUS_SRC_ALPHA);
```

Those are the methods I'm aware of. If you know of more please post them below.

这些都是我知道的方法。如果你了解更多的方法欢迎补?     