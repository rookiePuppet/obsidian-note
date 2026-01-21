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

