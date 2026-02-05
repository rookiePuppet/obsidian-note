
在某些情况下，必须手动编辑我们的资产文件，例如在发生合并冲突或文件损坏时。

本文会深入解读Unity的序列化系统，分享直接修改资产文件能实现的效果。

## YAML结构

YAML也就是“YAML Ain't Markup Language”（YAML不是标记语言），是人类可读数据序列化语言家族的一员。因为它相对其他语言（如XML和JSON）更加轻量化、相对直接，所以被认为更加易读。

Unity使用的是一个实现YAML标准子集的高性能序列化库，不支持空行、注释和其他一些语法，在一些特殊情况下，Unity的YAML格式与标准会有偏差。

下面是在Unity中创建一个默认立方体转换为预制体后，在文本编辑器中打开查看到的YAML代码。首先是文件头部，第一行定义的是使用的YAML版本，第二行为URI前缀“`tag:unity3d.com,2011:`”创建了一个叫做“`!u!`”的宏。

```YAML
%YAML 1.1
%TAG !u! tag:unity3d.com,2011:
```

在头部下面，是一系列的对象定义，例如在预制体或场景中的GameObject和每一个GameObject的组件，可能还有场景的Lightmap设置等其他对象。

```YAML
--- !u!1 &8389683831438724747
GameObject:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  serializedVersion: 6
  m_Component:
  - component: {fileID: 6723542853937848056}
  - component: {fileID: 6077688992829448166}
  - component: {fileID: 835210346175377398}
  - component: {fileID: 7638806868723225942}
  m_Layer: 0
  m_Name: Cube
  m_TagString: Untagged
  m_Icon: {fileID: 0}
  m_NavMeshLayer: 0
  m_StaticEditorFlags: 0
  m_IsActive: 1
```

每个对象定义都从一个两行的头部开始，第一行格式为`--- !u!{CLASS ID} &{FILE ID}`。

- `!u!{CLASS ID}`：描述对象所属的类，其中“!u!”就是文件头定义的宏，数字1指向的就是GameObject的类ID。每一个类的ID在Unity的源代码中定义，[这里可以查阅](https://docs.unity3d.com/Manual/ClassIDReference.html)。
- `&{FILE ID}`：定义该对象自己的ID，用于在各个对象直接相互引用。之所以叫做文件ID，是因为它表示的是在特定文件中的一个对象的ID。

第二行就是该对象类型的名称，可以让你能够识别该对象的类型。

在对象头部之后，是其所有的序列化属性，ScriptableObject、Animation、Material等类型的序列化使用的也是类似的格式。注意ScriptableObject使用MonoBehaviour作为其对象类型，是因为其内部使用MonoBehaviour类作为宿主。

## 局部引用

每个对象在YAML文件中都有一个唯一的“File ID”，用于对象之间的相互引用，YAML格式使用`{fileID: FILE ID}`来表示这种引用值，若File ID为0则代表空引用。

```YAML
Transform:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: 8389683831438724747}
```

想象一下在预制体中重新设定父子关系的情形，你可以更改Transform的`m_Father`属性的File ID为新的父变换ID，再把Transform的File ID加入新的父Transform的`m_Children`数组，并从原Transform的`m_Children`中移除。