你好，我是月影。今天，我们要讲WebGL。

WebGL是最后一个和可视化有关的图形系统，也是最难学的一个。为啥说它难学呢？我觉得这主要有两个原因。第一，WebGL这种技术本身就是用来解决最复杂的视觉呈现的。比如说，大批量绘制复杂图形和3D模型，这类比较有难度的问题就适合用WebGL来解决。第二，WebGL相对于其他图形系统来说，是一个更“开放”的系统。

我说的“开放”是针对于底层机制而言的。因为，不管是HTML/CSS、SVG还是Canvas，都主要是使用其API来绘制图形的，所以我们不必关心它们具体的底层机制。也就是说，我们只要理解创建SVG元素的绘图声明，学会执行Canvas对应的绘图指令，能够将图形输出，这就够了。但是，要使用WebGL绘图，我们必须要深入细节里。换句话说就是，我们必须要和内存、GPU打交道，真正控制图形输出的每一个细节。

所以，想要学好WebGL，我们必须先理解一些基本概念和原理。那今天这一节课，我会从图形系统的绘图原理开始讲起，主要来讲WebGL最基础的概念，包括GPU、渲染管线、着色器。然后，我会带你用WebGL绘制一个简单的几何图形。希望通过这个可视化的例子，能够帮助你理解WebGL绘制图形的基本原理，打好绘图的基础。

## 图形系统是如何绘图的？

首先，我们来说说计算机图形系统的主要组成部分，以及它们在绘图过程中的作用。知道了这些，我们就能很容易理解计算机图形系统绘图的基本原理了。

一个通用计算机图形系统主要包括6个部分，分别是输入设备、中央处理单元、图形处理单元、存储器、帧缓存和输出设备。虽然我下面给出了绘图过程的示意图，不过这些设备在可视化中的作用，我要再跟你多啰嗦几句。

- **光栅**（Raster）：几乎所有的现代图形系统都是基于光栅来绘制图形的，光栅就是指构成图像的像素阵列。
- **像素**（Pixel）：一个像素对应图像上的一个点，它通常保存图像上的某个具体位置的颜色等信息。
- **帧缓存**（Frame Buffer）：在绘图过程中，像素信息被存放于帧缓存中，帧缓存是一块内存地址。
- **CPU**（Central Processing Unit）：中央处理单元，负责逻辑计算。
- **GPU**（Graphics Processing Unit）：图形处理单元，负责图形计算。

![](https://static001.geekbang.org/resource/image/b5/56/b5e4f37e1c4fbyy6a2ea10624d143356.jpg?wh=1920%2A825)

知道了这些概念，我带你来看一个典型的绘图过程，帮你来明晰一下这些概念的实际用途。

首先，数据经过CPU处理，成为具有特定结构的几何信息。然后，这些信息会被送到GPU中进行处理。在GPU中要经过两个步骤生成光栅信息。这些光栅信息会输出到帧缓存中，最后渲染到屏幕上。

![](https://static001.geekbang.org/resource/image/9f/46/9f7d76cc9126036ef966dc236df01c46.jpeg?wh=1920%2A1080 "图形数据经过GPU处理最终输出到屏幕上")

这个绘图过程是现代计算机中任意一种图形系统处理图形的通用过程。它主要做了两件事，一是对给定的数据结合绘图的场景要素（例如相机、光源、遮挡物体等等）进行计算，最终将图形变为屏幕空间的2D坐标。二是为屏幕空间的每个像素点进行着色，把最终完成的图形输出到显示设备上。这整个过程是一步一步进行的，前一步的输出就是后一步的输入，所以我们也把这个过程叫做**渲染管线**（RenderPipelines）。

在这个过程中，CPU与GPU是最核心的两个处理单元，它们参与了计算的过程。CPU我相信你已经比较熟悉了，但是GPU又是什么呢？别着急，听我慢慢和你讲。

## GPU是什么？

CPU和GPU都属于处理单元，但是结构不同。形象点来说，CPU就像个大的工业管道，等待处理的任务就像是依次通过这个管道的货物。一条CPU流水线串行处理这些任务的速度，取决于CPU（管道）的处理能力。

实际上，一个计算机系统会有很多条CPU流水线，而且任何一个任务都可以随机地通过任意一个流水线，这样计算机就能够并行处理多个任务了。这样的一条流水线就是我们常说的**线程**（Thread）。

[![](https://static001.geekbang.org/resource/image/1e/80/1e6479ef37138f051b7a6e5de6977580.jpeg?wh=1920%2A615 "CPU")](https://thebookofshaders.com/)

这样的结构用来处理大型任务是足够的，但是要处理图像应用就不太合适了。这是因为，处理图像应用，实际上就是在处理计算图片上的每一个像素点的颜色和其他信息。每处理一个像素点就相当于完成了一个简单的任务，而一个图片应用又是由成千上万个像素点组成的，所以，我们需要在同一时间处理成千上万个小任务。

要处理这么多的小任务，比起使用若干个强大的CPU，使用更小、更多的处理单元，是一种更好的处理方式。而GPU就是这样的处理单元。

[![](https://static001.geekbang.org/resource/image/1a/e7/1ab1116e3742611f5cb26c942d67d5e7.jpeg?wh=1920%2A1080 "GPU")](https://thebookofshaders.com/)

GPU是由大量的小型处理单元构成的，它可能远远没有CPU那么强大，但胜在数量众多，可以保证每个单元处理一个简单的任务。即使我们要处理一张800 * 600大小的图片，GPU也可以保证这48万个像素点分别对应一个小单元，这样我们就可以**同时**对每个像素点进行计算了。

那GPU究竟是怎么完成像素点计算的呢？这就必须要和WebGL的绘图过程结合起来说了。

## 如何用WebGL绘制三角形？

浏览器提供的WebGL API是OpenGL ES的JavaScript绑定版本，它赋予了开发者操作GPU的能力。这一特点也让WebGL的绘图方式和其他图形系统的“开箱即用”（直接调用绘图指令或者创建图形元素就可以完成绘图）的绘图方式完全不同，甚至要复杂得多。我们可以总结为以下5个步骤：

1. 创建WebGL上下文
2. 创建WebGL程序（WebGL Program）
3. 将数据存入缓冲区
4. 将缓冲区数据读取到GPU
5. GPU执行WebGL程序，输出结果

别看这些步骤看起来很简单，但其中会涉及许多你没听过的新概念、方法以及各种参数。不过，这也不用担心，我们今天的重点还是放在理解WebGL的基本用法和绘制原理上，对于新的方法具体怎么用，参数如何设置，这些我们都会在后面的课程中详细来讲。

接下来，我们就用一个绘制三角形的例子，来讲一下这些步骤的具体操作过程。

### 步骤一：创建WebGL上下文

创建WebGL上下文这一步和Canvas2D的使用几乎一样，我们只要调用canvas元素的getContext即可，区别是将参数从’2d’换成’webgl’。

```
const canvas = document.querySelector('canvas');
const gl = canvas.getContext('webgl');
```

不过，有了WebGL上下文对象之后，我们并不能像使用Canvas2D的上下文那样，调用几个绘图指令就把图形画出来，还需要做很多工作。别着急，让我们一步一步来。

### 步骤二：创建WebGL程序

接下来，我们要创建一个WebGL程序。你可能会觉得奇怪，我们不是正在写一个绘制三角形的程序吗？为什么这里又要创建一个WebGL程序呢？实际上，这里的WebGL程序是一个WebGLProgram对象，它是给GPU最终运行着色器的程序，而不是我们正在写的三角形的JavaScript程序。好了，解决了这个疑问，我们就正式开始创建一个WebGL程序吧！

首先，要创建这个WebGL程序，我们需要编写两个**着色器**（Shader）。着色器是用GLSL这种编程语言编写的代码片段，这里我们先不用过多纠结于GLSL语言，在后续的课程中我们会详细讲解。那在这里，我们只需要理解绘制三角形的这两个着色器的作用就可以了。

```
const vertex = `
  attribute vec2 position;

  void main() {
    gl_PointSize = 1.0;
    gl_Position = vec4(position, 1.0, 1.0);
  }
`;


const fragment = `
  precision mediump float;

  void main()
  {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
  }    
`;
```

那我们为什么要创建两个着色器呢？这就需要我们先来理解**顶点和图元**这两个基本概念了。在绘图的时候，WebGL是以顶点和图元来描述图形几何信息的。顶点就是几何图形的顶点，比如，三角形有三个顶点，四边形有四个顶点。图元是WebGL可直接处理的图形单元，由WebGL的绘图模式决定，有点、线、三角形等等。

所以，顶点和图元是绘图过程中必不可少的。因此，WebGL绘制一个图形的过程，一般需要用到两段着色器，一段叫**顶点着色器**（Vertex Shader）负责处理图形的顶点信息，另一段叫**片元着色器**（Fragment Shader）负责处理图形的像素信息。

更具体点来说，我们可以把**顶点着色器理解为处理顶点的GPU程序代码。它可以改变顶点的信息**（如顶点的坐标、法线方向、材质等等），从而改变我们绘制出来的图形的形状或者大小等等。

顶点处理完成之后，WebGL就会根据顶点和绘图模式指定的图元，计算出需要着色的像素点，然后对它们执行片元着色器程序。简单来说，就是对指定图元中的像素点着色。

WebGL从顶点着色器和图元提取像素点给片元着色器执行代码的过程，就是我们前面说的生成光栅信息的过程，我们也叫它光栅化过程。所以，**片元着色器的作用，就是处理光栅化后的像素信息。**

这么说可能比较抽象，我 来举个例子。我们可以将图元设为线段，那么片元着色器就会处理顶点之间的线段上的像素点信息，这样画出来的图形就是空心的。而如果我们把图元设为三角形，那么片元着色器就会处理三角形内部的所有像素点，这样画出来的图形就是实心的。

![](https://static001.geekbang.org/resource/image/6c/6e/6c4390eb21e653274db092a9ba71946e.jpg?wh=1726%2A904)

这里你要注意一点，因为图元是WebGL可以直接处理的图形单元，所以其他非图元的图形最终必须要转换为图元才可以被WebGL处理。举个例子，如果我们要绘制实心的四边形，我们就需要将四边形拆分成两个三角形，再交给WebGL分别绘制出来。

好了，那让我们回到片元着色器对像素点着色的过程。你还要注意，这个过程是并行的。也就是说，**无论有多少个像素点，片元着色器都可以同时处理。**这也是片元着色器一大特点。

以上就是片元着色器的作用和使用特点了，关于顶点着色器的作用我们一会儿再说。说了这么多，你可别忘了，创建着色器的目的是为了创建WebGL程序，那我们应该如何用顶点着色器和片元着色器代码，来创建WebGL程序呢？

首先，因为在JavaScript中，顶点着色器和片元着色器只是一段代码片段，所以我们要将它们分别创建成shader对象。代码如下所示：

```
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertex);
gl.compileShader(vertexShader);


const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragment);
gl.compileShader(fragmentShader);
```

接着，我们创建WebGLProgram对象，并将这两个shader关联到这个WebGL程序上。WebGLProgram对象的创建过程主要是添加vertexShader和fragmentShader，然后将这个WebGLProgram对象链接到WebGL上下文对象上。代码如下：

```
const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
```

最后，我们要通过useProgram选择启用这个WebGLProgram对象。这样，当我们绘制图形时，GPU就会执行我们通过WebGLProgram设定的 两个shader程序了。

```
gl.useProgram(program);
```

好了，现在我们已经创建并完成WebGL程序的配置。接下来， 我们只要将三角形的数据存入缓冲区，也就能将这些数据送入GPU了。那实现这一步之前呢，我们先来认识一下WebGL的坐标系。

### 步骤三：将数据存入缓冲区

我们要知道WebGL的坐标系是一个三维空间坐标系，坐标原点是（0,0,0）。其中，x轴朝右，y轴朝上，z轴朝外。这是一个右手坐标系。

![](https://static001.geekbang.org/resource/image/yy/b1/yy3e873beb7743096e3cc7b641e718b1.jpeg?wh=1920%2A1080)

假设，我们要在这个坐标系上显示一个顶点坐标分别是（-1, -1）、（1, -1）、（0, 1）的三角形，如下图所示。因为这个三角形是二维的，所以我们可以直接忽略z轴。下面，我们来一起绘图。

![](https://static001.geekbang.org/resource/image/83/c3/8311b485131497ce59cd1600b9a7f7c3.jpeg?wh=1920%2A1080)

**首先，我们要定义这个三角形的三个顶点**。WebGL使用的数据需要用类型数组定义，默认格式是Float32Array。Float32Array是JavaScript的一种类型化数组（TypedArray），JavaScript通常用类型化数组来处理二进制缓冲区。

因为平时我们在Web前端开发中，使用到类型化数组的机会并不多，你可能还不大熟悉，不过没关系，类型化数组的使用并不复杂，定义三角形顶点的过程，你直接看我下面给出的代码就能理解。不过，如果你之前完全没有接触过它，我还是建议你阅读[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)，去详细了解一下类型化数组的使用方法。

```
const points = new Float32Array([
  -1, -1,
  0, 1,
  1, -1,
]);

```

**接着，我们要将定义好的数据写入WebGL的缓冲区**。这个过程我们可以简单总结为三步，分别是创建一个缓存对象，将它绑定为当前操作对象，再把当前的数据写入缓存对象。这三个步骤主要是利用createBuffer、bindBuffer、bufferData方法来实现的，过程很简单你可以看一下我下面给出的实现代码。

```
const bufferId = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, bufferId);
gl.bufferData(gl.ARRAY_BUFFER, points, gl.STATIC_DRAW);
```

### 步骤四：将缓冲区数据读取到GPU

现在我们已经把数据写入缓存了，但是我们的shader现在还不能读取这个数据，还需要把数据绑定给顶点着色器中的position变量。

还记得我们的顶点着色器是什么样的吗？它是按如下的形式定义的：

```
attribute vec2 position;

void main() {
  gl_PointSize = 1.0;
  gl_Position = vec4(position, 1.0, 1.0);
}
```

在GLSL中，attribute表示声明变量，vec2是变量的类型，它表示一个二维向量，position是变量名。接下来我们将buffer的数据绑定给顶点着色器的position变量。

```
const vPosition = gl.getAttribLocation(program, 'position');获取顶点着色器中的position变量的地址
gl.vertexAttribPointer(vPosition, 2, gl.FLOAT, false, 0, 0);给变量设置长度和类型
gl.enableVertexAttribArray(vPosition);激活这个变量
```

经过这样的处理，在顶点着色器中，我们定义的points类型数组中对应的值，就能通过变量position读到了。

### 步骤五：执行着色器程序完成绘制

现在，我们把数据传入缓冲区以后，GPU也可以读取绑定的数据到着色器变量了。接下来，我们只需要调用绘图指令，就可以执行着色器程序来完成绘制了。

我们先调用gl.clear将当前画布的内容清除，然后调用gl.drawArrays传入绘制模式。这里我们选择gl.TRIANGLES表示以三角形为图元绘制，再传入绘制的顶点偏移量和顶点数量，WebGL就会将对应的buffer数组传给顶点着色器，并且开始绘制。代码如下：

```
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, points.length / 2);
```

这样，我们就在Canvas画布上画出了一个红色三角形。

![](https://static001.geekbang.org/resource/image/cc/61/ccdd298c45f80a9a00d23082cf637d61.jpeg?wh=1920%2A1080)

为什么是红色三角形呢？因为我们在片元着色器中定义了像素点的颜色，代码如下：

```
precision mediump float;

void main()
{
  gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);
}
```

在**片元着色器**里，我们可以通过设置gl\_FragColor的值来定义和改变图形的颜色。gl\_FragColor是WebGL片元着色器的内置变量，表示当前像素点颜色，它是一个用RGBA色值表示的四维向量数据。在上面的代码中，因为我们写入vec4(1.0, 0.0, 0.0, 1.0)对应的是红色，所以三角形是红色的。如果我们把这个值改成vec4(0.0, 0.0, 1.0, 1.0)，那三角形就是蓝色。

我为什么会强调颜色这个事儿呢？你会发现，刚才我们只更改了一个值，就把整个图片的所有像素颜色都改变了。所以，我们必须要认识到一点，WebGL可以并行地对整个三角形的所有像素点同时运行片元着色器。并行处理是WebGL程序非常重要的概念，所以我就多强调一下。

我们要记住，不论这个三角形是大还是小，有几十个像素点还是上百万个像素点，GPU都是**同时处理**每个像素点的。也就是说，图形中有多少个像素点，着色器程序在GPU中就会被同时执行多少次。

到这里，WebGL绘制三角形的过程我们就讲完了。借助这个过程，我们加深了对顶点着色器和片元着色器在使用上的理解。不过，因为后面我们会更多地讲解片元着色器的绘图方法，那今天，我们正好可以借着这个机会，多讲讲顶点着色器的应用，我希望你也能掌握好它。

## 顶点着色器的作用

顶点着色器大体上可以总结为两个作用：一是通过gl\_Position设置顶点，二是通过定义varying变量，向片元着色器传递数据。这么说还是有点抽象，我们还是通过三角形的例子来具体理解一下。

### 1. 通过gl\_Position设置顶点

假如，我想把三角形的周长缩小为原始大小的一半，有两种处理方式法：一种是修改points数组的值，另一种做法是直接对顶点着色器数据进行处理。第一种做法很简单，我就不讲了，如果不懂你可以在留言区提问。我们来详细说说第二种做法。

我们不需要修改points数据，只需要在顶点着色器中，将 gl\_Position = vec4(position, 1.0, 1.0);修改为 gl\_Position = vec4(position * 0.5, 1.0, 1.0);，代码如下所示。

```
attribute vec2 position;

void main() {
  gl_PointSize = 1.0;
  gl_Position = vec4(position * 0.5, 1.0, 1.0);
}
```

这样，三角形的周长就缩小为原来的一半了。在这个过程中，我们不需要遍历三角形的每一个顶点，只需要是利用GPU的并行特性，在顶点着色器中同时计算所有的顶点就可以了。在后续课程中，我们还会遇到更加复杂的例子，但在那之前，你一定要理解并牢记WebGL可以**并行计算**这一特点。

### 2. 向片元着色器传递数据

除了计算顶点之外，顶点着色器还可以将数据通过varying变量传给片元着色器。然后，这些值会根据片元着色器的像素坐标与顶点像素坐标的相对位置做**线性插值**。这是什么意思呢？其实这很难用文字描述，我们还是来看一段代码：

```
attribute vec2 position;
varying vec3 color;

void main() {
  gl_PointSize = 1.0;
  color = vec3(0.5 + position * 0.5, 0.0);
  gl_Position = vec4(position * 0.5, 1.0, 1.0);
}
```

在这段代码中，我们修改了顶点着色器，定义了一个color变量，它是一个三维的向量。我们通过数学技巧将顶点的值映射为一个RGB颜色值（关于顶点映射RGB颜色值的方法，在后续的课程中会有详细介绍），映射公式是 vec3(0.5 + position * 0.5, 0.0)。

这样一来，顶点\[-1,-1]被映射为\[0,0,0]也就是黑色，顶点\[0,1]被映射为\[0.5, 1, 0]也就是浅绿色，顶点\[1,-1]被映射为\[1,0,0]也就是红色。这样一来，三个顶点就会有三个不同的颜色值。

然后我们将color通过varying变量传给片元着色器。片元着色器中的代码如下：

```
precision mediump float;
varying vec3 color;

void main()
{
  gl_FragColor = vec4(color, 1.0);
}  
```

我们将gl\_FragColor的rgb值设为变量color的值，这样我们就能得到下面这个三角形：

![](https://static001.geekbang.org/resource/image/5c/21/5c4c718eca069be33d8a1d5d1eb77821.jpeg?wh=1920%2A1080)

我们可以看到，这个三角形是一个颜色均匀（线性）渐变的三角形，它的三个顶点的色值就是我们通过顶点着色器来设置的。而且你会发现，中间像素点的颜色是均匀过渡的。这就是因为WebGL在执行片元着色器程序的时候，顶点着色器传给片元着色器的变量，会根据片元着色器的像素坐标对变量进行线性插值。利用线性插值可以让像素点的颜色均匀渐变这一特点，我们就能绘制出颜色更丰富的图形了。

好了，到这里，我们就在Canvas画布上用WebGL绘制出了一个三角形。绘制三角形的过程，就像我们初学编程时去写出一个Hello World程序一样，按道理来说，应该非常简单才对。但事实上，用WebGL完成这个程序，我们一共用了好几十行代码。而如果我们用Canvas2D或者SVG实现类似的功能，只需要几行代码就可以了。

那我们为什么非要这么做呢？而且我们费了很大的劲，就只绘制出了一个最简单的三角形，这似乎离我们用WebGL实现复杂的可视化效果还非常遥远。我想告诉你的是，别失落，想要利用WebGL绘制更有趣、更复杂的图形，我们就必须要学会绘制三角形这个图元。还记得我们前面说过的，要在WebGL中绘制非图元的其他图形时，我们必须要把它们划分成三角形才行。学习了后面的课程之后，你就会对这一点有更深刻的理解了。

而且，用WebGL可以实现的视觉效果，远远超越其他三个图形系统。如果用驾驶技术来比喻的话，使用SVG和Canvas2D时，就像我们在开一辆自动挡的汽车，那么使用WebGL的时候，就像是在开战斗机！所以，千万别着急，随着对WebGL的不断深入理解，我们就能用它来实现更多有趣的实例了。

## 要点总结

在这一节课，我们讲了WebGL的绘图过程以及顶点着色器和片元着色器的作用。

WebGL图形系统与用其他图形系统不同，它的API非常底层，使用起来比较复杂。想要学好WebGL，我们必须要从基础概念和原理学起。

一般来说，在WebGL中要完成图形的绘制，需要创建WebGL程序，然后将图形的几何数据存入数据缓冲区，在绘制过程中让WebGL从缓冲区读取数据，并且执行着色器程序。

WebGL的着色器程序有两个。一个是顶点着色器，负责处理图形的顶点数据。另一个是片元着色器，负责处理光栅化后的像素信息。此外，我们还要牢记，WebGL程序有一个非常重要的特点就是能够并行处理，无论图形中有多少个像素点，都可以通过着色器程序在GPU中被同时执行。

WebGL完整的绘图过程实在比较复杂，为了帮助你理解，我总结一个流程图，供你参考。

[![](https://static001.geekbang.org/resource/image/d3/30/d31e6c50b55872f81aa70625538fb930.jpg?wh=1196%2A960 "WebGL绘图流程")](https://juejin.im/post/5e7a042e6fb9a07cb96b1627)

那到这里，可视化的四个图形系统我们就介绍完了。但是，好戏才刚刚开始哦，在后续的文章中我们会围绕着这四个图形系统，尤其是Canvas2D和WebGL逐渐深入，来实现更多有趣的图形。

## 小试牛刀

1. WebGL通过顶点和图元来绘制图形，我们在上面的例子中，调用gl.TRIANGLES 绘制出了实心的三角形。如果要绘制空心三角形，我们又应该怎么做呢？有哪些图元类型可以帮助我们完成这个绘制？
2. 三角形是最简单的几何图形，如果我们要绘制其他的几何图形，我们可以通过用多个三角形拼接来实现。试着用WebGL绘制正四边形、正五边形和正六角星吧！

欢迎在留言区和我讨论，分享你的答案和思考，也欢迎你把这一节课分享给你的朋友，我们下节课再见！

* * *

## 源码

[WebGL绘制三角形示例代码](https://github.com/akira-cn/graphics/tree/master/webgl)

## 推荐阅读

\[1] [类型化数组 MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)  
\[2] [WebGL 的 MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API)
<div><strong>精选留言（15）</strong></div><ul>
<li><span>筑梦师刘渊</span> 👍（51） 💬（2）<p>作业一   
查了下资料，webgl支持的图元类型有七种，分别是 gl.POINTS(点), gl.LINES(线段), gl.LINE_STRIP(线条), gl.LINE_LOOP(回路), gl.TRIANGLES(三角形), gl.TRIANGLE_STRIP(三角带), gl.TRIANGLE_FAN(三角扇)。
要绘制空心三角形，gl.LINE_STRIP(线条)、gl.LINES(线段)、 gl.LINE_LOOP(回路)都可以实现。 但是gl.LINES(线段)需要写入六个顶点([-1, -1, 0, 1,   0, 1, 1, -1,   1, -1,-1, -1]), gl.LINE_STRIP(线条)也需要写入四个顶点([-1, -1, 0, 1, 1, -1,-1, -1]),而gl.LINE_LOOP(回路)，只需要是三个顶点([-1, -1, 0, 1, 1, -1])，因此gl.LINE_LOOP(回路)是最佳选择

作业二
a. 先封装一个生成多边形顶点坐标数组的函数
function createCircleVertex(x, y, r, n) {
    const sin = Math.sin;
    const cos = Math.cos;
    const perAngel = (2 * Math.PI) &#47; n;
    const positionArray = [];
    for (let i = 0; i &lt; n; i++) {
        const angel = i * perAngel;
        const nx = x + r * cos(angel);
        const ny = y + r * sin(angel);
        positionArray.push(nx, ny);
    }
    return new Float32Array(positionArray);
}
b. 封装一个生成正多角星顶点的数组函数
function create2CircleVertex(x, y, r, R, n) {
    const sin = Math.sin;
    const cos = Math.cos;
    const perAngel = Math.PI &#47; n;
    const positionArray = [];
    for (let i = 0; i &lt; 2 * n; i++) {
        const angel = i * perAngel;
        if (i % 2 !== 0) {
            const Rx = x + R * cos(angel);
            const Ry = y + R * sin(angel);
            positionArray.push(Rx, Ry);
        } else {
            const rx = x + r * cos(angel);
            const ry = y + r * sin(angel);
            positionArray.push(rx, ry);
        }
    }
    return new Float32Array(positionArray);
}
1. 正四边形  const points = createCircleVertex(0, 0, 0.5, 4); 
2. 正五边形  const points = createCircleVertex(0, 0, 0.5, 5); 
3. 正六角星 const points = create2CircleVertex(0, 0, 0.3, 0.6, 6);

以上要绘制空心用gl.LINE_LOOP图元，实心用gl.TRIANGLE_FAN图元   
1)空心:gl.drawArrays(gl.LINE_LOOP, 0, points.length &#47; 2);
2)实心:gl.drawArrays(gl.TRIANGLE_FAN, 0, points.length &#47; 2);
</p>2020-06-30</li><br/><li><span>Cailven</span> 👍（17） 💬（1）<p>补充：vs不仅仅只有postion值，一般通过attribute 进行属性赋值。在图形学管顶点操作叫做VAO(vertex array object)，而vao操作的float数据底层是vbo。不过如果用了threejs后很多图元操作就依赖引擎直接就解决了，但在Threejs中依然可以通过shaderMatiral通过setAttribute给bufferGeometry的顶点赋值。

不过个人在这几年的图形学学习中觉得vs相对还是简单的，fs对于像素的操作很像当年给photoshop写滤镜的过程。不过如果真的是玩片元着色炫技可以看看shadertoy里关于用remaching技术构建距离场用体素算法在片元里构建另一个三维引擎的效果。希望月影大大回头可以针对这方面有所加餐。感谢！

不过这课程来的太晚，如果几年前能早点接触到这门课，估计会在图形学方面少走很多弯路，谢谢月影大大。</p>2020-06-29</li><br/><li><span>xiao豪</span> 👍（8） 💬（2）<p>老师，将数据存入缓存再拿出来是有什么意义呢？</p>2020-07-06</li><br/><li><span>宁康</span> 👍（5） 💬（1）<p>正n边型，r是外接圆半径

getPolygonPoints( n, r ){
      const stepAngle = 2*Math.PI &#47; n
      let initAngle = 0
      const pointArray = []
      for(let i = 0; i &lt; n; i++) {
        &#47;&#47; 存储x坐标
        pointArray.push(r * Math.cos(initAngle))
        &#47;&#47; 存储y坐标
        pointArray.push(r * Math.sin(initAngle))

        initAngle += stepAngle
      }

      return pointArray
    }

&#47;&#47; 正十边型坐标点
const ponitsArray = getPolygonPoints(10, 1)

const ponits = new Float32Array(ponitsArray)

gl.drawArrays(gl.TRIANGLE_FAN, 0, ponits.length &#47; 2)</p>2020-06-29</li><br/><li><span>宁康</span> 👍（3） 💬（1）<p>1、gl_Position 设置顶点，这个我查了一下，第四个值设置为2.0也可以实现缩小一倍。
gl_Position = vec4(position, 0.0, 2.0);

2、空心三角形：
gl.drawArrays(gl. LINE_LOOP, 0, ponits.length &#47; 2)

3、绘制多边形
a.定义多边形的(x, y)坐标
const ponits = new Float32Array([
        -1, -1,
        0, -2,
        1, -1,
        1, 1,
        -1, 1
      ])
b.绘制多边形
gl.drawArrays(gl.TRIANGLE_FAN, 0, ponits.length &#47; 2)
</p>2020-06-29</li><br/><li><span>Kevin</span> 👍（2） 💬（1）<p>实现了一个正多边形的样例，动态修改边数。
https:&#47;&#47;codesandbox.io&#47;s&#47;practice-canvas-vme4k?file=&#47;src&#47;pages&#47;RegularPolygonWebGL.vue</p>2020-07-16</li><br/><li><span>Kevin</span> 👍（2） 💬（1）<p>问题一：
绘制空心三角形使用回路线条：gl.LINE_LOOP
    gl.drawArrays(gl.LINE_LOOP, 0, points.length &#47; 2);

WebGL可绘制的图元有以下7种，来源网络查找：https:&#47;&#47;www.jianshu.com&#47;p&#47;1e750f20ec23

点	gl.POINTS	
线段	gl.LINES	
线条	gl.LINE_STRIP
回路线条	gl.LINE_LOOP
三角形	gl.TRIANGLES
三角带	gl.TRIANGLE_STRIP	
三角扇	gl.TRIANGLE_FAN </p>2020-07-15</li><br/><li><span>国旗</span> 👍（1） 💬（1）<p>老师问下MDN文档里WebGL常数这儿，表格里十六进制的‘Value’表示的意思是类似于CPU指令寄存器么？
Getting GL parameter information这节BLEND_EQUATION，BLEND_EQUATION_RGB的value都是0x8009，也有点不理解
https:&#47;&#47;developer.mozilla.org&#47;en-US&#47;docs&#47;Web&#47;API&#47;WebGL_API&#47;Constants</p>2020-09-23</li><br/><li><span>不见飞刀</span> 👍（1） 💬（3）<p>&quot;图形中有多少个像素点，着色器程序在 GPU 中就会被同时执行多少次。&quot;

顶点着色器是否一样还是有几个顶点就执行多少次呢？
传给片元的varying变量会线性差值，那么插值这一步发生在哪呢？</p>2020-08-29</li><br/><li><span>Geek_gmfq9e</span> 👍（1） 💬（1）<p>请问编写glsl有什么智能提示插吗，我用的vscode？</p>2020-08-01</li><br/><li><span>我母鸡啊！</span> 👍（1） 💬（1）<p>作业1 ： gl.drawArrays传入gl.LINE_LOOP</p>2020-07-03</li><br/><li><span>我母鸡啊！</span> 👍（1） 💬（1）<p>所以在webgl中最小的图元是三角形？</p>2020-07-03</li><br/><li><span>miaomiao</span> 👍（1） 💬（4）<p>月影老师，你好，有个问题想请教下，如果我想实现12条心电波形折线图，每条波形图的每个脉搏段会依据脉搏类型展示不同的颜色，用户交互：用户可以选择特定的脉搏段，统一修改这个脉搏段类型，这样类型变化，12条心电图的对应脉搏段颜色也变化。这种用canvas好还是webgl好？用canvas的话，需要去获取用户选定的范围，对画布元素进行局部重绘，目前有没有可以只通过更改数据，根据数据变化进行自动重绘的图形库？</p>2020-07-03</li><br/><li><span>王子晨</span> 👍（1） 💬（2）<p>老师请问用webGL绘制复杂的图形，会不会设置多个顶点和片元着色程序？还是说一直在修改一个顶点和片元着色程序？</p>2020-07-01</li><br/><li><span>量子蔷薇</span> 👍（1） 💬（1）<p>我在codepen写了作业，https:&#47;&#47;codepen.io&#47;quantum-rose&#47;pen&#47;QWyqexL
不确定自己对WebGL绘图的机制理解对了没，关于封装复用代码那部分，感觉我可能复用了一些不需要复用的代码。期待之后的学习！
我的六角星只有描边，本质是在不清空画布的情况下画了两个三角形，不知道有没有更好的做法，如果是实心六角星，把我的代码中drawStar函数里的LINE_LOOP换成TRIANGLE_FAN就能画出来，但是其他n角星并不能正确画出来。</p>2020-06-30</li><br/>
</ul>