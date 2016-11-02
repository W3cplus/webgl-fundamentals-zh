## WebGL - Animation
## WebGL - 动画

This post is a continuation of a series of posts about WebGL. The first [started with fundamentals][1]. and the previous was about [3D cameras][2]. If you haven't read those please view them first.

这篇文章是 WebGL 系列文章的续篇。如果你没有阅读过第一篇[从基础开始][1]和先前的[3D 相机][2]，请先阅读它们。

How do we animate something in WebGL?

我们如何在 WebGL 中使用动画？

Actually this isn't specific to WebGL but generally if you want to animate something in JavaScript you need to change something over time and draw again.

其实在 WebGL 中没有什么特别的，如果你希望通过 JavaScript 使用动画，你需要不断的改变和重绘。

We can take one of our previous samples and animate it as follows.

我们可以采用以前的实例，使它如下运动。
```
	var fieldOfViewRadians = degToRad(60);
	var rotationSpeed = 1.2;
	 
	requestAnimationFrame(drawScene);
	 
	// Draw the scene.
	// 绘制场景。
	function drawScene() {
	  // Every frame increase the rotation a little.
	  // 每一帧旋转一点。
	  rotation[1] += rotationSpeed / 60.0;
	 
	  ...
	  // Call drawScene again next frame
	  // 下一帧再次调用 drawScene
	  requestAnimationFrame(drawScene);
	}
```
And here it is

效果如下

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-animation-not-frame-rate-independent.html"></iframe>

[click here to open in a separate window][3] 

[点击这里在新窗口打开][3]

There's a subtle problem though. The code above has a `rotationSpeed / 60.0`. We divided by 60.0 because we assumed the browser will respond to requestAnimationFrame 60 times a second which is pretty common.

但有一个小问题。上面的代码中有一个 `rotationSpeed / 60.0`。我们设置为 60.0 因为我们通常认为浏览器在一秒钟响应 requestAnimationFrame 60 次。

That's not actually a valid assumption though. Maybe the user is on a low-powered device like an old smartphone. Or maybe the user is running some heavy program in the background. There are all kinds of reasons the browser might not be displaying frames at 60 frames a second. Maybe it's the year 2020 and all machines run at 240 frames a second now. Maybe the user is a gamer and has a CRT monitor running at 90 frame a second.

实际上这不是一个有效的假设。用户可能使用的是旧的智能手机这样的低功率设备。或者用户可能在后台运行一些沉重的程序。有各种各样的原因，浏览器不能每秒 60 帧的显示。可能在 2020 年所有的都能每秒运行 240 帧。也可能用户是游戏玩家并且有一个每秒运行 90 帧的CRT显示器。

You can see the problem in this example

在这个例子中，你可以看到这个问题

<iframe class="webgl_example " style="width: 400px; height: 300px;" src="/webgl/lessons/../webgl-animation-frame-rate-issues.html"></iframe>

In the example above we want to rotate all of the 'F's at the same speed. The 'F' in the middle is running full speed and is frame rate independent. The one on the left and the right are simulating if the browser was only running at 1/8th max speed for the current machine. The one on the left is NOT frame rate independent. The one on the right IS frame rate independent.

上面的示例我们想所有的“F”能以同样的速度旋转。中间的“F”是以独立的帧速率全速运行的。左右两边的是模拟浏览器以当前机器最大速度的 1/8 运行的情况。左边的不是独立的帧速率。而右边的是独立的帧速率

Notice because the one on the left is not taking into account that the frame rate might be slow it's not keeping up. The one on the right though, even though it's running at 1/8 the frame rate it is keeping up with the middle one running at full speed.

请注意，左边的并没有考虑到帧速率可能是缓慢的且不稳定的。右边的虽然是以 1/8 的帧速率运行，却和中间的一样保持着全速运行。

The way to make animation frame rate independent is to compute how much time it took between frames and use that to calcuate how much to animate this frame.

使动画的帧数独立的方法是，计算两帧之间的时间差，用这个来计算每秒的帧动画次数。

First off we need to get the time. Fortunately `requestAnimationFrame` passes us the time since the page was loaded when it calls us.

首先我们需要得到帧动画次数。幸运的是，页面加载之后当我们可以调用`requestAnimationFrame`。

I find it easiest if we get the time in seconds but since the `requestAnimationFrame` passes us the time in milliseconds (1000ths of a second) we need to multiply by 0.001 to get seconds.

我发现我们很容易获得一秒内的帧动画次数，但`requestAnimationFrame`是以毫秒（一秒的千分之一）计算的，我们需要乘以 0.001 取得秒数。

So, we can then compute the delta time like this

因此，我们可以这样计算时差

```
	var then = 0;
	 
	requestAnimationFrame(drawScene);
	 
	// Draw the scene.
	// 绘制场景。
	function drawScene(now) {
	  // Convert the time to seconds
	  // 时间转换为秒
	  now *= 0.001;
	  // Subtract the previous time from the current time
	  // 当前时间减去之前时间
	  var deltaTime = now - then;
	  // Remember the current time for the next frame.
	  // 为下一帧记录当前时间
	  then = now;
	 
	   ...
```

Once we have the `deltaTime` in seconds then all our calculations can be in how many units per second we want something to happen. In this case `rotationSpeed` is 1.2 which means we want to rotate 1.2 radians per second. That's about 1/5 of a turn or in other words it will take about 5 seconds to turn around completely regardless of the frame rate.

一旦我们得到`时间差`，我们可以计算我们想要实现的事情在每秒中执行的次数。在这个示例中`rotationSpeed`为 1.2，意味着我们希望每秒旋转 1.2 圈。大约 1/5 圈，换句话说，无论帧数是多少都需要大约 5 秒的时间旋转完整。

```
  rotation[1] += rotationSpeed * deltaTime;
```

Here's that one working.

它运行了。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-animation.html"></iframe>



[click here to open in a separate window][4] 

[点击这里在新窗口打开][4]

You aren't likely to see a difference from the one at the top of this page unless you are on a slow machine but if you don't make your animations frame rate independent you'll likely have some users that are getting a very different experience than you planned.

除非你使用低速率设备，不然你可能看不出它与页面顶部示例的区别。如果你没有使动画帧独立，你的一些用户可能有与你计划的很不一样的体验。

Next up [how to apply textures][5].

下一步[如何运用纹理][5]

——————————

## Don't use setInterval or setTimeout!

## 不要使用 setInterval 或者 setTimeout！

If you've been programming animation in JavaScript in the past you might have used either `setInterval` or `setTimeout` to get your drawing function to be called. 

如果你曾经使用 JavaScript 编程动画，你可能是使用`setInterval`或者`setTimeout`来调用你的绘制函数。

The problems with using `setInterval` or `setTimeout` to do animation are two fold. First off both `setInterval` and `setTimeout` have no relation to the browser displaying anything. They aren't synced to when the browser is going to draw a new frame and so can be out of sync with the user's machine. If you use `setInterval` or `setTimeout` and assume 60 frames a second and the user's machine is actually running some other frame rate you'll be out of sync with their machine. 

使用`setInterval`或者`setTimeout`来实现动画会有两个问题。首先`setInterval`和`setTimeout`显示任何东西都与浏览器没有关联。当浏览器绘制新的帧时它们并不同步，因此与用户的设备也不同步。如果使用`setTinterval`或者`setTimeout`并且设定每秒 60 帧，用户的设备实际上还会运行其他的帧，你会与他们的设备不同步。

The other problem is the browser has no idea why you're using `setInterval` or `setTimeout`. So for example, even when your page is not visible, like when it's not the front tab, the browser still has to execute your code. Maybe you're using `setTimeout` or `setInterval` to check for new mail or tweets. There's no way for the browser to know. That's fine if you're just checking every few seconds for new messages but it's not fine if you're trying to draw 1000 objects in WebGL. You'll be effectively DOSing the user's machine with your invisible tab drawing stuff they can't even see. 

另一个问题是浏览器不知道你为什么会使用`setInterval`或者`setTimeout`。例如，即使你的页面不可见，比如它不是当前的标签，浏览器仍然必须执行你的代码。也许你正在使用` setTimeout`或`setInterval`查看新邮件或者 tweets。浏览器并不知道。如果你只是检查每几秒的新消息这没问题，但如果你正在尝试用 WebGL 绘制 1000 个对象就不好了。你那看不见的正在绘制东西的标签实际上是在攻击用户的设备，即使他们看不见。

`requestAnimationFrame` solves both of these issues. It calls you at just the right time to sync your animation with the screen and it also only calls you if your tab is visible. 

`requestAnimationFrame`解决了这两个问题。调用它时，动画会与你的显示屏同步，并且只会在标签可见的情况下调用。

[1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
[2]: http://webglfundamentals.org/webgl/lessons/webgl-3d-camera.html
[3]: http://webglfundamentals.org/webgl/webgl-animation-not-frame-rate-independent.html
[4]: http://webglfundamentals.org/webgl/webgl-animation.html
[5]: http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html