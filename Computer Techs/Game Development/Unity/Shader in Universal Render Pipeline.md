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

## Render Pipeline

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