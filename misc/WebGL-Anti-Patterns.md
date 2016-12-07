# WebGL Anti-Patterns
# WebGL 反模式

This is a list of anti patterns for WebGL. Anti patterns are things you should avoid doing

以下列表是 WebGL 反模式的例子。应该避免使用

1.	Putting `viewportWidth` and `viewportHeight` on the `WebGLRenderingContext`

    将`viewportWidth`和`viewportHeight`放到`WebGLRenderingContext`中

    Some code adds properties for their viewport width and height and sticks them on the `WebGLRenderingContext` something like this

    有些代码，会为视窗的宽高添加一些属性，并像以下那样放到`WebGLRenderingContext`中
    
    ```
    gl = canvas.getContext("webgl");
    gl.viewportWidth = canvas.width;    // BAD!!!
    gl.viewportHeight = canvas.height;  // BAD!!!
    ```
    
    Then later they might do something like this
    
    接着可能会这样
    
    ```
    gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
    ```
    
    **Why it's Bad:**
    
    **缺点**
    
    It's objectively bad because you now have 2 properties that need to be updated anytime you change the size of the canvas. For example if you change the size of the canvas when the user resizes the window `gl.viewportWidth` & `gl.viewportHeight` will be wrong unless you set them again.
    
    客观上，当改变 canvas 尺寸，你需要更新 2 个属性。例如，当用户调整窗口时，改变了 canvas 的尺寸，除非重新设置`gl.viewportWidth` 和`gl.viewportHeight`，不然它们会出错。
    
    It's subjectively bad because any new WebGL programmer will glance at your code and likely think `gl.viewportWidth` and `gl.viewportHeight` are part of the WebGL spec, confusing them for months.
    
    主观上，任何一个新的 WebGL 开发者看到你的代码，可能会认为`gl.viewportWidth`和`gl.viewportHeight`是 WebGL 规范的一部分，并在接下来的几个月中将它们混淆。
    
    **What to do instead:**
    
    **替代方法**
    
    Why make more work for yourself? The WebGL context has its width and height directly on it. Just use that.
    
    为什么要多此一举？WebGL 上下文中保存了其宽高。直接使用就好了。
    
    ```
    // When you need to set the viewport to match the size of the canvas's
    // drawingBuffer this will always be correct
    // 当你需要设置 viewport 使其与 canvas 的 drawingBuffer 相匹配时，以下做法总是正确的。
    
    gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
    ```
    Even better it will handle extreme cases whereas using `gl.canvas.width` and `gl.canvas.height` will not. [As for why see here][1].
    
    更好的是，它能够处理一些使用`gl.canvas.width`和`gl.canvas.height`不能处理的特殊情况。[至于原因请参考这里][1]
    
2.	Using `canvas.width` and `canvas.height` for aspect ratio

    使用`canvas.width`和`canvas.height`设置纵横比

    Often code uses `canvas.width` and `canvas.height` for aspect ratio like this
    
    通常会这样使用`canvas.width`和`canvas.height`来设置纵横比
    
    ```
    var aspect = canvas.width / canvas.height;
    perspective(fieldOfView, aspect, zNear, zFar);
    ```
    
    **Why it's Bad:**
    
    **缺点**
    
    The width and height of the canvas have nothing to do with the size the canvas is displayed. CSS controls the size the canvas is displayed.
    
    canvas 的宽高与 canvas 的大小无关。控制 canvas 大小的是 CSS。
    
    **What to do instead:**
    
    **替代方法**
    
    Use `canvas.clientWidth` and `canvas.clientHeight`. Those values tell you what size your canvas is actually being displayed on the screen. Using those values you'll always get the correct aspect ratio regardless of your CSS settings.
    
    使用`canvas.clientWidth`和`canvas.clientHeight`。它们保存着 canvas 在屏幕上的实际尺寸。不管 CSS 如何设置，使用它们，总能得到正确的纵横比。
    
    ```
    var aspect = canvas.clientWidth / canvas.clientHeight;
    perspective(projectionMatrix, fieldOfView, aspect, zNear, zFar);
    ```

    Here are examples of a canvas that's the same size (`width="400" height="300"`) but using CSS we've told the browser to display the canvas a different size. Notice the samples all display the 'F' in the correct aspect ratio.
    
    以下例子中，canvas 的尺寸都是相同（`width="400" height="300"`），通过 CSS 告诉浏览器显示不同尺寸的 canvas 。注意所有的例子的 “F” 的纵横比例都正确显示了。

    <iframe class="webgl_example " style="width: 150px; height: 200px;" src="/webgl/lessons/../webgl-canvas-clientwidth-clientheight.html"></iframe>

    <iframe class="webgl_example " style="width: 400px; height: 150px;" src="/webgl/lessons/../webgl-canvas-clientwidth-clientheight.html"></iframe>

    If we had used `canvas.width` and `canvas.height` that would not be true.
    
    如果我们使用`canvas.width`和`canvas.height`，情况就不一样了。

    <iframe class="webgl_example " style="width: 150px; height: 200px;" src="/webgl/lessons/../webgl-canvas-width-height.html"></iframe> 

    <iframe class="webgl_example " style="width: 400px; height: 150px;" src="/webgl/lessons/../webgl-canvas-width-height.html"></iframe>
    
3.	Using `window.innerWidth` and `window.innerHeight` to compute anything
    使用`window.innerWidth`和`window.innerHeight`计算
    
    Many WebGL programs use window.innerWidth and window.innerHeight in many places. For example:
    
    许多 WebGL 程序在许多地方使用了 window.innerWidth 和 window.innerHeight。例如：

    ```
    canvas.width = window.innerWidth;                    // BAD!!
    canvas.height = window.hinnerHeight;                 // BAD!!
    ```
    
    **Why it's Bad:**
    
    **缺点**
    
    It's not portable. Yes, it can work for WebGL pages where you want to make the canvas fill the screen. The problem comes when you don't. Maybe you decide to make an article like these tutorials where your canvas is just some small diagram in a larger page. Or maybe you need some property editor on the side or a score for a game. Sure you can fix your code to handle those cases but why not just write it so it works in those cases in the first place? Then you won't have to go change any code when you copy it to a new project or use an old project in a new way.
    
    这样做本身没有问题。是的，在 WebGL 页面中这样做能够让 canvas 填满屏幕。这之后不填充满屏幕时问题就来了。可能你决定写一篇像这些教程一样的文章，但在大大的页面中，你的 canvas 只有一些小小的图表。或者你可能需要在一边放置属性编辑器或显示游戏分数。当然，你可以调整你的代码来处理这些例子，但为什么不只在开始的地方编写呢？那么你将它们复制到一个新的项目或在旧的项目上使用新方法时，就不需要改变任何代码了。
    
    **What to do instead:**
    
    **替代方法**
    
    Instead of fighting the Web platform, use the Web platform as it was designed to be used. Use CSS and `clientWidth` and `clientHeight`.
    
    正如网络平台所设计那样使用它，而不是对抗它。使用 CSS 和`clientWidth`和`clientHeight`。
    
    ```
    var width = gl.canvas.clientWidth;
    var height = gl.canvas.clientHeight;

    gl.canvas.width = width;
    gl.canvas.height = height;
    ```
    
    Here are 9 cases. They all use exactly the same code. Notice that none of them reference `window.innerWidth` nor `window.innerHeight`.
    
    这里有 9 个例子。使用了完全一样的代码。注意它们都没有引用`window.innerWidth`或者`window.innerHeight`。
    
    [A page with nothing but a canvas using CSS to make it fullscreen][2]

    [A page with a canvas set to using 70% width so there is room for editor controls][3]
    
    [A page with a canvas embedded in a paragraph][4]

    [A page with a canvas embedded in a paragraph using `box-sizing: border-box`][5];
    
    `box-sizing: border-box;` makes borders and padding take space from the element they're defined on rather than outside it. In other words, in normal box-sizing mode a 400x300 pixel element with 15 pixel border has a 400x300 pixel content space surrounded by a 15 pixel border making its total size 430x330 pixels. In box-sizing: border-box mode the border goes on the inside so that same element would stay 400x300 pixels, the content would end up being 370x270. This is yet another reason why using `clientWidth` and `clientHeight` is so important. If you set the border to say 1em you'd have no way of knowing what size your canvas will turn out. It would be different with different fonts on different machines or different browsers.
    
    `box-sizing:border-box;`使 borders 和 padding 置于定义的区域内部而不是外部。换句话说，在普通的 box-sizing 模式中，定义一个 400x300 像素并带有 15 像素边框的元素，它有一个 400x300 像素的内容区域，这个区域被 15 像素的边框包围，使该元素总大小为 430x330 像素。在 box-sizing: border-box 模式中，边框位于内部，因此元素保持为 400x300 像素大小，内容区域最终大小为 370x270 。这也是为什么要使用`clientWidth`和`clientHeight`的另一个重要原因。如果你将边框设置为 1em ，你没有办法知道 canvas 最终的大小。它会因各种设备或浏览器中的字体不同而有所不同。
    
    [A page with nothing but a container using CSS to make it fullscreen into which the code will insert a canvas][6]
    
    [A page with a container set to using 70% width so there is room for editor controls into which the code will insert a canvas][7]

    [A page with a container embedded in a paragraph into which the code will insert a canvas][8]
    
    [A page with a container embedded in a paragraph using box-sizing: border-box; into which the code will insert a canvas][9]
    
    [A page with no elements with CSS setup to make it fullscreen into which the code will insert a canvas][10]
    
    Again, the point is, if you embrace the web and write your code using the techniques above you won't have to change any code when you run into different use cases.

    同样的，如果你拥抱网络，使用上述的技术，能够在不同用例中运行而不必修改。


4.	Using the `'resize'` event to change the size of your canvas.

    使用`“resize”`事件来改变 canvas 的大小。

    Some apps check for the window 'resize' event like this to `resize` their canvas.
    
    一些 app 通过监听 window 的 “resize” 事件来 “resize” canvas 的尺寸。
    
    ```
    window.addEventListener('resize', resizeTheCanvas);
    ```
    or this
    
    或者
    
    ```
    window.onresize = resizeTheCanvas;
    ```
    
    **Why it's Bad:**
    
    **缺点**
    
    It's not bad per se, rather, for most WebGL programs it fits less use cases. Specifically `'resize'` only works when the window is resized. It doesn't work if the canvas is resized for some other reason. For example let's say you're making a 3D editor. You have your canvas on the left and your settings on the right. You've made it so there's a draggable bar separating the 2 parts and you can drag that bar to make the settings area larger or smaller. In this case you won't get any `'resize'` events. Similarly if you've got a page where other content gets added or removed and the canvas changes size as the browser re-lays out the page you won't get a resize event.
    
    这样做本身并不是不好，然而，对于大多数 WebGL 程序并不适合。这样做`resize`只会在窗口重新调整大小时才会起作用。如果其他使 canvas 改变大小的事件，不会起效。例如，你正在编辑一个 3D 编辑器。canvas 位于左边，编辑器位于右边。编辑器中添加一个拖动条，你可以拖动它来调整大小。在这个例子中，不会有任何`“resize”`事件发生。同样，如果在一个页面中，添加或者删除一些东西，或者 canvas 改变了大小，使得浏览器回流，resize 事件也不会被触发。
    
    **What to do instead:**
    
    **替代方法**
    
    Like many of the solutions to anti-patterns above there's a way to write your code so it just works for most cases. For WebGL apps that constantly draw every frame the solution is to check if you need to resize every time you draw like this
    
    同上面的许多的反模式的解决方法一样，有一个方法适用于大多数情况。在 WebGL 应用中，连续地绘制每一帧并检查是否需要重新调整。
    
    ```
    function resize() {
        var width = gl.canvas.clientWidth;
        var height = gl.canvas.clientHeight;
        if (gl.canvas.width != width ||
            gl.canvas.height != height) {
            gl.canvas.width = width;
            gl.canvas.height = height;
        }
    }

    function render() {
        resize();
        drawStuff();
        requestAnimationFrame(render);
    }
    render();
    ```
    
    Now in any of those cases your canvas will scale to the right size. No need to change any code for different cases. For example using the same code from #3 above here's an editor with a sizable editing area.
    
    现在任何情况下，你的画布将缩放到正确的大小。在不同的用例中都不需要改变代码。例如使用上面第 3 个例子的代码，有一个相当大的编辑区域。
    
    <iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-same-code-resize.html"></iframe> 

    [click here to open in a separate window][11]
    
    [点击这里在新窗口打开][11]
 
    There would be no resize events for this case nor any other where the canvas gets resized based on the size of other dynamic elements on the page.
    
    这个例子中没有 resize 事件，canvas 也不会随着页面中的其它元素的变化动态调整大小。
    
    For WebGL apps that don't re-draw every frame the code above is still correct, you'll just need to trigger a re-draw in every case where the canvas can possibly get resized. One easy way to do that would be to setup a requestAnimationFrame loop like this.
    
    对于 WebGL 应用，没有重新绘制每一帧，以上的代码仍然是正确的，只需要在可能会重新调整 canvas 大小的情况下触发重绘事件。一个简单的方法是像这样设置一个 requestAnimationFrame 循环。
    
    ```
    function resize() {
        var width = gl.canvas.clientWidth;
        var height = gl.canvas.clientHeight;
        if (gl.canvas.width != width ||
            gl.canvas.height != height) {
            gl.canvas.width = width;
            gl.canvas.height = height;
            return true;
        }
        return false;
    }

    var needToRender = true;  // draw at least once
    function checkRender() {
        if (resize() || needToRender) {
            needToRender = false;
            drawStuff();
        }
        requestAnimationFrame(checkRender);
    }
    checkRender();
    ```
    
    This would only draw if the canvas has been resized or if `needToRender` is true. This would handle the resize case for apps that don't render the scene every frame. Just set `needToRender` any time you've changed something in the scene and you want the scene to be rendered incorporating your changes.
    
    这样只会在 canvas 需要重新调整或者`needToRender`为 true 时重绘。这样会为应用程序处理调整事件，而不会每一帧都重绘。任何时候，改变了场景中的东西，并且你希望渲染这些改变，只需要设置`needToRender`。
    
5.	Adding properties to `WebGLObjects`

    为`webGLObjects`添加属性

    `WebGLObjects` are the various types of resources in WebGL like a `WebGLBuffer` or `WebGLTexture`. Some apps add properties to those objects. For example code like this:
    
    `webGLObjects`指的是在 WebGL 中的各种类型，如`webGLBuffer`、`webGLTexture`。一些应用将属性添加到这些对象上。例如以下代码：
    
    ```
    var buffer = gl.createBuffer();
    buffer.itemSize = 3;        // BAD!!
    buffer.numComponents = 75;  // BAD!!

    var program = gl.createProgram();
    ...
    program.u_matrixLoc = gl.getUniformLocation(program, "u_matrix");  // BAD!!
    ```
    
    **Why it's Bad:**
    **缺点**
    
    The reason this is bad is that WebGL can "lose the context". This can happen for any reason but the most common reason is if the browser decides too many GPU resources are being used it might intentionally lose the context on some `WebGLRenderingContexts` to free up space. WebGL programs that want to always work have to handle this. Google Maps handles this for example.
    
    糟糕的原因是 WebGL 会“丢失上下文”。有很多原因会导致这个问题，但最常见的原因是，如果浏览器使用了过多的 GPU 资源，它可能会在一些`webGLRenderingContexts`上主动地丢失上下文，以释放空间。WebGL 程序必须处理这个问题才能一直工作下去。Google Maps 就是其中之一。
    
    The problem with the code above is that when the context is lost the WebGL creation functions like `gl.createBuffer()` above will return `null`. That effectively makes the code this
    
    以上代码的问题是，当上下文丢失了，webGL 创建如`gl.createBuffer()`方法时，将会返回 null。代码实际上是这样的
    
    ```
    var buffer = null;
    buffer.itemSize = 3;        // ERROR!
    buffer.numComponents = 75;  // ERROR!
    ```
    
    That will likely kill your app with an error like
    
    就等同于以下的错误终止了应用
    
    ```
    TypeError: Cannot set property 'itemSize' of null
    ```
    
    While many apps don't care if they die when the context is lost it seems like a bad idea to write code that will have to be fixed later if the developers ever decide to update their app to handle context lost events.
    
    很多应用程序中，在编写代码时不关心丢失上下文，程序是否会挂掉，这似乎是一个糟糕的主意，如果开发者决定更新应用程序，处理丢失上下文的事情，就不得不修改这些代码。
    
    **What to do instead:**
    
    **替代方法**
    
    If you want to keep `WebGLObjects` and some info about them together one way would be to use JavaScript objects. For example:
    
    如果你希望将`webGLObjects`和一些信息保存在一起，一种方法是使用 JavaScript 对象。例如：
    
    ```
    var bufferInfo = {
        id: gl.createBuffer(),
        itemSize: 3,
        numComponents: 75,
    };

    var programInfo = {
        id: program,
        u_matrixLoc: gl.getUniformLocation(program, "u_matrix"),
    };
    ```
    
    Personally I'd suggest [using a few simple helpers that make writing WebGL much simpler][12].
    
    就个人而言，我建议[利用一些辅助器使编写 WebGL 更加简单][12]
    
Those are a few of what I consider WebGL Anti-Patterns in code I've seen around the net. Hopefully I've made the case why to avoid them and given solutions that are easy and useful.

这些都是我在网上看到过的认为是 WebGL 反模式的代码。希望我列出的这些避免原因和解决方法都是简单并且有用的。

-----

## What is drawingBufferWidth and drawingBufferHeight?

## 什么是 drawingBufferWidth 和 drawingBufferHeight ？

GPUs have a limit on how big a rectangle of pixels (texture, renderbuffer) they can support. Often this size is the next power of 2 larger than whatever a common monitor resolution was at the time the GPU was made. For example if the GPU was designed to support 1280x1024 screens it might have a size limit of 2048. If it was designed for 2560x1600 screens it might have a limit of 4096. 

GPUs 对于它们支持的矩形有限制像素（纹理、渲染）的大小。这个大小通常为大于 GPU 使用的显示器分辨率的最小的 2 的幂。例如，如果 GPU 设计为支持 1280x1024 的屏幕，它可能有一个 2048 大小的限制。如果它设计为支持 2560x1600 屏幕，它可能有一个 4096 大小的限制。

That seems reasonable but what happens if you have multiple monitors? Let's say I have a GPU with a limit of 2048 but I have two 1920x1080 monitors. The user opens a browser window with a WebGL page, they then stretch that window across both monitors. Your code tries to set the `canvas.width` to `canvas.clientWidth` which in this case is 3840. What should happen? 

这似乎是合理的，但如果有多个显示器会发生什么呢？如果 GPU 极限为 2048，我有两个 1920x1080 的显示器。用户在浏览器窗口中打开一个 WebGL 页面，将窗口拉伸横跨到两个显示屏。代码尝试将`canvas.width`设置为`canvas.clientWidth`，也就是 3840 像素，会发生什么？

Off the top of my head there are only 3 options

我脑海中有 3 个选项

1.	Throw an exception.


1. 抛出一个异常

    That seems bad. Most web apps won't be checking for it and the app wil crash. If the app had user data in it the user just lost their data
    
    这似乎很糟糕。大多数的 web 应用程序不会测试这种情况，这种情况会导致程序崩溃。如果期间程序保存了用户数据，数据将丢失。
    
2.	Limit the size of the canvas to the GPUs limit


2. canvas 被限制为 GPUs 的极限

    The problem with this solution is it will also likely lead to a crash or possibly a messed up webpage because the code expects the canvas to be the size they requested and they expect other parts of the UI and elements on the page to be in the proper places.
    
    这个解决方法的问题是，它也可能导致程序崩溃或者网页错乱，代码希望 canvas 大小为它们所要求的，同时也希望页面中其他的 UI 和 元素在适当的地方。
    
3.	Let the canvas be the size the user requested but make its drawingbuffer the limit


3. canvas 为用户所要求的大小，但限制 drawingbuffer 的大小

    This is the solution WebGL uses. If your code is written correctly the only thing the user might notice is the image in the canvas is being scaled slightly. Otherwise it just works. In the worst case most WebGL programs that don't do the right thing will just have a slightly off display but if the user sizes the window back down things will return to normal.
    
    这是 WebGL 使用的解决方法。如果编写的代码正确，那么用户唯一注意到的，是 canvas 中的图片略微放大了。程序照常运行。最坏的情况下，大多数 WebGL 程序只是不显示画面，但如果用户将窗口调整回正常大小，它会恢复正常。
    
Most people don't have multiple monitors so this issue rarely comes up. Or at least it used to. Chrome and Safari, at least as of January 2015, had a hardcoded limit on canvas size of 4096. Apple's 5k iMac is past that limit. Lots of WebGL apps were having strange displays because of this. Similarly many people have started using WebGL with multiple monitors for installation work and have been hitting this limit.

大多数人都不会使用多个显示屏，所以这个问题很少出现。至少曾经是这样。Chrome 和Safari，最晚在 2015 年 1 月起，对 canvas 的大小强制限制在 4096 以内。Apple的 5k iMac 接纳了这个限制。许多 WebGL 应用程序显示怪异都是因为这个原因。同样，许多人已经开始在工作中将 WebGL 运行在多个显示器中，并且达到了极限。

So, if you want to handle these cases use `gl.drawingBufferWidth` and `gl.drawingBufferHeight` as shown in #1 above. For most apps if you follow the best practices above things will just work. Be aware though if you are doing calculations that need to know the actual size of the drawingbuffer you need to take that into account. Examples off the top of my head, picking, in other words converting from mouse coordinates into canvas pixel coordinates. Another would be any kind of post processing effects that want to know the actual size of the drawingbuffer. 

因此，如果你希望解决这些问题，就像第一个例子那样使用`gl.drawingBufferWidth`和`gl.drawingBufferHeight`。如果你遵循以上的最好的做法，对于大多数应用程序都是起效的。要注意的是，如果你计算的时候需要考虑到 drawingbuffer 的实际尺寸。选择我脑海中的例子，换言之，将鼠标坐标转换为像素坐标。或者是知道 drawingbuffer 处理后的任何类型的大小。

[1]: http://webglfundamentals.org/webgl/lessons/webgl-anti-patterns.html#drawingbuffer
[2]: http://webglfundamentals.org/webgl/webgl-same-code-canvas-fullscreen.html
[3]: http://webglfundamentals.org/webgl/webgl-same-code-canvas-partscreen.html
[4]: http://webglfundamentals.org/webgl/webgl-same-code-canvas-embedded.html
[5]: http://webglfundamentals.org/webgl/webgl-same-code-canvas-embedded-border-box.html
[6]: http://webglfundamentals.org/webgl/webgl-same-code-container-fullscreen.html
[7]: http://webglfundamentals.org/webgl/webgl-same-code-container-partscreen.html
[8]: http://webglfundamentals.org/webgl/webgl-same-code-container-embedded.html
[9]: http://webglfundamentals.org/webgl/webgl-same-code-container-embedded-border-box.html
[10]: http://webglfundamentals.org/webgl/webgl-same-code-body-only-fullscreen.html
[11]: http://webglfundamentals.org/webgl/webgl-same-code-resize.html
[12]: http://webglfundamentals.org/webgl/lessons/webgl-less-code-more-fun.html

