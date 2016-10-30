## WebGL and Alpha

## WebGL ��  Alpha

I've noticed some OpenGL developers having issues with how WebGL treats alpha in the backbuffer (ie, the canvas), so I thought it might be good to go over some of the differences between WebGL and OpenGL related to alpha.

�����⵽һЩ OpenGL �����߶��� WebGL ����˫�����е� alpha ͨ���������ʣ���������Ϊ����һ�� alpha ͨ���� WebGL ��OpenGL ֮��Ĳ�����б�Ҫ��

The biggest difference between OpenGL and WebGL is that OpenGL renders to a backbuffer that is not composited with anything, or effectively not composited with anything by the OS's window manager, so it doesn't matter what your alpha is.

OpenGL �� WebGL ���Ĳ����ǣ�OpenGL ��Ⱦһ��˫���治��Ҫ�����κζ���������Ч�ز��� OS �Ĵ��ڹ��������κζ������ϣ����� alpha  ͨ���Ƕ��ٲ���Ҫ��

WebGL is composited by the browser with the web page and the default is to use pre-multiplied alpha the same as .png `<img>` tags with transparency and 2D canvas tags.

�� WebGL ����Ҫ�������ҳ���ϵģ�Ĭ�ϻ�ʹ�� pre-multiplied alpha ������`<img>`��ǩ��.png��ʽ�� 2D ������ǩ�� alpha ͨ��һ����

WebGL has several ways to make this more like OpenGL.

WebGL �ж��ַ�ʽʹ alpha ͨ����Ч�����ӽ� OpenGL��

**#1) Tell WebGL you want it composited with non-premultiplied alpha**

**#1��֪ͨ WebGL ��ʹ�� pre-multiplied alpha**

```
	gl = canvas.getContext("webgl", {
	  premultipliedAlpha: false  // Ask for non-premultiplied alpha   // ����ʹ�� pre-multiplied alpha
	});
```

The default is true.

Ĭ��ֵ�� true��

Of course the result will still be composited over the page with whatever background color ends up being under the canvas (the canvas's background color, the canvas's container background color, the page's background color, the stuff behind the canvas if the canvas has a z-index > 0, etc....) in other words, the color CSS defines for that area of the webpage.

��Ȼ���ܱ���ɫ�ڻ���֮�£����������ҳ�����ˣ�������������ɫ��������������ɫ��ҳ�汳��ɫ����������� z-index ���� > 0����������������Ķ����ı���ɫ���ȵȡ������������������������ CSS �������ɫ��

A really good way to find if you have any alpha problems is to set the canvas's background to a bright color like red. You'll immediately see what is happening.

��������κι��� alpha ͨ�������⣬��һ���ܺõİ취�����ǽ������ı���ɫ����Ϊ���ɫһ������������ɫ��

```
<canvas style="background: red;"><canvas>
```

You could also set it to black which will hide any alpha issues you have.

��Ҳ���Խ�������Ϊ��ɫ������Խ� alpha ͨ������������������

**#2) Tell WebGL you don't want alpha in the backbuffer**

**#2��֪ͨ WebGL ˫���治��Ҫ alpha **

```
gl = canvas.getContext("webgl", { alpha: false }};
```

This will make it act more like OpenGL since the backbuffer will only have RGB. This is probably the best option because a good browser could see that you have no alpha and actually optimize the way WebGL is composited. Of course that also means it actually won't have alpha in the backbuffer so if you are using alpha in the backbuffer for some purpose that might not work for you. Few apps that I know of use alpha in the backbuffer. I think arguably this should have been the default.

˫����ֻ�� RGB ֵ������ WebGL ����Ϊ���ӽӽ� OpenGL�����������õ�ѡ����Ϊ�õ�������ܹ�������û������ alpha ͨ�������Ż� WebGL �ĸ��ϡ���Ȼ��Ҳ��ζ����˫��������û�� alpha ͨ���ġ���������ĳЩԭ����˫������ʹ���� alpha��ͨ�������ܲ��������á���֪��������˫����ʹ����͸���ȵ�Ӧ�ó���

**#3) Clear alpha at the end of your rendering**

**#3������Ⱦ�Ľ���ʱ��� alpha**

```
	..
	renderScene();
	..
	// Set the backbuffer's alpha to 1.0
	// ˫�����е� alpha ͨ������Ϊ 1.0
	gl.clearColor(1, 1, 1, 1);
	gl.colorMask(false, false, false, true);
	gl.clear(gl.COLOR_BUFFER_BIT);
```

Clearing is generally very fast as there is a special case for it in most hardware. I did this in most of my demos. If I was smart I'd switch to method #2 above. Maybe I'll do that right after I post this. It seems like most WebGL libraries should default to this method. Those few developers that are actually using alpha for compositing effects can ask for it. The rest will just get the best perf and the least surprises.

���һ������¶��Ǻܿ�ģ���Ϊ�������Ӳ���ж���Ϊ������һ��λ�á��ҵĴ���� demo �ж�������һ�����ġ�������㹻�������Һ�ʹ������ڶ��ֵķ�����Ҳ����������ƪ����֮���һ�ʹ����������һЩ�����߿���ʹ����һ������Ч�ظ��� alpha ͨ����ʣ�µľ��ǻ����ѵ����ܺ�СС�ľ�ϲ��

**#4) Clear the alpha once then don't render to it anymore**

**#4������Ⱦʱ������� alpha ͨ��**

```
	// At init time. Clear the back buffer.
	// �ڳ�ʼʱ�������������
	gl.clearColor(1,1,1,1);
	gl.clear(gl.COLOR_BUFFER_BIT);
	 
	// Turn off rendering to alpha
	// �ر� alpha ͨ������Ⱦ
	gl.colorMask(true, true, true, false);
```

Of course if you are rendering to your own framebuffers you may need to turn rendering to alpha back on and then turn it off again when you switch to rendering to the canvas.

��Ȼ�����������Ⱦ֡���棬�������Ҫ���´� alpha ͨ������Ⱦ����Ⱦ����ʱ�ٹرա�

**#5) Handling Images**

**#5������ͼ��**

My default if you are loading images with alpha into WebGL. WebGL will provide the values as they are in the PNG file with color values not premultiplied. This is generally what I'm used to for OpenGL programs because it's lossless whereas pre-multiplied is lossy.

��Ĭ�����ǽ� alpha ͨ�����ؽ��� WebGL��WebGL ���ṩ������ PNG �ļ���û��Ԥ�˵���ɫֵ��ͨ���� OpenGL ���һ�ʹ����һ��������Ϊ��������Ķ� pre-multiplied ������ġ�

```
1, 0.5, 0.5, 0  // RGBA  
```

Is a possible value un-premultiplied whereas pre-multiplied it's an impossible value because `a = 0` which means `r`,`g`, and `b` have to be zero.

����һ�� un-premultiplied ֵ�������� pre-multiplied ����Ϊ `a = 0`��ζ��`r`��`g`��`b`������Ϊ 0��

You can have WebGL pre-multiply the alpha if you want. You do this by setting `UNPACK_PREMULTIPLY_ALPHA_WEBGL` to true like this

�����Ԥ�� alpha��ͨ��������������`UNPACK_PREMULTIPLY_ALPHA_WEBGL`Ϊ true

```
gl.pixelStorei(gl.UNPACK_PREMULTIPLY_ALPHA_WEBGL, true);
```

The default is un-premultiplied.

Be aware that most if not all Canvas 2D implementations work with pre-multiplied alpha. That means when you transfer them to WebGL and `UNPACK_PREMULTIPLY_ALPHA_WEBGL` is false WebGL will convert them back to un-premultipiled.

Ĭ���� un-premultiplied �ġ�

Ҫ֪���������е� Canvas 2D ��ʵ����pre-multiplied alpha ͨ���ġ�����ζ�ŵ��㽫���Ǵ��ݵ� WebGL������`UNPACK_PREMULTIPLY_ALPHA_WEBGL` ��ֵΪ false ʱ��WebGL �Ὣ����ת���� un-premultipiled��

**#6) Using a blending equation that works with pre-multiplied alpha.**

**#6��ʹ�� pre-multiplied alpha ��Ϸ���**

Almost all OpenGL apps I've writing or worked on use

�������е� OpenGL Ӧ�ó����Ҷ�ʹ����һ������

```
gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA);
```

That works for non pre-multiplied alpha textures.

�����ڷ� pre-multiplied alpha ����

If you actually want to work with pre-multiplied alpha textures then you probably want

�������ʹ�� pre-multiplied alpha �������������Ҫ

```
gl.blendFunc(gl.ONE, gl.ONE_MINUS_SRC_ALPHA);
```

Those are the methods I'm aware of. If you know of more please post them below.

��Щ������֪���ķ�����������˽����ķ�����ӭ��?     