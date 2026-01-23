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

## 渲染纹理（Render Texture）

渲染纹理是每帧都会更新的摄像机视图的快照纹理，通过Assets>Create>Rendering创建，在摄像机的Output菜单下引用。

在UI Toolkit中可以用这些纹理来显示小地图、角色选择界面和其他需要集成到UI中的元素。反过来，还可以将UI Toolkit的界面渲染到渲染纹理中，再应用到3D模型使用的材质上。

## 2D PSD Importer

PSD文件常用于在单个文件中分层存储多张图片，安装2D PSD Importer包之后可以直接导入Unity中，而不需要将每一层分别导出为独立的文件。

作为UI资源使用的图片，需要在Character Rig栏下将*Use as Rig*选项取消，Rig用于2D角色骨骼动画，对于UI来说是不必要的。

## 矢量图像（Vector Images）

矢量图像的支持仍在开发中，目前可通过安装Vector Graphics包测试使用，在Generated Asset Type设置中，选择*UI Toolkit Vector Image*即可在UI Toolkit中使用。

目前SVG文件是切分成多边形进行渲染的，不能发挥矢量图像的优势。当图像放大时，可以看到多边形边界，而且UI Toolkit暂时还不能使用抗锯齿。

## 字体（Fonts）

- Font：标准的字体格式，例如TTF或OTF，支持向后兼容，在后台会自动转换为FontAsset。
- FontAsset：推荐使用的格式，允许对字距或基线进行微调，而无需修改原始字体资源，对于字体高度风格化的游戏很有用。对图集创建提供精确的控制，包括文字集、分辨率和图集填充设置（atlas population options），这些设置可以帮助降低内存使用，特别是对于包含大量文字的Unicode字体。

## 纹理打包器

将多个2D图形组合进同一张纹理是减少Draw Call和提升内存使用效率的优化手段，UI Toolkit支持以下两种图集系统。

### Sprite Atlas

Sprite Atlas是用于精灵的图集工具，也可用于UI图形，它会自动将同一个文件夹中的资源（精灵、法线/遮罩贴图）打包成图集，一般用于编辑器中，而非运行时。

### Dynamic Atlas

对于没有被打包到精灵图集中的UI图形，在一个预处理通道期间也会被UI Toolkit的Dynamic Atlas特性自动打包。

被Visual Element引用的图像会根据UI Document的Panel Settings设置的标准进行图集打包，例如，可以设置纳入图集的最小或最大纹理尺寸，或者对图像过滤。在UI Toolkit Debugger的Texture Atlas Viewer中可以预览生成的图集。

Dynamic Atlas在编辑器和运行时均可使用，对于动态生成的UI元素很有用。

---

一些好的实践做法：

- 在制作原型图时就确定好最高的目标分辨率，避免之后反复修改
- 避免在位图图像制作完成后放大它们，这会导致像素化和模糊，而是从支持的最高分辨率开始，一步一步地缩小。
- 对于矢量图形，虽然随时调整大小不会有什么问题，但还是要根据参考分辨率进行设计，使每一个资源有正确的相对大小，例如保持描边粗细统一。
- 如果图像分辨率低于需求，尝试使用2D Enhancers，以及Sprite Editor中基于AI的放大功能。
- 使用2D PSD Importer将PSD直接导入Unity，每次更改保存PSD文件，在Unity中都会立即刷新。
- 利用预设（**Preset**）功能将导入流程自动化，避免每次添加图像资源时都要手动修改资源设置。
- 如果需要更深度的自动化流程，例如运行时检查，或大规模的设置修改，可以使用**Asset PostProcessor** API。

# 布局

## 核心运行时组件

要将UI Toolkit元素渲染到Game View中，必须要让一个GameObject挂载**UI Document**组件，引用一个**Panel Settings**资产和**Visual Tree（UXML）**资产。

**UI Document**组件定义了要显示的UXML，并附带一个默认的Panel Settings，**Sort Order**决定了使用相同Panel Settings的UI Document之间的显示顺序。

**Panel Settings**定义UI Document组件在运行时如何被实例化和显示，可以给UI设置不同Panel Settings来实现不同样式。

## 响应式布局：Flexbox

UI Toolkit基于Yoga定位UI元素，Yoga是实现了Flexbox的一个子集的HTML/CSS布局引擎。Flexbox是一种将按行列组织元素的方法，有如下优势：

- 响应式UI：Flexbox将一切组织进一个由盒子或容器组成的网络，可以将元素以父级或子级方式嵌套排列，或利用简单的规则在空间中分散排列。响应式布局可以适应不同分辨率的屏幕，针对多个平台开发时更加容易。
- 有组织的复杂性：样式是控制UI呈现效果的简单规则，一个样式可以同时被成百上千个元素所使用，并在整个UI上立即响应更改。这样可以将UI设计的重心放在一致、可复用的样式上，而不是专注于单个元素。
- 逻辑与设计解耦：UI布局和样式不与代码耦合，有利于设计师和开发者并行工作。

## 可视元素（Visual elements）

在UI Toolkit中，每个界面最基本的组成元素叫做**Visual Element**，Visual Element是所有UI Toolkit元素的基类。一个由多个可视元素组成的UI层次结构叫做**Visual Tree**。视觉元素的布局、样式和层级相关信息会被存储到UXML文件中。

