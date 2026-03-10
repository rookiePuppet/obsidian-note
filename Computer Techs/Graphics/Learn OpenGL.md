# Getting started

## OpenGL

OpenGL经常被认为是一种给我们提供操作图形和图像的函数集的API，但是OpenGL本身并不是API，而仅仅是由Khronos开发和维护的一种规范标准。

OpenGL规范指明了每一个函数的输出结果和表现方式，然后再由开发者实现函数的具体操作。只要这些函数的结果遵循了规范，不同的OpenGL版本都可以有不同的实现。

通常是由显卡制造商开发实际的OpenGL库，每一张显卡所支持的特定OpenGL版本都是针对于该显卡系列开发的版本。这意味着当OpenGL出现异常行为时，很大可能是显卡制造商或库开发者的过错。

### 核心模式vs即时模式

在3.2版本之前，OpenGL使用一种叫做即时模式的方法来绘制图形，虽然易于理解和使用，但大多数功能隐藏于库中，开发者对OpenGL没有太多的控制权，而且效率极低。

从3.2版本开始，即时模式被弃用，并鼓励开发者转向使用核心模式，这也是OpenGL规范移除所有老旧废弃功能的分界线。

使用核心模式时，OpenGL会强制我们使用更加灵活高效的现代方法，当然也更难学习，因为着要求开发者真正理解OpenGL和图形编程。

从3.3之后的版本都只是增加了一些高效的新特性，并没有改变核心机制，因此没有必要一开始就选择最新版本进行学习。

最新特性往往只有在最现代的显卡上才能运行，所以大多数开发者普遍针对较低版本的OpenGL进行开发，再选择性地启用高版本功能。

### 扩展

当图形公司研究出一项新技术或渲染上的重大优化时，常常会在驱动中添加扩展。如果程序运行的硬件支持某种扩展，开发者就可以使用该扩展所提供的功能。

这样图形开发者不需要等待OpenGL将新功能收纳，就可以使用这些新的渲染技术。当一项扩展流行起来后，OpenGL就会将其收纳到未来的版本当中。

### 状态机

OpenGL本身就是一个超大的状态机，其中包含了一堆变量来定义当前OpenGL应该如何操作。OpenGL的状态一般指的是OpenGL上下文（context），当使用OpenGL时，我们会通过设置一些选项、操控一些缓冲区来改变其状态，然后使用当前上下文进行渲染。

在使用OpenGL时会遇到一些用于更改上下文的**状态改变**(state-changing)方法，和一些基于当前状态执行操作的**状态使用**(stage-using)方法，只要记住OpenGL本质上是一个巨大的状态机，那么它的大多数功能就更容易理解了。

### 对象

OpenGL的核心是C语言库，由于C语言的结构不能很好地翻译成其他更高级的语言，OpenGL在开发时考虑了几个抽象，对象就是其中之一。

在OpenGL中，一个对象就是代表一个OpenGL状态的子集。例如代表绘制窗口的设置就是一个对象，我们可以设置窗口大小以及支持的颜色范围等等。

一般都会像下面这样去使用对象：

```C
// The State of OpenGL
struct OpenGL_Context {
	...
	object_name* object_Window_Target;
	...
};

// create object
unsigned int objectId = 0;
glGenObject(1, &objectId);
// bind/assign object to context
glBindObject(GL_WINDOW_TARGET, objectId);
// set options of object currently bound to GL_WINDOW_TARGET
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// set context target back to default
glBindObject(GL_WINDOW_TARGET, 0);
```

使用这些对象的好处就是我们可以在应用当中定义多个对象，设置它们的参数，无论何时使用OpenGL状态进行操作时，就可以将预先的设置绑定到对象上。

## 创建窗口

第一件事就是创建一个OpenGL上下文和用于绘制图形的窗口，因为这些操作是特定于不同操作系统的，OpenGL对此做了抽象，我们必须自己创建窗口，定义上下文以及处理用户输入。

有一些库可以帮助我们做这些系统特定的工作，例如GLFW、GLUT、SDL、SFML，在这里我们选择使用GLFW。

### GLFW

GLFW是一个用C语言写的库，专用于OpenGL，可以从他们的[网站](https://www.glfw.org/download.htm)上获得。

GLFW提供了编译好的二进制和头文件，但下面我们会从源文件进行编译。对于开源的库，不一定有提供适用于你系统的预编译文件，并且每个人的操作系统和开发环境不同，自己编译源代码可以得到更适合自己系统的版本。

### CMake

CMake是一个能够根据用户选择，使用预定义的CMake脚本从源代码文件中生成项目/解决方案文件的工具，可以让我们用GLFW的源代码包生成一个Visual Studio的项目文件，用以编译GLFW库。

```shell
cmake -S /glfw-path ./build
```

使用IDE打开生成的项目，然后进行编译，在`build/src/Debug`文件中可以找到一个名为`glfw3.lib`的文件。



### GLAD

因为OpenGL只是一个标准/规范，具体的实现是驱动制造商为特定的显卡提供的。

由于OpenGL的驱动有太多不同的版本了，太多数函数的位置在编译时是未知的，需要在运行时查询。开发者需要检索定位这些函数，并将它们存储在函数指针中。这个操作也是系统特定的，在Windows上像这样：

```C
// define the function’s prototype
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);
// find the function and assign it to a function pointer
GL_GENBUFFERS glGenBuffers =
(GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");
// function can now be called as normal
unsigned int buffer;
glGenBuffers(1, &buffer);
```

GLAD是一个开源库，用于完成上述繁琐工作。GLAD的设置和大多数开源库有些不同，它使用一个Web服务让我们选择要定义和加载相关OpenGL函数的OpenGL版本

完成所有准备工作后，我们将得到如下文件。

- include
	- glad
		- glad.h
	- GLFW
		- glfw3.h
		- glfw3native.h
	- KHR
		- khrplatform.h
- libs
	- glfw3.lib
- glad.c

## Hello Window

新建项目后，将glfw和glad相关文件复制到工程中，或利用其他方式进行关联。

首先创建一个.cpp文件，添加以下include语句，注意，GLAD引用必须在GLFW之前。

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
```

然后在main函数中实例化GLFW窗口，使用`glfwInit`初始化GLFW，之后通过`glfwWindowHint`配置GLFW，第一个参数用于选择配置选项，第二个参数用于设置选项的值。前两个`glfwWindowHint`语句用于配置我们想要使用的OpenGL版本（3.3），以便于GLFW可以正确创建OpenGL上下文，确保GLFW在用户使用不恰当的OpenGL版本时运行失败。然后我们告诉GLFW要使用核心模式，意味着我们将只能访问OpenGL特性的一个更小的子集，不需要使用向后兼容的特性。

```cpp
int main()
{
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
	//glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); // Mac OS X额外添加该行代码
	return 0;
}
```

下一步是创建一个窗口对象，它拥有大部分GLFW函数需要的所有窗口数据。

```cpp
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
if (window == NULL)
{
	std::cout << "Failed to create GLFW window" << std::endl;
	glfwTerminate();
	return -1;
}
glfwMakeContextCurrent(window);
```

### GLAD

在调用任何OpenGL函数之前，需要先初始化GLAD。

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
	std::cout << "Failed to initialize GLAD" << std::endl;
	return -1;
}
```

GLFW提供了`glfwGetProcAddress`定义当前系统的函数位置，将它传递给`gladLoadGLLoader`函数来加载OpenGL函数指针的地址。

### 视口

在开始渲染之前，我们还需要做最后一件事，告诉OpenGL我们的渲染窗口尺寸，让OpenGL知道如何显示相对于窗口的数据和坐标。

可以通过`glViewport`函数来设置窗口大小，前两个参数设置窗口左下角的位置，后两个参数设置窗口的宽高。

```cpp
glViewport(0, 0, 800, 600);
```

当用户调整窗口大小时，视口也应该跟随变化。可以在窗口上注册一个回调函数来更新视口大小，这个回调函数的原型如下：

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height);
```

```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
}
```

注册回调函数，让每当窗口大小变化时执行视口更新函数。

```cpp
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback)
```

### 准备你的引擎

我们不想让程序只是绘制了一张图像后就立即退出，而是持续绘制图像并处理用户输入。为此，我们需要创建一个while循环，它被称为`渲染循环`（render loop），以下代码是一个简单的渲染循环。

```cpp
while(!glfwWindowShouldClose(window))
{
	glfwSwapBuffers(window);
	glfwPollEvents();
}
```

`glfwWindowShouldClose`函数检查GLFW是否被指示关闭，`glfwPollEvents`函数检查是否用时间被触发，更新窗口状态和调用相关函数，`glfwSwapBuffers`函数用于交换颜色缓冲区（包含GLFW窗口中每一个像素颜色数据的2D缓冲区）。

> [!note] 双缓冲区
> 如果程序在单个缓冲区上绘制，可能会导致最终图像有闪烁问题。这是因为图像不是一瞬间绘制完成的，而通常是从左到右、从上到下地逐像素绘制的。<br>
> 为了解决该问题，窗口会应用双缓冲区来渲染，**前缓冲区**包含显示在屏幕上的最终输出图像，所有的渲染指令都会应用到**后缓冲区**。当所有指令完成后，将后缓冲区的内容**替换**到前缓冲区，从而消除伪影。

### 最后一件事

当我们退出渲染循环时，需要将所有已分配的GLFW资源正确清除，可以在main函数的最后通过调用`glfwTerminate`函数来完成。

```cpp
glfwTerminate();
return 0;
```

### 输入

GLFW的`glfwGetKey`函数可以返回某个按键是否正在被按下，创建一个`processInput`函数来组织所有输入相关的代码。

```cpp
void processInput(GLFWwindow *window)
{
	if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
		glfwSetWindowShouldClose(window, true);
}
```

将该函数加入到渲染循环中，这样一旦按下Escape键，窗口就会退出了。

```cpp
while (...)
{
	processInput(window);
	...
}
```

### 渲染

由于我们要在循环的每一帧执行所有的渲染指令，所以将所有渲染指令都放在渲染循环中，就像这样：

```cpp
// render loop
while(!glfwWindowShouldClose(window))
{
	// input
	processInput(window);
	// rendering commands here
	...
	// check and call events and swap the buffers
	glfwPollEvents();
	glfwSwapBuffers(window);
}
```

为了测试，下面我们使用一种颜色来填充屏幕，并且在每一帧开始时需要先清除屏幕，否则仍然会看到前一帧的渲染结果。可以使用`glClear`函数清除屏幕的颜色缓冲区，传入缓冲区位来指定要清除的缓冲区，可设置的位有`GL_COLOR_BUFFER_BIT`， `GL_DEPTH_BUFFER_BIT`和`GL_STENCIL_BUFFER_BIT`。

### 目前为止的代码

```cpp
#include <iostream>
#include <glad\glad.h>
#include <GLFW\glfw3.h>

void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
	glViewport(0, 0, width, height);
}

void processInput(GLFWwindow* window)
{
	if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
		glfwSetWindowShouldClose(window, true);
}

int main()
{
	// 初始化GLFW
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

	// 创建GLFW窗口
	GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
	if (window == NULL)
	{
		std::cout << "Failed to create GLFW window" << std::endl;
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(window);

	// GLAD加载函数指针
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

	// 设置视口，注册窗口大小调整回调函数
	glViewport(0, 0, 800, 600);
	glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

	// 渲染循环
	while (!glfwWindowShouldClose(window))
	{
		processInput(window);

		glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
		glClear(GL_COLOR_BUFFER_BIT);

		glfwPollEvents();
		glfwSwapBuffers(window);
	}

	// 释放资源并退出
	glfwTerminate();
	return 0;
}
```

## Hello Triangle

所有东西在OpenGL中都是在3D空间下的，但是屏幕和窗口是一个像素二维数组，所以OpenGL的工作是关于将所有3D坐标转换为2D像素的，这个过程由OpenGL的**图形管线**所管理。

图形管线可以分为两大部分：将3D坐标转换为2D坐标和将2D坐标转换为真实可见的有色像素。

图形管线可以划分为多个步骤，每个步骤依赖前一个步骤的输出作为输入，可以轻易地并行化执行。GPU上的处理核心会为管线的每一个步骤运行程序，这些程序叫做**着色器**。

一些着色器是可以由开发者自己配置的，让我们可以编写自己的着色器来替换已存在的默认着色器。在OpenGL中，着色器是使用**OpenGL着色语言**（GLSL）来编写的。

下图是图形管线所有阶段的抽象表示，蓝色部分表示可以注入我们自己的着色器。

![[Pasted image 20260309170657.png]]

顶点数据是整个图形管线的输入，一个**顶点**代表了一个3D坐标的数据集合，用**顶点属性**（vertex attributes）表示，它不止包含坐标和颜色，还有一些其他数据。

> [!note]
> 为了让OpenGL知道要用提供的坐标集和颜色生成什么样的图形，我们需要给OpenGL提示，即我们想要将数据作为一个点的集合，还是一个三角形的集合，甚至可能是一条长线。这些提示称为**图元**，在调用任何绘制命令时都要给出。

渲染管线的第一部分就是**顶点着色器**，将单个顶点作为输入，其主要目的是将3D坐标转换成不同的3D坐标，并且可以让我们在顶点属性上做一些基本处理。

**图元装配**阶段将来自顶点着色器的所有顶点作为输入，根据给定的图元形状将所有的点进行组装，形成图元。

**几何着色器**将形成图元的顶点几何作为输入，能够通过生成新顶点来形成新的图元，生成其他形状。

**光栅化阶段**将产生的图元映射到最终屏幕上对应的像素，并产生片元。之再会执行**裁剪**，丢弃所有视野之外的片元，提高性能。

**片元**包含了OpenGL渲染一个像素所需的全部数据，**片元着色器**利用它计算对应像素的最终颜色。

**透明度测试和混合阶段**会检查片元的深度（和模板）值，判断其是否应该被丢弃，并检查**透明度**值，相应地对像素进行**混合**。

## 顶点输入

OpenGL只会处理三个轴在`[-1.0, 1.0]`范围内的3D坐标，这种坐标称为`标准化设备坐标`。

```cpp
float vertices[] = {
	-0.5f, -0.5f, 0.0f,
	0.5f, -0.5f, 0.0f,
	0.0f, 0.5f, 0.0f
};
```

> [!note] 标准化设备坐标（NDC）
> 顶点坐标一旦经过顶点着色器处理后，就会变成标准化设备坐标，x、y和z值范围都是`[-1.0, 1.0]`。然后视口变换使用你提供给`glViewport`的数据将NDC变换到屏幕空间，最后转换成片元。

为了将顶点数据发送给顶点着色器，我们需要在GPU上创建存储顶点数据的内存，配置OpenGL解析内存的方式，并指定如何发送数据给显卡。

**顶点缓冲对象**（VBO）可以在GPU内存中存储大量顶点，使用这些缓冲对象的好处是我们可以一次性向显卡发送大批量的数据，而不是一次发送一个顶点。

OpenGL中的任何对象都一样，VBO拥有一个和缓冲区对应的唯一ID，我们可以使用一个缓冲区ID通过`glGenBuffers`函数创建。

```cpp
unsigned int VBO;
glGenBuffers(1, &VBO);
```

OpenGL有多种类型的缓冲区对象，顶点缓冲对象的类型是`GL_ARRAY_BUFFER`。OpenGL允许我们一次绑定多个缓冲区，只要他们的类型不同，使用glBindBuffer函数可以将刚刚创建的缓冲区绑定到`GL_ARRAY_BUFFER`。

```cpp
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

这样以后，针对`GL_ARRAY_BUFFER`的任何函数调用就会被用于配置当前绑定的缓冲区，即`VBO`。然后我们通过`glBufferData`函数将之前定义好的顶点数据复制到缓冲区内存，第一个参数是目标缓冲区类型，第二个参数是数据的字节大小，第三个参数就是实际要拷贝的数据，第四个参数指定显卡如何管理该数据，有三种形式：

- `GL_STREAM_DRAW`：数据只设置一次，最多被使用几次
- `GL_STATIC_DRAW`：数据只设置一次，被多次使用
- `GL_DYNAMIC_DRAW`：数据频繁改变，被多次使用

```cpp
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

因为这里顶点的位置数据不会改变，会被频繁使用，并且每一次渲染调用都保持相同，所以最好使用`GL_STATIC_DRAW`。