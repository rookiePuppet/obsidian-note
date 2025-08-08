# 实例化

在渲染管线中，CPU和GPU之间交换数据的过程会造成主要的性能瓶颈。如果某些使用相同网格和材质的模型需要被多次渲染，可以使用Unity提供的一些工具来优化它。

## GPU Resident Drawer和GPU occlusion culling

**GPU Resident Drawer**是一个由GPU驱动的渲染系统，用于优化CPU时间。它可以让游戏对象借助**BatchRenderGroup** API，加速合批以提升CPU性能。

使用GPU Resident Drawer时，游戏对象的数据会被导入到特殊的快速路径进行实例化并渲染。

该功能可在URP Asset的Rendering部分开启，将GPU Resident Drawer设置为**Instanced Drawing**。

当场景规模越大，实例对象数量越多时，获得的提升就越显著。

GPU Resident Drawer只对MeshRenderer有效，使用自定义着色器时，需要确保它们兼容[DOTS实例化](https://github.com/Unity-Technologies/EntityComponentSystemSamples/blob/master/GraphicsSamples/URPSamples/Assets/SampleScenes/6.%20Misc/SimpleDotsInstancingShader/SceneAssets/CustomDotsInstancingShader.shader)。

> [!NOTE]
> GPU Resident Drawer要求使用**Forward+**渲染器，并在Project Settings > Graphics> BatchRendererGroup Variants处设置为**Keep All**。

GPU occlusion culling也是使用一种GPU驱动的技术来确保不渲染屏幕上不可见的内容，从而减少CPU负担。

为了检验GPU occlusion culling是否有效，可以打开Window > Analysis > Rendering Debugger，选择GPU Resident Drawer > Occlusion Test Overlay，该工具会显示被剔除实例的热图，颜色越红表示被剔除的实例越多。

## SRP Batcher

**SRP Batcher**技术可以使大量相同的游戏对象分批合并渲染，例如一块草地上有62500棵草，都使用了同一个兼容SRP Batcher的着色器。

那么在实际渲染时，62500棵草就会被划分为20个批次（观察Frame Debugger得出，不确定分批数量的依据是什么）依次渲染。

即使有这种批处理技术，在面对这种大规模实例化对象的场景时，游戏帧率还是非常低。

### GPU Instancing

GPU Instancing可在材质面板的Advanced Options部分开启，需要注意的是它和SRP Batcher会相互冲突，也就是说在开启SRP Batcher的情况下，即使启用GPU Instancing也是无效的。

因此要使用GPU Instancing，首先得禁用SRP Batcher。

SRP Batcher是通过在Shader中将暴露在外的变量放进CBUFFER块中启用的，所以对于自定义着色器，可以直接将CBUFFER移除。对于使用Shader Graph创建的着色器，需要通过View Generated Shader，手动注释CBUFFER块，然后修改着色器名称另存Shader。

### Render Mesh Primitives

Unity的Graphics API中有许多能绕开GameObject直接渲染网格的方法，例如**RenderMeshPrimitives**，这种方法需要使用**Compute Buffer**来为每一个独立网格分配其位置信息。

在草地的例子中，将草的位置作为一个材质属性，使渲染草所需的数据保存在GPU，从而借助GPU强大的并行计算能力将整块草地以极快的速度渲染出来。

```CS
_positionsCount = _positions.Count;
_positionBuffer?.Release();
if (_positionsCount == 0) return;
_positionBuffer = new ComputeBuffer(_positionsCount, 8);
_positionBuffer.SetData(_positions);
_instanceMaterial.SetBuffer(Shader.PropertyToID("PositionsBuffer"), _positionBuffer);
```

在C#代码中，需要创建一个ComputeBuffer对象，缓冲区大小即草的数量，由于每棵草只需存储其在XZ平面上的坐标（两个float），因此每个单元大小需要8字节。将位置数据通过**SetData**方法存入缓冲区后，再把缓冲区传入材质属性。

![[Pasted image 20250808162155.png]]

以下是世界位置节点之后的自定义函数内容：

```C
#pragma instancing_options procedural:ConfigureProcedural
Out = In;
```

ShaderGraphFunction定义在如下HLSL文件：

```C
#if defined(UNITY_PROCEDURAL_INSTANCING_ENABLED)
StructuredBuffer<float2> PositionsBuffer;
#endif

float2 position;

void ConfigureProcedural () {
	#if defined(UNITY_PROCEDURAL_INSTANCING_ENABLED)
	position = PositionsBuffer[unity_InstanceID];
	#endif
}

void ShaderGraphFunction_float (out float2 PositionOut) {
	PositionOut = position;
}
```

草地初始化代码：

```CS
_grassEntities = grassEntities;  
_grassBounds = new Bounds(transform.position, new Vector3(fieldSize.x, 0.0f, fieldSize.y));  
_positions = new List<Vector2>();  
_renderParams = new RenderParams(_instanceMaterial);  
_renderParams.worldBounds = _grassBounds;  
_renderParams.shadowCastingMode = ShadowCastingMode.Off;
```

实际的渲染步骤通过Update函数完成：

```CS
private void Update() {  
    if (_positionsCount == 0) return;  
    Graphics.RenderMeshPrimitives(_renderParams, _instanceMesh, 0, _positionsCount);  
}
```

# 环境光遮蔽

![[Pasted image 20250808164914.png]]

环境光遮蔽是一种用于加深褶皱、缝隙以及表面相交处的颜色的后处理技术。在现实世界中，这些区域会阻挡或遮蔽环境光，看起来会更暗。

Unity中将实时的屏幕空间环境光遮蔽（SSAO）效果作为一个Renderer Feature提供。

[Unity - Manual: Configure screen space ambient occlusion in URP](https://docs.unity3d.com/6000.0/Documentation/Manual/urp/ssao-renderer-feature-reference.html)