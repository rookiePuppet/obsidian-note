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

## 视觉元素（Visual elements）

在UI Toolkit中，每个界面最基本的组成元素叫做**Visual Element**，Visual Element是所有UI Toolkit元素的基类。一个由多个可视元素组成的UI层次结构叫做**Visual Tree**。视觉元素的布局、样式和层级相关信息会被存储到UXML文件中。

## 定位视觉元素

UI Builder提供两种定位选项：

1. 相对定位：默认设置，子元素遵循父容器的Flexbox规则。相对定位动态移动和调整元素尺寸的依据：
	1. 父元素的尺寸和样式规则
	2. 子元素本身的大小和样式规则
2. 绝对定位：子元素锚定/覆盖在父容器上，Grow、Shrink、Margin这类flex相关的设置会被忽略，可以使用Left、Top、Right和Bottom设定锚点。

相对布局适用于永久可见，组合复杂的元素，绝对定位适用于类似弹出窗口的临时UI，装饰性元素或需要参照其他元素定位的元素。

## Size设置

视觉元素只是容器，在Unity6中，它们默认的Grow设置为1，意味着它们会占据容器所有的可用空间，除非其或子元素设定了固定的宽高。

Width和Height参数决定了元素的尺寸，Max Width和Max Height限制元素的最大尺寸，Min Width和Min Height限制元素的最小尺寸，尺寸单位可以设置为像素单位或百分比，这些会影响Flex设置如何根据可用空间调整元素的大小。

## Flex设置

Flex设置可以在使用相对定位时影响元素的尺寸。

**Basis**指在任何Grow或Shrink比例操作发生前，元素的默认宽高：

- Grow=1时，元素会占据父容器所有纵向和横向空间，Grow=0.5时则会占据一半空间
- Grow=0时，元素不会扩展并超出其基础大小
- Shrink=1时，元素会尽可能收缩，以适应父容器的可用空间
- Shrink=0时，元素不会收缩，可能会发生溢出

> [!NOTE] 计算元素尺寸
> 当使用相对定位时，布局引擎结合Size和Flexbox设置决定元素大小。
> 1. 根据Width和Height属性计算元素尺寸
> 2. 检查父容器是否有额外的可用空间，或子元素是否已经溢出
> 3. 如果有额外空间，查找Flex/Grow为非零值的元素，按比例扩展子元素尺寸
> 4. 如果子元素溢出，查找Flex/Shrink为非零值的元素，按比例收缩子元素尺寸
> 5. 考虑其他影响元素尺寸的属性，例如Min Width，Flex Basis等

Direction决定子元素在父容器内部如何排列，位于越高层级的子元素越先出现。

Wrap决定子元素是否需要尝试适应单行或单列布局（No Wrap），否则将溢出的元素排列到新的行列上（Wrap或Wrap reverse）。

## Align设置

Align设置决定子元素在排列时如何对齐父元素。设置父元素的**Align Items**，让子元素对齐首端（start）、中心（center）或尾端（end）进行排列，这些选项会影响交叉轴（垂直于主轴Flex Direction）。

**Stretch**选项也会影响交叉轴，但是Size中的Min和Max值会限制其效果。同时，**Auto**选项表示布局引擎可以根据其他参数动态选择其他一个选项，但是建议只在特殊情况下使用Auto选项。

**Justify Content**决定布局引擎如何将父容器中的元素分隔开。

## Margin和Padding

Margin和Padding用于设置元素周围的空白空间，Unity使用的是标准CSS盒子模型的一个变体。

- Content Space（内容空间）：容纳视觉元素的空间
- Padding（内边距）：在边框内部，围绕主空间的空白区域
- Border（边框）：内边距和外边距之间的边界，厚度向内扩展，可以设置颜色和圆角
- Margin（外边距）：在边框外部，围绕主空间的空白区域

## 背景和图像

在UI Toolkit中，任何视觉元素都可以显示一张图像，只需要将background属性设置为一张纹理或精灵即可，还可以填入颜色改变元素的外观。

## 可变或固定测量单位

在UI Builder中会遇到决定元素间距和尺寸的四种参数：

- Auto：默认选项，布局引擎根据父元素和子元素的信息计算数值
- Percentage：相当于容器的一个百分比单位，随着父元素的宽高动态改变
- Pixels：像素单位，在希望元素大小固定时有用
- Initial：将属性设置回默认状态（Unity自己的默认样式规则），忽略当前样式

如果希望对整个UI应用一个缩放规则，可以设置Panel Settings的Scale Mode参数：

- Constant Pixel Size：将元素保持在一个固定的像素大小，不受屏幕大小影响
- Constant Physical Size：将元素保持在各种屏幕上的相同物理大小，当实际的屏幕DPI不同于参考DPI时，基于参考DPI缩放UI元素
- Scale with Screen Size：根据分辨率动态调整元素尺寸。Screen Match Mode决定当缩放时优先考虑宽或高还是两者的混合；Reference Resolution设置UI的基础尺寸。

## UI Builder中的覆盖属性

被修改的属性在Inspector中会以粗体和白线高亮显示，表示它们现在覆盖了默认值或USS中的值，这种行为也叫做“内联样式”。

如果一个值不需要修改，最好保持其为默认状态。

## 将UXML作为模板

UXML文件可以像预制体一样使用，在任意UXML右键选择Create Template即可创建模板，之后就可以将模板添加到任何的视觉元素中或通过代码实例化，模板可以在Project视图下找到。

# 样式

## USS选择器

在没有创建USS文件时，所有的样式更改都会作为内联样式直接嵌入在UXML中，通过USS定义选择器，就可以让样式表在整个项目中共享和复用。

### 将内联样式转换为选择器

使用Add Style Class To List按钮可以将一个元素的所有内联样式转换为选择器，对选择器的更改会自动反映到所有关联的元素上。

要将特定的内联样式提取到一个新的选择器，可以点击属性旁边的三点符号，选择Extract Inlined Sytle to Selector/Add Class。

### 创建新的选择器

选择器会查询视觉树中匹配给定搜索条件的元素，然后UI Toolkit将样式应用到所有匹配的元素上，在UI Builder左上角点击Add new selector输入框添加新的选择器。

USS选择器通过以下方式匹配元素：

- 元素的C#类型：匹配Library面板中可用的默认类型名称，例如`Button`匹配所有Button类型的元素
- 名称或ID：匹配元素的名称，名称选择器以`#`作为前缀，例如`#title`匹配所有名字为title的元素
- 样式类：匹配元素的类列表中的类名，样式类选择器以`.`作为前缀，例如`.smallFont`匹配所有使用了smallFont样式的元素
- 直接子元素：在匹配条件后添加一个`>`，只有满足第二个的条件的直接子元素会受到影响，例如`#title>Lable`匹配名称为title的元素里面（第一层）任何的Label类型元素
- 伪类：伪类选择器用于定义元素在不同状态下的不同样式，例如`Button:focus`定义元素被聚焦时的样式

若一个元素匹配多个选择器，拥有最高特异性的选择器享有更高的优先级，在USS中的特异性层级如下：

1. 内联样式：直接应用在元素上的样式具有最高优先级，能覆盖所有的USS选择器
2. ID选择器：最明确的USS选择器，应用于特定名称的元素
3. 类选择器：应用于Class List中持有相关类的元素
4. C#类型选择器：应用于所有指定类型的元素

若多个选择器尝试应用于同一个属性，并且拥有相同级别的特异性，那么在Class List中最低层的选择器会被应用。

## USS过渡动画

通过配置Property、Duration、Easing和Delay来设置动画，当相关样式变化时，过渡会自动进行。

- Property：定义插值对象（属性）
- Duration：过渡时长，以秒或毫秒为单位
- Easing：定义动画如何随时间进行，模拟自然运动，例如加速、加速或弹性
- Delay：指定过渡开始前的等待时长
- Add Transition：每一个属性都可以分别用不同的参数进行动画，通过添加多个重叠的过渡动画，可以让这些动画同时触发，效果更自然

> [!NOTE] 过渡事件
> 可以为元素添加过渡事件回调，实现更加高级的工作流，例如序列和循环。以下是一些常见的过渡事件：
> 
> - TransitionRunEvent：过渡被创建时发送
> - TransitionStartEvent：过渡的延迟阶段结束，将要开始时发送
> - TransitionEndEvent：过渡结束时发送
> - TransitionCancelEnvent：过渡被取消时发送

对于视觉元素来说，由于伪类可以可以独立定义样式规则，动画不需要额外代码。当一个伪类触发了样式变化，所有定义的过渡会自动处理变化动画。

伪类是预定义的，不能自己创造新的伪类。

## 按需切换样式

使用UI Element API可以在代码中更换样式，例如根据角色稀有度来改变样式，可以使用`RemoveFromClassList`和`AddToClassList`方法。

```CS
if (character.rarity == RarityType.Legendary)
{
	visualElement.RemoveFromClassList("common");
	visualElement.AddToClassList("legendary");
}
```

另外，可以通过触发`:active`或`:inactive`伪类（根据元素的enabled状态），在改变状态时进行USS过渡。

## 主题

如果想根据季节变化调整UI或提供不同颜色的样式，使用Theme Style Sheets（TSS）可以简化这个过程。

TSS是一种操作像普通USS文件的资产文件，由USS选择器和属性、变量设置组成，提供一个自定义主题的切入点。

根据已有主题创建新主题的工作流可以是这样：

1. 创建新的TSS，添加要继承的主题以及新的USS文件
2. 在UI Builder > StyleSheets中，点击Add Existing USS，选择新主题使用的USS
3. 复制要覆盖的选择器到新主题的USS列表，选择Set as Active USS
4. 在新的USS中编辑选择器

在运行时，在PanelSettings的Theme Style Sheet处引用新主题。

# 命名规范

在UI Toolkit中需要使用字符串标识符来查找视觉元素和USS，因此使用一套命名的标准有利于代码的可读性并减少错误。

**Block Element Modifier**(BEM)是在现代网站开发中广泛使用的一种命名规范，在UI Toolkit中也推荐使用它。

BEM名称可以描述一个元素是做什么的，出现在哪里，以及如何与周围的其他元素关联，BEM有三个主要成分：**block-name__element-name--modifier-name**，例如navbar-menu__shop-button--small。

每一部分可以由拉丁字母、数字和短线组成，并由双下划线或双短线连接。

- block-name：代表高层组件，像是一个navbar-menu、玩家状态，任何明确有意义的UI组件。除非是不属于任何特定模块的通用组件，该部分可以省略，例如button-small
- element-name：代表一个模块的子元素或其中一部分元素，这些元素依赖于模块而存在，例如shop-button和navbar-menu__shop-button是位于不同模块中的
- modifier-name：代表模块或元素的一种变体或状态，可以是当按钮被按下，文本框选项被选择等

BEM命名的例子：

- menu__button-home
- menu__button-shop
- navbar-menu__shop-button--small
- navbar-menu__shop-button--large

# 文本

UI Toolkit使用一种基于TextMeshPro的字体渲染技术，叫做TextCore，它能在各种分辨率下渲染出清晰的文本并提供高级的样式功能。TextCore具备有向距离场字体渲染的优势，即使经过缩放或其他变换，也能生成锐利的文本。

## 源字体文件

常见的字体格式如TTF和OTF，在Unity中需要转换为字体资产才能够使用。字体资产是一种Unity专用的资源，包含渲染字体所需的数据，包括字形、字体度量参数和渲染设置（如大小、粗细和样式）。

选中源字体文件，在右键菜单中选择Create>Text Core > Font Asset > SDF即可创建相关字体资产。

## 字体资产设置

将源字体文件转换为字体资产后，可以进行设置以控制字体的生成。以下是一些关键选项：

- Face Info：间距和缩放，用于对源字体进行微调
- Generation Settings：包括源字体、字体样式、图集生成模式和渲染模式等基本配置
- Atlas and Material：设置图集为静态/动态，渲染模式为bitmap/SDF，控制生成的图集大小
- Font Weights：可模拟不同字体粗细程度
- Fallback Font Asset：备用字体，以防当前字体资产缺少文字或字形
- Character and Glyph Tables：包含在字体资产中所有文字和字形的详细列表
- Glyph Adjestments：定义逐文字或字形覆盖

> [!NOTE] 间距和图集分辨率
> 在字体纹理中的文字之间需要一些间距，避免在渲染时重叠。间距也为SDF渐变过渡创造空间，间距越大，过渡就越平滑，以提供高质量的渲染和厚描边等效果。
> 
> 若只使用ASCII字符，512x512的图集分辨率和5边距一般就足够了。对于需要更多字符的字体，就需要更大的分辨率或多个图集。一般情况下，间距和采样大小的关系为1:10。

### 字体资产变体

在需要对字体显示参数做变化时，可以创建变体，而无需创建一个新的字体图集。

变体引用原图集，但是它会存储自己的字体设置，例如行高和下标位置。这样，它就可以区别于原字体资产有自己的样式，不需要消耗额外的纹理空间。

## 富文本

富文本标签是一种通过在文本中补充额外标签来改变文本外观和布局的方法，标签可以让文本在运行时被格式化。

富文本标签可以改变文本的颜色或对齐方式，而无需修改字体属性或样式，可用于在视觉上强调你所想要传达的内容。

### 渐变

在UI Toolkit中使用`<gradient>`标签来应用渐变，以下是操作步骤：

1. 通过Create>Text Core>Gradient Color创建渐变色资产，文件需要放在Resources文件夹或其子文件夹
2. 创建一个Text Settings资产，在Panel Settings中引用，填写渐变色资产所在文件夹路径
3. 在UI Builder中开启Rich Text，添加富文本标签，如`<color=white><gradient="testColorGradient">Gradient Test</gradient></color>`

### 精灵资产和表情符号

还可以通过富文本标签在文本中添加表情符号，这需要使用和渐变色资产类似的精灵资产。当导入多个精灵时，将它们打包到一张图集以减少绘制调用，确保图集的分辨率设置对于目标平台是适合的。

按下列步骤导入精灵：

1. 导入包含表情或图标的精灵或PSD文件
2. 将图像切分成多个精灵，通过Create>Text Core>Sprite Asset从文件创建精灵资产，放置在Resources文件夹中
3. 可以调整Face Info和自定义每一个“字形”的外观和名称

在UI Toolkit中使用精灵资产：

1. 选择UI Document的Panel Settings，打开Text Settings资产
2. 将精灵资产关联到Text Settings，保存并进入Play Mode，更新设置
3. 使用富文本标签添加精灵，`<sprite index=0> or <sprite name="name">`

### 文本样式表

如果你的应用需要处理大量我呢本，可以考虑通过Assets>Text Core>Text Stylesheet创建一个文本样式表来管理格式，这样可以用富文本标签`<style>`来设置自定义的文本样式。

# 数据绑定

本质上，用户界面就是玩家与驱动游戏的数据之间的连接，是玩家与游戏内部状态和逻辑交互的首要方式。

视图与模型之间的关注点分离是UI架构中的核心原则，将界面从背后的数据中解耦可以让代码更加灵活、可复用。然而一旦分离之后，为了将模型和视觉连接起来又需要进行同步。传统做法是直接更新或使用事件驱动系统，这些同步操作会带来重复的样板代码。随着项目规模的增长，添加新元素或依赖经常需要额外的更新逻辑或事件处理，这些系统变得难以管理。

在Unity6中的运行时数据绑定提供一种简洁高效的解决方案，它将应用数据直接与UI元素连接，确保一方改变能自动反映给另一方。

Model-view-viewmodel（MVVM）架构在view和model之间添加了一层表示逻辑，viewmodel扮演一个中介者，将model中的数据进行格式化后提供给view展示。例如，一个生命条能够自动展示玩家的生命值，一个分数标签能实时更新，而无需额外脚本逻辑或手动处理事件。

![[Pasted image 20260129155050.png]]

## 数据绑定概念

Unity6引入了运行时数据绑定系统，提供一种结构化的方式来连接UI元素和应用数据。要将一个元素的属性绑定到数据源，需要创建一个**DataBinding**实例。

以下是几个重要概念：

- Data source（数据源）：持有UI绑定数据的对象
- Data source path（数据源路径）：UI元素连接到数据源的属性或字段
- Binding mode（绑定模式）：控制数据流单向/双向流动

## 准备数据源

任何C#对象都可以作为数据源，包括ScriptableObjects，MonoBehaviours，或自定义的C#对象。使用结构体作为数据源可以提高性能，因为其内存分配更加轻量化，能减少内存垃圾。数据绑定可以通过代码或Inspector完成。

### 使用CreateProperty特性

为了将属性提供给绑定使用，UI Toolkit需要依靠由**Unity Properties**模块生成的属性包（property bags），属性包定义了数据源中哪一些属性能够被UI绑定所访问。

使用`CreateProperty`特性，显式地为绑定系统标记属性，让属性可被绑定。

```CS
[SerializeField, DontCreateProperty]
int m_Value;

[CreateProperty]
public int Value
{
	get => m_Value;
	set => m_Value = value;
}
```

> [!NOTE]
> 运行时数据绑定使用属性包来高效遍历和操纵一个类型的数据，默认情况下，在首次访问某个类型时，Unity利用反射来生成属性包，这会带来一定的运行时开销。为了避免这个开销，在定义属性时使用CreateProperty特性，可以在编译时生成绑定代码，从而消除运行时反射的需求。

### 数据源和路径

数据源路径用于指定你想要连接到UI元素的数据源中的属性或字段，例如数据源中有一个“health”属性，路径会直接指向该属性。

#### 在UI Builder中

选择某个元素，在Inspector中从某个属性的左侧三点菜单中使用Add Binding，引用数据源对象，例如一个ScriptableObject，然后指定数据源路径。

#### 在UXML中

通过UI Builder设置数据绑定后，会自动生成相应的UXML。你也可以在文本编辑器中手动添加和数据源路径。

```XML
<Bindings>
	<ui:DataBinding property="text" data-source-path="Health"/>
</Bindings>
```

#### 使用C\#

可以在脚本中实例化或引用数据源对象，将其赋值给根元素的dataSource属性，再使用dataSourcePath指定绑定的属性。

```CS
var label = new Label();
var parentData = ScriptableObject.CreateInstance<PlayerDataSO>();
playerData.Health = 100;

label.SetBinding("text", new DataBinding()
{
	dataSource = playerData,
	dataSourcePath = new PropertyPath(nameof(PlayerDataSO.Health)),
});
```

注意：同时使用多种方式为同一个UI元素定义数据绑定，可能会出现冲突，因此：

- 对于不需要运行时调整的静态或默认配置数据，使用UI Builder/UXML绑定
- 对于动态更新或数据源在运行时需要改变的数据，使用C#绑定

### 继承数据源

视觉元素会自动从父级继承数据源，也就是说如果父元素由一个数据源，其所有子元素都会默认使用它，需要时可以给子元素重新指定数据源。

## 绑定模式

- TwoWay：变化可以从数据源传递到UI，也可以从UI传递到数据源。在用户可交互更改数据的元素上使用该模式，例如滑动条或文本框
- ToTarget：数据仅从数据源流向UI，在只读的UI元素上使用
- ToSource：数据仅从UI流向数据源，用于一开始不需要显示当前值的输入
- ToTargetOnce：数据仅从数据源流向UI一次

