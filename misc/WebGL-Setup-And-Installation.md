# WebGL Setup and Installation
# WebGL的设置与安装

Techincally you don't need anything other than a web browser to do WebGL development. Go to [jsfiddle.net][1] or [jsbin.com][2] or [codepen.io][3] and just start applying the lessons here.

从技术上讲，你仅仅需要一个网页浏览器做 WebGL 开发。可以到 [jsfiddle.net][1]、 [jsbin.com][2] 或者 [codepen.io][3] 应用这里的教程内容。

On all of them you can reference external scripts by adding a <script src="..."></script> tag pair if you want to use external scripts.

如果你想使用外部脚本，所有的内容你都可以通过引用外部脚本标签 <script src="..."> 来使用。

Still, there are limits. WebGL has stronger restrictions than Canvas2D for loading images which means you can't easily access images from around the web for your WebGL work. On top of that it's just faster to work with everything local.

不过，这里有限制。WebGL 相对于 Canvas2D 在加载图片方面有更强的约束，这意味着你在 WebGL 开发中不能轻易的访问来自网络的图片。最重要的是，它与本地资源合作会更快。

Let's assume you want to run and edit the samples on this site. The first thing you should do is download the site. [You can download it here][4].

让我们假设你想在这个网站运行和编辑示例代码。首先你应该下载这个网站。[你可以在这里下载][4]。

![download][5]

Unzip the files into some folder.
在文件夹中解压文件。

## Using a small simple easy Web Server
## 使用一个简单的 Web 服务器

Next up you should install a small web server. I know "web server" sounds scary but the truth is [web servers are actually extremely simple][6].

下一步你应该安装一个小的 web 服务器。我知道“web 服务器”听起来很吓人，但事实是 [web 服务器实际上非常简单][6]。

If you're on Chrome here's a simple solution. [Here's a small chrome extension that's a web server][7].

如果你用 Chrome，有一个简单的解决方法。[这里有一个小的 Web 服务器 chrome 扩展][7]。

![chrome extension][8]

Just point it at the folder where you unzipped the files and click one of the web server URLs.

只需点击解压了的文件夹中的文件，单击其中一个 web 服务器的 URL。

If you're not on chrome or if you don't want to use the extension another way is to use [node.js][9]. Download it, install it, then open a command prompt / console / terminal window. If you're on Windows the installer will add a special "Node Command Prompt" so use that.

如果你不是使用 chrome 或者你不想使用扩展，另一种方法是使用 [node.js][9]。下载，安装，然后在命令窗口/控制台/终端窗口中打开 。如果你在 Windows 中安装，将会添加一个特需的 “Node Command Prompt” 以供使用。

Then install the http-server by typing。

然后通过命令行输入安装 http-server。

```
npm -g install http-server
```

If you're on OSX use
如果你在 OSX 中使用

```
sudo npm -g install http-server
```

Once you've done that type

安装完成之后输入

```
http-server path/to/folder/where/you/unzipped/files
```

It should print something like

命令窗口应该打印出类似的东西

![http-server][10]

Then in your browser go to http://localhost:8080.

然后在你的浏览器中输入 http://localhost:8080。

If you don't specify a path then http-server will server the current folder.

如果你没有指定路径，http-server 将会在指向当前的文件夹。

## Using your Browsers Developer Tools
## 使用你的浏览器开发者工具

Most browser have extensive developer tools built in.
大多数的浏览器都内置了开发者工具。

![developer tools][11]

[Docs for Chrome's are here][12], [Firefox's are here][13].
Chrome 的参考文档在这里，Firefox 的在这里。

Learn how to use them. If nothing else always check the JavaScript console. If there is an issue it will often have an error message. Read the error message closely and you should get a clue where the issue is.

学习如果使用它们。如果没有其它的事一直检查 JavaScript 控制台。如果有问题，它会显示错误信息。仔细阅读错误信息，你会得到问题的线索。

![chrome console][14]

## WebGL Helpers
## WebGL 助手

There are various WebGL Inspectors / Helpers. [Here's one for Chrome][15].

这里有各种各样的 WebGL 检验员/助手。[这个是 Chrome 的][15]。

![WebGL Inspectors / Helper][16]

They may or may not be helpful. Most of them are designed for animated samples and will capture a frame and let you see all the WebGL calls that made that frame. That's great if you already have something working or if you had something working and it broke. But it's not so great if your issue is during initialization which they don't catch or if you're not using animation, as in drawing something every frame. Still they can be very useful. I'll often click on a draw call, and check the uniforms. If I see a bunch of NaN (NaN = Not a Number) then I can usually track down the code that set that uniform and find the bug.

他们可能会也可能不会有帮助。它们中的大多数是专为动画示例的，它们捕获并展示一个调用 WebGL 的框。如果你已经有了一些示例起效了或者有一些示例起效但又失败了，这很好。如果你的问题是在初始化过程中助手不能捕获框架或者是你在绘制框架时没有使用动画，这就不是很好了。但它们任然可以非常有用的。我经常会点击一个调用，以检查 uniforms。如果我看到一堆 NaN（NaN = Not a Number）我就可以跟踪代码，设置 uniform 并找出错误。

## Inspect the Code
## 检查代码

Also always remember you can inspect the code. You can usually just pick view source

永远记住你可以检查代码。你可以选择只查看源码。

![helper][17]

Even if you can't right click a page or if the source is in a separate file you can always view the source in the devtools

即使你不能右击页面或者源码是在一个单独的文件中，你仍然可以在开发者工具中查看源码。

![devtools][18]

## Get Started
## 开始

Hopefully that helps you get started. [Now back to the lessons][19].

希望这有助于你开始。[现在回到教程][19]。

[1]: https://jsfiddle.net/
[2]: http://jsbin.com/
[3]: http://codepen.io/
[4]: https://github.com/greggman/webgl-fundamentals/
[5]: http://webglfundamentals.org/webgl/lessons/resources/download-webglfundamentals.gif
[6]: http://games.greggman.com/game/saving-and-loading-files-in-a-web-page/
[7]: https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb?hl=en
[8]: http://webglfundamentals.org/webgl/lessons/resources/chrome-webserver.png
[9]: https://nodejs.org/en/
[10]: http://webglfundamentals.org/webgl/lessons/resources/http-server-response.png
[11]: http://webglfundamentals.org/webgl/lessons/resources/chrome-devtools.png
[12]: https://developers.google.com/web/tools/chrome-devtools/
[13]: https://developer.mozilla.org/en-US/docs/Tools
[14]: http://webglfundamentals.org/webgl/lessons/resources/javascript-console.gif
[15]: https://benvanik.github.io/WebGL-Inspector/
[16]: https://benvanik.github.io/WebGL-Inspector/images/screenshots/1-Trace.gif
[17]: http://webglfundamentals.org/webgl/lessons/resources/view-source.gif
[18]: http://webglfundamentals.org/webgl/lessons/resources/devtools-source.gif
[19]: http://webglfundamentals.org/