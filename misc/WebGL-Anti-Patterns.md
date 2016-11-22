# WebGL Anti-Patterns
# WebGL ����ģʽ

This is a list of anti patterns for WebGL. Anti patterns are things you should avoid doing

�����б��� WebGL ����ģʽ�����ӡ�Ӧ�ñ���ʹ��

1.	Putting `viewportWidth` and `viewportHeight` on the `WebGLRenderingContext`

    Some code adds properties for their viewport width and height and sticks them on the `WebGLRenderingContext` something like this


1. ��`viewportWidth`��`viewportHeight`�ŵ�`WebGLRenderingContext`��
    
    ��Щ���룬��Ϊ�Ӵ��Ŀ�����һЩ���ԣ��������������ŵ�`WebGLRenderingContext`��
    
    ```
    gl = canvas.getContext("webgl");
    gl.viewportWidth = canvas.width;    // BAD!!!
    gl.viewportHeight = canvas.height;  // BAD!!!
    ```
    
    Then later they might do something like this
    
    ���ſ��ܻ�����
    
    ```
    gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
    ```
    
    **Why it's Bad:**
    
    **ȱ��**
    
    It's objectively bad because you now have 2 properties that need to be updated anytime you change the size of the canvas. For example if you change the size of the canvas when the user resizes the window `gl.viewportWidth` & `gl.viewportHeight` will be wrong unless you set them again.
    
    �͹��ϣ����ı� canvas �ߴ磬����Ҫ���� 2 �����ԡ����磬���û���������ʱ���ı��� canvas �ĳߴ磬������������`gl.viewportWidth` ��`gl.viewportHeight`����Ȼ���ǻ����
    
    It's subjectively bad because any new WebGL programmer will glance at your code and likely think `gl.viewportWidth` and `gl.viewportHeight` are part of the WebGL spec, confusing them for months.
    
    �����ϣ��κ�һ���µ� WebGL �����߿�����Ĵ��룬���ܻ���Ϊ`gl.viewportWidth`��`gl.viewportHeight`�� WebGL �淶��һ���֣����ڽ������ļ������н����ǻ�����
    
    **What to do instead:**
    
    **�������**
    
    Why make more work for yourself? The WebGL context has its width and height directly on it. Just use that.
    
    ΪʲôҪ���һ�٣�WebGL �������б��������ߡ�ֱ��ʹ�þͺ��ˡ�
    
    ```
    // When you need to set the viewport to match the size of the canvas's
    // drawingBuffer this will always be correct
    // ������Ҫ���� viewport ʹ���� canvas �� drawingBuffer ��ƥ��ʱ����������������ȷ�ġ�
    
    gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
    ```
    Even better it will handle extreme cases whereas using `gl.canvas.width` and `gl.canvas.height` will not. [As for why see here][1].
    
    ���õ��ǣ����ܹ�����һЩʹ��`gl.canvas.width`��`gl.canvas.height`���ܴ���ļ��������[����ԭ����ο�����][1]
    
2.	Using `canvas.width` and `canvas.height` for aspect ratio


2. ʹ��`canvas.width`��`canvas.height`�����ݺ��

    Often code uses `canvas.width` and `canvas.height` for aspect ratio like this
    
    ͨ��������ʹ��`canvas.width`��`canvas.height`�����ݺ��
    
    ```
    var aspect = canvas.width / canvas.height;
    perspective(fieldOfView, aspect, zNear, zFar);
    ```
    
    **Why it's Bad:**
    
    **ȱ��**
    
    The width and height of the canvas have nothing to do with the size the canvas is displayed. CSS controls the size the canvas is displayed.
    
    canvas �Ŀ���� canvas �Ĵ�С�޹ء����� canvas ��С���� CSS��
    
    **What to do instead:**
    
    **�������**
    
    Use `canvas.clientWidth` and `canvas.clientHeight`. Those values tell you what size your canvas is actually being displayed on the screen. Using those values you'll always get the correct aspect ratio regardless of your CSS settings.
    
    ʹ��`canvas.clientWidth`��`canvas.clientHeight`�����Ǳ����� canvas ����Ļ�ϵ�ʵ�ʳߴ硣ʹ�����ǣ����� CSS ������ã����ܵõ���ȷ���ݺ�ȡ�
    
    ```
    var aspect = canvas.clientWidth / canvas.clientHeight;
    perspective(projectionMatrix, fieldOfView, aspect, zNear, zFar);
    ```

    Here are examples of a canvas that's the same size (`width="400" height="300"`) but using CSS we've told the browser to display the canvas a different size. Notice the samples all display the 'F' in the correct aspect ratio.
    
    ���������У�canvas ������ͬ�ĳߴ磨`width="400" height="300"`��������ͨ�� CSS �����������ʾ��ͬ�ߴ�� canvas ��ע�����е����ӵ� ��F�� ���ݺ��������ȷ��ʾ�ˡ�

    <iframe class="webgl_example " style="width: 150px; height: 200px;" src="/webgl/lessons/../webgl-canvas-clientwidth-clientheight.html"></iframe>

    <iframe class="webgl_example " style="width: 400px; height: 150px;" src="/webgl/lessons/../webgl-canvas-clientwidth-clientheight.html"></iframe>

    If we had used `canvas.width` and `canvas.height` that would not be true.
    
    �������ʹ��`canvas.width`��`canvas.height`������Ͳ��������ˡ�

    <iframe class="webgl_example " style="width: 150px; height: 200px;" src="/webgl/lessons/../webgl-canvas-width-height.html"></iframe> 

    <iframe class="webgl_example " style="width: 400px; height: 150px;" src="/webgl/lessons/../webgl-canvas-width-height.html"></iframe>
    
3.	Using `window.innerWidth` and `window.innerHeight` to compute anything
Many WebGL programs use window.innerWidth and window.innerHeight in many places. For example:


3. ʹ��`window.innerWidth`��`window.innerHeight`����

    ��� WebGL ���������ط�ʹ���� window.innerWidth �� window.innerHeight�����磺

    ```
    canvas.width = window.innerWidth;                    // BAD!!
    canvas.height = window.hinnerHeight;                 // BAD!!
    ```
    
    **Why it's Bad:**
    
    **ȱ��**
    
    It's not portable. Yes, it can work for WebGL pages where you want to make the canvas fill the screen. The problem comes when you don't. Maybe you decide to make an article like these tutorials where your canvas is just some small diagram in a larger page. Or maybe you need some property editor on the side or a score for a game. Sure you can fix your code to handle those cases but why not just write it so it works in those cases in the first place? Then you won't have to go change any code when you copy it to a new project or use an old project in a new way.
    
    ����������û�����⡣�ǵģ��� WebGL ҳ�����������ܹ��� canvas ������Ļ����֮���������Ļʱ��������ˡ����������дһƪ����Щ�̳�һ�������£����ڴ���ҳ���У���� canvas ֻ��һЩСС��ͼ�������������Ҫ��һ�߷������Ա༭������ʾ��Ϸ��������Ȼ������Ե�����Ĵ�����������Щ���ӣ���Ϊʲô��ֻ�ڿ�ʼ�ĵط���д�أ���ô�㽫���Ǹ��Ƶ�һ���µ���Ŀ���ھɵ���Ŀ��ʹ���·���ʱ���Ͳ���Ҫ�ı��κδ����ˡ�
    
    **What to do instead:**
    
    **�������**
    
    Instead of fighting the Web platform, use the Web platform as it was designed to be used. Use CSS and `clientWidth` and `clientHeight`.
    
    ��������ƽ̨����Ƶ�ʹ�����������ǶԿ�����ʹ�� CSS ��`clientWidth`��`clientHeight`��
    
    ```
    var width = gl.canvas.clientWidth;
    var height = gl.canvas.clientHeight;

    gl.canvas.width = width;
    gl.canvas.height = height;
    ```
    
    Here are 9 cases. They all use exactly the same code. Notice that none of them reference `window.innerWidth` nor `window.innerHeight`.
    
    ������ 9 �����ӡ����Ƕ���ʹ����ȫһ���Ĵ��롣ע�����Ƕ�û������`window.innerWidth`����`window.innerHeight`��
    
    [A page with nothing but a canvas using CSS to make it fullscreen][2]

    [A page with a canvas set to using 70% width so there is room for editor controls][3]
    
    [A page with a canvas embedded in a paragraph][4]

    [A page with a canvas embedded in a paragraph using `box-sizing: border-box`][5];
    
    `box-sizing: border-box;` makes borders and padding take space from the element they're defined on rather than outside it. In other words, in normal box-sizing mode a 400x300 pixel element with 15 pixel border has a 400x300 pixel content space surrounded by a 15 pixel border making its total size 430x330 pixels. In box-sizing: border-box mode the border goes on the inside so that same element would stay 400x300 pixels, the content would end up being 370x270. This is yet another reason why using `clientWidth` and `clientHeight` is so important. If you set the border to say 1em you'd have no way of knowing what size your canvas will turn out. It would be different with different fonts on different machines or different browsers.
    
    `box-sizing:border-box`��ʹ borders �� padding ���ڶ���������ڲ��������ⲿ�����仰˵���ڲ�ͬ�� box-sizing ģʽ�У�һ������Ϊ 400x300���ز����� 15 ���ر߿��Ԫ�أ���һ�� 400x300 ���ص���������������� 15 ���صı߿��Χ��ʹ��Ԫ���ܴ�СΪ 430x330 ���ء��� box-sizing: border-box ģʽ�У��߿�λ���ڲ������Ԫ�ر���Ϊ 400x300 ���ش�С�������������մ�СΪ 370x270 ����Ҳ��ΪʲôҪʹ��`clientWidth`��`clientHeight`����һ����Ҫԭ������㽫�߿�����Ϊ 1em ����û�а취֪�� canvas ���յĴ�С�����ڲ�ͬ�����壬��������������ǲ�ͬ�ġ�
    
    [A page with nothing but a container using CSS to make it fullscreen into which the code will insert a canvas][6]
    
    [A page with a container set to using 70% width so there is room for editor controls into which the code will insert a canvas][7]

    [A page with a container embedded in a paragraph into which the code will insert a canvas][8]
    
    [A page with a container embedded in a paragraph using box-sizing: border-box; into which the code will insert a canvas][9]
    
    [A page with no elements with CSS setup to make it fullscreen into which the code will insert a canvas][10]
    
    Again, the point is, if you embrace the web and write your code using the techniques above you won't have to change any code when you run into different use cases.

4.	Using the `'resize'` event to change the size of your canvas.


4. ʹ��`��resize��`�¼����ı� canvas �Ĵ�С��

    Some apps check for the window 'resize' event like this to `resize` their canvas.
    
    һЩ app ͨ������ window �� ��resize�� �¼��� ��resize�� canvas �ĳߴ硣
    
    ```
    window.addEventListener('resize', resizeTheCanvas);
    ```
    or this
    
    ����
    
    ```
    window.onresize = resizeTheCanvas;
    ```
    
    **Why it's Bad:**
    
    **ȱ��**
    
    It's not bad per se, rather, for most WebGL programs it fits less use cases. Specifically `'resize'` only works when the window is resized. It doesn't work if the canvas is resized for some other reason. For example let's say you're making a 3D editor. You have your canvas on the left and your settings on the right. You've made it so there's a draggable bar separating the 2 parts and you can drag that bar to make the settings area larger or smaller. In this case you won't get any `'resize'` events. Similarly if you've got a page where other content gets added or removed and the canvas changes size as the browser re-lays out the page you won't get a resize event.
    
    �������������ǲ��ã�Ȼ�������ڴ���� WebGL ���򲢲��ʺϡ�������`��resize��`ֻ���� window ���µ�����Сʱ�Ż������á��������ʹ canvas �ı��С���¼���������Ч�����磬�����ڱ༭һ�� 3D �༭����canvas λ����ߣ��༭��λ���ұߡ��༭�������һ���϶�����������϶�����������С������������У��������κ�`��resize��`�¼�������ͬ���������һ��ҳ���У���ӻ���ɾ��һЩ���������� canvas �ı��˴�С��ʹ�������������resize �¼�Ҳ���ᱻ������
    
    **What to do instead:**
    
    **�������**
    
    Like many of the solutions to anti-patterns above there's a way to write your code so it just works for most cases. For WebGL apps that constantly draw every frame the solution is to check if you need to resize every time you draw like this
    
    ͬ��������ķ���ģʽ�Ľ������һ������һ�����������ڴ����������� WebGL Ӧ���У������ػ���ÿһ֡������Ƿ���Ҫ���µ�����
    
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
    
    �����κ�����£���Ļ��������ŵ���ȷ�Ĵ�С���ڲ�ͬ������¶�����Ҫ�ı���롣����ʹ������� 3 �����ӵĴ��룬��һ���൱��ı༭����
    
    <iframe class="webgl_example" style=" " src="/webgl/resources/editor.html?url=/webgl/lessons/../webgl-same-code-resize.html"></iframe> 

    [click here to open in a separate window][11]
    
    [����������´��ڴ�][11]
 
    There would be no resize events for this case nor any other where the canvas gets resized based on the size of other dynamic elements on the page.
    
    ���������û�� resize �¼���canvas Ҳ��������ҳ���е�����Ԫ�ض�̬������С��
    
    For WebGL apps that don't re-draw every frame the code above is still correct, you'll just need to trigger a re-draw in every case where the canvas can possibly get resized. One easy way to do that would be to setup a requestAnimationFrame loop like this.
    
    ���� WebGL Ӧ�ã�û�����»���ÿһ֡�����ϵĴ�����Ȼ����ȷ�ģ�ֻ��Ҫ�ڿ��ܻ����µ��� canvas ��С������´����ػ��¼���һ���򵥵ķ���������������һ�� requestAnimationFrame ѭ����
    
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
    
    ����ֻ���� canvas ��Ҫ���µ�������`needToRender`Ϊ true ʱ�ػ档������ΪӦ�ó���������¼���������ÿһ֡���ػ档�κ�ʱ�򣬸ı��˳����еĶ�����������ϣ����Ⱦ��Щ�ı䣬ֻ��Ҫ����`needToRender`��
    
5.	Adding properties to `WebGLObjects`


5. Ϊ`webGLObjects`�������

    `WebGLObjects` are the various types of resources in WebGL like a `WebGLBuffer` or `WebGLTexture`. Some apps add properties to those objects. For example code like this:
    
    `webGLObjects`ָ������ WebGL �еĸ������ͣ���`webGLBuffer`��`webGLTexture`��һЩӦ�ý�������ӵ���Щ�����ϡ��������´��룺
    
    ```
    var buffer = gl.createBuffer();
    buffer.itemSize = 3;        // BAD!!
    buffer.numComponents = 75;  // BAD!!

    var program = gl.createProgram();
    ...
    program.u_matrixLoc = gl.getUniformLocation(program, "u_matrix");  // BAD!!
    ```
    
    **Why it's Bad:**
    **ȱ��**
    
    The reason this is bad is that WebGL can "lose the context". This can happen for any reason but the most common reason is if the browser decides too many GPU resources are being used it might intentionally lose the context on some `WebGLRenderingContexts` to free up space. WebGL programs that want to always work have to handle this. Google Maps handles this for example.
    
    ����ԭ���� WebGL �ᡰ��ʧ�����ġ����кܶ�ԭ��ᵼ��������⣬�������ԭ���ǣ���������ʹ���˹���� GPU ��Դ�������ܻ���һЩ`webGLRenderingContexts`�������ض�ʧ�����ģ����ͷſռ䡣WebGL ������봦������������һֱ������ȥ��Google Maps ��������֮һ��
    
    The problem with the code above is that when the context is lost the WebGL creation functions like `gl.createBuffer()` above will return `null`. That effectively makes the code this
    
    ���ϴ���������ǣ��������Ķ�ʧ�ˣ�webGL ������`gl.createBuffer()`����ʱ�����᷵�� null������ʵ������������
    
    ```
    var buffer = null;
    buffer.itemSize = 3;        // ERROR!
    buffer.numComponents = 75;  // ERROR!
    ```
    
    That will likely kill your app with an error like
    
    �͵�ͬ�����µĴ�����ֹ��Ӧ��
    
    ```
    TypeError: Cannot set property 'itemSize' of null
    ```
    
    While many apps don't care if they die when the context is lost it seems like a bad idea to write code that will have to be fixed later if the developers ever decide to update their app to handle context lost events.
    
    �ܶ�Ӧ�ó����У��ڱ�д����ʱ�����Ķ�ʧ�����ģ������Ƿ��ҵ������ƺ���һ���������⣬��������߾�������Ӧ�ó���������ʧ�����ĵ����飬�Ͳ��ò��޸���Щ���롣
    
    **What to do instead:**
    
    **�������**
    
    If you want to keep `WebGLObjects` and some info about them together one way would be to use JavaScript objects. For example:
    
    �����ϣ����`webGLObjects`��һЩ��Ϣ������һ��һ�ַ�����ʹ�� JavaScreip �������磺
    
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
    
    �͸��˶��ԣ��ҽ���[����һЩ������ʹ��д WebGL ���Ӽ�][12]
    
Those are a few of what I consider WebGL Anti-Patterns in code I've seen around the net. Hopefully I've made the case why to avoid them and given solutions that are easy and useful.

��Щ�����������Ͽ���������Ϊ�� WebGL ����ģʽ�Ĵ��롣ϣ�����г�����Щ����ԭ��ͽ���������Ǽ򵥲������õġ�

-----

## What is drawingBufferWidth and drawingBufferHeight?

## ʲô�� drawingBufferWidth �� drawingBufferHeight ��

GPUs have a limit on how big a rectangle of pixels (texture, renderbuffer) they can support. Often this size is the next power of 2 larger than whatever a common monitor resolution was at the time the GPU was made. For example if the GPU was designed to support 1280x1024 screens it might have a size limit of 2048. If it was designed for 2560x1600 screens it might have a limit of 4096. 

GPUs ��������֧�ֵľ������������أ�������Ⱦ���Ĵ�С�������Сͨ��Ϊ���� GPU ʹ�õ���ʾ���ֱ��ʵ���С�� 2 ���ݡ�

That seems reasonable but what happens if you have multiple monitors? Let's say I have a GPU with a limit of 2048 but I have two 1920x1080 monitors. The user opens a browser window with a WebGL page, they then stretch that window across both monitors. Your code tries to set the `canvas.width` to `canvas.clientWidth` which in this case is 3840. What should happen? 

���ƺ��Ǻ���ģ�������ж����ʾ���ᷢ��ʲô�أ���� GPU ����Ϊ 2048���������� 1920x1080 ����ʾ�����û�������������д�һ�� WebGL ҳ�棬�����������絽������ʾ�������볢�Խ�`canvas.width`����Ϊ`canvas.clientWidth`��Ҳ���� 3840 ���أ��ᷢ��ʲô��

Off the top of my head there are only 3 options

���Ժ����� 3 ��ѡ��

1.	Throw an exception.


1. �׳�һ���쳣

    That seems bad. Most web apps won't be checking for it and the app wil crash. If the app had user data in it the user just lost their data
    
    ���ƺ�����⡣������� web Ӧ�ó��򲻻���������������������ᵼ�³������������ڼ���򱣴����û����ݣ����ݽ���ʧ��
    
2.	Limit the size of the canvas to the GPUs limit


2. canvas ������Ϊ GPUs �ļ���

    The problem with this solution is it will also likely lead to a crash or possibly a messed up webpage because the code expects the canvas to be the size they requested and they expect other parts of the UI and elements on the page to be in the proper places.
    
    �����������������ǣ���Ҳ���ܵ��³������������ҳ���ң�����ϣ�� canvas ��СΪ������Ҫ��ģ�ͬʱҲϣ��ҳ���������� UI �� Ԫ�����ʵ��ĵط���
    
3.	Let the canvas be the size the user requested but make its drawingbuffer the limit


3. canvas Ϊ�û���Ҫ��Ĵ�С�������� drawingbuffer �Ĵ�С

    This is the solution WebGL uses. If your code is written correctly the only thing the user might notice is the image in the canvas is being scaled slightly. Otherwise it just works. In the worst case most WebGL programs that don't do the right thing will just have a slightly off display but if the user sizes the window back down things will return to normal.
    
    ���� WebGL ʹ�õĽ�������������д�Ĵ�����ȷ����ô�û�Ψһע�⵽�ģ��� canvas �е�ͼƬ��΢�Ŵ��ˡ������ճ����С��������£������ WebGL ����ֻ�ǲ���ʾ���棬������û������ڵ�����������С������ָ�������
    
Most people don't have multiple monitors so this issue rarely comes up. Or at least it used to. Chrome and Safari, at least as of January 2015, had a hardcoded limit on canvas size of 4096. Apple's 5k iMac is past that limit. Lots of WebGL apps were having strange displays because of this. Similarly many people have started using WebGL with multiple monitors for installation work and have been hitting this limit.

������˶�����ʹ�ö����ʾƵ���������������ٳ��֡�����������������Chrome ��Safari�������� 2015 �� 1 ���𣬶� canvas �Ĵ�Сǿ�������� 4096 ���ڡ�Apple�� 5k iMac ������������ơ���� WebGL Ӧ�ó�����ʾ���춼����Ϊ���ԭ��ͬ����������Ѿ���ʼ�ڹ����н� WebGL �����ڶ����ʾ���У����Ҵﵽ�˼��ޡ�

So, if you want to handle these cases use `gl.drawingBufferWidth` and `gl.drawingBufferHeight` as shown in #1 above. For most apps if you follow the best practices above things will just work. Be aware though if you are doing calculations that need to know the actual size of the drawingbuffer you need to take that into account. Examples off the top of my head, picking, in other words converting from mouse coordinates into canvas pixel coordinates. Another would be any kind of post processing effects that want to know the actual size of the drawingbuffer. 

��ˣ������ϣ�������Щ���⣬�����һ����������ʹ��`gl.drawingBufferWidth`��`gl.drawingBufferHeight`���������ѭ���ϵ���õ����������ڴ����Ӧ�ó�������Ч�ġ�Ҫע����ǣ����������ʱ����Ҫ���ǵ� drawingbuffer ��ʵ�ʳߴ硣ѡ�����Ժ��е����ӣ�����֮�����������ת��Ϊ�������ꡣ������֪�� drawingbuffer �������κ����͵Ĵ�С��

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

