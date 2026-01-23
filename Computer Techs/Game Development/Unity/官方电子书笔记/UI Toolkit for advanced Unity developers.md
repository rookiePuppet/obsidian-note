# 简介

UI Toolkit相比旧的UI系统的优势：

- 更快的迭代：全局样式管理，即使编辑能力
- 渲染性能：使用Render Hints和动态纹理图集，对游戏性能有更好地掌控
- 更好的协作：UI结构、逻辑和样式分离，减少冲突
- 复用性：跨项目共享复用样式和组件

## UI资产

在UI Toolkit中，构建UI的主要元素包含UXML和USS文件。UXML用于表示UI的内容和结构，是类似于HTML和XML的标记语言；USS用于定义UI内容的外观和样式，类似于CSS。

![[Pasted image 20260121112449.png]]

## UI Builder

UI资产可以像代码一样通过IDE去编辑，也可以使用UI Builder进行可视化编辑。

# 图形字体资源准备

## 位图图像（Bitmap Graphics）

Unity支持大多数的图像文件类型，如PNG、BMP、TIF、TGA、JPG和PSD。当你将这些格式的文件添加进Assets文件夹之后，Unity会将它们导入为Texture 2D（3D项目）或Sprites（2D项目），可以在Texture Type处更改类型，UI Toolkit对这两种格式的位图图形都支持。

纹理只会包含图像的尺寸和格式信息，而精灵则会包含UI Toolkit所使用的一些额外属性。

## 精灵（Sprites）

精灵在2D游戏中作为纹理，由Sprite Renderer组件所使用。2D精灵可以平铺、绑定骨骼和蒙皮，在2D灯光中可以包含自定义几何或额外贴图。

大多数UI图形资产会被渲染在屏幕空间，而不受Unity的世界缩放影响（一单位代表3D空间中的一立方米），UI Toolkit会管理这些图形的缩放。精灵的PPU（Pixels Per Unit）影响着它们在UI上的大小，例如，如果你的精灵在每个网格单元上有128个像素，那就把PPU设置为128。

精灵是映射在平面矩形3D网格上的2D纹理，在导入时默认使用**Mesh Type**：*Tight*设置，使网格紧贴着精灵上非透明像素的边缘，这样可以减少重绘（Overdraw）。可以在Sprite Editor的Outline部分调整优化该网格。

> [!NOTE] 重绘 Overdraw
> 由于透明区域的重叠，GPU可能在一帧内对同一个像素重复多次绘制。

**Sprite Modes**包括以下选项：

- Single：默认模式，当纹理源文件中仅包含单个图片元素时选择
- Multiple：当纹理源文件中包含多个图片元素时选择。在Sprite Editor中设置每个元素的位置，以便将纹理分割为多个子图片，然后在UI Toolkit中就可以分别使用它们。
- Polygon：适用于圆形或规则多边形，该模式可帮助设置贴合图像形状的边缘。

## 渲染纹理（Render Texture）资产

渲染纹理是每帧都会更新的摄像机视图的快照纹理，通过Assets>Create>Rendering创建，在摄像机的Output菜单下引用。在UI Toolkit中可以用这些纹理来显示小地图、角色选择界面和其他需要集成到UI中的元素。


