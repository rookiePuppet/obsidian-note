# ShaderLab

ShaderLab是Unity中专门用于定义着色器属性、子着色器和通道的语言，实际的着色器代码用的是HLSL(High Level Shading Language)。

所有的Shader都由Shader块开始，用于指定Shader显示在材质Inspector下拉框中的路径和名称。

```C
Shader "Custom/Example" {}
```

## Properties

Properties块用于定义需要暴露在材质Inspector中的属性值。

```C
Properties
{
//  [name] ("[name in inspector]", [type]) = [default value]
    _BaseMap ("Base Texture", 2D) = "white" {}
    _BaseColor ("Base Colour", Color) = (0, 0.66, 0.73, 1)
//  _ExampleDir ("Example Vector", Vector) = (0, 1, 0, 0)
//  _ExampleFloat ("Example Float (Vector1)", Float) = 0.5
```

这些属性值可以通过C#脚本来设置，例如`material.SetColor/SetFloat/SetVector`。如果这些属性值在各个材质中是不同的，必须将它们包含到`UnityPerMaterial CBUFFER`中，以正确支持SRP Batcher。

如果使用该Shader的所有材质共享相同的值，就不需要将属性暴露出来，只需要在HLSL代码中声明，然后在C#脚本中通过`Shader.SetGlobalColor/SetGlobalFloat`来设置属性值。

## SubShader

Shader块中可以包含多个SubShader块，Unity会使用GPU支持的第一个SubShader。如果所有SubShader都不支持，就会使用Fallback指定的Shader。

```C
Shader "Custom/UnlitShaderExample" {
    Properties { ... }
	SubShader { ... }
	FallBack "Path/Name"
}
```

之后可以在SubShader中定义通道，通道中可以指定Shader编译目标，越高级别的编译目标支持更多的GPU特性，但可能不在所有平台上都被支持。

在URP v10之前，通常在所有通道中指定：

```c
// Required to compile gles 2.0 with standard SRP library
// All shaders must be compiled with HLSLcc and currently only gles is not using HLSLcc by default
#pragma prefer_hlslcc gles
#pragma exclude_renderers d3d11_9x
#pragma target 2.0
```

在v10之后，由于延迟渲染开始被支持，一般会提供两个SubShader，第一个SubShader的所有通道都会指定：

```c
#pragma exclude_renderers gles gles3 glcore
#pragma target 4.5
```

以排除使用OpenGL的平台。

而第二个SubShader指定：

```c
#pragma only_renderers gles gles3 glcore d3d11
#pragma target 2.0
```

### Render Pipeline

RenderPipeline标签用于说明SubShader针对的渲染管线，如果当前渲染管线（`Shader.globalRenderPipeline`）不是其指定的，则不会使用该SubShader。

如果没有设置Render Pipeline标签，则说明该SubShader可以在任何管线中使用。

```c
Shader "Custom/UnlitShaderExample" {
    Properties { ... }
	SubShader {
		Tags { "RenderPipeline"="UniversalPipeline" "Queue"="Geometry" }
		...
	}
	SubShader {
		Tags { "RenderPipeline"="HDRenderPipeline" "Queue"="Geometry" }
		...
	}
	SubShader {
		Tags { }
		...
	}
	FallBack "Path/Name"
}
```

### Queue

Queue标签用于确定物体渲染的时机，必须设置成下面这些预定义的名称（分别对应一个渲染队列值）：

- Background - 1000
- Geometry - 2000
- AlphaTest - 2450
- Transparent - 3000
- Overlay - 4000

也可使用“+n"或“-n”的形式，例如“Geometry+1”即2001。

队列值在2500以下的物体为不透明物体，相同队列值的物体是从前往后渲染的。

2500以上队列值的物体为透明物体，从后往前渲染。

## Pass

Pass块定义在每一个SubShader块中，每个Pass块都会包含一个叫做**LightMode**的特殊标签，其决定何时以及如何使用该Pass。

可以用**Name**为Pass命名，之后在其他的Shader中就可以通过**UsePass**来使用该Pass。如果Pass中使用了CBUFFER，则不建议在其他Shader中使用UsePass来调用该Pass。这是为了保持SRP Batcher的兼容性，Shader中所有的Pass必须共用相同的**UnityPerMaterial CBUFFER**，而UsePass可能会破坏这个规则。

当Shader只需要一个单独的Pass时，可以完全省略LightMode标签，例如一个实现全屏效果的Blit Render Feature所使用的Shader。

### LightMode

URP所使用的LightMode如下：

- UniversalForward：用于在前向（Forward）渲染路径中渲染物体
- ShadowCaster：用于投射阴影
- DepthOnly：被Depth Prepass用于创建深度纹理，当启用MSAA或平台不支持拷贝深度缓冲时
- DepthNormals：被Depth Normals Prepass用于创建深度纹理和法线纹理，需要在渲染器中通过`ConfigureInput(ScriptableRenderPassInput.Normal)`开启
- Meta：在烘焙光照贴图时使用
- Universal2D：用于在2D渲染器启用时渲染物体
- SRPDefaultUnlit：当未指定任何LightMode时所使用的默认值，可用于渲染额外通道，但可能会破坏SRP Batcher的兼容性

为内置渲染管线设计的LightMode不受URP支持，如Always、ForwardAdd、PrepassBase等。

### Cull

Cull指令用于控制三角形面的剔除。

- Off：不剔除，两面都渲染
- Back：剔除背面，默认值
- Front：剔除正面

```c
Pass
{
	Cull Off
	...
}
```

### Depth Test/Write

ZTest指令用于控制深度测试时决定片元渲染的深度值比较方式，例如默认值LEqual，仅当片元深度值小于或等于缓冲区值时渲染片元。

ZWrite指令用于控制当片元通过深度测试时是否将其深度值写入缓冲区。

Offset指令用于偏移深度值。

```cs
Pass {
	ZTest LEqual	// Default
	// ZTest Less | Greater | GEqual | Equal | NotEqual | Always
	
	ZWrite On		// Default
	// ZWrite Off
	...
	
	Offset 0, -1
}
```

### Blend & Transparency

Blend指令和BlendOp指令用于控制源颜色（着色器计算出的颜色）和目标颜色（缓冲区中的颜色）如何进行混合。

Blend指令可指定对源颜色和目标颜色所使用的混合因子，BlendOp指令用于指定使用混合因子对源颜色和目标颜色进行混合的计算模式。

可选的计算因子有：One、Zero、SrcColor、SrcAlpha、DstColor、DstAlpha、OneMinusSrcColor.....

常用的混合因子组合如下：

- SrcAlpha OneMinusSrcAlpha - 传统透明
- One OneMinusSrcAlpha - 预乘透明
- One One - 叠加
- OneMinusDstColor One - 柔和叠加
- DstColor Zero - 正片叠底
- DstColor SrcColor - 2倍正片叠底

## Multi-Pass

LightMode设置为SRPDefaultUnlit的通道会破坏SRP Batcher的兼容性，因此不推荐使用。

实现多通道的推荐方式如下：

- 将额外的Pass写到新的Shader中，再将多个材质应用到网格渲染器上。
- 前向渲染器中的RenderObjects特性可以使用一个覆盖材质在特定的层级对物体再次渲染，但是覆盖材质不会保留上一个Shader的属性和纹理。
- 使用一个自定义的LightMode标签，在RenderObjects特性上使用Shader Tag ID设定。

# HLSL

## HLSLPRORAM & HLSLINCLUDE

在每一个Pass中使用`HLSLPROGRAM`和`ENDHLSL`来定义HLSL的代码块，HLSL代码块中必须包含一个顶点着色器和一个片元着色器，使用`#pragma vertex/fragment`来指定顶点和片元着色器对应的函数。

在内置渲染管线中通常使用vert和frag作为顶点片元着色器函数的名称，但是在URP中会使用更具体的名称如“UnlitPassVertex”和“UnlitPassFragment”。

在SubShader块中可以使用`HLSLINCLUDE`来包含一些在每个Pass中共享的代码，这里面的代码也可以单独放在另一个文件中。

```c
SubShader {
	Tags { "RenderPipeline"="UniversalPipeline" "Queue"="Geometry" }
	
	HLSLINCLUDE
	...
	ENDHLSL

	Pass {
		Name "Forward"
		// LightMode tag. Using default here as the shader is Unlit
		// Cull, ZWrite, ZTest, Blend, etc

		HLSLPROGRAM
		#pragma vertex UnlitPassVertex
		#pragma fragment UnlitPassFragment
		...
		ENDHLSL
	}
}
```

## 变量

### 标量

- bool：布尔值
- half：16位浮点数，常用于短向量、方向、模型空间位置和颜色
- float：32位浮点数，常用于世界空间位置、纹理坐标和包含复杂函数（三角/幂）的标量计算
- double：64位浮点数，不能用于输入输出
- real：在URP/HDRP中使用，当平台支持half时默认为half，或指定`#define PREFER_HALF 0`时为float。ShaderLibrary中的许多公共数学函数都使用该类型。
- int：32位有符号整数
- uint：32位无符号整数，GLES2不支持
- fixed：11位定点数，范围从-2到2，来自于旧版CG，通常用于LDR色彩，会自动转换为half。

### 矢量

矢量类型是通过在标量类型后添加一个1-4的整数后缀来指定的，例如：

- float4
- half3
- int2

使用`.x/y/z/w`或`.r/g/b/a`的形式来得到矢量的分量，还可以使用`.xy`来得到一个二维向量。

```c
float3 vector = float3(1, 2, 3); // defined a 3 dimensional float vector

float3 a = vector.xyz;  // or .rgb,		a = (1, 2, 3)
float3 b = vector3.zyx; // or .bgr,		b = (3, 2, 1)
float3 c = vector.xxx;  // or .rrr,		c = (1, 1, 1)
float2 d = vector.zy;   // or .bg,		d = (3, 2)
float4 e = vector.xxzz; // or .rrbb,	e = (1, 1, 3, 3)
float  f  = vector.y;   // or .g,		f = 2

// Note that mixing xyzw/rgba is not allowed.
```

### 矩阵

矩阵类型是通过在标量类型后面添加两个1-4的整数（分别代表行列数），并以`x`分隔来指定的，例如：

- float4x4
- int4x3
- half2x1

Unity中内置了用于在各个空间之间进行变换的矩阵，例如：

- UNITY_MATRIX_M：模型矩阵（模型空间-世界空间）
- UNITY_MATRIX_V：视图矩阵（世界空间-视图空间）
- UNITY_MATRIX_P：投影矩阵（视图空间-裁剪空间）
- UNITY_MATRIX_VP：视图投影矩阵（世界空间-裁剪空间）

以及它们的转置矩阵，如UNITY_MATRIX_I_M等。

可以利用`mul`函数做矩阵乘法来进行空间变换，也可以使用SRP Core ShaderLibrary [SpaceTransforms.hlsl](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl)中的函数。

需要注意的是矩阵/向量相乘的顺序，一般是矩阵左乘向量。

访问矩阵分量的方式：

1. 从0开始的行列索引，如`._m01`对应第一行第二列
2. 从1开始的行列索引，如`._m11`对应第一行第二列
3. 从0开始的数组索引形式，如`[0][0]`对应第一行第一列

### 纹理对象

纹理存储了每一个像素的颜色，这里的像素通常叫做**纹素**（纹理元素）。

纹理可以有不同的尺寸（宽度/高度/深度），但用于采样纹理的坐标范围是`[0,1]`，称为纹理坐标或UV（U对应横轴，V对应纵轴）。

最常见的是2D纹理，在URP中可使用如下宏在全局范围下定义：

```c
TEXTURE2D(textureName)
SAMPLER(sampler_textureName) // 每一个纹理对象需要定义一个采样器状态，其中包含了纹理的wrap和filter模式设置
```

> FilterModes

- Point：颜色取自最近纹素，结果是块状或像素化的
- Linear/Bilinear：颜色取自根据邻近纹素距离的加权平均值
- Trilinear：和Linear/Bilinear相同，但它还会在mipmap级别间进行混合

> Wrap Modes

- Repeat：在`[0,1]`范围之外的UV值使纹理重复平铺
- Clamp：在`[0,1]`范围之外的UV值使纹理边缘被拉伸
- Mirror：纹理重复平铺，并在每个整数边界上产生镜像
- Mirror Once：只产生一次镜像，UV被限制在`[-1,2]`

---

在片元着色器中使用`SAMPLE_TEXTURE2D`来采样纹理:

```c
float4 color = SAMPLE_TEXTURE2D(textureName, sampler_textureName, uv);
```

`SAMPLE_TEXTURE2D`只能在片元着色器中使用，若需要在顶点着色器中采样纹理，则使用下面这个LOD版本，指定一个mipmap（如0代表全分辨率）。

```c
float4 color = SAMPLE_TEXTURE2D_LOD(textureName, sampler_textureName, uv, 0);
```

其他纹理类型的采样方法如下：

```c
// Texture2DArray
TEXTURE2D_ARRAY(textureName);
SAMPLER(sampler_textureName);
// ...
float4 color = SAMPLE_TEXTURE2D_ARRAY(textureName, sampler_textureName, uv, index);
float4 color = SAMPLE_TEXTURE2D_ARRAY_LOD(textureName, sampler_textureName, uv, lod);

// Texture3D
TEXTURE3D(textureName);
SAMPLER(sampler_textureName);
// ...
float4 color = SAMPLE_TEXTURE3D(textureName, sampler_textureName, uvw);
float4 color = SAMPLE_TEXTURE3D_LOD(textureName, sampler_textureName, uvw, lod);
// uses 3D uv coord (commonly referred to as uvw)

// TextureCube
TEXTURECUBE(textureName);
SAMPLER(sampler_textureName);
// ...
float4 color = SAMPLE_TEXTURECUBE(textureName, sampler_textureName, dir);
float4 color = SAMPLE_TEXTURECUBE_LOD(textureName, sampler_textureName, dir, lod);
// uses 3D uv coord (named dir here, as it is typically a direction)

// TextureCubeArray
TEXTURECUBE_ARRAY(textureName);
SAMPLER(sampler_textureName);
// ...
float4 color = SAMPLE_TEXTURECUBE_ARRAY(textureName, sampler_textureName, dir, index);
float4 color = SAMPLE_TEXTURECUBE_ARRAY_LOD(textureName, sampler_textureName, dir, lod);
```

### 数组

```c
float4 _VectorArray[10]; // Vector array
float _FloatArray[10]; // Float array

void ArrayExample_float(out float Out){
	float add = 0;
	[unroll]
    for (int i = 0; i < 10; i++){
		add += _FloatArray[i];
	}
	Out = add;
}
```

若循环次数固定，并且不提前退出，则“展开”循环可以获得更好的性能。

声明其他类型的数组在技术上也是可行的，但是只有float向量和float标量的数组可以通过C#脚本来设置。

建议使用全局的Set方法，如`Shader.SetGlobalVectorArray`，原因是数组不能被正确地包含到UnityPerMaterial CBUFFER中