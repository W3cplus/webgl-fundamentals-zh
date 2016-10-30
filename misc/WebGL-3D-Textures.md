## WebGL 3D - Textures

## WebGL 3D - ����

This post is a continuation of a series of posts about WebGL. The first [started with fundamentals][1] and the previous was about [animation][2].

��ƪ������ WebGL ϵ�����µ���ƪ����һƪ��[�ӻ���ԭ��ʼ][1]��ǰһƪ��[����][2]

How do we apply textures in WebGL? You could probably derive how by reading [the articles on image processing][3] but it will probably be easier to understand if we go over it in more detail.

������ WebGL ����δ�������������Ķ�[����ͼ���������][3]�˽⵽����������Ǹ���ϸ�����¿��ܻ�����ס�

The first thing we need to do is adjust our shaders to use textures. Here are the changes to the vertex shader. We need to pass in texture coordinates. In this case we just pass them straight through to the fragment shader.

���ȣ�ʹ���������Ǿ���Ҫ����ɫ�����е����������ǶԶ�����ɫ����vertex shader���� ������������Ҫʹ���������ꡣ������ʾ��������ֱ�ӽ����Ǵ��ݸ�ƬԪ��ɫ����fragment shader����

```
	attribute vec4 a_position;
	attribute vec2 a_texcoord;
	 
	uniform mat4 u_matrix;
	 
	varying vec2 v_texcoord;
	 
	void main() {
	  // Multiply the position by the matrix.
	  // position ���� matrix��
	  gl_Position = u_matrix * a_position;
	 
	  // Pass the texcoord to the fragment shader.
	  // ���� texcoord ��ƬԪ��ɫ����
	  v_texcoord = a_texcoord;
	}
```
In the fragment shader we declare a uniform sampler2D which lets us reference a texture. We use the texture coordinates passed from the vertex shader and we call texture2D to look up a color from that texture.

��ƬԪ��ɫ������������һ�� uniform sampler2D ������������ʹ�ö�����ɫ���������������� texture2D ������������

```
	precision mediump float;
	 
	// Passed in from the vertex shader.
	// �Ӷ�����ɫ���д��롣
	varying vec2 v_texcoord;
	 
	// The texture.
	// ����
	uniform sampler2D u_texture;
	 
	void main() {
	   gl_FragColor = texture2D(u_texture, v_texcoord);
	}
```
We need to setup the texture coordinates

������Ҫ�����������ꡣ

```
	// look up where the vertex data needs to go.
	// ���Ҷ������ݵ�λ�á�
	var positionLocation = gl.getAttribLocation(program, "a_position");
	var texcoordLocation = gl.getAttribLocation(program, "a_texcoords");
	 
	...
	 
	// Create a buffer for texcoords.
	// �������괴����������
	var buffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
	gl.enableVertexAttribArray(texcoordLocation);
	 
	// We'll supply texcoords as floats.
	// �ṩ�����͵��������ꡣ
	gl.vertexAttribPointer(texcoordLocation, 2, gl.FLOAT, false, 0, 0);
	 
	// Set Texcoords.
	// �����������ꣻ
	setTexcoords(gl);
```
And you can see the coordinates we're using which are mapping the entire texture to each quad on our 'F'.

����Կ�������ʹ�õ����꽫��������ӳ�䵽��F����ÿһ�������ϡ�

```
	// Fill the buffer with texture coordinates for the F.
	// �� F ������������仺������
	function setTexcoords(gl) {
	  gl.bufferData(
	      gl.ARRAY_BUFFER,
	      new Float32Array([
	        // left column front
	        // �������
	        0, 0,
	        0, 1,
	        1, 0,
	        0, 1,
	        1, 1,
	        1, 0,
	 
	        // top rung front
	        // ��������
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

������Ҫһ���������Դ�ͷ��ʼ��һ����������ķ��������Ǽ���һ��ͼƬ��

Here's the image we're going to use

�������ǽ�Ҫʹ�õ�ͼƬ

![textrue][4]

What an exciting image! Actually an image with an 'F' on it has a clear direction so it's easy to tell if it's turned or flipped etc when we use it as a texture.

��ô�����˷ܵ�ͼƬ��ʵ����ͼƬ�еġ�F���Ѿ���һ����ȷ�ķ����ˣ�������ʹ������Ϊһ������ʱ���������ж����Ƿ���ת��ת��

The thing about loading an image is it happens asynchronously. We request the image to be loaded but it takes a while for the browser to download it. There are generally 2 solutions to this. We could make the code wait until the texture has downloaded and only then start drawing. The other solution is to make up some texture to use until the image is downloaded. That way we can start rendering immediately. Then, once the image has been downloaded we copy the image to the texture. We'll use that method below.

ͼƬ���첽���صġ������������ͼ���������Ҫһ��ʱ��ȥ���ء�һ�������ֽ��������һ�ַ����ǵȴ�����������ٿ�ʼ���ơ���һ�ַ�����ͼ��������ٲ��������������ǿ���������ʼ��Ⱦ��һ��ͼ������꣬�ͽ�ͼ���Ƶ������С����ǽ�ʹ�����·�����

```
	// Create a texture.
	//��������
	var texture = gl.createTexture();
	gl.bindTexture(gl.TEXTURE_2D, texture);
	 
	// Fill the texture with a 1x1 blue pixel.
	// �� 1x1 ����ɫ������䵽�����С�
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE,
	              new Uint8Array([0, 0, 255, 255]));
	 
	// Asynchronously load an image
	// �첽����ͼ��
	var image = new Image();
	image.src = "resources/f-texture.png";
	image.addEventListener('load', function() {
	  // Now that the image has loaded make copy it to the texture.
	  // ͼ���������Ƶ������С�
	  gl.bindTexture(gl.TEXTURE_2D, texture);
	  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
	  gl.generateMipmap(gl.TEXTURE_2D);
	});
```
And here it is

Ч������

<iframe class="webgl_example"  style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures.html"></iframe>

[click here to open in a separate window][5] 


[����������´��ڴ�][5]

What if we wanted to just use a part of the texture across the front of the 'F'? Textures are referenced with "texture coordinates" and texture coordinates go from 0.0 to 1.0 from left to right across the texture and 0.0 to 1.0 from bottom to top up the texture.

�������ֻ���ڡ�F��������ʹ�������һ�����أ������Ǳ����������ꡯ���õģ���������������������ҵķ�Χ�� [0.0��1.0]�����µ��ϵķ�Χ�� [0.0��1.0]��

![svg][6]

So if we match up our vertices to the texture we can figure out what texture coordinates to use.

��ˣ�������ǽ�����ƥ�䵽�����У����ǿ��Լ������Ҫʹ�õ��������ꡣ

![svg1][7]

Here are the texture coordinates for the front.

�����������������

```
	    // left column front
	    // �������
	    0.22, 0.19,
	    0.22, 0.79,
	    0.34, 0.19,
	    0.22, 0.79,
	    0.34, 0.79,
	    0.34, 0.19,
	 
	    // top rung front
	    // ��������
	    0.34, 0.19,
	    0.34, 0.31,
	    0.62, 0.19,
	    0.34, 0.31,
	    0.62, 0.31,
	    0.62, 0.19,
	 
	    // middle rung front
	    // �м����
	    0.34, 0.43,
	    0.34, 0.55,
	    0.49, 0.43,
	    0.34, 0.55,
	    0.49, 0.55,
	    0.49, 0.43,
```

And here it is.

Ч�����¡�

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-texture-coords-mapped.html"></iframe> 

[click here to open in a separate window][8]

[����������´��ڴ�][8]

Not a very exciting display but hopefully it demonstrates how to use texture coordinates. If you're making geometry in code (cubes, spheres, etc) it's usually pretty easy to compute whatever texture coordinates you want. On the other hand if you're getting 3d models from 3d modeling software like Blender, Maya, 3D Studio Max, then your artists (or you) will adjust texture coordinates in those packages.

����һ�����˷ǳ��˷ܵ�չʾ������������ʾ�����ʹ���������ꡣ������ڴ�����ʹ���˼����壨�����壬����ȣ�������ͨ�������׾��ܼ�����κ�����Ҫ���������ꡣ��һ���棬������ 3d ��ģ����� Blender��Maya��3D Studio Max �л�ȡ 3d ģ�ͣ�������ʦ�������㣩������Щ�������У׼�������ꡣ

So what happens if we use texture coordinates outside the 0.0 to 1.0 range. By default WebGL repeats the texture. 0.0 to 1.0 is one 'copy' of the texture. 1.0 to 2.0 is another copy. even -4.0 to -3.0 is yet another copy. Let's display a plane using these texture coordinates.

�������ʹ�� [0.0��1.0] ֮�����������ᷢ��ʲô��WebGL Ĭ�����ظ�����ġ�[0.0�� 1.0] ��һ���������ơ�������[1.0��2.0] ��һ����������ʹ�� [��4.0����3.0] Ҳ����һ��������������ʹ����Щ��������չʾƽ��ɡ�

```
 -3, -1,
  2, -1,
 -3,  4,
 -3,  4,
  2, -1,
  2,  4,
```
and here it is

Ч������

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-repeat-clamp.html"></iframe> 

[click here to open in a separate window][9]

[����������´��ڴ�][9]

You can tell WebGL to not repeat the texture in a certain direction by using CLAMP_TO_EDGE. For example

��������� CLAMP_TO_EDGE ���� WebGL ���ض��ķ����ظ���������

```
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
```
Click the buttons in the sample above to see the difference.

�������ʾ���еİ�ť���鿴���졣

You might have noticed a call to `gl.generateMipmap` back when we loaded the texture. What is that for?

�����ע�⵽�ˣ������Ǽ���������֮�������`gl.generateMipmap`������ʲô���ã�

Imagine we had this 16x16 pixel texture.

������������� 16x16 ���ص�����


![16x16][10]

Now imagine we tried to draw that texture on a polygon 2x2 pixels big on the screen. What colors should we make those 4 pixels? There are 256 pixels to choose from. In Photoshop if you scaled a 16x16 pixel image to 2x2 it would average the 8x8 pixels in each corner to make the 4 pixels in a 2x2 image. Unfortunately reading 64 pixels and averaging them all together would be way too slow for a GPU. In fact imagine if you had a 2048x2084 pixel texture and you tried to draw it 2x2 pixels. To do what Photoshop does for each of the 4 pixels in the 2x2 result it would have to average 1024x1024 pixel or 1 million pixels times 4. That's way way too much to do and still be fast.

���ڼ������ǳ�������Ļ�н������Ƶ� 2x2 ���صĶ�����С�����Ӧ�ø��� 4 ������ʲô��ɫ����256�����ص���ɫ����ѡ���� Photoshop �У�����㽫һ�� 16x16 ��ͼ����С�� 2x2�����Ὣÿ������� 8x8 ����ƽ�����䵽һ�� 2x2 ͼ��� 4 �����С����ҵ��ǣ�һ�� GPU ��ȡ 64 �����ز�ƽ�����䣬�ٶ�̫����ʵ���ϣ���������һ�� 2048x2048 ���ص���������ͼ�������Ƶ� 2x2 �����У�������ƽ������ 1024x1024 ���ػ� 100 �����س��� 4���Դﵽ Photoshop ����� 2x2 �� 4 ���ص�Ч���������ķ����ܶಢ�Ҷ��ܿ��١�

So what the GPU does is it uses a mipmap. A mipmap is a collection of progressively smaller images, each one 1/4th the size of the previous one. The mipmap for the 16x16 texture above would look something like this.

GPU ʹ�� mipmap ʵ�֡�mipmap ��һ��ͼ��С�ļ��ϣ�ÿһ�� mip �Ĵ�С��ǰһ���� 1/4������������ 16x16 �������� mipmap ����������ӡ�


![mipmap][11]

Generally each smaller level is just a bilinear interpolation of the previous level and that's what gl.generateMipmap does. It looks at the biggest level and generates all the smaller levels for you. Of course you can supply the smaller levels yourself if you want.

һ����˵��ÿһ����С��ֻ��ǰһ����һ��˫���Բ�ֵ������� gl.generateMipmap ���������顣��������������� mip �Ļ����ϣ��������н�С�� mip����Ȼ������ṩ��С�� mip��

Now if you try to draw that 16x16 pixel texture only 2x2 pixels on the screen WebGL can select the mip that's 2x2 which has already been averaged from the previous mips.

�������������Ļ�е� 2x2 �����ϻ��� 16x16 ���ص�����WebGL ���Դ��Ѿ�ƽ������õ� mips ��ѡ�� 2x2 �� mip��

You can choose what WebGL does by setting the texture filtering for each texture. There are 6 modes

�����Ϊÿһ��������������������Ӷ�ѡ�� WebGL ����Ϊ��һ���� 6 ��ģʽ��

*	NEAREST = choose 1 pixel from the biggest mip
*	NEAREST = ������ mip ��ѡ�� 1 ������
*	LINEAR = choose 4 pixels from the biggest mip and blend them
*	LINEAR = ������ mip ��ѡ�� 4 �����ز���������
*	NEAREST_MIPMAP_NEAREST = choose the best mip, then pick one pixel from that mip
*	NEAREST_MIPMAP_NEAREST = ����ǡ���� mip��ѡ�� 1 ������
*	LINEAR_MIPMAP_NEAREST = choose the best mip, then blend 4 pixels from that mip
*	LINEAR_MIPMAP_NEAREST = ����ǡ���� mip��ѡ�� 4 �����ز�����
*	NEAREST_MIPMAP_LINEAR = choose the best 2 mips, choose 1 pixel from each, blend them
*	NEAREST_MIPMAP_LINEAR = ����ǡ���� 2 �� mip�У�����ѡ�� 1 �����أ���������
*	LINEAR_MIPMAP_LINEAR = choose the best 2 mips. choose 4 pixels from each, blend them
*	LINEAR_MIPMAP_LINEAR = ����ǡ���� 2 ���� mip�У�����ѡ�� 4 �����أ���������

You can see the importance of mips in these 2 examples. The first one shows that if you use `NEAREST` or `LINEAR` and only pick from the largest image then you'll get a lot of flickering because as things move, for each pixel it draws it has to pick a single pixel from the largest image. That changes depending on the size and position and so sometimes it will pick one pixel, other times a different one and so it flickers.

�������������ʾ���п��� mips ����Ҫ�ԡ���һ��չʾ�����ʹ��`NEAREST`����`LINEAR`��WebGL ֻ��ѡ������ͼ����ῴ���ܶ����˸����Ϊ����ģ�͵��ƶ������Ƶ�ÿһ�����ض����������ͼ����ѡ�� 1 �����ء����ֱ仯ȡ���ڴ�С��λ�õı仯����ʱ����ѡ��һ�����أ�����ʱ��ѡ����������أ����������˸��

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-mips.html"></iframe> 

[click here to open in a separate window][12] 

[����������´��ڴ�][12]

Notice how much the ones on the left and middle flicker where as the ones on the right flicker less. The ones on the right also have blended colors since they are using the mips. The smaller you draw the texture the further apart WebGL is going to pick pixels. That's why for example the bottom middle one, even though it's using LINEAR and blending 4 pixels it flickers because those 4 pixels are from different corners of the 16x16 image depending on which 4 are picked you'll get a different color. The one on the bottom right though stays a consistent color because it's using the 2nd to the smallest mip.

ע����ߺ��м����˸���ұߵĶࡣ���ұ�ʹ�õ� mips Ҳ�л����ɫֵ��WebGL ����ԽԶ�Ĳ��ֻ�ѡ��ԽС���ص�������䡣Ϊʲôʾ���е����²��֣���ʹ��ʹ�� LINEAR �������� 4 ���أ���Ȼ��˸��������Ϊ�� 4 ���������� 16x16 ͼ���в�ͬ�Ľ��䣬��ɫȡ����ѡ������ 4 �����ء����µı���һ�µ���ɫ������Ϊ��ʹ���˵ڶ�����С�� mip��

The second example shows polygons that go deep into the screen.

�ڶ���ʾ��չʾ������Ļ��Ķ���Ρ�

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-mips-tri-linear.html"></iframe> 

[click here to open in a separate window][13]

[����������´��ڴ�][13]

The 6 beams going into the screen are using the 6 filtering modes listed above. The top left beam is using `NEAREST` and you can see it's clearly very blocky. The top middle is using `LINEAR` and it's not much better. The top right is using `NEAREST_MIPMAP_NEAREST`. Click on the image to switch to a texture where every mip is a different color and you'll easily see where it chooses to use a specific mip. The bottom left is using `LINEAR_MIPMAP_NEAREST` meaning it picks the best mip and then blends 4 pixels within that mip. You can still see a clear area where it switches from one mip to the next mip. The bottom middle is using `NEAREST_MIPMAP_LINEAR` meaning picking the best 2 mips, picking one pixel from each and blending them. If you look close you can see how it's still blocky, especially in the horizontal direction. The bottom right is using `LINEAR_MIPMAP_LINEAR` which is picking the best 2 mips, picking 4 pixels from each, and blends all 8 pixels.

������Ļ�� 6 �������ֱ�ʹ�������г��� 6 �ֹ���ģʽ�����ϵĹ�ʹ����`NEAREST`������Կ����ǳ������Ŀ�״�����ϵĹ�ʹ����`LINEAR`�����Ϳ�����û��ô���������ϵĹ�ʹ����`NEAREST_MIPMAP_NEAREST`�����ͼ���л�����ÿһ�� map ����ɫ����ͬ��������׵Ŀ��� WebGL ʹ�����ĸ� mip�����µĹ�ʹ����`LINEAR_MIPMAP_NEAREST`���������ǡ���� mip ��ѡ�� 4 �����ز�����������Ȼ��������Ŀ�����һ�� mip �л�����һ�� mip ���������µĹ�ʹ����`NEAREST_MIPMAP_LINEAR`���������ǡ���� 2 �� mip �и���ѡ�� 1 �����أ��������ǡ������ϸ��������Կ�������������ˮƽ��������Ȼ�ǿ�״�ģ������µĹ�ʹ����`LINEAR_MIPMAP_LINEAR`����ѡ�� 2 ����ǡ���� mip���Ӹ�����ѡ�� 4 �����أ��������� 8 �����ء�

![different colored mips][14]

different colored mips

��ͬ��ɫ�� mips

You might be thinking why would you ever pick anything other than `LINEAR_MIPMAP_LINEAR` which is arguably the best one. There are many reasons. One is that `LINEAR_MIPMAP_LINEAR` is the slowest. Reading 8 pixels is slower than reading 1 pixel. On modern GPU hardware it's probably not an issue if you are only using 1 texture at a time but modern games might use 2 to 4 textures at once. 4 textures * 8 pixels per texture = needing to read 32 pixels for every pixel drawn. That's going to be slow. Another reason is if you're trying to achieve a certain effect. For example if you want something to have that pixelated retro look maybe you want to use `NEAREST`. Mips also take memory. In fact they take 33% more memory. That can be a lot of memory especially for a very large texture like you might use on a title screen of a game. If you are never going to draw something smaller than the largest mip why waste memory for those mips. Instead just use `NEAREST` or `LINEAR` as they only ever use the first mip.

�����Ϊʲôѡ���`LINEAR_MIPMAP_LINEAR`֮���ѡ����������õġ����кܶ��ԭ������һ����`LINEAR_MIPMAP_LINEAR`����Ӧ�ٶ��������ġ���ȡ 8 �����رȶ�ȡ 1 ���������������һ��ֻʹ��1�����������ִ�GPU��Ӳ�����ܲ������⣬�����ִ���Ϸ�п��ܻ�ͬʱʹ�� 2 �� 4 ������4 ������ * 8 ������ = ÿ��һ���������ȡ 32 �����ء�����Ӧ�ٶȽ����������һ��ԭ����������ͼ�ﵽһ����Ч��ʱ�����磬�����ϣ��ͼ���������ţ�����ܻ�ʹ��`NEAREST`��Mips ��ռ���ڴ�ġ�ʵ�������ǻ�ռ�� 33% ���ϵ��ڴ档���������Ϸ��һ����С����Ļ��ʹ��һ���ܴ��������ռ�ø�����ڴ档�������Ƶ�ͼ������� mip ������ΪʲôҪ�˷��ڴ����洢��Щ mips �ء�`NEAREST`����`LINEAR`ֻʹ�õ�һ�� mip��

To set filtering you call `gl.texParameter` like this

�������������`gl.texParameteri`���ù�������

```
	gl.texParameter(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
	gl.texParameter(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
```
`TEXTURE_MIN_FILTER` is the setting used when the size you are drawing is smaller than the largest mip. `TEXTURE_MAG_FILTER` is the setting used when the size you are drawing is larger than the largest mip. For `TEXTURE_MAG_FILTER` only `NEAREST` and `LINEAR` are valid settings.

`TEXTURE_MIN_FILTER`������ͼƬ�ߴ�С������ mip �ġ�`TEXTURE_MAG_FILTER`������ͼƬ�ߴ�������� mip �ġ�`TEXTURE_MAG_FILTER`ֻ��`NEAREST`��`LINEAR`������Чֵ��

Let's say we wanted to apply this texture.

������Ӧ���������

![keyboard][15]

Here it is.

Ч�����¡�

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-bad-npot.html"></iframe> 

[click here to open in a separate window][16]

[����������´��ڴ�][16]

Why doesn't the keyboard texture show up? That's beacuse WebGL has a kind of severe restriction on textures that are not a power of 2 in both dimensions. Powers of 2 are 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, etc. The 'F' texture was 256x256. 256 is a power of 2. The keyboard texture is 320x240. Neither of those are a power of 2 so trying to display the texture fails. In the shader when `texture2D` is called and when the texture referenced is not setup correctly WebGL will use color (0, 0, 0, 1) which is black. If you open up the JavaScript console or Web Console, depending on the browser you might see errors pointing out the problem like this

Ϊʲô��������û����ʾ������������Ϊ WebGL ������������ƺ��ϸ������ÿ�������ϳߴ綼��Ϊ 2 ��ָ�������������ÿ�������϶������ϡ�2 �ı����� 1��2��4��8��16��32��64��128��256��512��1024��2048���ȵȡ���F��������ߴ��� 256x256��256 �� 2 ��ָ������������ߴ��� 320x240�����߾����� 2 ��ָ����������ʾ����ʧ�ܡ�������`texture2D`�����õ�����û�б���ȷ����ʱ��WebGL ��ʹ�ú�ɫ��0��0��0��1�����档������ JavaScript ����̨���� WebGL ����̨����ȡ������ʹ�õ������������ܻῴ�������Ĵ�����Ϣ

```
	WebGL: INVALID_OPERATION: generateMipmap: level 0 not power of 2
	   or not all the same size
	WebGL: drawArrays: texture bound to texture unit 0 is not renderable.
	   It maybe non-power-of-2 and have incompatible texture filtering or
	   is not 'texture complete'.
```
To fix it we need to set the wrap mode to `CLAMP_TO_EDGE` and turn off mip mapping by setting filtering to `LINEAR` or `NEAREST`.

Ϊ�˽��������⣬������Ҫ���û���ģʽΪ `CLAMP_TO_EDGE`�����ҹرչ��������õ�`LINEAR`����`NEAREST`�� mip ӳ�䡣

Let's update our image loading code to handle this. First we need a function that will tell us if a value is a power of 2.

�����Ǹ���ͼƬ���صĴ��롣����������Ҫһ�������ж��Ƿ�Ϊ 2 ��ָ����

```
	function isPowerOf2(value) {
	  return (value & (value - 1)) == 0;
	}
```
I'm not going to go into the binary math on why this works. Since it does work though, we can use it as follows.

�����Ҳ��������������ѧ����Ȼ����Ч�����ǿ�������ʹ������

```
	// Asynchronously load an image
	// �첽����ͼ��
	var image = new Image();
	image.src = "resources/keyboard.jpg";
	image.addEventListener('load', function() {
	  // Now that the image has loaded make copy it to the texture.
	  // ����ͼ���Ѿ������꣬�������Ƶ������С�
	  gl.bindTexture(gl.TEXTURE_2D, texture);
	  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,gl.UNSIGNED_BYTE, image);
	 
	  // Check if the image is a power of 2 in both dimensions.
	  // ���ͼ����ÿ���������Ƿ��Ϊ 2 ��ָ����
	  if (isPowerOf2(image.width) && isPowerOf2(image.height)) {
	     // Yes, it's a power of 2. Generate mips.
	     // �� 2 ��ָ���������� mips��
	     gl.generateMipmap(gl.TEXTURE_2D);
	  } else {
	     // No, it's not a power of 2. Turn of mips and set wrapping to clamp to edge
	     // ���� 2 ��ָ���������� wrapping ���� mips �ı�Ե��
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
	     gl.texParameteri(gl.TETXURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
	  }
	}
```
And here's that

Ч������

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-good-npot.html"></iframe>

[click here to open in a separate window][17]

[����������´��ڴ�][17]

A common question is "How do I apply a different image to each face of a cube?". For example let's say we had these 6 images.

һ�������������ǡ���������������ÿһ����Ӧ�ò�ͬ��ͼ�񣿡������磬�������� 6 ��ͼ��

![images of cube][18]

3 answers come to mind

�� 3 �ַ���ʵ��

1��  make a complicated shader that references 6 textures and pass in some extra per vertex info into the vertex shader that gets passed to the fragment shader to decide which texture to use. DON'T DO THIS! A little thought would make it clear that you'd end up having to write tons of different shaders if you wanted to do the same thing for different shapes with more sides etc.

1��ʹ��һ�����ӵ���ɫ���������ɫ�������� 6 ���������򶥵���ɫ����vertex shader������ÿ������ĸ�����Ϣ����ƬԪ��ɫ����fragment shader������ʹ���ĸ�������Ҫ����������΢˼��һ�¾ͻ����ף���������ڸ������������Ӧ��ͬ���ķ�������������д��������ɫ����

2�� draw 6 planes instead of a cube. This is a common solution. It's not bad but it also only really works for small shapes like a cube. If you had a sphere with 1000 quads and you wanted to put a different texture on each quad you'd have to draw 1000 planes and that would be slow.

2������ 6 ��ƽ�������һ�������塣���ǳ����Ľ������������������������ֻ������������������С��״�ϲ�������Ч���������һ��ӵ�� 1000 �����ε����壬����Ϊÿһ���������ò�ͬ�������������� 1000 ��ƽ�棬�⽫�������Ĺ��̡�

3�� The, dare I say, best solution is to put all of the images in 1 texture and use texture coordinates to map a different part of the texture to each face of the cube. This is the technique that pretty much all high performance apps (read games) use. So for example we'd put all the images in one texture possibly like this.

3����õĽ���취�ǽ����е�ͼ����� 1 �������У�ʹ����������Ϊ�������ÿһ��ӳ�䲻ͬ�������֡��⼸�������и�����Ӧ�ó�����Ϸ����ʹ�õļ�������ˣ����ǻ����´������ͼ��һ�������С�

![all the images in one texture ][19]

and then use a different set of texture coordinates for each face of the cube.

Ȼ��Ϊ�������ÿһ�����ò�ͬ���������ꡣ

```
	    // select the bottom left image
	    // ѡ�����µ�ͼ��
	    0   , 0  ,
	    0   , 0.5,
	    0.25, 0  ,
	    0   , 0.5,
	    0.25, 0.5,
	    0.25, 0  ,
	    // select the bottom middle image
	    // ѡ�����µ�ͼ��
	    0.25, 0  ,
	    0.5 , 0  ,
	    0.25, 0.5,
	    0.25, 0.5,
	    0.5 , 0  ,
	    0.5 , 0.5,
	    // select to bottom right image
	    // ѡ�����µ�ͼ��
	    0.5 , 0  ,
	    0.5 , 0.5,
	    0.75, 0  ,
	    0.5 , 0.5,
	    0.75, 0.5,
	    0.75, 0  ,
	    // select the top left image
	    // ѡ�����ϵ�ͼ��
	    0   , 0.5,
	    0.25, 0.5,
	    0   , 1  ,
	    0   , 1  ,
	    0.25, 0.5,
	    0.25, 1  ,
	    // select the top middle image
	    // ѡ�����ϵ�ͼ��
	    0.25, 0.5,
	    0.25, 1  ,
	    0.5 , 0.5,
	    0.25, 1  ,
	    0.5 , 1  ,
	    0.5 , 0.5,
	    // select the top right image
	    // ѡ�����ϵ�ͼ��
	    0.5 , 0.5,
	    0.75, 0.5,
	    0.5 , 1  ,
	    0.5 , 1  ,
	    0.75, 0.5,
	    0.75, 1  ,
```
And we get

���ǵõ�����Ч��

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-3d-textures-texture-atlas.html"></iframe> 

[click here to open in a separate window][20]

[����������´��ڴ�][20]

This style of applying multiple images using 1 texture is often called a texture atlas. It's best because there's just 1 texture to load, the shader stays simple as it only has to reference 1 texture, and it only requires 1 draw call to draw the shape instead of 1 draw call per texture as it might if we split it into planes.

����ʹ��һ������Ӧ�ö��ͼƬ�ķ�����Ϊ������������õķ�������Ϊֻ��Ҫ����һ��������ɫ�������ż򵥣���ֻ��������һ������ֻ����һ�λ��Ʒ�������ͼ�Σ�������ǽ����ֽ�Ϊ���ƽ�����ҪΪÿһ���������һ�λ��Ʒ�����

Next up [lets start simplifying with less code more fun][21].

��һ����[һ��򻯴��룬��ø�����Ȥ][21]

-----------
## UVs vs. Texture Coordinates
## UVs vs. Texture Coordinates
Texture coordinates are often shortened to texture coords, texcoords or UVs (pronounced Ew-Vees). I have no idea where the term UVs came from except that vertex positions often use `x, y, z, w` so for texture coordinates they decided to use `s, t, u, v` to try to make it clear which of the 2 types you're refering to. Given that though you'd think they'd be called Es-Tees and in fact if you look at the texture wrap settings they are called `TEXTURE_WRAP_S` and `TEXTURE_WRAP_T` but for some reason as long as I've been working in graphics people have called them Ew-Vees. 
So now you know if someone says UVs they're talking about texture coordinates.

Texture coordinates ��������дΪtexture coords��texcoords ���� UVs (pronounced Ew-Vees)���Ҳ�֪�� UVs ��һ����Ӻζ��������Ƕ���λ��ͨ��ʹ�� x��y��z��w ��ʾ������������ʹ��s��t��u��v ��ʾ����ͼ��������ָ�ĸ����ꡣ�������������Ӧ�ñ���Ϊ Es-Tees����ʵ���ϣ�����鿴����� wrap ���ã��ᷢ�����Ǳ���Ϊ `TEXTURE_WRAP_S`��`TEXTURE_WRAP_T`��������ĳ��ԭ������ͼѧ�����ǳ�����Ϊ Ew-Vees����������������ˣ��������˵ UVs������ָ�����������ꡣ

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