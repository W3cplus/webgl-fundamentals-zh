## WebGL - Rasterization vs 3D libraries

## WebGL－栅格化 vs 3D 库

This post is a kind of side topic on a series of posts about WebGL. The first [started with fundamentals][1].

这篇文章是 WebGL 一系列文章中的一个的周边话题。本系列文章的第一篇是[从基础原理开始][1]。

I'm writing this because my claim that WebGL is a Rasterization API and not a 3D API touches a nerve with some people. I'm not sure why they feel threatened or whatever it is that makes them so upset I called WebGL a Rasterization API.

我写这篇文章是因为我认为 WebGL 是个栅格化 API 而不是 3D API，这触动了一些人的神经。我不知道他们为什么感觉受到威胁，可能是我将 WebGL 称为栅格化 API 让他们不安。

Arguably everything is a matter of perspective. I might say a knife is an eating utensil, someone else might say a knife is a tool and yet another person might say a knife is a weapon.

这一切都是看角度的问题。好比，我认为刀是一种饮食器具，有的人可能认为刀是一种工具，还有另一些人可能认为刀是一种武器。

In the case of WebGL though there's a reason I think it's important to call WebGL a rasterization API and that is specifically because of the amount of 3D math knowledge you need to know to use WebGL to draw anything in 3D.

在 WebGL 的案例中，有一个我认为 WebGL 称为栅格化 API 的重要原因，就是使用 WebGL 绘制 3D 的任何东西时，你需要运用大量的 3D 数学知识。

I would argue that anything that calls itself a 3D library should do the 3D parts for you. You should be able to give the library some 3D data, some material parameters, some lights and it should draw 3D for you. WebGL (and OpenGL ES 2.0+) are both used to draw 3D but neither fits this description.

我会说，任何自称是 3D 的库都应该为你完成的 3D 的绘制部分。你要做的就是为这些库提供 3D 数据。一些材料参数，一些灯光，然后它就能够绘制。WebGL（和 OpenGL ES 2.0＋）都是用来绘制 3D，这样的描述并不符合。

To give an anology, C++ does not "process words" out of the box. We don't call C++ a "word processor" even though word processors can be written in C++. Similarly WebGL does not draw 3D graphics out of the box. You can write a library that will draw 3D graphics with WebGL but by itself it does not do 3D graphics.

打个比喻，C＋＋不是开箱即用的“过程语言”。我们也不会称 C＋＋ 为“语言处理器”，即使语言处理器能够用 C＋＋ 来写。同样的，WebGL 不是开箱即用的 3D 绘制图形语言。你可以用 WebGL 写一个用于绘制 3D 图形的库，但 WebGL 本身并不是 3D 图形语言。

To give a further example, assume we want to draw a cube in 3D with lights.

进一步的例子，假设我们想画一个带有灯光效果的三维立方体。

Here's the code in three.js to display this

这是 three.js 代码绘制的

```
	  // Setup WebGL.
	  // 设置 WebGL。
	  var c = document.getElementById("c");
	  renderer = new THREE.WebGLRenderer();
	  renderer.setSize(c.clientWidth, c.clientHeight);
	  c.appendChild(renderer.domElement);
	 
	  // Make and setup a camera.
	  // 创建并设置一个相机。
	  camera = new THREE.PerspectiveCamera(
	      70, c.clientWidth / c.clientHeight, 1, 1000);
	  camera.position.z = 400;
	  camera.updateProjectionMatrix();
	 
	  // Make a scene
	  // 创建一个场景
	  scene = new THREE.Scene();
	 
	  // Make a cube.
	  // 创建一个立方体。
	  var geometry = new THREE.BoxGeometry(200, 200, 200);
	 
	  // Make a material
	  // 创建一种材料
	  var material = new THREE.MeshPhongMaterial({
	    ambient: 0x555555,
	    color: 0x555555,
	    specular: 0xffffff,
	    shininess: 50,
	    shading: THREE.SmoothShading
	  });
	 
	  // Create a mesh based on the geometry and material
	  // 创建一个基于几何图形和材料的网格
	  mesh = new THREE.Mesh(geometry, material);
	  scene.add(mesh);
	 
	  // Add 2 lights.
	  // 添加 2 中灯光。
	  light1 = new THREE.PointLight(0xff0040, 2, 0);
	  light1.position.set(200, 100, 300);
	  scene.add(light1);
	 
	  light2 = new THREE.PointLight(0x0040ff, 2, 0);
	  light2.position.set(-200, 100, 300);
	  scene.add(light2);
```

and here it is displayed.

显示效果如下。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/resources/three-js-cube-with-lights.html"></iframe>

[click here to open in a separate window][2] 

[点击这里在新窗口打开][2]

Here's similar code in OpenGL (not ES) to display a cube with 2 lights.

这是用 OpenGL（不是 ES）实现类似效果的代码。

```
  // Setup
  // 设置
    glViewport(0, 0, width, height);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    gluPerspective(70.0, width / height, 1, 1000);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
   
    glClearColor(0.0, 0.0, 0.0, 0.0);
    glEnable(GL_DEPTH_TEST);
    glShadeModel(GL_SMOOTH);
    glEnable(GL_LIGHTING);
   
    // Setup 2 lights
    // 设置 2 中灯光
    glEnable(GL_LIGHT0);
    glEnable(GL_LIGHT1);
    float light0_position[] = {  200, 100, 300, };
    float light1_position[] = { -200, 100, 300, };
    float light0_color[] = { 1, 0, 0.25, 1, };
    float light1_color[] = { 0, 0.25, 1, 1, };
    glLightfv(GL_LIGHT0, GL_DIFFUSE, light0_color);
    glLightfv(GL_LIGHT1, GL_DIFFUSE, light1_color);
    glLightfv(GL_LIGHT0, GL_POSITION, light0_position);
    glLightfv(GL_LIGHT1, GL_POSITION, light1_position);
  ...
   
    // Draw a cube.
    // 绘制一个立方体。
    static int count = 0;
    ++count;
   
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glLoadIdentity();
    double angle = count * 0.1;
    glTranslatef(0, 0, -400);
    glRotatef(angle, 0, 1, 0);
   
    glBegin(GL_TRIANGLES);
    glNormal3f(0, 0, 1);
    glVertex3f(-100, -100, 100);
    glVertex3f( 100, -100, 100);
    glVertex3f(-100,  100, 100);
    glVertex3f(-100,  100, 100);
    glVertex3f( 100, -100, 100);
    glVertex3f( 100,  100, 100);
   
    /*
    ...
    ... repeat for 5 more faces of cube
    ...
    */
   
    glEnd();
```

Notice how we need almost no knowledge of 3D math for either of those examples. Compare that to WebGL. I'm not going to write the code required for WebGL. The code is not that much larger. It's not about the amount of lines required. It's about the amount of `knowledge` required. In the two 3D libraries they take care of the 3D. You give them a camera position and field of view, a couple of lights, and a cube. They deal with all the rest. In other words: They are 3D libraries.

注意这些例子中，我们几乎不需要任何 3D 数学知识。与 WebGL 进行对比。但是我并不打算写 WebGL 的代码。这与代码量无关。而是关乎“知识”量的要求。上面 2 个 3D 库中，它们专注于 3D 效果。你只需设置相机的位置和视角，两个灯光效果，和一个立方体。其余工作由它们解决。换句话说，它们是 3D 库。

In WebGL on the other hand you need to know matrix math, normalized coordinates, frustums, cross products, dot products, varying interpolation, lighting specular calculations and all kinds of other stuff that often take months or years to fully understand.

在 WebGL 中你需要知道矩阵数学方面的知识，归一化坐标，平截头体，叉乘，点乘，多种插值计算，照明反射计算以及其他各种事情，这往往需要几个月或几年的时间来充分了解。

A 3D library's entire point is to have that knowledge built in so you don't need that knowledge yourself, you can just rely on the library to handle it for you. This was true for the original OpenGL as shown above. It's true of other 3D libraries like three.js. It is NOT true of OpenGL ES 2.0+ or WebGL.

一个 3D 库的出发点是将这些知识建立起来，因此你不需要了解这些知识，你可以依赖这些库来解决。以上展示的是原生的 OpenGL 代码。其他的 3D 库与 three.js 的编写类似。但与 OpenGL ES 2.0＋ 和 WebGL 并不相同。

It seems misleading to call WebGL a 3D library. A user coming to WebGL will think "oh, 3D library. Cool. This will do 3D for me" and then find out the hard way that no, that's not the case at all.

将 WebGL 称为 3D 库似乎是在误导。一个用户看到 WebGL 会认为“喔，3D 库。酷。它会为我完成 3D 绘制”，然后痛苦的发现它并不是这样的。

We can even take it one step further. Here's drawing 3D wireframe cube in Canvas.

我们更进一步分析。这是在 Canvas 中绘制的 3D 线框图。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/resources/3d-in-canvas.html"></iframe>

[click here to open in a separate window][3] 

[点击这里在新窗口打开][3]

And here is drawing a wireframe cube in WebGL.
这是在 WebGL 中绘制的 3D 线框图。

<iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/resources/3d-in-webgl.html"></iframe>

[click here to open in a separate window][4] 

[点击这里在新窗口打开][4]

If you inspect the code you'll see there's not a whole lot of difference in terms of the amount of knowledge or for that matter even the code. Ultimately the Canvas version loops over the vertices, does the math WE SUPPLIED and draws some lines in 2D. The WebGL version does the same thing except the math WE SUPPLIED is in GLSL and executed by the GPU.

如果你查看了代码，你会发现从知识量和代码量上对比，并没有很大的差异。基本上 Canvas 在循环绘制顶点，根据我们提供的数学方法绘制 2D 线。而 WebGL 做了同样的事情，除了我们提供数学方法的是 GLSL 中内置的，程序是在 GPU 中执行的。

The point of this last demostration is to show that effectively WebGL is just a rasterization engine, similar to Canvas 2D. Sure WebGL does have features that help you implement 3D. WebGL has a depth buffer which makes depth sorting far easier than a system without. WebGL also has various math functions built in that are very useful for doing 3D math although there is argubly nothing that makes them 3D. They're a math library. You use them for math whether or not that math is 1D, 2D, 3D, whatever. But ultimately, WebGL only rasterizes. You have to provide it with clipspace coordinates that represent what you want drawn. Sure you provide a x,y,z,w and it divides by W before rendering but that's hardly enough to qualify WebGL as a 3D library. In the 3D librares you supply 3D data, the libraries take care of calculating clipspace points from 3D.

这一点以上示例有效的表明 WebGL 是一个栅格化引擎，类似于 Canvas 2D。当然 WebGL 也有实现 3D 的功能。WebGL 有一个深度缓冲，相比于没有深度缓冲的系统，深度排序更容易实现。WebGL 也有各种高效绘制 3D 图形的内置数学方法，绘制 3D 是无可争辩的。它们是一个数学库。无论 1D，2D 还是 3D 的数学你都可以使用它。但最终，WebGL 只是光栅化的 API。你必须提供需要绘制的有效的裁剪坐标。当然，你提供 x，y，z，w，WebGL 在渲染之前将它们各自除以 w ，但这难以将 WebGL 视为 3D 库。在 3D 库中，你提供 3D 数据，它们会这些 3D 数据转换成裁剪坐标。

I hope you at least understand where I'm coming from when I say WebGL is not a 3D library. I hope you'll also realize that a 3D library should handle the 3D for you. OpenGL did. Three.js does. OpenGL ES 2.0 and WebGL do not. Therefore they argubly don't belong in the same broad category of "3D libraries".

我希望你至少能够明白，我说的是 WebGL 而不是 3D 库。我也希望你能意识到，3D 库是能够像为你处理 3D 绘制的。例如 OpenGL 和 Three.js 就是 3D 库。而 OpenGL ES 2.0 和 WebGL 则不是。因此他们可以说是不属于同一广义范畴的“3D 库”。

The point of all of this is to give a developer that is new to WebGL an understanding of WebGL at its core. Knowing that WebGL is not a 3D library and that they have to provide all the knowledge themselves lets them know what's next for them and whether they want to pursue that 3D math knowledge or instead choose a 3D library to handle it for them. It also removes much of the mystery of how it works.

本文的重点是，让开发者重新认识 WebGL，理解 WebGL 的核心。明白 WebGL 并不是 3D 库，在 WebGL 开发中，开发者必须根据自身所拥有的知识，决定下一步做什么，或者是否继续，而 3D 数学知识或者 3D 库都能够帮助开发者解决这些问题。3D 数学知识和 3D 库同时也隐藏了许多的工作原理。

[1]: http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html
[2]: http://webglfundamentals.org/webgl/lessons/resources/three-js-cube-with-lights.html
[3]: http://webglfundamentals.org/webgl/lessons/resources/3d-in-canvas.html
[4]: http://webglfundamentals.org/webgl/lessons/resources/3d-in-webgl.html