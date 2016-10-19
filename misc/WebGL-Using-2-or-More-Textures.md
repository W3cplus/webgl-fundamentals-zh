## WebGL Using 2 or More Textures

## WebGL 使用两个以上纹理

This article is a continuation of [WebGL Image Processing][1]. If you haven't read that I suggest [you start there][2].

这篇文章是[WebGL 图像处理][1]的延续。如果你没有阅读过，我建议[你先从这里开始][2]。

Now might be a good time to answer the question, "How do I use 2 or more textures?"

现在可能是回答“我如何使用 2 个以上的纹理”的好时机了。

It's pretty simple. Let's [go back a few lessons to our first shader that draws a single image][3] and update it for 2 images.

这很简单。[回到前几节我们利用着色器绘制一张图片的示例][3]，更新为 2 个图片。

The first thing we need to do is change our code so we can load 2 images. This is not really a WebGL thing, it's a HTML5 JavaScript thing, but we might as well tackle it. Images are loaded asynchronously which can take a little getting used to.

首先我们需要改变代码，加载 2 张图片。这真的不是 WebGL 要做的事情，这是 HTML5 和 JavaScript 做的事情。可以利用图片异步加载的特性很好的解决这个问题。

There are basically 2 ways we could handle it. We could try to structure our code so that it runs with no textures and as the textures are loaded the program updates. We'll save that method for a later article.

基本上有 2 种方法可以处理。我们可以尝试构建没有纹理的代码，作为加载纹理的更新程序。保存起来下文使用。

In this case we'll wait for all the images to load before we draw anything.

这个示例中，等待所有你图片加载完再开始绘制。

First let's change the code that loads an image into a function. It's pretty straightforward. It creates a new `Image` object, sets the URL to load, and sets a callback to be called when the image finishes loading.

首先，将加载图片的代码放进一个函数中。这非常的简单。函数里创建了一个`Image`对象，设置加载的 URL，最后设置图片加载完成后调用的回调函数。

```
	function loadImage(url, callback) {
	  var image = new Image();
	  image.src = url;
	  image.onload = callback;
	  return image;
	}
```

Now let's make a function that loads an array of URLs and generates an array of images. First we set `imagesToLoad` to the number of images we're going to load. Then we make the callback we pass to `loadImage` decrement `imagesToLoad`. When `imagesToLoad` goes to 0 all the images have been loaded and we pass the array of images to a callback.

现在创建一个函数，用来加载一组 URL ，并生成一组图片。首先我们设置`imagesToLoad`变量表示需要加载的图片数量。然后创建传递给`loadImage`的递减`imagesToLoad`的回调函数。当`imagesToLoad`等于 0 时，代表所有图片已经加载完，将图片数组传递给 callback。

```
	function loadImages(urls, callback) {
	  var images = [];
	  var imagesToLoad = urls.length;
	 
	  // Called each time an image finished loading.
	  // 每加载完一张图片调用一次。
	  var onImageLoad = function() {
	    --imagesToLoad;
	    // If all the images are loaded call the callback.
	    // 如果所有图片都加载完就调用 callback。
	    if (imagesToLoad == 0) {
	      callback(images);
	    }
	  };
	 
	  for (var ii = 0; ii < imagesToLoad; ++ii) {
	    var image = loadImage(urls[ii], onImageLoad);
	    images.push(image);
	  }
	}
```

Now we call loadImages like this

我们这样调用 loadImages

```
	function main() {
	  loadImages([
	    "resources/leaves.jpg",
	    "resources/star.jpg",
	  ], render);
	}
```

Next we change the shader to use 2 textures. In this case we'll multiply 1 texture by the other.

接下来让着色器使用 2 个纹理。这里我们将两个纹理相乘。

```
	<script id="2d-fragment-shader" type="x-shader/x-fragment">
	precision mediump float;
	 
	// our textures
	// 两个的纹理
	uniform sampler2D u_image0;
	uniform sampler2D u_image1;
	 
	// the texCoords passed in from the vertex shader.
	// 从顶点着色器传入的纹理坐标。
	varying vec2 v_texCoord;
	 
	void main() {
	   vec4 color0 = texture2D(u_image0, v_texCoord);
	   vec4 color1 = texture2D(u_image1, v_texCoord);
	   gl_FragColor = color0 * color1;
	}
	</script>
```

We need to create 2 WebGL texture objects.

我们需要创建 2 个WebGL 纹理对象。

```
	  // create 2 textures
	  // 创建 2 个纹理
	  var textures = [];
	  for (var ii = 0; ii < 2; ++ii) {
	    var texture = gl.createTexture();
	    gl.bindTexture(gl.TEXTURE_2D, texture);
	 
	    // Set the parameters so we can render any size image.
	    // 设置参数，这样就可以使用任何尺寸的图片了。
	    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
	    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
	    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
	    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
	 
	    // Upload the image into the texture.
	    // 更新图片到纹理中。
	    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, images[ii]);
	 
	    // add the texture to the array of textures.
	    // 添加纹理到纹理数组中。
	    textures.push(texture);
	  }
```

WebGL has something called "texture units". You can think of it as an array of references to textures. You tell the shader which texture unit to use for each sampler.

WebGL中被称为“纹理单元”（texture units）的东西。你可以认为它是引用了纹理的数组。让着色器知道每个示例使用了哪个纹理坐标。

```
	  // lookup the sampler locations.
	  // 查看示例的位置。
	  var u_image0Location = gl.getUniformLocation(program, "u_image0");
	  var u_image1Location = gl.getUniformLocation(program, "u_image1");
	 
	  ...
	 
	  // set which texture units to render with.
	  // 设置渲染哪个纹理单元。
	  gl.uniform1i(u_image0Location, 0);  // texture unit 0
	  gl.uniform1i(u_image1Location, 1);  // texture unit 1
```

Then we have to bind a texture to each of those texture units.

然后我们要为每一个纹理单元绑定纹理。

```
	  // Set each texture unit to use a particular texture.
	  // 设置每个纹理单元使用的纹理．
	  gl.activeTexture(gl.TEXTURE0);
	  gl.bindTexture(gl.TEXTURE_2D, textures[0]);
	  gl.activeTexture(gl.TEXTURE1);
	  gl.bindTexture(gl.TEXTURE_2D, textures[1]);
```

The 2 images we're loading look like this

加载以下的 2 张图片

![i1][4]

![i2][5]

And here's the result if we multiply them together using WebGL.

这是我们在 WebGL 中一起使用它们的效果。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-2-textures.html"></iframe>

[click here to open in a separate window][6] 

[点击这里在新窗口打开][6]

Some things I should go over.

我应该回顾一些事情。

The simple way to think of texture units is something like this: All of the texture functions work on the "active texture unit". The "active texture unit" is just a global variable that's the index of the texture unit you want to work with. Each texture unit has 2 targets. The TEXTURE_2D target and the TEXTURE_CUBE_MAP target. Every texture function works with the specified target on the current active texture unit. If you were to implement WebGL in JavaScript it would look something like this

可以这样简单理解纹理单元：所有的纹理函数都是在“活动纹理单元”中工作的。“活动纹理单元”仅仅是一个全局变量，是你想要使用的纹理单元的索引。每一个纹理单元都有 2 个目标。TEXTURE_2D 目标和 TEXTURE_CUBE_MAP 目标。在当前活动纹理单元上的每个纹理函数都与指定的目标工作。如果你用 JavaScript 实现 WebGL，代码可能类似这样

```
	var getContext = function() {
	  var textureUnits = [];
	  var activeTextureUnit = 0;
	 
	  var activeTexture = function(unit) {
	    // convert the unit enum to an index.
	    // 传递 unit enum 给 index。
	    var index = unit - gl.TEXTURE0;
	    // Set the active texture unit
	    // 设置活动纹理单元
	    activeTextureUnit = index;
	  };
	 
	  var bindTexture = function(target, texture) {
	    // Set the texture for the target of the active texture unit.
	    // 为活动纹理单元的目标设置纹理
	    textureUnits[activeTextureUnit][target] = texture;
	  };
	 
	  var texImage2D = function(target, ... args ...) {
	    // Call texImage2D on the current texture on the active texture unit
	    // 在活动纹理单元中调用当前纹理的 texImage2D 方法
	    var texture = textureUnits[activeTextureUnit][target];
	    texture.image2D(...args...);
	  };
	 
	  // return the WebGL API
	  // 返回 WebGL API
	  return {
	    activeTexture: activeTexture,
	    bindTexture: bindTexture,
	    texImage2D: texImage2D,
	  }
	};
```

The shaders take indices into the texture units. Hopefully that makes these 2 lines clearer.

着色器将索引存入纹理单元。希望这使这 2 行会更明了

```
	  gl.uniform1i(u_image0Location, 0);  // texture unit 0
	  gl.uniform1i(u_image1Location, 1);  // texture unit 1
```

One thing to be aware of, when setting the uniforms you use indices for the texture units but when calling gl.activeTexture you have to pass in special constants gl.TEXTURE0, gl.TEXTURE1 etc. Fortunately the constants are consecutive so instead of this

需要注意的一件事是，用 uniform 变量设置纹理单元的索引，调用 gl.activeTexture 时你必须传入 gl.TEXTURE0，gl.TEXTURE1 等等这样的特殊常量。幸运的是，这个常量是连续的，所以替代这段代码

```
	  gl.activeTexture(gl.TEXTURE0);
	  gl.bindTexture(gl.TEXTURE_2D, textures[0]);
	  gl.activeTexture(gl.TEXTURE1);
	  gl.bindTexture(gl.TEXTURE_2D, textures[1]);
```

We could have done this

我们可以这样做

```
	  for (var ii = 0; ii < 2; ++ii) {
	    gl.activeTexture(gl.TEXTURE0 + ii);
	    gl.bindTexture(gl.TEXTURE_2D, textures[ii]);
	  }
```

Hopefully this small step helps explain how to use mutliple textures in a single draw call in WebGL.

希望这篇短文能够解释清楚 WebGL 在一次绘制中如何使用多个纹理。

[1]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
[2]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
[3]: http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html
[4]: http://webglfundamentals.org/webgl/resources/leaves.jpg
[5]: http://webglfundamentals.org/webgl/resources/star.jpg
[6]: http://webglfundamentals.org/webgl/webgl-2-textures.html