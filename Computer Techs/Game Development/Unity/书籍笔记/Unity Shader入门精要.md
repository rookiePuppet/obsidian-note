
# 渲染流水线

渲染流水线的最终目的：生成或者渲染一张二维纹理。

## 何为流水线

将产品的生产流程分为**多个步骤**，每个步骤**并行进行**，以提高整体效率。

比如，将一个生产流程分为n个流水线阶段来同时进行，若每个阶段所花费时间相同，那么相比原来的非流水线流程就得到了n倍的速度提升。

流水线系统中决定最终生产速度的是最慢的工序所需的时间，这个**最慢的工序就是性能的瓶颈**。

## 什么是渲染流水线

将流水线的思想应用到计算机的**图像渲染**上就得到了渲染流水线。

渲染所做的工作就是将一个三维场景转换成一张二维图像，是由CPU和GPU共同完成的。

渲染流水线可以分为三个阶段：**应用阶段**、**几何阶段**和**光栅化阶段**。

### 应用阶段

该阶段由应用主导，由CPU负责实现，开发者在此阶段有绝对控制权。

开发者在该阶段的三个任务：

1. 准备场景数据，例如摄像机、模型、光源等。
2. 剔除不可见的物体，以提高渲染性能。
3. 设置渲染状态，即模型的材质、纹理和Shader等。

该阶段的输出是渲染所需的几何信息，即渲染图元（点、线、三角面等）。

### 几何阶段

该阶段主要处理绘制的几何相关的事情，在GPU上进行。

一个重要任务是将顶点坐标变换到屏幕空间，再交给光栅器进行处理。

### 光栅化阶段

该阶段利用几何阶段的数据产生屏幕上的像素，并渲染出最终图像，在GPU上进行。

光栅化的主要任务：决定每个渲染图元中的哪些像素应该被绘制在屏幕上。

## CPU和GPU之间的通信

### 将数据加载到显存中

渲染所需的数据一开始是存放在硬盘上的，首先需要将它们加载到系统内存中，再加载到显存中。因为显卡对显存的访问速度更快，而且显卡没有直接访问内存的权力。

### 设置渲染状态

渲染状态决定场景中的网格如何被渲染，例如着色器、光源属性、材质等。

只有设置了正确的渲染状态，模型才能以我们预期的效果进行渲染。

### 调用Draw Call

Draw Call就是一个CPU向GPU发出的渲染指令，让GPU根据顶点数据和渲染状态进行渲染，最终将图像输出到屏幕上。

## GPU流水线

GPU流水线指的是几何阶段和光栅化阶段的统称，这两个阶段还可以细分为更小的流水线阶段.

不同阶段具有不同的可配置性和可编程性，如下，粗体部分表示完全可编程，斜体部分表示可配置，其他部分由GPU固定实现。

> 几何阶段

**顶点着色器**$\rightarrow$**曲面细分着色器**$\rightarrow$**几何着色器**$\rightarrow$*裁剪*$\rightarrow$屏幕映射

> 光栅化阶段

三角形设置$\rightarrow$三角形遍历$\rightarrow$**片元着色器**$\rightarrow$*逐片元操作*

### 顶点着色器

输入到顶点着色器的每一个顶点都会执行一次顶点着色器，每个顶点之间是相互独立的。

顶点着色器主要的工作是坐标变换和逐顶点光照。

**坐标变换**指的是将顶点坐标从模型空间变换到另一个指定的空间。

在顶点着色器中必须完成的是**将顶点坐标从模型空间变换到齐次裁剪空间**，然后由硬件做透视除法得到归一化的设备坐标（Normalized Device Coordinates，NDC）。

### 裁剪

裁剪就是将在摄像机视野范围外的部分移除，让这些看不到的位置不参与渲染。

例如一条线段的一个顶点在视野范围内，而另一条不在，那么视野外的这个顶点就会被使用一个新的顶点所替代，新的顶点是线段和视野边界的交点。

### 屏幕映射

屏幕映射的任务是把每个图元的x、y坐标转换到屏幕坐标系中，而z坐标不做任何处理。

### 三角形设置

该步骤主要计算光栅化一个三角网格所需的信息，因为从上一阶段输出的是三角网格的顶点，我们要计算出各个三角网格覆盖到了哪些像素。

### 三角形遍历

该步骤会检查每个像素是否被一个三角网格所覆盖，如果被覆盖就会生成一个片元。

这里生成的片元并不是真正意义上的像素，它包含了屏幕坐标、深度以及法线等等的信息，这些信息用于计算像素的最终颜色。

### 片元着色器

片元着色器的输入是上一阶段对顶点信息插值得到的结果，输出的是颜色。

### 逐片元操作

该阶段主要的任务是：

1. 决定每个片元的**可见性**，例如进行模板测试、深度测试等。
2. 将通过测试的片元的颜色和颜色缓冲区中的颜色进行**合并**（混合）。

> 模板测试

GPU首先读取片元位置对应在模板缓冲区中的模板值，将该值和参考值进行比较，没有通过测试的片元会被舍弃。

还可以根据模板测试的结果来修改模板缓冲区，例如在测试通过时将对应模板缓冲区的值加1。

模板测试通常用于限制渲染的区域，另外还可以用于渲染阴影、轮廓等。

> 深度测试

通过了模板测试的片元会进行深度测试。

GPU将片元的深度和深度缓冲区中的深度值进行比较，通常情况下，深度值大于或等于已经存在于深度缓冲区中的值时，片元被舍弃。

通过深度测试后，可以选择是否将新的深度值写入深度缓冲区。

> 合并

如果在某次渲染时，发现颜色缓冲区对应位置上已经存在颜色值，这时就需要进行合并操作。

对于不透明物体，需要关闭混合，让片元着色器的结果直接覆盖颜色缓冲区中的值。

对于半透明物体，需要开启混合，来达到透明效果。

如果开启了混合，GPU会将源颜色和目标颜色进行混合（使用一个混合函数），得到混合之后的颜色再存进颜色缓冲区。

---

GPU会希望尽可能地在执行片元着色器之前进行各项的测试，以尽早知道哪些片元是要舍弃的，以**提高性能**。

在Unity的渲染流水线中，深度测试在片元着色器之前进行，这种技术被称为**Early-Z**。

但是，将这些测试提前可能会与片元着色器中的一些操作产生冲突，例如透明度测试。

假设在片元着色器中进行透明度测试，对于没有通过透明度测试的片元，我们会调用API来手动将其舍弃。但是，由于现代GPU会判断片元着色器中的操作是否和提前测试产生冲突，如果产生冲突，则会禁用提前测试。

不能进行提前测试也就意味着可能有许多不必要的片元被处理了，从而导致性能下降。

模型的图片经过一系列的计算和测试后，就会显示到屏幕上。但是，一帧画面可能无法一瞬间就完整地绘制出来，为了避免用户看到光栅化的过程，GPU会采用**双重缓冲**策略。

在渲染场景时，渲染的结果会先存放在**后置缓冲**中，渲染完成后再和**前置缓冲**交换内容，以此来保证图像的连续性。

## 容易困惑的地方

### OpenGL/DirectX

OpenGL和DirectX都是图形编程的**接口**，用于渲染二维或三维图形，这些接口方便了我们和GPU的交流。

应用程序通过调用这些图形接口，将渲染所需的数据存放在显存中，然后发出渲染指令（Draw Call）。

**显存**是显卡中的一块内存区域，可以存放任何类型的数据，对于渲染来说，一般有图像缓冲、深度缓存、纹理等等。

另外，在图形接口和GPU之间其实还有一个重要角色——**显卡驱动**。显卡驱动扮演的是一个中间人，它可以将开发者通过图形接口调用的渲染指令翻译成GPU能够理解的语言，让GPU进行实际的绘制工作。

### HLSL/GLSL/CG

在渲染管线中，有一些可编程的阶段，例如顶点着色器和片元着色器，HLSL/GLSL/CG就是这些着色器所使用的语言。

**着色语言**（Shading Language）专门用于编写着色器程序，而且它们是一种高级语言，首先会被编译成汇编语言，最终会被显卡驱动编译成机器语言。

> 区别是什么

GLSL是OpenGL使用的语言，其优点在于**跨平台**性。但由于OpenGL本身不提供着色器编译器，而是由显卡驱动来进行着色器编译，也就是说它**依赖于硬件**。这意味着GLSL的编译结果取决于硬件供应商，可能导致编译结果不一致的情况。

HLSL是DirectX使用的语言，由微软提供着色器编译器，**不依赖于硬件**。但这样造成的结果是，HLSL几乎**只支持微软**的产品，例如Windows、Xbox等。

CG是由NVIDIA开发的着色语言，它会根据不同平台将着色器编译成对应的中间语言，做到了真正意义上的**跨平台**。CG可以从HLSL代码无缝移植，缺点是可能无法完全发挥OpenGL的最新特性。

### Draw Call

Draw Call就是CPU调用图像编程接口的指令，例如OpenGL中的glDrawElements命令。

> CPU和GPU如何并行工作

为了让CPU和GPU并行工作，会使用一个**命令缓冲区**（命令队列）。

当需要渲染时，CPU会向缓冲区中添加命令，然后GPU不断从缓冲区中读取命令并执行。

Draw Call就是这些命令中的一种，除此之外还有改变渲染状态的命令。

> 为什么Draw Call影响帧率

在每次调用Draw Call之前，CPU都要向GPU发送很多内容，相对来说比较耗时，而GPU渲染是极快的，远远快于CPU提交命令的速度。

如果Draw Call数量过多，CPU就会把大量时间花费在提交Draw Call上，造成CPU过载。

> 如何减少Draw Call

**批处理**（Batching）是减少Draw Call的一种方法，主要思想就是将多次Draw Call合并成一次。

合并网格的过程是需要消耗时间的，所以批处理比较适合静态物体。对于动态物体，每一帧都需要重新进行合并再发送给GPU，会在空间和时间上造成一定的开销。

在游戏开发中，为了减少Draw Call需要注意：

1. 避免使用大量很小的网格，不可避免时考虑合并它们。
2. 避免使用过多材质，不同的网格尽量共用材质。

### 固定管线渲染

**固定函数流水线**（Fixed Function Pipeline），通常是指在比较旧的GPU上实现的渲染流水线，开发者只有一些可配置的操作，无法完全控制。

随着GPU的发展，固定管线已经逐渐过时。除非要兼容旧设备，否则不建议使用。

# Unity Shader

编写Unity Shader需要使用Unity提供的ShaderLab语言。

## 基础结构

一个Unity Shader的基础结构如下：

```C
// Shader名字中可以添加"/"，控制该Shader在材质面板中的目录位置
Shader ".../ShaderName"
{

	// 定义显示在材质面板中的属性
	Properties 
	{
		// 属性名字一般以“_"为前缀
		// 属性的类型有Int、Float、Range、Color、Vector、2D、3D、Cube
		// 对于Int、Float和Range类型，属性的默认值就是一个数字
		// 对于Color和Vector，默认值是一个四维向量，使用圆括号包裹
		// 对于2D、3D和Cube，默认值是一个字符串加一对花括号，字符串要么是空，要么是内置的纹理名称
		PropertyName("Display Name", PropertyType) = DefaultValue
		// ...
	}
	
	// SubShader至少要有一个，可以有多个
	// Unity会从SubShader中选择能够在目标平台运行的一个使用，如果都不支持，就会使用Fallback指定的Shader
	// 因为不同显卡有不同的能力，对于高级的显卡，我们希望尽可能地发挥更强大渲染能力
	SubShader 
	{
		// 标签（可选）
		// 使用字符串键值对形式定义
		// 1. Queue：指定渲染队列，控制渲染顺序
		// 2. RenderType：着色器分类，用于着色器替换
		// 3. DisableBatching：禁用批处理
		// 4. ForceNoShadowCasting： 控制阴影投射
		// 5. IgnoreProjector：忽略投影器
		// 6. CanUseSpriteAtlas： 是否用于精灵
		// 7. PreviewType：材质面板的默认预览形状
		[Tags]
		
		// 状态（可选）
		// 在SubShader中设置将会应用到该SubShader所有的Pass
		// 1. Cull 剔除模式：Back | Front | Off
		// 2. ZTest  深度测试函数：Less Greater | LEqual | GEqual ...
		// 3. ZWrite 深度写入：On | Off
		// 4. Blend 混合模式：SrcFactor DstFactor
		[RenderSetup]
		
		Pass
		{
			// Pass的名称
			// 可以使用UsePass命令直接使用其他Shader中的Pass
			[Name]
			
			// 标签
			// 1. LightMode：定义该Pass在渲染流水线中的角色
			// 2. RequireOptions：指定当满足某些条件时才渲染该Pass
			[Tags]
			
			// 和SubShader中的一样
			[RenderSetup]
			
			// 其他代码
		}
	}
	// 其他SubShader
	
	// 指定备用Shader
	// 当上面的SubShader都无法运行时，就会使用这个备用的Shader
	Fallback "AnotherShader"
}
```

## 不同类型的着色器

### 表面着色器

表面着色器是Unity自己创造的一种着色器类型，代码量更少，但渲染代价较大。

表面着色器最终都会转换成顶点/片元着色器，所以它们并没有本质上的区别，它的主要加载在于为我们处理了很多光照细节。

下面是一个简单的表面着色器示例：

```C
Shader "Custom/Simple surface Shader"
{
	SubShader
	{
		Tags { "RenderType"="Opaque" }
		CGPROGRAM
		#pragma surface surf Lambert
		
		struct Input
		{
			float4 color: COLOR;
		}
		
		void surf(Input IN, inout SurfaceOutput o)
		{
			o.Albedo = 1;
		}
		ENDCG
	}
	Fallback "Diffuse"
}
```

### 顶点/片元着色器

顶点/片元着色器相对更复杂，但灵活性也更高。

下面是一个简单的顶点/片元着色器示例：

```C
Shader "Custom/Simple VertexFragment Shader"
{
	SubShader
	{
		Pass
		{
			Tags { "RenderType"="Opaque" }
			
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
	
			float4 vert(float4 v: POSITION): SV_POSITION
			{
				return mul(UNITY_MATRIX_MVP, v);
			}
			
			fixed4 frag(): SV_Target
			{
				return fixed4(1.0, 1.0, 1.0, 1.0);
			}
			ENDCG
		}
	}
	Fallback "Diffuse"
}
```

## 最简单的顶点/片元着色器

```C
CGPROGRAM
#pragma vertex vert
#pragma fragment frag

float4 vert(float4 v: POSITION): SV_POSITION
{
	return mul(UNITY_MATRIX_MVP, v);
}

fixed4 frag(): SV_Target
{
	return fixed4(1.0, 1.0, 1.0, 1.0);
}
ENDCG
```

### 获取模型的更多数据

要获取更多和模型相关的数据，例如顶点的纹理坐标、法线等，就需要先声明一个结构体。

```C
struct a2v
{
	float4 vertex: POSITION;
	float3 normal: NORMAL;
	float4 texcoord: TEXCOORD0;
}
```

将该结构体类型作为顶点着色器函数的变量类型即可。

通过语义来指定的数据是由`Mesh Renderer`组件提供的。每次调用Draw Call时，`Mesh Renderder`就会把它负责渲染的模型数据发送给Unity Shader。

### 顶点与片元着色器之间的数据传递

可以将模型的法线、纹理坐标等数据通过顶点着色器传递给片元着色器，也是需要声明一个结构体。

```C
struct v2f
{
	float4 pos: SV_POSITION;
	fixed3 color: COLOR0;
}
```

将该结构体类型作为顶点着色器函数的返回类型，并在函数中将相应数据存储在结构体变量中，最后返回。

另外在片元着色器函数中，将该结构体类型作为输入变量类型。

顶点着色器是逐顶点调用的，而片元着色器是逐片元调用的，因此片元着色器的输入实际上是顶点着色器的输出进行插值的结果。

### 使用属性

材质提供的参数需要在`Properties`语句块中定义。

比如在材质面板上显示一个颜色拾取器，则需要定义如下属性。

```C
Properties
{
	_Color("Color Tint", Color) = (1,1,1,1)
}
...
SubShader
{
	Pass
	{
		CGPROGRAM
		...
		fixed4 _Color;
		...
		ENDCG
	}
}
...
```

除了在`Properties`语句块中定义属性，还需要再CG代码块中声明相应类型的变量，如上使用`fixed4`类型对应`Color`类型。

ShaderLab属性类型和CG变量类型的对应关系如下表所示。

| ShaderLab属性类型 | CG变量类型                |
| ------------- | --------------------- |
| Color, Vector | float4, half4, fixed4 |
| Range, Float  | float, half, fixed    |
| 2D            | sampler2D             |
| Cube          | samplerCube           |
| 3D            | sampler3D             |

## CG/HLSL语义

### 语义的概念

**语义**实际上是一个字符串，用于表达**参数的含义**，让Shader知道从哪里读取数据，以及把数据输出到哪里。

通常情况下，输入输出变量不需要有特别的意义，我们可以自行决定变量的用途。

但在Unity中也有例外情况，比如顶点着色器输入结构体中用`TEXCOORD0`来描述texcoord变量，这表示将模型的第一组纹理坐标填充到texcoord中。

有一种以`SV`开头的语义称为**系统数值语义**，在渲染管线中有特殊含义。

例如使用`SV_POSITION`去修饰顶点着色器的输出变量pos，就表示它包含了可用于光栅化的变换后的顶点坐标（齐次裁剪空间中的坐标）。

在大多数平台上，`SV_POSITION`和`POSITION`是等价的，有一些Shader可能会使用`POSITION`语义修饰顶点着色器输出，但是为了更好的跨平台性，对于这种有特殊含义的变量，最好使用`SV`开头的语义。

### Unity支持的语义

> 应用程序传递给顶点着色器支持的语义

| 语义        | 描述                      |
| --------- | ----------------------- |
| POSITION  | 模型空间中的顶点坐标，float4类型     |
| NORMAL    | 顶点法线，float3类型           |
| TANGENT   | 顶点切线，float4类型           |
| TEXCOORDn | 顶点的纹理坐标，float2或float4类型 |
| COLOR     | 顶点颜色，float4或fixed4类型    |

>  顶点着色器传递给片元着色器支持的语义

| 语义          | 描述                |
| ----------- | ----------------- |
| SV_POSITION | 裁剪空间中的顶点坐标        |
| COLOR0      | 通常用于输出第一组顶点颜色，非必需 |
| COLOR1      | 通常用于输出第二组顶点颜色，非必需 |
| TEXCOORDn   | 通常用于输出纹理坐标，非必需    |

> 片元着色器输出支持的语义

| 语义        | 描述            |
| --------- | ------------- |
| SV_Target | 输出值将会存储到渲染目标中 |

## 调试

### 假彩色图像

使用假彩色技术生成的图像称为假彩色图像，用于可视化数据。

将需要调试的变量映射到`[0,1]`，作为颜色输出到屏幕上，再根据颜色来判断值是否正确，一个变量可以对应一个颜色分量。

### Visual Studio

### Frame Debugger

## Shader整洁之道

### float、half、fixed

这三种类型对应不同精度的浮点数，大多数现代桌面GPU都会使用最高精度来计算，而在移动平台GPU上，使用不同精度的浮点数会对运算速度造成影响。

尽量使用较低精度的浮点数，以优化Shader的性能，这在移动平台上尤为重要。

### 规范语法

DirectX相比OpenGL对Shader的语法要求可能会更严格，比如在声明一个float4类型变量时，必须提供和变量类型相匹配的参数数目，即4个float值，而不能是一个。

### 避免不必要的计算

如果在Shader中进行了过多的运算，使需要使用的寄存器数量或指令数目超过了限制，就会收到错误提示。

通常可以指定更高等级的`Shader Target`来消除错误，以下是Unity支持的`Shader Target`。

| 指令                  | 描述                                                        |
| ------------------- | --------------------------------------------------------- |
| \#pragma target 2.0 | 默认的等级，相当于DirectX 9的Shader Model 2.0，不支持顶点纹理采样和显式的LOD纹理采样。 |
| \#pragma target 3.0 | 相当于DirectX 9的Shader Model 3.0，支持顶点纹理采样。                   |
| \#pragma target 4.0 | 相当于DirectX 10的Shader Model 4.0，支持几何着色器                    |
| \#pragma target 5.0 | 相当于DirectX 11的Shader Model 5.0。                           |

> Shader Model

Shader Model是微软提出的一套着色器的规范，定义着色器各个特性的能力，例如能使用的运算指令数目、寄存器数目等。

### 慎用分支和循环

GPU使用了和CPU不同的技术来实现分支语句，如果在Shader中使用大量的流程控制语句将会导致Shader性能大幅下降。

不可避免时，建议条件变量使用常数，尽量减少分支中的操作指令和嵌套层数。

### 不要除以0

除以0可能会导致Shader的崩溃或渲染结果的错误，对于除数可能为0的情况，可以强制截取到非0范围。

# 基础光照

从宏观上来说，渲染包含两大部分：决定像素的**可见性**和决定像素上的**光照计算**。

要模拟真实的光照环境来生成图像，就需要考虑三种物理现象：

1. 光线从光源发射出来
2. 光线和物体相交，一些被吸收，另一些被散射
3. 摄像机吸收了一些光，产生图像

> 光源

在光学里，使用**辐照度**来量化光。

对于平行光，辐照度等于在垂直于光线方向的单位面积上单位时间内穿过的能量。

当光线与表面不垂直时，可以使用光源方向和表面法线夹角的余弦值来计算辐照度。

辐照度和照射到物体表面时光线之间的距离`d/cos`成反比，因此辐照度和`cos`成正比，`cos`通过光源方向和表面法线点积得到。

> 吸收和散射

光线和物体相交的结果通常有两个：散射和吸收。

散射只改变光线的方向，不影响光线密度和颜色。吸收只改变光线的密度和颜色，不影响光线的方向。

光线经过散射会产生两种方向，一种指向物体内部，即**折射**现象，另一种指向外部，即**反射**现象。

对于不透明物体，折射进物体内的光线还会和内部的颗粒相交，其中一些光线还会反射出物体表面，这些光线将具有和入射光线不同的方向和颜色。

为了区分两种不同的散射方向，在光照模型中使用不同部分来计算它们：漫反射和高光反射。

**漫反射**部分表示有多少光线会被折射、吸收和散射出表面，**高光反射**部分表示物体表面如何反射光线。

根据入射光线的数量和方向，可以计算出射光线的数量和方向，即**出射度**。

辐照度和出射度之间是线性关系，它们的比值就是材质的漫反射和高光反射属性。

> 着色

着色指的是根据材质属性、光源信息，使用一个公式去计算沿某个观察方向的出射度，这个公式称为**光照模型**。

> BRDF光照模型

BRDF是用来计算表面和光照交互情况的，例如计算光线的反射情况。

在图形学中，BRDF大多使用一个数学公式来表示，并提供一些参数来调整材质属性。

通俗来讲，当给定入射光线的方向和辐照度后，BRDF可以给出在某个出射方向上的光照能量分布。

## 标准光照模型

裴祥风提出，**标准光照模型只关心直接光照**，即直接从光源发射出来照射到物体表面后，经过物体表面的一次反射直接进入摄像机的光线。

基本方法是把进入摄像机的光线分为4个部分，每个部分使用一种方法来计算其贡献度。

1. 自发光$C_{emissive}$：描述表面本身向一个指定的方向发射的辐射量。
2. 高光反射$C_{specular}$：描述表面在完全镜面反射方向上的散射辐射量。
3. 漫反射$C_{diffuse}$：描述表面每个方向的散射辐射量。
4. 环境光$C_{ambient}$：描述其他所有的间接光照。

### 环境光

物体除了被直接光照所照亮，也可以被**间接光照**所照亮。

间接光照指光线经过多次在物体之间的反射，最后进入摄像机的光线。

在标准光照模型中，使用**环境光模拟间接光照**。环境光通常是一个全局变量，场景中所有物体都是用这个环境光。

$$
c_{ambient}=g_{ambient}
$$

### 自发光

从光源射出的光线可以不经过任何反射，直接进入摄像机。

自发光部分直接使用材质的自发光颜色。

$$
c_{emissive}=m_{emissive}
$$

### 漫反射

漫反射光照符合兰伯特定律，即反射光线的强度与表面法线和光源方向之间夹角的余弦值成正比。

$$
c_{diffuse}=(\overset{光源颜色}{c_{light}}\cdot \overset{材质漫反射颜色}{m_{diffuse}})max(0,\overset{表面法线}{n}\cdot \overset{光源方向}{l})
$$

防止法线和光源方向点乘结果为负值，而导致物体被背面的光源照亮，可使用max函数将其截取到0。

### 高光反射

高光反射用于计算沿着完全镜面反射方向被反射的光线，可以让物体看起来有光泽。

Phong模型公式如下：

$$
c_{specular}=(\overset{光源颜色}{c_{light}}\cdot \overset{材质高光反射颜色}{m_{specular}})max(0,\overset{视角方向}{v}\cdot \overset{反射方向}{r})^{\overset{材质光泽度}{m_{gloss}}}
$$

反射方向可以由法线和光源方向计算得到：

$$
r=2(n\cdot l)n-l
$$

Blinn提出了更为简单的方法来实现高光反射，即使用视角方向与光源方向的半程向量，避免了计算反射方向。

Blinn Phong模型公式如下：

$$
c_{specular}=(\overset{光源颜色}{c_{light}}\cdot \overset{材质高光反射颜色}{m_{specular}})max(0,\overset{表面法线}{n}\cdot \overset{半程向量}{\frac{v+l}{\lvert {v+l}\rvert}})^{\overset{材质光泽度}{m_{gloss}}}
$$

### 逐像素与逐顶点

逐像素光照指的是在片元着色器中计算光照，而逐顶点光照是在顶点着色器中计算光照。

在逐像素光照中，法线是基于像素的，可以是对顶点法线插值得到，也可以从法线纹理采样得到，这种对顶点法线插值的技术被称为**Phong着色**，Phong插值或法线插值着色技术。

在逐顶点光照中，每个顶点计算光照后在图片内部进行插值，最终输出成像素颜色。由于顶点数目往往远小于像素数目，因此计算量也就更小。但是它依赖于线性插值来得到像素光照，因此当光照模型中有非线性的计算（例如高光反射时），逐顶点光照就会出问题，而且图元内部颜色会比顶点处颜色更暗。逐顶点光照也被称为**高洛德着色**。

## Unity中的环境光和自发光

Unity场景中的环境光可以通过Window/Lighting/Ambient Source/Ambient Color/Ambient Intensity控制，在Shader中，使用`UNITY_LIGHTMODEL_AMBIENT`获得环境光的颜色和强度。

大多数物体没有自发光，如果要计算自发光，只需要将材质的自发光颜色添加到片元着色器输出的颜色中即可。

# 基础纹理

使用纹理的最初目的是用一张图片来控制模型的外观，通过**纹理映射技术**可以让一张图片贴在模型表面，逐**纹素**地控制模型颜色。

建模时通常利用纹理展开技术将**纹理映射坐标**存储在每个顶点上，该坐标定义了该顶点在纹理中对应的二维坐标u和v，分别代表横向和纵向的坐标，因此纹理映射坐标也叫做UV坐标。

## 单张纹理

通常使用一张纹理作为物体的漫反射颜色，部分代码如下。

```C
struct a2v
{
	...
	float4 texcoord: TEXCOORD0;
}

struct v2f
{
	...
	float2 uv: TEXCOORD0;
}

v2f vert(a2v v)
{
	v2f o;
	// _MainTex_ST是纹理的一个属性，用于对纹理坐标进行缩放和平移变换
	o.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
	// 或直接使用该函数
	// o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
	...
}

fixed4 frag(v2f i): SV_Target
{
	fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
	// 计算光照部分
	...
}
```

### 纹理的属性

- Texture Type：如Texture、Normal map，选择合适的纹理类型，才能为Shader提供正确的纹理，并且可以让Unity对纹理进行优化。
- Wrap Mode：当纹理坐标超过$[0,1]$范围后的覆盖模式。
- Filter Mode：纹理产生拉伸时采用的滤波模式。
- Max Size：如果纹理大小超过该值，那么Unity会将纹理缩放为这个最大的分辨率。纹理的长宽尺寸尽量使用2的幂，否则会占用更多的内存，也影响GPU的读取速度。

# 非真实感渲染

非真实感渲染的一个主要目标是，使用一些渲染方法使画面达到和某些特殊的绘画风格相似的效果，例如水彩、卡通风格等。

## 卡通风格渲染

卡通风格的游戏画面通常有一些共同特点，例如黑色的线条描边和明显的明暗变化等。

**基于色调的着色技术**是实现卡通风格渲染的一种方法，使用漫反射系数对一张一维纹理进行采样来控制漫反射的色调，模型的高光是一块分界明显的纯色区域，另外还需要在物体边缘部分绘制轮廓。

### 渲染轮廓线

在《Real Time Rendering》一书中，作者将绘制模型轮廓线的方法分为五种类型。

1. **基于观察角度和表面法线的轮廓线渲染。** 使用视角方向和表面法线的点乘结果来得到轮廓线信息。优点是简单快速，可以在一个Pass中得到结果；缺点是局限性很大，效果不好。
2. **过程式几何轮廓线渲染。** 使用两个Pass渲染，第一个Pass渲染背面，并让其轮廓可见，第二个Pass渲染正面。优点是快速有效，适用于绝大多数表面平滑的模型；缺点是不适合立方体这种平整的模型。
3. **基于图像处理的轮廓线渲染。** 优点是适用于任何种类的模型；缺点是一些深度和法线变化很小的轮廓无法被检测出来，例如桌面上的纸张。
4. **基于轮廓边检测的轮廓线渲染。** 使用算法精确地检测出一条边是否为轮廓边。优点是可以渲染出独特风格的轮廓线；缺点是实现相对复杂，会影响动画的连贯性。
5. **以上方法的结合使用。** 首先找出精确的轮廓边，把模型和轮廓边渲染到纹理中，再使用图像处理方法识别轮廓线，并在图像空间中进行风格化渲染。

> 轮廓边检测公式

$$
(n_0 \cdot v > 0) \neq (n_1 \cdot v > 0)
$$
$n_0$和$n_1$分别是两个相邻三角面片的法线，$v$是从视角到边上任意顶点的方向。

公式的本质在于检查两个向量的三角面片是否一个朝正面，一个朝背面。

---

下面使用过程式几何轮廓线渲染方法来对模型进行轮廓描边，第一个Pass中，使用轮廓线颜色渲染整个背面的面片，并在视角空间下将模型顶点沿法线方向向外扩张一段距离，让背部轮廓线可见。

```C
viewPos = viewPos + viewNormal * _Outline;
```

对于一些内凹的模型，直接使用顶点法线进行扩展，可能会出现背面面片遮挡正面面片的情况。

解决方法是，在扩展背面顶点前，首先将顶点法线的z分量固定，然后归一化，再扩展。这样做的好处是让扩展后的背面更加扁平化，从而降低遮挡正面面片的可能性。

```C
viewNormal.z = -0.5;
viewNormal = normalize(viewNormal);
viewPos += viewNormal * _Outline;
```

### 添加高光

首先计算法线和半程向量的点乘，与一个阈值进行比较，若小于该阈值，则高光反射系数为0，否则为1。

```C
float spec = dot(worldNormal, halfDir);
spec = step(threshlod, spec);
```

step函数的第一个参数是参考值，第二个参数是待比较的值，若第二个参数大于第一个参数，返回1，否则返回0。

但是，这种方法会在高光区域的边界产生锯齿，因为边缘不是平滑渐变的，即从0直接突变为1，下面对边界处一块区域做平滑处理。

```C
float spec = dot(worldNormal, halfDir);
spec = lerp(0, 1, smoothstep(-w, w, spec - threshold));
```

在smoothstep函数中，w是一个很小的值，当$spec-threshld < -w$时，返回0，大于w时返回1，否则在0到1之间进行插值。这样做的效果是，在$[-w, w]$区间中，即高光区域的边界处，得到一个从0到1平滑变化的值，从而实现抗锯齿的目的。

### 实现

首先声明使用到的属性。

```C
Properties
{
	_Color ("Color", Color) = (1, 1, 1, 1)
	_MainTex ("Texture", 2D) = "white" { }
	// 控制漫反射色调的渐变纹理
	_RampTex ("Ramp Texture", 2D) = "white" { }
	_Specular ("Specular", Color) = (1, 1, 1, 1)
	_Glossiness ("Glossiness", Range(0, 0.1)) = 0.01
	// 轮廓线宽度和颜色
	_Outline ("Outline Width", Range(0, 1)) = 0.1
	_OutlineColor ("Outline Color", Color) = (0, 0, 0, 1)
}
```

第一个Pass，只渲染背面。

使用Name命令定义Pass名称，方便其他Shader直接使用。

```C
Pass
{
	Name "OUTLINE"
	Cull Front

	CGPROGRAM
	#pragma vertex vert
	#pragma fragment frag

	#include "UnityCG.cginc"

	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
		float3 normal : NORMAL;
	};

	struct v2f
	{
		float2 uv : TEXCOORD0;
		float4 vertex : SV_POSITION;
	};

	sampler2D _MainTex;
	float4 _MainTex_ST;
	fixed _Outline;
	fixed4 _OutlineColor;

	v2f vert(appdata v)
	{
		v2f o;

		float4 viewPos = mul(unity_MatrixMV, v.vertex);
		float3 viewNormal = mul(UNITY_MATRIX_IT_MV, v.normal);
		viewNormal.z = -0.5;
		viewPos += float4(normalize(viewNormal), 0) * _Outline;
		
		o.vertex = mul(UNITY_MATRIX_P, viewPos);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		
		return o;
	}

	fixed4 frag(v2f i) : SV_Target
	{
		return fixed4(_OutlineColor.rgb, 1);
	}
	ENDCG
}
```

第二个Pass，渲染正面。

对于漫反射，首先计算了半兰伯特漫反射系数，和阴影值相乘，再对渐变纹理进行采样，和材质反射率、光照颜色相乘得到漫反射光照。

对于高光反射，使用fwidth函数对高光区域边界做抗锯齿，再将高光反射系数和高光反射颜色相乘得到高光反射光照。

```C
Pass
{
	Tags { "LightMode" = "ForwardBase" }
	Cull Back

	CGPROGRAM
	#pragma vertex vert
	#pragma fragment frag
	#pragma multi_compile_fwdbase

	#include "UnityCG.cginc"
	#include "Lighting.cginc"
	#include "AutoLight.cginc"

	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
		float3 normal : NORMAL;
	};

	struct v2f
	{
		float2 uv : TEXCOORD0;
		float4 pos : SV_POSITION;
		float4 worldPos : TEXCOORD1;
		float3 worldNormal : TEXCOORD2;
		SHADOW_COORDS(3)
	};

	sampler2D _MainTex;
	float4 _MainTex_ST;
	sampler2D _RampTex;
	float4 _RampTex_ST;
	fixed4 _Color;
	fixed4 _Specular;
	fixed _Glossiness;

	v2f vert(appdata v)
	{
		v2f o;

		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		o.worldPos = mul(unity_ObjectToWorld, v.vertex);
		o.worldNormal = UnityObjectToWorldNormal(v.normal);
		TRANSFER_SHADOW(o);
		return o;
	}

	fixed4 frag(v2f i) : SV_Target
	{
		fixed3 albedo = tex2D(_MainTex, i.uv) * _Color;
		
		float3 normal = normalize(i.worldNormal);
		float3 lightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
		float3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
		float3 halfDir = normalize(lightDir + viewDir);

		UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

		// Ambient
		fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo.rgb;

		// Diffuse
		fixed diff = dot(normal, lightDir) * 0.5 + 0.5;
		diff *= atten;
		fixed3 diffuse = _LightColor0.rgb * albedo * tex2D(_RampTex, fixed2(diff, diff)).rgb;

		// Specular
		fixed spec = dot(normal, halfDir);
		fixed w = fwidth(spec) * 2;
		fixed3 specular = _Specular.rgb * lerp(0, 1, smoothstep(-w, w, spec + _Glossiness - 1)) * step(0.0001, _Glossiness);

		return fixed4(ambient + diffuse + specular, 1);
	}
	ENDCG
}
```