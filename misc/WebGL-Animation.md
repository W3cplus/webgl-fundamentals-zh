## WebGL - Animation
## WebGL - ����

This post is a continuation of a series of posts about WebGL. The first [started with fundamentals][1]. and the previous was about [3D cameras][2]. If you haven't read those please view them first.

��ƪ������ WebGL ϵ�����µ���ƪ�������û���Ķ�����һƪ[�ӻ�����ʼ][1]����ǰ��[3D ���][2]�������Ķ����ǡ�

How do we animate something in WebGL?

��������� WebGL ��ʹ�ö�����

Actually this isn't specific to WebGL but generally if you want to animate something in JavaScript you need to change something over time and draw again.

��ʵ�� WebGL ��û��ʲô�ر�ģ������ϣ��ͨ�� JavaScript ʹ�ö���������Ҫ���ϵĸı���ػ档

We can take one of our previous samples and animate it as follows.

���ǿ��Բ�����ǰ��ʵ����ʹ�������˶���
```
	var fieldOfViewRadians = degToRad(60);
	var rotationSpeed = 1.2;
	 
	requestAnimationFrame(drawScene);
	 
	// Draw the scene.
	// ���Ƴ�����
	function drawScene() {
	  // Every frame increase the rotation a little.
	  // ÿһ֡��תһ�㡣
	  rotation[1] += rotationSpeed / 60.0;
	 
	  ...
	  // Call drawScene again next frame
	  // ��һ֡�ٴε��� drawScene
	  requestAnimationFrame(drawScene);
	}
```
And here it is

Ч������

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-animation-not-frame-rate-independent.html"></iframe>

[click here to open in a separate window][3] 

[����������´��ڴ�][3]

There's a subtle problem though. The code above has a `rotationSpeed / 60.0`. We divided by 60.0 because we assumed the browser will respond to requestAnimationFrame 60 times a second which is pretty common.

����һ��С���⡣����Ĵ�������һ�� `rotationSpeed / 60.0`����������Ϊ 60.0 ��Ϊ����ͨ����Ϊ�������һ������Ӧ requestAnimationFrame 60 �Ρ�

That's not actually a valid assumption though. Maybe the user is on a low-powered device like an old smartphone. Or maybe the user is running some heavy program in the background. There are all kinds of reasons the browser might not be displaying frames at 60 frames a second. Maybe it's the year 2020 and all machines run at 240 frames a second now. Maybe the user is a gamer and has a CRT monitor running at 90 frame a second.

ʵ�����ⲻ��һ����Ч�ļ��衣�û�����ʹ�õ��Ǿɵ������ֻ������ĵ͹����豸�������û������ں�̨����һЩ���صĳ����и��ָ�����ԭ�����������ÿ�� 60 ֡����ʾ�������� 2020 �����еĶ���ÿ������ 240 ֡��Ҳ�����û�����Ϸ��Ҳ�����һ��ÿ������ 90 ֡��CRT��ʾ����

You can see the problem in this example

����������У�����Կ����������

<iframe class="webgl_example " style="width: 400px; height: 300px;" src="/webgl/lessons/../webgl-animation-frame-rate-issues.html"></iframe>

In the example above we want to rotate all of the 'F's at the same speed. The 'F' in the middle is running full speed and is frame rate independent. The one on the left and the right are simulating if the browser was only running at 1/8th max speed for the current machine. The one on the left is NOT frame rate independent. The one on the right IS frame rate independent.

�����ʾ�����������еġ�F������ͬ�����ٶ���ת���м�ġ�F�����Զ�����֡����ȫ�����еġ��������ߵ���ģ��������Ե�ǰ��������ٶȵ� 1/8 ���е��������ߵĲ��Ƕ�����֡���ʡ����ұߵ��Ƕ�����֡����

Notice because the one on the left is not taking into account that the frame rate might be slow it's not keeping up. The one on the right though, even though it's running at 1/8 the frame rate it is keeping up with the middle one running at full speed.

��ע�⣬��ߵĲ�û�п��ǵ�֡���ʿ����ǻ������Ҳ��ȶ��ġ��ұߵ���Ȼ���� 1/8 ��֡�������У�ȴ���м��һ��������ȫ�����С�

The way to make animation frame rate independent is to compute how much time it took between frames and use that to calcuate how much to animate this frame.

ʹ������֡�������ķ����ǣ�������֮֡���ʱ�������������ÿ���֡����������

First off we need to get the time. Fortunately `requestAnimationFrame` passes us the time since the page was loaded when it calls us.

����������Ҫ�õ�֡�������������˵��ǣ�ҳ�����֮�����ǿ��Ե���`requestAnimationFrame`��

I find it easiest if we get the time in seconds but since the `requestAnimationFrame` passes us the time in milliseconds (1000ths of a second) we need to multiply by 0.001 to get seconds.

�ҷ������Ǻ����׻��һ���ڵ�֡������������`requestAnimationFrame`���Ժ��루һ���ǧ��֮һ������ģ�������Ҫ���� 0.001 ȡ��������

So, we can then compute the delta time like this

��ˣ����ǿ�����������ʱ��

```
	var then = 0;
	 
	requestAnimationFrame(drawScene);
	 
	// Draw the scene.
	// ���Ƴ�����
	function drawScene(now) {
	  // Convert the time to seconds
	  // ʱ��ת��Ϊ��
	  now *= 0.001;
	  // Subtract the previous time from the current time
	  // ��ǰʱ���ȥ֮ǰʱ��
	  var deltaTime = now - then;
	  // Remember the current time for the next frame.
	  // Ϊ��һ֡��¼��ǰʱ��
	  then = now;
	 
	   ...
```

Once we have the `deltaTime` in seconds then all our calculations can be in how many units per second we want something to happen. In this case `rotationSpeed` is 1.2 which means we want to rotate 1.2 radians per second. That's about 1/5 of a turn or in other words it will take about 5 seconds to turn around completely regardless of the frame rate.

һ�����ǵõ�`ʱ���`�����ǿ��Լ���������Ҫʵ�ֵ�������ÿ����ִ�еĴ����������ʾ����`rotationSpeed`Ϊ 1.2����ζ������ϣ��ÿ����ת 1.2 Ȧ����Լ 1/5 Ȧ�����仰˵������֡���Ƕ��ٶ���Ҫ��Լ 5 ���ʱ����ת������

```
  rotation[1] += rotationSpeed * deltaTime;
```

Here's that one working.

�������ˡ�

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-animation.html"></iframe>



[click here to open in a separate window][4] 

[����������´��ڴ�][4]

You aren't likely to see a difference from the one at the top of this page unless you are on a slow machine but if you don't make your animations frame rate independent you'll likely have some users that are getting a very different experience than you planned.

������ʹ�õ������豸����Ȼ����ܿ���������ҳ�涥��ʾ�������������û��ʹ����֡���������һЩ�û�����������ƻ��ĺܲ�һ�������顣

Next up [how to apply textures][5].

��һ��[�����������][5]

��������������������

## Don't use setInterval or setTimeout!

## ��Ҫʹ�� setInterval ���� setTimeout��

If you've been programming animation in JavaScript in the past you might have used either `setInterval` or `setTimeout` to get your drawing function to be called. 

���������ʹ�� JavaScript ��̶������������ʹ��`setInterval`����`setTimeout`��������Ļ��ƺ�����

The problems with using `setInterval` or `setTimeout` to do animation are two fold. First off both `setInterval` and `setTimeout` have no relation to the browser displaying anything. They aren't synced to when the browser is going to draw a new frame and so can be out of sync with the user's machine. If you use `setInterval` or `setTimeout` and assume 60 frames a second and the user's machine is actually running some other frame rate you'll be out of sync with their machine. 

ʹ��`setInterval`����`setTimeout`��ʵ�ֶ��������������⡣����`setInterval`��`setTimeout`��ʾ�κζ������������û�й�����������������µ�֡ʱ���ǲ���ͬ����������û����豸Ҳ��ͬ�������ʹ��`setTinterval`����`setTimeout`�����趨ÿ�� 60 ֡���û����豸ʵ���ϻ�������������֡����������ǵ��豸��ͬ����

The other problem is the browser has no idea why you're using `setInterval` or `setTimeout`. So for example, even when your page is not visible, like when it's not the front tab, the browser still has to execute your code. Maybe you're using `setTimeout` or `setInterval` to check for new mail or tweets. There's no way for the browser to know. That's fine if you're just checking every few seconds for new messages but it's not fine if you're trying to draw 1000 objects in WebGL. You'll be effectively DOSing the user's machine with your invisible tab drawing stuff they can't even see. 

��һ���������������֪����Ϊʲô��ʹ��`setInterval`����`setTimeout`�����磬��ʹ���ҳ�治�ɼ������������ǵ�ǰ�ı�ǩ���������Ȼ����ִ����Ĵ��롣Ҳ��������ʹ��` setTimeout`��`setInterval`�鿴���ʼ����� tweets�����������֪���������ֻ�Ǽ��ÿ���������Ϣ��û���⣬����������ڳ����� WebGL ���� 1000 ������Ͳ����ˡ����ǿ����������ڻ��ƶ����ı�ǩʵ�������ڹ����û����豸����ʹ���ǿ�������

`requestAnimationFrame` solves both of these issues. It calls you at just the right time to sync your animation with the screen and it also only calls you if your tab is visible. 

`requestAnimationFrame`��������������⡣������ʱ���������������ʾ��ͬ��������ֻ���ڱ�ǩ�ɼ�������µ��á�

[1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
[2]: http://webglfundamentals.org/webgl/lessons/webgl-3d-camera.html
[3]: http://webglfundamentals.org/webgl/webgl-animation-not-frame-rate-independent.html
[4]: http://webglfundamentals.org/webgl/webgl-animation.html
[5]: http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html