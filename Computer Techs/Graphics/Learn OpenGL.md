# Getting started

## OpenGL

OpenGL经常被认为是一种给我们提供操作图形和图像的函数集的API，但是OpenGL本身并不是API，而仅仅是由Khronos开发和维护的一种规范标准。

OpenGL规范指明了每一个函数的输出结果和表现方式，然后再由开发者实现函数的具体操作。只要这些函数的结果遵循了规范，不同的OpenGL版本都可以有不同的实现。

通常是由显卡制造商开发实际的OpenGL库，每一张显卡所支持的特定OpenGL版本都是针对于该显卡系列开发的版本。这意味着当OpenGL出现异常行为时，很大可能是显卡制造商或库开发者的过错。

## 核心模式vs即时模式

在3.2版本之前，OpenGL使用一种叫做即时模式的方法来绘制图形，虽然易于理解和使用，但大多数功能隐藏于库中，开发者对OpenGL没有太多的控制权，而且效率极低。

从3.2版本开始，即时模式被弃用，并鼓励开发者转向使用核心模式，这也是OpenGL规范移除所有老旧废弃功能的分界线。

使用核心模式时，OpenGL会强制我们使用更加灵活高效的现代方法，当然也更难学习，因为着要求开发者真正理解OpenGL和图形编程。

从3.3之后的版本都只是增加了一些高效的新特性，并没有改变核心机制，因此没有必要一开始就选择最新版本进行学习。

最新特性往往只有在最现代的显卡上才能运行，所以大多数开发者普遍针对较低版本的OpenGL进行开发，再选择性地启用高版本功能。

## 扩展

当图形公司研究出一项新技术或渲染上的重大优化时，常常会在驱动中添加扩展。如果程序运行的硬件支持某种扩展，开发者就可以使用该扩展所提供的功能。

这样图形开发者不需要等待OpenGL将新功能收纳，就可以使用这些新的渲染技术。当一项扩展流行起来后，OpenGL就会将其收纳到未来的版本当中。

## 状态机

OpenGL本身就是一个超大的状态机，其中包含了一堆变量来定义当前OpenGL应该如何操作。OpenGL的状态一般指的是OpenGL上下文（context），当使用OpenGL时，我们会通过设置一些选项、操控一些缓冲区来改变其状态，然后使用当前上下文进行渲染。

在使用OpenGL时会遇到一些用于更改上下文的**状态改变**(state-changing)方法，和一些基于当前状态执行操作的**状态使用**(stage-using)方法，只要记住OpenGL本质上是一个巨大的状态机，那么它的大多数功能就更容易理解了。

## 对象

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

