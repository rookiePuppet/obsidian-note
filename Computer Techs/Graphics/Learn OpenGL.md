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

### 顶点输入

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

### 顶点着色器

现代OpenGL要求必须至少配置一个顶点着色器和一个片元着色器，我们首先要用GLSL编写一个顶点着色器，然后编译它。

```c
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
	gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

第一行声明GLSL的版本，这和OpenGL版本是相对应的，因为我们使用OpenGL 3.3，所以这里是330，另外还显式声明了使用核心模式。

接着我们使用`in`关键字声明所有输入顶点属性，目前我们只关心位置数据，所以只需要一个顶点属性即可。我们还通过`layout (location = 0)`，设置了输入变量的位置。

在`main`函数最后，为`gl_Position`设定的值会被用作顶点着色器的输出，它是`vec4`类型的，而我们的输入是`vec3`，所以要进行一个转换。

### 编译着色器

我们将顶点着色器的源代码存储在文件顶部的一个C字符串常量中：

```c
const char *vertexShaderSource = "#version 330 core\n"
	"layout (location = 0) in vec3 aPos;\n"
	"void main()\n"
	"{\n"
	" gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
	"}\0";
```

OpenGL必须在运行时从源代码动态编译该着色器，我们首先需要使用`glCreateShader`创建一个着色器对象，传入`GL_VERTEX_SHADER`指定着色器类型。

```c
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```

然后我们将着色器源代码附加到该着色器对象上，编译着色器。

```c
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

> 检查着色器编译是否成功

```c
int success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);

if(!success)
{
	glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" <<
	infoLog << std::endl;
}
```

### 片元着色器

片元着色器只需要一个vec4输出变量用于定义最终的颜色输出，我们可以用`out`关键字声明输出值。

```c
#version 330 core
out vec4 FragColor;

void main()
{
	FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

编译片元着色器的过程和顶点着色器是类似的，不过这次我们使用`GL_FRAGMENT_SHADER`作为着色器类型。

最后我们要将两个着色器对象链接到一个着色器程序。

### 着色器程序

着色器程序对象表示的是多个着色器组合并链接形成的程序，为了使用刚刚编译好的着色器，我们必须将它们**链接**到一个着色器程序对象，然后在渲染物体时激活这个着色器程序。

链接着色器其实就是将每一个着色器的输出链接到其下一个着色器的输入，如果输入输出不匹配就会发生链接错误。

首先创建一个着色器程序对象：

```c
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
```

然后将编译好的着色器附加到程序对象上，再使用`glLinkProgram`函数链接它们：

```c
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
```

> 检查着色器程序链接是否成功

```c
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if(!success) {
	glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
	...
}
```

之后通过调用`glUseProgram`函数，将程序对象激活。

```c
glUseProgram(shaderProgram);
```

着色器链接完成后，别忘记将着色器对象删除。

```c
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

### 链接顶点属性

顶点着色器允许我们以顶点属性的形式指定需要的输入，这意味着我们必须手动指定输入数据和顶点属性的对应关系。

我们的顶点缓冲数据格式如下：

![[Pasted image 20260310154429.png]]

- 位置数据以32位浮点值存储
- 每一个位置由3个这样的值组成
- 数值紧密排列在数组中
- 数据中第一个值位于缓冲区的开头

知道了这些，我们就可以使用`glVertexAttribPointer`告诉OpenGL如何理解顶点数据。

```c
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

`glVertexAttribPointer`函数的各个参数：

- 指定要配置的顶点属性，我们在顶点着色器中通过`layout(location = 0)`指定了顶点属性`position`的位置，因此这里是0。
- 指定顶点属性的大小，vec3由3个值组成。
- 指定数据类型为`GL_FLOAT`，因为`vec*`由浮点值组成。
- 指定是否对数据标准化。
- 指定顶点属性之间的间隔，因为这里每一个位置数据占据3个浮点数大小，也就是说从一个位置数据的开始到下一个位置数据的开始之间间隔3个浮点数。另外，由于我们已知数组是紧密排列的，也可以指定为0，让OpenGL去决定间隔。
- 位置数据在缓冲区的起始偏移

顶点属性默认是关闭的，我们需要使用`glEnableVertexAttribArray`将它启用。

到此为止，我们已经准备好所有东西了：使用顶点缓冲对象初始化缓冲区中的顶点数据，准备顶点和片元着色器并告诉OpenGL如何链接顶点数据和顶点属性。在OpenGL中绘制物体就像这样：

```c
// 0. copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float),
(void*)0);
glEnableVertexAttribArray(0);
// 2. use our shader program when we want to render an object
glUseProgram(shaderProgram);
// 3. now draw the object
someOpenGLFunctionThatDrawsOurTriangle();
```

每次绘制一个物体，我们都必须重复这个过程。一旦顶点属性和物体数量多起来，正确地绑定缓冲对象并为每一个对象配置顶点属性很快变得枯燥。是否有某种方式可以将所有状态配置存储到一个对象，并简单地绑定该对象来恢复它的状态？

#### 顶点数组对象

顶点数组对象（VAO）可以像顶点缓冲对象一样绑定，之后所有的顶点属性调用都会存储在VAO中。每当我们绘制物体时，只需要绑定对应的VAO，就能在不同的顶点数据和属性配置之间切换。

顶点数组对象存储以下内容：

- 对`glEnableVertexAttribArray`和`glDisableVertexattribAray`的调用
- 通过`glVertexAttribPointer`进行的顶点属性配置
- 通过向`glVertexAttribPointer`调用与顶点属性关联的顶点缓冲对象

![[Pasted image 20260310170824.png]]

创建VAO的过程和VBO类似：

```c
unsigned int VAO;
glGenVertexArrays(1, &VAO);
```

要使用VAO，只需通过`glBindVertexArray`绑定VAO即可，然后绑定/配置相应的VBO和属性指针，再解绑VAO。

```c
// ..:: Initialization code (done once (unless your object frequently changes)) :: ..
// 1. bind Vertex Array Object
glBindVertexArray(VAO);
// 2. copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. then set our vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float),
(void*)0);
glEnableVertexAttribArray(0);
// ..:: Drawing code (in render loop) :: ..
// 4. draw the object
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
someOpenGLFunctionThatDrawsOurTriangle();
```

#### 等待已久的三角形

为了绘制我们想要的物体，OpenGL提供了`glDrawArrays`函数，可以使用当前激活的着色器以及先前定义的顶点属性配置和顶点数据绘制图元。

```c
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, 3);
```

`glDrawArray`函数的第一个参数为我们希望绘制的OpenGL图元类型，第二个参数指定顶点数组的起始索引，第三个参数指定绘制的顶点数量。

### 元素缓冲对象

假设要绘制一个矩形，可以通过绘制两个三角形得到，顶点数据会是这样的：

```c
float vertices[] = {
	// first triangle
	0.5f, 0.5f, 0.0f, // top right
	0.5f, -0.5f, 0.0f, // bottom right
	-0.5f, 0.5f, 0.0f, // top left
	// second triangle
	0.5f, -0.5f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, // bottom left
	-0.5f, 0.5f, 0.0f // top left
};
```

可以看到有一些重复的顶点，有没有办法只存储唯一的顶点，然后指定顶点绘制的顺序呢？这样我们就只需要存储四个顶点来绘制矩形。

元素缓冲对象就可以解决这个问题，它可以存储OpenGL用来决定要绘制的顶点索引，这被称为**索引绘制**。

```c
float vertices[] = {
	0.5f, 0.5f, 0.0f, // top right
	0.5f, -0.5f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, // bottom left
	-0.5f, 0.5f, 0.0f // top left
};
unsigned int indices[] = { // note that we start from 0!
	0, 1, 3, // first triangle
	1, 2, 3 // second triangle
};
```

声明顶点和索引之后，创建元素缓冲对象：

```c
unsigned int EBO;
glGenBuffers(1, &EBO);
```

和VBO类似，我们会绑定EBO，然后使用`glBufferData`将索引拷贝到缓冲区。

```c
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices,
GL_STATIC_DRAW);
```

最后，我们要将`glDrawArrays`调用替换为`glDrawElements`，以表明我们要从索引缓冲渲染三角形。第一个参数指定绘制模式，和`glDrawArrays`类似。第二个参数是绘制元素的数量，我们总共需要绘制6个顶点。第三个参数是索引的类型。最后一个参数指定EBO的起始偏移。

```c
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

`glDrawElements`函数从当前绑定到`GL_ELEMENT_ARRAY_BUFFER`的EBO中取出索引，这意味着每次我们想用索引绘制时都要绑定对应的VBO。当VAO绑定时，最后绑定的EBO会作为该VAO的元素缓冲对象，绑定到该VAO也会自动绑定到对应的VBO。

![[Pasted image 20260311113504.png]]

最终的初始化和绘制代码会变成这样：

```c
// ..:: Initialization code :: ..
// 1. bind Vertex Array Object
glBindVertexArray(VAO);
// 2. copy our vertices array in a vertex buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. copy our index array in a element buffer for OpenGL to use
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices,
GL_STATIC_DRAW);
// 4. then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
[...]
// ..:: Drawing code (in render loop) :: ..
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0)
glBindVertexArray(0);
```

> [!note] 线框模式
> 要以线框模式绘制三角形，可以通过`glPolygonmode(GL_FRONT_AND_BACK, GL_LINE)`配置OpenGL如何绘制图元。第一个参数表示我们想将其应用于所有三角形的正面和背面，第二个参数表示绘制成线段。所有之后的绘制调用都会在线框模式下进行，直到我们通过`glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)`设置回默认。

## 着色器

着色器是运行在GPU上的程序，在图形管线的特定部分工作。简单来说，着色器只不过是将输入进行变换从而产生输出的程序，它们之间不能相互通信。

### GLSL

OpenGL中的着色器使用类似C语言的GLSL编写，GLSL是为图形专门设计的，包含向量和矩阵操作等有用的特性。

编写着色器从版本声明开始，然后是一系列的输入输出变量、uniform和`main`函数。每个着色器的入口点就是它的`main`函数，在这里面我们对输入变量进行处理，然后输出结果到它的输出变量中。

```c
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

void main()
{
	// process input(s) and do some weird graphics stuff
	...
	// output processed stuff to output variable
	out_variable_name = weird_stuff_we_processed;
}
```

对于顶点着色器来说，输入变量也叫做**顶点属性**。可声明顶点属性的数量受限于硬件，OpenGL会保证至少可以使用16个4分量的顶点属性，一些硬件可能允许声明更多，这可以通过`GL_MAX_VERTEX_ATTRIBS`来查询。

```c
int nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
std::cout << "Maximum nr of vertex attributes supported: " << nrAttributes
<< std::endl;
```

### 类型

GLSL提供的基础类型有int，float，double，uint和bool，另外还有两种容器类型是向量和矩阵。

#### 向量

在GLSL中，向量是一个包含最多四个分量的基本类型容器，可以有以下形式（n表示分量的数量）：

- vecn：float类型
- bvecn：bool类型
- ivecn：int类型
- uvecn：uint类型
- dvecn：double类型

向量的分量可以通过`vec.x/y/z/w`的形式进行访问，对于颜色可以使用`rgba`，对于纹理坐标可以使用`stpq`。

向量类型支持`swizzling`，一种灵活的分量选择方式，语法像这样：

```c
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

你可以使用最多4个字母的任意组合来创建一个新的向量，只要原向量拥有对应的分量。还可以利用它来实现更简便的向量构造函数调用：

```c
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

### in和out

着色器可以使用`in`和`out`关键字指定输入和输出，当输出变量和下一个着色器阶段的输入变量匹配时，数据就可以传递下去。顶点和片元着色器有一些不同。

顶点着色器由于效率问题，需要直接从顶点数据接收输入。我们会用位置元数据指定输入变量，定义顶点数据的组织结构，在CPU上配置顶点属性。因此顶点着色器需要一个额外的布局说明，以便我们链接顶点数据。

由于片元着色器需要生成最终输出的颜色，所以它需要一个`vec4`颜色输出变量。如果在片元着色器中没有成功指定输出颜色，相应片元的颜色缓冲输出就会未定义。

所以，如果我们想要从一个着色器发送数据给另一个着色器，我们就需要在对应着色器上定义输出和输入。当输出变量类型和名称和输入变量一致时，OpenGL会在链接着色器程序对象时将变量链接在一起。

### Uniform

**uniform**是另一种从CPU向GPU上的着色器传递数据的方式，和顶点属性有些不同。首先，uniform是全局的，意味着它在一个着色器程序对象中是唯一的，可以被着色器程序中任何阶段的着色器所访问。其次，无论你（在着色器中）如何设置uniform的值，它都会保持不变，直到其被重新赋值或更新。

在GLSL中，我们使用`uniform`关键字在着色器中声明uniform变量。

```c
#version 330 core
out vec4 FragColor;

uniform vec4 ourColor; // we set this variable in the OpenGL code.

void main()
{
	FragColor = ourColor;
}
```

> [!note]
> 如果声明了没有在任何GLSL代码中使用过的uniform，编译器会自动将其移除。

要修改uniform变量，首先需要使用`glGetUniformLocation`函数在着色器中找到它的索引/位置，再使用`glUniform*`函数设置它的值，在设置uniform值之前还需要使用着色器程序（调用`glUseProgram`）。

```c
float timeValue = glfwGetTime();
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

> [!note]
> 由于OpenGL不支持函数重载，所以分别定义了不同参数类型的函数。例如这里的`glUniform`，要对特定类型的uniform调用该函数，需要通过一个对应的后缀来指定：
> - f：float
> - i：int
> - ui：unsigned int
> - 3f：3个float
> - fv：float向量/数组

### 更多属性

现在我们向顶点数据中添加颜色数据，分别指定三角形的三个顶点的颜色为红、绿、蓝。

```c
float vertices[] = {
	// positions       // colors
	0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f, // bottom left
	0.0f, 0.5f, 0.0f, 0.0f, 0.0f, 1.0f // top
};
```

相应地，我们需要在顶点着色器中添加颜色属性。

```c
#version 330 core
layout (location = 0) in vec3 aPos; // position has attribute position 0
layout (location = 1) in vec3 aColor; // color has attribute position 1

out vec3 ourColor; // output a color to the fragment shader

void main()
{
	gl_Position = vec4(aPos, 1.0);
	ourColor = aColor; // set ourColor to input color from the vertex data
}
```

直接使用`outColor`变量作为片元着色器的输出。

```c
#version 330 core
out vec4 FragColor;
in vec3 ourColor;

void main()
{
	FragColor = vec4(ourColor, 1.0);
}
```

现在VBO内存布局变成如下这样，我们还需要重新配置顶点属性指针。

![[Pasted image 20260313111609.png]]

```c
// position attribute
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
// color attribute
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float)));
glEnableVertexAttribArray(1);
```

我们只提供了3个颜色，但输出的图像是一个巨大的调色盘。这其实是片元着色器当中的**片元插值**导致的，当渲染三角形时，光栅化阶段通常会输出比一开始指定的顶点更多的片元，光栅化器会根据片元在三角形形状上的位置来决定它们的坐标，进而根据这些坐标对片元着色器的输入变量进行**插值**。

### 我们自己的着色器类

```c
#ifndef SHADER_H
#define SHADER_H
#include <glad/glad.h> // include glad to get the required OpenGL headers
#include <string>
#include <fstream>
#include <sstream>
#include <iostream>

class Shader
{
public:
	// the program ID
	unsigned int ID;
	// constructor reads and builds the shader
	Shader(const char* vertexPath, const char* fragmentPath);
	// use/activate the shader
	void use();
	// utility uniform functions
	void setBool(const std::string& name, bool value) const;
	void setInt(const std::string& name, int value) const;
	void setFloat(const std::string& name, float value) const;
};
#endif
```

该着色器类持有着色器程序的ID，由于我们希望将顶点和片元着色器源代码以普通文本文件的形式存储在硬盘上，所以其构造函数需要传入着色器源代码的文件路径。此外还添加了一些功能函数方便我们的使用，`use`函数负责激活着色器程序，`set`函数用于查询uniform位置并设置值。

### 读取文件

在构造函数中，我们使用C++的filestream读取着色器源代码文件，然后编译和链接着色器，并检查编译和链接是否成功。

```c
Shader(const char* vertexPath, const char* fragmentPath)
{
	// 1. retrieve the vertex/fragment source code from filePath
	std::string vertexCode;
	std::string fragmentCode;
	std::ifstream vShaderFile;
	std::ifstream fShaderFile;
	// ensure ifstream objects can throw exceptions:
	vShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);
	fShaderFile.exceptions (std::ifstream::failbit | std::ifstream::badbit);
	try
	{
		// open files
		vShaderFile.open(vertexPath);
		fShaderFile.open(fragmentPath);
		std::stringstream vShaderStream, fShaderStream;
		// read file’s buffer contents into streams
		vShaderStream << vShaderFile.rdbuf();
		fShaderStream << fShaderFile.rdbuf();
		// close file handlers
		vShaderFile.close();
		fShaderFile.close();
		// convert stream into string
		vertexCode = vShaderStream.str();
		fragmentCode = fShaderStream.str();
	}
	catch(std::ifstream::failure e)
	{
		std::cout << "ERROR::SHADER::FILE_NOT_SUCCESFULLY_READ" << std::endl;
	}
	const char* vShaderCode = vertexCode.c_str();
	const char* fShaderCode = fragmentCode.c_str();
	[...]
	// 2. compile shaders
	unsigned int vertex, fragment;
	int success;
	char infoLog[512];
	// vertex Shader
	vertex = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertex, 1, &vShaderCode, NULL);
	glCompileShader(vertex);
	// print compile errors if any
	glGetShaderiv(vertex, GL_COMPILE_STATUS, &success);
	if(!success)
	{
		glGetShaderInfoLog(vertex, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" <<
		infoLog << std::endl;
	};
	// similiar for Fragment Shader
	[...]
	// shader Program
	ID = glCreateProgram();
	glAttachShader(ID, vertex);
	glAttachShader(ID, fragment);
	glLinkProgram(ID);
	// print linking errors if any
	glGetProgramiv(ID, GL_LINK_STATUS, &success);
	if(!success)
	{
		glGetProgramInfoLog(ID, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" <<
		infoLog << std::endl;
	}
	
	// delete shaders; they’re linked into our program and no longer necessary
	glDeleteShader(vertex);
	glDeleteShader(fragment);
}
```

`use`函数：

```c
void use()
{
	glUseProgram(ID);
}
```

uniform的`set`函数：

```c
void setBool(const std::string &name, bool value) const
{
	glUniform1i(glGetUniformLocation(ID, name.c_str()), (int)value);
}

void setInt(const std::string &name, int value) const
{
	glUniform1i(glGetUniformLocation(ID, name.c_str()), value);
}

void setFloat(const std::string &name, float value) const
{
	glUniform1f(glGetUniformLocation(ID, name.c_str()), value);
}
```

## 纹理

纹理是一种二维图像，用于给物体添加细节，也可以存储任何用于发送给着色器的数据。

为了将纹理映射到三角形上，我们需要告诉三角形的每一个顶点，它们各自对应纹理的哪一部分。每个顶点都有一个对应的**纹理坐标**，用于采样纹理图像。

纹理坐标的x和y轴范围从0到1，左下角为(0,0)，右上角坐标为(1,1)，使用纹理坐标取出纹理颜色的过程叫做**采样**。

### 纹理环绕

当纹理坐标超出范围时，OpenGL默认会对纹理进行重复（忽略坐标的整数部分），其他选项包括：

- `GL_REPEAT`：重复纹理图像，默认行为
- `GL_MIRRORED_REPEAT`：每次重复会镜面翻转
- `GL_CLAMP_TO_EDGE`：将坐标收缩在0到1之间
- `GL_CLAMP_TO_BORDER`：超出范围的部分变成指定的边框颜色

可以使用`glTexParameteri`函数对每个坐标轴分别设置纹理环绕模式。

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```

如果选择`GL_CLAMP_TO_BORDER`模式，还需要额外指定边框颜色。

```c
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

### 纹理过滤

纹理坐标是与分辨率无关的任意浮点值，所以OpenGL必须清楚哪一个纹理像素（也叫做**纹素**）应该映射到一个纹理坐标，这就是**纹理过滤**。现在我们只讨论最重要的两种选项：`GL_NEAREST`和`GL_LINEAR`。

`GL_NEAREST`（也叫做**最近邻过滤**或**点过滤**）是OpenGL纹理过滤的默认方式，OpenGL会选择中心最接近纹理坐标的纹素。

`GL_LINEAR`（也叫做**线性过滤**）会从纹理坐标的邻近纹素中计算插值，得到一个近似颜色。纹素中心越靠近纹理坐标，对采样颜色的贡献就越大。

纹理过滤可以设置放大和缩小操作，所以可以在纹理缩小时使用最近邻过滤，在纹理放大时使用线性过滤。

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

#### 多级纹理

由于离摄像机很远的物体可能只产生几个片元，OpenGL很难从高分辨率的纹理中取出正确的颜色。在小物体上使用高分辨率问题不仅会浪费内存带宽，还会造成伪影。

为了解决这个问题，OpenGL使用了一种叫做`多级纹理`的概念，简单来说，它是一个纹理图像的集合，每一个纹理的大小是前一个的一半。

当观察者距离物体超出一个特定阈值后，OpenGL就会使用一个最适合该物体当前距离的纹理。由于物体非常远，用户不会注意到分辨率降低。这样OpenGL就能采样到正确的纹素，并且在采样时需要的缓存更少。

在创建纹理之后，通过`glGenerateMipmaps`函数，OpenGL就能帮我们自动创建出多级纹理。

在不同的多级纹理之间切换时，可能会出现一些像尖锐边缘的伪影。和普通纹理过滤一样，我们可以在切换多级纹理时使用纹理过滤，有以下四种选项：

- `GL_NEAREST_MIPMAP_NEAREST`: 选取最近的多级纹理匹配像素尺寸，使用最近邻插值进行纹理采样
- `GL_LINEAR_MIPMAP_NEAREST`: 选取最近的多级纹理级别，使用线性插值
- `GL_NEAREST_MIPMAP_LINEAR`: 在最匹配像素尺寸的两个多级纹理之间线性插值，通过最近邻插值采样插值级别
- `GL_LINEAR_MIPMAP_LINEAR`: 在两个最接近的多级纹理之间线性插值，通过线性插值采样插值级别

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### 加载和创建纹理

在使用纹理之前，我们必须将它们加载进来。纹理图像的格式有很多种，自己写加载方法是一件麻烦事，所以我们会使用一个图像加载库——stb_iamge.h。

### stb_iamge.h

[stb_image.h](https://github.com/nothings/stb/blob/master/stb_image.h)支持加载大多数流行文件格式，容易集成到项目中，下载后只需要将stb_iamge.h文件添加到工程中，然后创建一个C++文件：

```c
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```

通过定义`STB_IMAGE_IMPLEMENTATION`，让预处理器修改头文件，使其只包含相关的定义源代码，以便捷地将头文件变成`.cpp`文件。

使用`stbi_load`函数加载图像：

```c
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
```

### 创建纹理

和其他OpenGL对象一样，纹理也通过ID引用。

```c
unsigned int texture;
glGenTextures(1, &texture);
```

绑定纹理：

```c
glBindTexture(GL_TEXTURE_2D, texture);
```

使用`glTexImage2D`函数创建纹理并生成多级纹理：

```c
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```

最后释放图像内存：

```c
stbi_image_free(data);
```

### 使用纹理

我们要将纹理绘制在一个矩形上，首先在顶点数据中添加纹理坐标：

```c
float vertices[] = {
	// positions      // colors         // texture coords
	0.5f, 0.5f, 0.0f, 1.0f, 0.0f, 0.0f, 1.0f, 1.0f, // top right
	0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, 0.0f, 0.0f, 1.0f, 0.0f, 0.0f, // bottom left
	-0.5f, 0.5f, 0.0f, 1.0f, 1.0f, 0.0f, 0.0f, 1.0f // top left
};
```

配置顶点属性：

```c
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
glEnableVertexAttribArray(2);
```

在顶点着色器中接收纹理坐标并传递给片元着色器：

```c
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
	gl_Position = vec4(aPos, 1.0);
	ourColor = aColor;
	TexCoord = aTexCoord;
}
```

在片元着色器中，使用GLSL提供的纹理对象类型`sampler`来访问纹理，通过声明一个`uniform sampler2D`就可以添加一张纹理，并使用GLSL内置的`texture`函数对纹理进行采样。

```c
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main()
{
	FragColor = texture(ourTexture, TexCoord);
}
```

最后，在调用`glDrawElements`之前绑定纹理，纹理将自动赋值给片元着色器的sampler。

```c
glBindTexture(GL_TEXTURE_2D, texture);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

### 纹理单元

`glUniformli`可以为纹理采样器指定一个位置，让我们可以在一个片元着色器中一次使用多个纹理。纹理所在的位置称为一个**纹理单元**，默认为0。

因为默认激活的纹理单元为0，所以在绑定多张纹理时，要先使用`glActiveTexture`激活对应的纹理单元。

```c
glActiveTexture(GL_TEXTURE0); // activate texture unit first
glBindTexture(GL_TEXTURE_2D, texture);
```

> [!note]
> OpenGL至少可以使用16个纹理单元，从`GL_TEXTURE0`到`GL_TEXTURE15`。在访问`GL_TEXTURE8`时也可以使用`GL_TEXURE0+8`，这在循环纹理单元时非常有用。

在片元着色器中添加另一个纹理采样器：

```c
#version 330 core

uniform sampler2D texture1;
uniform sampler2D texture2;

void main()
{
	FragColor = mix(texture(texture1, TexCoord),
	texture(texture2, TexCoord), 0.2);
}
```

然后加载第二张纹理：

```c
unsigned char *data = stbi_load("awesomeface.png", &width, &height, &nrChannels, 0);

if (data)
{
	glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGBA,
	GL_UNSIGNED_BYTE, data);
	glGenerateMipmap(GL_TEXTURE_2D);
}
```

由于这次是`.png`格式的图片，我们需要使用`GL_RGBA`说明图片数据中包含透明度通道。

```c
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

glBindVertexArray(VAO);
```

我们还需要告诉OpenGL纹理单元和每一个着色器中的采样器的对应关系。

```c
// before enter the render loop:
shader.use(); // don’t forget to activate the shader first!
glUniform1i(glGetUniformLocation(shader.ID, "texture1"), 0); // manually
shader.setInt("texture2", 1); // or with shader class
```

由于图像原点位于左上角，而OpenGL的原点位于左下角，还需要在加载图像时对y轴进行翻转。

```c
stbi_set_flip_vertically_on_load(true);
```

## 变换

### GLM

GLM即OpenGL Mathematics，是一个只包含头文件的库。

大多数GLM功能都可以在以下3个头文件中找到：

```c
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```

如何在着色器中使用变换矩阵呢？我们可以声明一个`mat4`类型的uniform变量：

```c
uniform mat4 transform;
```

构建矩阵：

```c
glm::mat4 trans = glm::mat4(1.0f);
trans = glm::translate(trans, glm::vec3(0.5f, -0.5f, 0.0f));
trans = glm::rotate(trans, (float)glfwGetTime(), glm::vec3(0.0f, 0.0f, 1.0f));
```

然后将变换矩阵发送给着色器：

```c
unsigned int transformLoc = glGetUniformLocation(ourShader.ID,"transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```

## 坐标系统

OpenGL希望在执行完顶点着色器之后，所有的顶点都在标准设备坐标下。

将顶点坐标转换到标准设备坐标是一个分步进行的过程，因为其中的一些操作在特定的坐标系统下会更简单，所以我们会使用一些中间坐标系统，对我们来说最重要的是以下5个：

- 本地空间（模型空间）
- 世界空间
- 视图空间
- 裁剪空间
- 屏幕空间

在各个坐标空间之间转换需要用到一些变换矩阵，最关键的是模型、视图和投影矩阵。顶点坐标首先以**本地坐标**位于**本地空间**，然后依次变换为**世界坐标**、**视图坐标**、**裁剪坐标**，最终结束于**屏幕坐标**。

下图展示了整个过程和相应的变换矩阵。

![[Pasted image 20260317103004.png]]

要绘制3D场景，我们首先要创建一个模型矩阵，将物体的顶点变换到世界空间。这里将平面绕x轴旋转一些，让它看起来像是倒在地面上：

```c
glm::mat4 model = glm::mat4(1.0f);
model = glm::rotate(model, glm::radians(-55.0f), glm::vec3(1.0f, 0.0f, 0.0f));
```

然后我们再创建一个视图矩阵，为了让物体稍微往前移动而可见，换一种思考方式就是让摄像机往后移动。

```c
glm::mat4 view = glm::mat4(1.0f);
// note that we’re translating the scene in the reverse direction
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f));
```

最后是投影矩阵：

```c
glm::mat4 projection;
projection = glm::perspective(glm::radians(45.0f), 800.0f / 600.0f, 0.1f, 100.0f);
```

在顶点着色器中定义这三个矩阵的uniform变量，并对顶点进行变换输出到`gl_Position`。

```c
#version 330 core
layout (location = 0) in vec3 aPos;
...
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
	// note that we read the multiplication from right to left
	gl_Position = projection * view * model * vec4(aPos, 1.0);
	...
}
```

将矩阵发送给着色器：

```c
int modelLoc = glGetUniformLocation(ourShader.ID, "model");
glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));
... // same for View Matrix and Projection Matrix
```

---

下面我们将二维平面扩展为三维的立方体。

要渲染一个立方体，我们总共需要36个顶点（6个面，每个面两个三角形三个顶点）。

```c
float vertices[] = {
-0.5f, -0.5f, -0.5f, 0.0f, 0.0f,
0.5f, -0.5f, -0.5f, 1.0f, 0.0f,
0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
...
}
```

让立方体随时间旋转：

```c
model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), glm::vec3(0.5f, 1.0f, 0.0f));
```

使用`glDrawArrays`绘制立方体。

```c
glDrawArrays(GL_TRIANGLES, 0, 36);
```

这时会发现立方体的绘制并不完整，这是因为OpenGL是逐三角形逐片元进行绘制的，它会覆盖已经绘制过的像素。由于OpenGL不能保证三角形渲染的顺序，一些三角形的会相互重叠绘制。

幸运的是，OpenGL会将深度信息存储在一个叫做**z缓冲区**（深度缓冲）的地方，它可以让OpenGL决定何时在某个像素上绘制以及何时不进行绘制。我们可以使用z缓冲配置OpenGL，进行深度测试。

GLFW会自动为我们创建深度缓冲区，每个片元都会将深度值存储为它的z值，当片元将要输出颜色时，OpenGL会将它的深度值和深度缓冲进行比较，以决定是否丢弃已有的片元，这个过程就是**深度测试**。

因为OpenGL默认是将深度测试关闭的，我们需要使用`glEnable(GL_DEPTH_TEST)`开启深度测试。在每一次渲染循环中，我们还要清除深度缓冲区，这和颜色缓冲区是一个道理。

```c
glEnable(GL_DEPTH_TEST);

// in render loop
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

## 摄像机

OpenGL中并没有摄像机的概念，但是我们可以尝试通过将场景中的物体朝相反方向移动来模拟，制造一种我们自身在移动的假象。

### 摄像机/观察空间

摄像机/观察空间指的是顶点以摄像机视角作为场景原点时的相对坐标，观察矩阵将世界坐标变换为相对摄像机位置的方向的观察坐标。

定义一个摄像机，我们需要其世界空间坐标，看向的方向，指向右端的方向以及指向顶端的方向，实际上就是将摄像机位置作为原点，然后用3个相互垂直的轴构建一个坐标系统。

![[Pasted image 20260319104949.png]]

#### 摄像机位置

摄像机的位置就是一个世界空间中的向量，其指向摄像机所在位置。

别忘了正z轴是指向屏幕外的自己的，所以如果想让摄像机向后移动，应该向正z轴移动。

```c
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
```

#### 摄像机方向

摄像机方向是摄像机指向的方向，用摄像机位置减去场景的原点即可得到。

```c
glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
```

#### 右方向轴

右方向向量用于表示摄像机空间的正x轴，我们可以先指定一个世界空间的上方向向量，然后用它和摄像机方向叉乘。

```c
glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f);
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
```

#### 上方向轴

上方向轴表示的是摄像机空间的正y轴，用右方向和摄像机方向叉乘即可得到。

```c
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
```

### Look At

使用上面三个轴和一个平移向量，我们可以组合出一个LookAt矩阵。

$$
LookAt=
\left [
\begin{matrix}
R_x & R_y & R_z & 0 \\
U_x & U_y & U_z & 0 \\
D_x & D_y & D_z & 0 \\
0 & 0 & 0 & 1 \\
\end{matrix}
\right ]
*
\left [
\begin{matrix}
1 & 0 & 0 & -P_x \\
0 & 1 & 0 & -P_y \\
0 & 0 & 1 & -P_z \\
0 & 0 & 0 & 1 \\
\end{matrix}
\right ]
$$

```c
glm::mat4 view;
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f));
```

### 移动

首先在我们的程序顶部定义一些摄像机变量，用于配置摄像机系统。

```c
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 3.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);
```

LookAt函数现在变成这样：

```c
view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
```

在`processInput`函数中处理移动输入：

```c
void processInput(GLFWwindow *window)
{
	...
	const float cameraSpeed = 0.05f; // adjust accordingly
	if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
		cameraPos += cameraSpeed * cameraFront;
	if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
		cameraPos -= cameraSpeed * cameraFront;
	if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
		cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
	if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
		cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
}
```

### 移动速度

目前我们使用一个常量作为移动速度，由于每个人的电脑配置不一样，有的人每秒渲染的帧数可能更多，这会导致有些人移动得更快，而有些人比较慢。

图形应用和游戏通常会记录一个`deltatime`变量，用来保存上一帧的渲染时间。我们将这个变量乘以移动速度，就能确保每个人的体验是相同的。

我们需要记录两个全局变量来计算`deltatime`：

```c
float deltaTime = 0.0f; // Time between current frame and last frame
float lastFrame = 0.0f; // Time of last frame
```

每一帧计算新的`deltatime`：

```c
float currentFrame = glfwGetTime();
deltaTime = currentFrame - lastFrame;
lastFrame = currentFrame;
```