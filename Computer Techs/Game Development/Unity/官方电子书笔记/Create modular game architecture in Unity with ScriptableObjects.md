# 引入

就像Unity的Component一样，ScriptableObject在场景背后默默工作，你可能直到需要使用时才会注意到它们的存在。

你会经常听说ScriptableObject是一种“数据容器”，但这个标签并不能完全地描述它。合理地使用ScriptableObject可以帮助你提高Unity工作流效率，减少内存使用，并简化代码。

这篇指南集成了专业开发者在生产中应用ScriptableObject的一些提示和技巧，也包括了如何将其应用于特定的设计模式的例子，来避免常见的潜在问题。

对于有经验的Unity开发者，ScriptableObject能通过分离数据和逻辑来促成干净的代码。这使修改变得更加容易，也不会产生意料之外的副作用，提高可测试性和模块性。

由于你可以在编辑器中可交互式地操作它们，和非程序员（例如美术和策划）一起合作时会非常有用。

接下来让我们来探索这个游戏架构的无名英雄，不起眼的ScriptableObject。

# 什么是ScriptableObject?

ScriptableObject是不属于游戏对象（GameObject）实例一部分的一个Unity对象，你可以用它来创建一个拥有自己的变量和方法的自定义类，而且开销会比MonoBehaviour小。

ScriptableObject没有Transform组件，它不存在于场景层级之中，而是作为一个项目级别的资产（asset），就像一个材质或3D模型一样。

可以像下面这样声明一个ScriptableObject：

```C#
[CreateAssetMenu(fileName="MyScriptableObject")] 
public class MyScriptableObject: ScriptableObject
{ 
	public int someVariable; 
}
```

但是，现在这个ScriptableObject还不能在GameObject上面使用。CreateAssetMenu特性会在项目窗口右键菜单中提供一个创建该ScriptableObject的操作，点击它，然后就可以从你的ScriptableObject类实例化出一个自定义的资产了。

其他的脚本可以使用一个字段在场景中引用该ScriptableObject资产：

```C#
public class MyMonoBehaviour : MonoBehaviour 
{ 
	public ScriptableObject soInstance; 
}
```

对于那些不需要在运行时更新的事物，ScriptableObject是特别有用的。

由于Unity将ScirptableObject作为第一类对象，你可以：

- 将它们存储在变量中
- 在运行时动态地创建和销毁它们
- 将它们作为参数传递
- 从方法中返回它们
- 在数据结构中引入它们
- 序列化/反序列化它们

最后一点强调了ScriptableObject的一个重要特性，即可以显示在Inspector窗口中，这意味着它里面的字段在编辑器当中是容易被查看和修改的。

在运行时改变ScriptableObjet里的值，游戏也会随之立刻更新。即使退出Play mode，更改后的值依然保留。也就是说，ScriptableObject在程序运行时也能保存更改，这对需要平衡游戏设置而无需编写代码的游戏设计师非常有用。

这比单独使用MonoBehaviour多了一些优势，利用ScriptableObject重构项目可以帮助你更好地进行团队协作，以及提高内存使用率。

> 序列化(Serialization)

序列化是一个自动化的流程，将数据结构或者对象转化成一种容易存储和重建的格式。

Unity的序列化后端接受分散在内存上的数据，然后将它们按顺序排列。这将数据流重新组织，之后存储在数据库，文件，或者内存中。反序列化（Deserialization）就是相反的过程。

![[Pasted image 20240227223820.png]]

虽然内存布局在C#开发中很容易被忽视，但我们应该注意到一些使用了序列化的Unity内置功能：

- 保存和加载：如果使用文本编辑器打开一个.unity后缀的场景文件，并在Unity中设置”fore text serialization“，那么序列化器会使用YAML后端运行。
- Inspector窗口：这个接口不需要通过C# API来计算它正在检查的值，而是让对象序列化自身并显示到窗口中。
- 预制体：在内部，预制体是游戏对象和组件的序列化数据流。一个预制体实例是应该在序列化的数据之上进行的修改列表。
- 实例化：当你实例化一个预制体（或者一个存在于场景中的游戏对象）时，首先会序列化对象，创建一个新对象，然后将数据反序列化进新对象。

# ScriptableObject 对比 MonoBehaviour

从表面上来看，ScriptableObject是简单的，其API只有几个方法，而简单也意味着出错的可能性更小。

![[Pasted image 20240227224438.png]]

和MonoBehaviour一样，ScriptableObject也继承于UnityEngine.Object。

## 比较

也许理解ScriptableObject的最好方式就是和它的兄弟MonoBehaviour进行比较，下面这个图表列出了它们的相同与不同之处。

| MonoBehaviour                                            | ScriptableObject                                                                                                                |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 都是脚本，都继承于UnityEngine.Object。                             |                                                                                                                                 |
| 接收来自Unity的回调，根据MonoBehaviours的事件函数命名方法，将它们连接到游戏引擎的玩家循环中。 | 不接收来自Unity的大部分回调，在运行时支持少数事件函数，包括Awake，OnEnable，OnDestroy和OnDisable。编辑器还会从Inspector中调用OnValidate和Reset。你可以创建其他方法，但玩家循环不会自动地调用它们。 |
| MonoBehaviour在运行时必须依附于游戏对象，在运行时使用AddComponent API来创建。    | 每一个ScriptableObject实例都可以保存到自己的项目级别的文件中。                                                                                         |
| 在编辑器中，退出Play mode时会重置对MonoBehaviour的修改。                  | 在编辑器中，退出Play mode不会重置对SciptableObject的修改。在独立构建中，运行时对ScriptableObject值的修改不会被保存。                                                  |
| 都是可序列化的，能够在Inspector中查看。                                 |                                                                                                                                 |

## 回调与消息

ScriptableObject拥有MonoBehaviour可用事件函数的一个子集，你可以为ScriptableObject自定义方法，但是得你自己去调用它们。只有下面这些方法会在PlayerLoop中自动被调用：


| 事件函数（运行时）  | 执行时机                                                                                                      |
| ---------- | --------------------------------------------------------------------------------------------------------- |
| Awake      | 当ScriptableObject脚本启动时被调用，类似于MonoBehaviour的Awake回调。当游戏启动或者一个场景加载时引用了ScriptableObject资源时，也会执行。             |
| OnEnable   | 当ScriptableObject被加载或实例化时，在Awake回调之后立即调用。OnEnable会在ScriptableObject.CreateInstance期间或者脚本重新编译完成后执行。        |
| OnDisable  | 当ScriptableObject超出范围时被调用。这在当你加载一个没有ScriptableObject资源引用的场景时，或在ScriptableObject的OnDestroy之前发生。            |
| OnDestroy  | 当ScriptableObject被销毁时，即从编辑器或者代码中删除它时会调用。当你在运行时创建ScriptableObject，OnDestroy也会在程序退出时执行。注意：这只会销毁该对象的原生C++部分。 |
| 仅编辑器下的函数   |                                                                                                           |
| OnValidate | 在脚本被加载或在Inspector中某个值变化时执行，这可以用来确保你的数据维持在一个特定的范围。                                                         |
| Reset      | 当你点击Inspector上下文菜单中的Reset按钮时调用。                                                                           |
销毁一个ScriptableObject，可以将它从编辑器中删除或在运行时调用Destroy或DestroyImmediate。

这是一个ScriptableObject事件函数和生命周期的简要概述，试着将它与MonoBehaviour事件函数的执行顺序进行对比。

![[Pasted image 20240227235918.png]]

## 文件

MonoBehaviour和ScriptableObject之间的最大区别之一，就是它们保存数据的方式。

Unity将MonoBeviour序列化在一个Scene或者Prefab文件中，其保存的数据包含：

- MonoBehaviour自身
- 依附的游戏对象
- 它的Transform
- 其他组件以及依附的游戏对象上的MonoBehaviour

相比之下，Unity将ScriptableObject保存在它们自己的资产文件中，这些文件比MonoBeviour更小，更分离。

如果你在`Project Settings>Assets Serialization`窗口中选择使用`Mode：Force Text`，你可以在一个文本编辑器中打开一个ScriptableObject资产，它可能是像这样的：

![[Pasted image 20240228000953.png]]

> YAML不是标记语言

Unity使用的是一个实现YAML规范子集的高性能序列化库，YAML是一种与XML和JSON相关的轻量、容易阅读的语言。

在YAML中，数据被组织为嵌套元素的层次结构。每一个对象都有一个Class ID，File ID，和Object type。注意，ScriptableObject使用MonoBehaviour作为它的Object type，而没有声明自己的类型。

![[Pasted image 20240228001752.png]]

在每一个对象下面，是它的序列化属性，以键值对的形式表示。

关于YAML更多的信息，查看[“Understanding Unity’s serialization language, YAML.”](https://blog.unity.com/engine-platform/understanding-unitys-serialization-language-yaml?utm_source=demand-gen&utm_medium=pdf&utm_campaign=clean-code&utm_content=scriptable-objects-ebook)。

## 创建与生命周期

ScriptableObject的生命周期和在你的项目中的其他资产（材质，纹理等等）是相似的。

在之前的例子中，使用`CreateAssetMenu`特性可以在编辑器中增加一个自定义的菜单选项，你可以选择性地指定默认文件名称和菜单项顺序，这是一种创建ScriptableObject资产最常用的方式。

如果你需要在运行时创建ScriptableObject，你可以调用这个静态的方法：

`ScriptableObejct.CreateInstance<MyScriptableObejctClass>();`

> 销毁ScriptableObject

和其他Unity对象类似，ScriptableObject由原生C++部分和C#托管部分组成。你可以直接销毁原生C++部分，但C#托管部分需要等资产垃圾回收器去清除。当你改变场景或调用`Resources.UnloadUnusedAssets`时，会触发GC的清理。

![[Pasted image 20240228003113.png]]

显式地将对ScriptableObject的引用置为null，可以避免垃圾回收的延迟。在调用`Destroy`或`DestroyImmediate`之前，这个操作是很重要的。

一旦你掌握了创建和销毁ScriptableObject的技巧，是时候去探索在游戏程序中使用它们的一些创造性方法了。

# 数据容器

ScriptableObject最普遍的使用就是作为数据容器共享数据，尤其是不会在运行时改变的静态的游戏配置数据。

ScriptableObject的典型使用案例包括：

- 库存清单
- 敌人，玩家，或物品数据
- 音频集合

在运行时，你可以在MonoBehaviour上存储这些数据，但这样是低效的。正如你在前面的比较所看到的，MonoBeviour由于需要GameObject（默认带有一个Transform）来作为宿主，需要额外的开销。这意味着，你需要在存储某个数据之前创建很多未使用的数据。

为了让你亲眼看看，现在我们生成一个新的GameObject，附带一个空的MonoBehaviour，然后将序列化的对象用文本编辑器打开：

对比一个空的ScriptableObject：

ScriptableObject减少了内存占用，删除了GameObject和Transform，同时它在项目级别上存储着数据，这在你想从多个场景中访问一份相同的数据时非常有用。

MonoBehaviour中额外的数据可能在一开始并不会影响程序性能，但随着游戏的扩展，有了更多的对象时，将会明显影响游戏性能。

> ScriptableObject对比持久化数据

当ScriptableObject作为数据容器时，这通常指的是在运行时不需要修改的数据。

虽然对ScriptableObject数据做的修改确实会被编辑器保存（就像修改一个材质资产一样），但在应用构建的运行时不会保存。

需要从一个会话保存然后加载到另一个会话中的持久化数据，通常被存储在不同格式的文件中（例如JSON，XML，MessagePack，Protocol Buffers等等）。

在游戏构建的运行时修改ScriptableObject数据是可行的，但这些修改时临时。重新启动游戏后，ScriptableObject的数据会重置到（构建时的）原始状态。

相对于保存在一个外部文件中可读可写的持久化数据，可将ScriptableObject数据视为只读的。

## 减少重复数据

想象一下你有一千个附带自定义MonoBehaviour的GameObject，每一个都有几个字段。

如果每一个组件都有它自己的那些成员变量的拷贝，那意味着你可能会携带大量的重复数据。

![[Pasted image 20240228143257.png]]

因为有一些数据没必要在运行时改变，所以这有点低效。相反，你可以将数据汇集到ScriptableObject中，这样成千个对象都可以指向这份共享的数据资产，每一个对象存储着对这份数据的引用而不是重新拷贝一份数据。

![[Pasted image 20240228143303.png]]

在软件设计中，这是一种被称为享元模式的优化。利用这种方式重构你的代码，可以避免大量数据的拷贝，减少内存占用。

一个对ScriptableObject的引用对于一份完整数据的拷贝，是相对小的。随着规模的扩大，由于不复制数据而节省的内存会变得可观。

![[Pasted image 20240228145447.png]]

用这种方式，你可以存储大量的共享数据，考虑将ScriptableObject用于：

- 在编辑器中保存和存储数据
- 将数据保存为资产在运行时使用

不像MonoBehaviour，ScriptableObject不能附加在GameObject上。相反，你可以在项目中将它们作为资产进行保存，当你有一个在MonoBehaviour中使用不会更改的数据的预制体时，这非常有用。

## 重构举例

考虑一个控制NPC生命值的MonoBehaviour，你可能将它定义为：

```C#
public class NPCHealthUnrefactored : MonoBehaviour
{
	[Range(10, 100)]
	public int maxHealth;
	
	[Range(10, 100)]
	public int healthThreshold;
	
	public NPCAIStateEnum goodHealthAi;
	public NPCAIStateEnum lowHealthAi;
	
	public int currentHealth;
}
```

这能工作，但这里面有不会在运行时改变的数据。如果你有很多挂载了该组件的对象，这会导致大量不必要的重复数据。

![[Pasted image 20240228150441.png]]

任何不需要改变的数据都可以放进一个ScriptableObject：

```C#
[CreateAssetMenu(fileName="NPCConfig")]
public class NPCConfigSO : ScriptableObject
{
	[Range(10, 100)]
	public int maxHealth;
	
	[Range(10, 100)]
	public int healthThreshold;
	
	public NPCAIStateEnum goodHealthAi;
	public NPCAIStateEnum lowHealthAi;
}
```

使用`CreateAssetMenu`特性来配置菜单操作，你可以选择性地指定默认的文件名和菜单项次序。

> 本篇指南中的代码约定

在本指南中的许多代码样例都是为了便于说明和容易阅读，而做了简化，例如`public`字段。

在生产中，会使用`private`字段和`public`属性来做额外的封装和获得灵活性，对`private`字段应用`SerializeField`特性可以让其显示在编辑器Inspector中。

对命名的约定也可以帮助区分ScriptableObject和MonoBehaviour，一种方式是在类名末尾添加“Data”或者“SO”后缀，虽然这不是必要的，但它可以使你的项目保持条理性和减少歧义。

重构之后的NPCHealth组件简化成了这样：

```C#
public class NPCHealth: MonoBehaviour
{
	// Reference to our ScriptableObject
	public NPCConfigSO config;
	public int currentHealth;
}
```

现在MonoBahaviour包含了对这个新的ScriptableObject的引用，在Inspector中，重构后的内容看起来都很相似，只是数据被拆分了。

![[Pasted image 20240228151730.png]]

## 自定义Inspector

当把数据拆分进ScriptableObject，数据被放在了两个地方，ScriptableObject资源以及引用它的的MonoBehaviour。

为了使MonoBahaviour易于导航，考虑创建一个自定义编辑器。下面是关于NPCHealth的一个例子：

![[Pasted image 20240228152038.png]]

这使MonoBehaviour中的属性和NPCConfig中的变量放在一起，让你查看。如果你选择原始的NPCHealth预制体，你可以方便地在两个对象中编辑数据。

一个自定义的编辑器仅仅需要几行代码：

- 从`Editor`派生一个新的类，并将它存放在名为Editor的文件夹中，在该类上应用`NPCHealth`类型的`CustomEditor`特性。
- 为`NPCConfig`ScriptableObject保留一个临时编辑器。
- 在`OnInspectorGUI`中，为NPCHealth组件创建编辑器。
- 从基类和新的自定义Inspector绘制inspector。

```C#
using UnityEditor;

[CustomEditor(typeof(NPCHealth))]
public class NPCHealthEditor : Editor
{
	private Editor editorInstance;
	
	private void OnEnable()
	{
		// reset the editor instance
		editorInstance = null;
	}
	
	public override void OnInspectorGUI()
	{
		// the inspected target component
		NPCHealth npcHealth = (NPCHealth)target;
		
		if (editorInstance == null)
			editorInstance = Editor.CreateEditor(npcHealth.config);
		
		// show the variables from the MonoBehaviour
		base.OnInspectorGUI();
		
		// draw the ScriptableObjects inspector
		editorInstance.DrawDefaultInspector();
	}
}
```

你可以自定义属性抽屉和编辑器属性来扩展此示例，甚至可以在使用ScriptableObject有更好的用户体验。

## 架构优势

使用ScriptableObject，你可以将共享数据和非共享数据清晰地分离。GameObject实例中任何唯一的东西都保留在MonoBehaviour中，而共享数据存放在ScriptableObject中。

使用SciptableObject不仅仅是节约内存，使用ScriptableObject重构代码架构有如下好处：

- **设计师可以更独立于软件开发者工作**
	- 将数据和逻辑存储在一个单独的MonoBehaviour中，会造成开发者和游戏设计师跨越彼此工作的可能。如果两个人修改了同一个预制体或场景的不同部分，会造成合并冲突，解决这种问题既耗费时间又令人沮丧。
	- 将共享的数据拆分到更小的文件和资产中，能减少这些问题。使用ScriptableObject进行架构设计，可以让设计师不依赖于程序员构建游戏玩法，每个人都会更高兴。
	- 在使用共享数据时，要在团队中定义一个清晰的工作流。好的沟通和边界的建立可以防止出现上述情况（合并冲突）。一些额外的错误检查或数据验证也许也是必要的，例如使用`Range`特性或`OnValidate`防止错误值。
- **编辑共享数据会更快，更不容易出错**
	- 现在修改共享数据一触即发。如果你需要为一个NPC修改它的设定，你只需要在一个地方做修改即可，然后在每一个场景以及每一个受ScriptableObject影响的组件都会得到更新。
	- 这减少了大量手动编辑单个GameObject带来的潜在错误。将数据转移到ScriptableObject中对版本控制也有帮助，能够防止当团队成员在同一场景或预制体上工作时造成的合并冲突。
- **在播放模式保存游戏玩法调整**
	- 在编辑器中，播放模式是设计师实验游戏玩法和配置的机会。但是当退出播放模式时，任何对MonoBehaviour的修改都会丢失，因为Unity丢弃了场景的临时拷贝。
	- 由于ScriptableObject是资产，不管是否处于播放模式，对它们的修改都会随时更新，这在你需要在运行时调整游戏是非常有用的。
	- 但是，如果你想要重置那些修改，这可能是一种负担。只要记住依靠版本控制系统，你就总是可以在必要时候恢复你的工作。
- **改善场景加载时间**
	- 当保存一个场景或预制体时，Unity会序列化它们内部的一切，包括每一个GameObject，每一个依附于这些GameObject的组件，以及每一个`public`字段，Unity不会检查是否有数据重复。
	- 将数据转移至ScriptableObject中可以降低你的场景和预制体的大小。因为硬盘的I/O是相对慢的，这会显著影响加载和保存。

> ScriptableObject变量

你可以利用ScriptableObject制作仅表示一个值的更精细化的共享数据容器，例如，你可以创建只有一个`public`字段，名为`IntVariable`的ScriptableObject类：

```C#
[CreateAssetMenu(menuName = "Variables/Int", order = 1)]
public class IntVariableSO : ScriptableObject
{
	public int value;
}
```

然后你可以在MonoBehaviour中使用`IntVariable`，构建一个`PlayerHealth`类可以是像这样的。

尽管我们通常认为ScirptableObject保存不变的值，但你可以提供一些方法，在运行时更新数据（然后再退出播放模式时重置为初始值）。通过这种方式，你可以使ScriptableObject在本质上充当各种包含整型，浮点型，布尔型等等的变量。

这样，设计师们可以为游戏逻辑保留数据，而不必每次都需要软件开发人员处理。然而，这需要计划才能成功，也就要和你的设计师去决定如何划分创作玩法数据。

在如何协作上划分一些界限。例如，程序团队可能为库存系统做了ScriptableObject的初始化设置。然后，设计团队就可以使用它们来补充每一个物品的游戏属性或行为。

使用一些额外的编辑器脚本，这可以成为一种近乎无缝的体验。另一种可能性是让Inspector中的字段可以在使用ScriptableObject的共享值和常量之间切换，这使游戏设计团队可以更自由地重写每一个实例的ScriptableObject数据。

![[Pasted image 20240228164741.png]]

## 双重序列化

你可以混合在Unity中序列化数据的方式，这使你能够在编辑器中使用ScriptableObject，然后将数据存储在另一个位置，例如JSON或XML文件，从而利用不同文件格式的优势。

像JSON和XML的文件格式可能很难在编辑器中使用，但是可以在Unity外部使用任意的文本编辑器进行修改。

相比之下，ScriptableObject在编辑器中通过快速的拖拽操作是非常好用的。但是它在Unity外部就不容易修改或者在玩家社区中分享了。

将不同的序列化格式混合使用可以为你的游戏扩展新的可能性，比如关卡编辑或模组。在构建时，一个脚本可以将其他格式的文件转换成ScriptableObject，从而比纯文本更快地加载。

虽然你可能希望将一些敏感数据安全地隐隐藏在服务器上（如虚拟货币，账户信息），但将游戏的部分数据在社区公开出来也许能增强游戏玩法。如果你打开一个沙盒关卡给玩家进行实验，你可能会为玩家能够构建他们自己的关卡并在网络上分享而感到惊喜。

想象一个定义游戏关卡布局的ScriptableObject，它可能只是简单包含了一些定义预制体位置的Transfrom，启动配置等等。你的游戏脚本将使用该数据组装每一个关卡。

想象将游戏中墙壁和玩家的起始位置被存储在一个ScriptableObject中：

```C#
[CreateAssetMenu(fileName ="LevelLayout")]
public class LevelLayout : ScriptableObject
{
	public Vector3[] wallPositions = new Vector3[2];
	public Vector3[] playerPositions = new Vector3[2];
	public Vector3[] goalPositions = new Vector3[2];
	public Vector3 ballPosition;
}
```

它定义了关卡的设置方式，你的关卡管理脚本可以从`LevelLayout`对象读取数据，然后在正确的位置实例化你的预制体。

一个自定义脚本可以使用`JsonUtility`将相同数据导出到硬盘，在编辑器外部生成一个文本文件，然后你的用户可以使用外部工具对其进行修改。

为了加载一个自定义修改的关卡，可以利用`ScriptableObject.CreateInstance`在运行时生成一个ScriptableObject，然后从JSON文件中读取文本填充进去。这个`LoadLevelFromJson`示例方法展示了上面的操作：

```C#
using System.IO;
public class LevelManager : MonoBehaviour
{
	public ScriptableObject levelLayout;
	public void LoadLevelFromJson(string jsonFile)
	{
		if (levelLayout == null)
		{
			levelLayout = ScriptableObject.CreateInstance<LevelLayout>();
		}
		
		var importedFile = File.ReadAllText(jsonFile);
		JsonUtility.FromJsonOverwrite(importedFile, levelLayout);
	}
}
```

自定义数据被替换进了ScriptableObject中，这使你能够使用这个外部编辑的关卡，就像在游戏中的其他关卡一样。

![[Pasted image 20240229104429.png]]

注：使用JsonUtility将JSON反序列化成ScriptableObject时，必须使用`FromJsonOverwrite·方法`。JsonUtility会将JSON数据加载进一个已存在的对象，而不会创建一个新的对象再加载数据进去，从而无需任何内存分配。

> 保护你的数据

这个简单的mod示例演示了ScriptableObject的一种可能应用。但是，当暴露游戏数据以供修改时，应当谨慎行事，避免玩家篡改程序的其余部分。

以下是一些保护数据的常用方法：

- 加密：使用加密手段保护数据文件不被轻易阅读和修改，这能使用户更改关键数据变得更加困难。
- 电子签名：可以使用指纹算法来验证你的数据文件没有被篡改。
- 服务端验证：如果你的游戏依赖于存储在服务器上的数据，可以使用前检查服务器上的数据，然后丢弃被修改过的数据。

没有任何方法是万无一失的，所以将这些计数结合起来使用是一个好办法，防止玩家将任何漏洞或错误引入游戏中。

# 可扩展的枚举

在游戏开发中，通常需要解决反复出现或类似的问题。幸运的是，你可以利用设计模式，从已经经历过这些问题的工程师那里汲取积累下来的知识和经验。

设计模式是帮助你构建大型的可扩展的应用的通用解决方案，它们可以提供代码的可读性，使代码更加干净，还可以减少重构和花费在测试上的时间。

将设计模式作为一种模板来解决类似这些普遍的问题：

- 高效地存储大量数据
- 让对象可以在不同游戏系统中交互
- 在运行时动态地切换行为

ScriptableObject可以帮助实现其中的一些模式，你已经看到了它们使如何作为数据容器使用的，但它们能做的比单纯保存值或设置还要多。

## 类似枚举的类别

事实上，SctiptableObject不一定要包含任何有用的东西。如果你创建一个空的ScriptableObject，你会发现它仍然具有实用性，即使它仅用于和其他ScriptableObject进行比较。

在你的游戏应用中，假设你从`GameItemSO` ScriptableObject创建了若干的资产，像这样：

```C#
Using UnityEngine;
[CreateAssetMenu(fileName=”GameItem”)]
public class GameItemSO : ScriptableObject { }
```

![[Pasted image 20240229112223.png]]

这允许你在项目中生成许多的资产，即使它们不包含任何数据，但ScriptableObject本身就可以代表一种类别或物品的类型。如果用来判断是否相等，它的工作原理和枚举非常类似。

如果两个变量引用同一个ScriptableObject，那它们就是同一种类型，否则不是。

因此，你可以用一个ScriptableObject定义特殊的伤害效果（例如冰冻，灼烧，点击，魔法等等）或你最喜欢的零和游戏——剪刀石头布。

![[Pasted image 20240229113326.png]]

如果你的应用需要一个库存系统来配置游戏玩法物品，ScriptableObject可以代表物品类型或武器格子，从而在Inspector中可以通过拖拽操作设置它们。


![[Pasted image 20240229113803.png]]

使用ScriptableObject作为枚举，当你想要扩展它们以及增加更多数据时会变得更加有趣。不像普通的枚举，ScriptableObject可以有额外的字段和方法。

下面是适用于剪刀石头布游戏的`GameItem`，ScriptableObject本身仍然定义为类枚举类别，但这次它不再是空的。

```C#
public class GameItem : ScriptableObject
{
	public GameItem weakness;
	
	public bool IsWinner(GameItem other)
	{
		return other.weakness == this;
	}
}
```

现在这个ScriptableObject包含了一个`weakness`字段，用于决定在和其他物品进行潜在的交互时，哪个将获胜。除了存储数据，每一个ScriptableObject在`IsWinner`包含还包含了简单的比较逻辑。

每一个游戏玩法的物品需要一个MonoBehaviour，引用一个特定的ScriptableObject资产。

```C#
public class GameItemController : MonoBehaviour
{
	// rock, paper, scissors
	public GameItem gameItem;
	
	private void OnTriggerEnter(Collider other)
	{
		GameItemController otherController = other.GetComponent<GameItemController>();
		GameItem otherGameItem = otherController.gameItem;
		
		if (gameItem.IsWinner(otherGameItem))
		{
			Debug.Log(gameItem.name + " beats " + otherGameItem.name);
		}
	}
}
```

与枚举类型不同的是，ScriptableObject易于扩展，它不需要单独的查找表或者与新的数据数组相关联，只是简单地添加一个额外的字段或者处理逻辑的方法就行了。

![[Pasted image 20240229115557.png]]
对比维护一个传统的枚举，如果你有一个很长的没有明确编号的枚举值列表，添加或删除一个枚举值都会改变它们的顺序，这可能会引入微妙的漏洞或意料之外的行为。

基于ScriptableObject的枚举就没有这样的问题，不管是新增还是删除，都没有影响。

假设你想在一个RPG游戏中使物品可装备，你可以为ScriptableObject增加一个布尔类型字段，它可以表示某种角色是否允许拿取某种物品，某些物品是具有魔法的或有特殊能力的。

# 模式：委托对象

由于你可以在ScriptableObject中创建方法，所以它除了用于保存数据，还可以包含逻辑或操作，因此你可以把它们用于委托对象。

> 委托对比事件

委托是一种定义方法签名的类型，它允许你将方法作为参数传递给其他方法，可以将它们视为类似一种保存对方法而非值的引用的变量。

另一方面，事件在本质上是一种特殊的委托，使类与类之间以一种松耦合的方式相互通信。

委托与事件的更多信息，查看[Distinguishing Delegates and Events in C#]([Delegates vs. events - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/distinguish-delegates-events))。

委托对象的思想是，如果你需要执行特定的任务，你可以将执行任务的这些算法封装到它们自己的对象中，原始的四人帮将这种设计称为策略模式。

假设你想要一个寻路对象，用于计算迷宫中的路线。该对象本身实际并不包含任何寻路逻辑，相反它只保持对另一个包含寻路算法的对象的引用。

如果你想使用特定的路径搜索技术（如A*，Dijkstra等等）来解决迷宫问题，在单独的“策略”对象中实现正确的解决方案。在运行时，你可以通过交换“策略”对象来切换不同的算法。

## ScriptableObject方法

在Unity中，实现该模式的一种方法就是用以MonoBehaviour引用一个包含必要逻辑的ScriptableObject。当MonoBehaviour执行一个任务，它会调用ScriptableObject上的外部方法而不是自己的。

但是，这样也会有一些限制：

- ScriptableObject上的方法不会被MonoBehaviour的玩家循环自动调用，你需要自己调用它们。
- 像预制体一样，ScriptableObject不能直接引用场景中的物体。如果它们需要在一个场景中的物体上工作，那么你需要将那个物体作为参数传入。

当调用ScriptableObject的方法时，一个MonoBehaviour通常可以将自身作为参数或将其他依赖传入。这给你带来执行逻辑，运行协程等等的灵活性，尽管ScriptableObject存在于项目级别。

![[Pasted image 20240229142103.png]]

例如，你可以定义游戏中拥有不同移动行为的敌人单位，让我们假设它们其中一些需要追击，待机，或从玩家中逃脱的行为。

一个单独的`EnemyUnit`MonoBehaviour可以引用一个包含`MoveUnit`方法的`EnemyAI`ScriptableObject。`EnemyUnit`自身不包含任何移动或行为逻辑，它仅仅在恰当的时机执行ScriptableObject的`MoveUnit`方法。

如果方法需要场景中的数据，`EnemyUnit`对象可以将引用作为参数传递给它，场景中任何必要的依赖都可以被传入。

> 修改ScriptableObject数据

在运行时，你确实可以修改ScriptableObject的数据，但是要注意在何时进行。多个MonoBahaviour共享同一个ScirptableObject，如果修改相同的数据，可能会出现问题。

记住，你可以在运行时创建一个ScriptableObject的实例来避免这个问题，原始的ScriptableObject表现为类似一种包含逻辑和数据的模板。每一个MonoBehaviour都可以创建它自己的能够随意修改的ScriptableObject。

## 可插拔行为

你可以通过将`EnemyAI`ScriptableObject定义为抽象类，使这个模式更加有用。这允许它作为与各种`EnemyUnit`MonoBehaviour兼容的ScriptableObject模板，所以抽象的ScriptableObject可以代表多种算法。

![[Pasted image 20240229143821.png]]

因此，你可以拥有具体的ScriptableObejct类，从`EnemyAI`基类派生出像巡逻，待机或逃脱行为。尽管它们都实现了相同的`MoveUnit`方法，但每一个方法都能产生非常不一样的结果。

在编辑器中，每一个资产都是可互换的，你可以选择地将ScriptableObject拖拽到`EnemyAI`字段，任何兼容的ScriptableObject都可以用这种方式“插入”。

`EnemyUnit`或其他组件可以表现为“大脑“，监视何时切换ScriptableObject，以及在运行时交换行为。这是`EnemyUnit`可以对游戏玩法做出反应的一种方式，只是在每一个状态改变时切换`EnemyAI`ScriptableObject。

在生产中，其他开发者或设计师可以在ScriptableObject中实现实际的移动或AI逻辑。随着更多的移动或行为被添加进游戏（例如躲避，追击等等），而原始的`EnemyUnit`脚本保持不变。这个模式可以帮助你使代码更加具有扩展性，以遵循SOLID编程的开闭原则。

> AI与ScriptableObject

关于使用ScriptableObject驱动行为的更详细的例子，查看[Pluggable AI With Scriptable Objects]([Pluggable AI With Scriptable Objects - Intro and Goals [1/10] Live 2017/3/08 - YouTube](https://www.youtube.com/watch?v=cHUXh5biQMg?utm_source=demand-gen&utm_medium=pdf&utm_campaign=clean-code&utm_content=scriptable-objects-ebook))视频系列，视频中演示了一种基于有限状态机的AI系统，可以使用ScriptableObject来配置状态，操作和这些状态之间的切换。

## 举例：音频代理

包含在ScriptableObject中的行为没必要非常复杂，它可以是像播放自定义音效一样基础的东西。

下面是一个”音频代理“ScriptableObject的例子，可以帮助你增加音频切片的多样性。`AudioDelegateSO`是一个抽象类，定义了一个使用AudioSource作为参数的`Play`方法。

```C#
[Serializable]
public struct RangedFloat
{
	public float minValue;
	public float maxValue;
}

public abstract class AudioDelegateSO: ScriptableObject
{
	public abstract void Play(AudioSource source);
}
```

具体的`SimpleAudioDelegate`ScriptableObject可以随机选择一个切片，并在播放时改变它的音量和音调，这减少了重复播放相同音效的单调。

```C#
[CreateAssetMenu(fileName ="AudioDelegate")]
public class SimpleAudioDelegateSO : AudioDelegateSO
{
	public AudioClip[] clips;
	public RangedFloat volume;
	public RangedFloat pitch;
	
	public void Play(AudioSource source)
	{
		if (clips.Length == 0 || source == null)
			return;
			
		source.clip = clips[Random.Range(0, clips.Length)];
		source.volume = Random.Range(volume.minValue,volume.maxValue);
		source.pitch = Random.Range(pitch.minValue, pitch.maxValue);
		source.Play();
	}
}
```

任何MonoBehaviour都可以使用一个从`AudioDelegateSo`派生的ScriptableObject实例，你还可以为不同的音频效果制作不同的`AudioDelegate`。

在ScriptableObject上拥有方法打开了许多可能性，除了执行操作，还可以给场景中的任意对象发送消息。

# 模式：观察者

在游戏开发中，有多个游戏对象需要相互共享数据或状态的普遍的。在一个小型游戏中，你可以在这些对象之间建立直接的引用，但这是不利于扩展的。管理这些依赖需要大量的精力，它们通常也是漏洞的来源。

当你的应用规模增长时，你需要一个更好的解决方案。

## 避免单例

许多开发选择使用单例，一种场景加载时就存在的某个类的全局实例，但是在引入全局状态的同时也造成单元测试变得困难。

如果你正在使用一个引用单例的预制体，你最终会导入它的所有依赖项，而你只需要去测试一个单独的函数，这降低了模块性和可调试性。

考虑使用一种替代的解决方案来帮助你的对象通信：基于ScriptableObject的事件。

> 关于单例的更多信息

单例这一话题在Unity游戏开发中经常引起争论。对于小型项目或原型，单例可能是一种合适的解决方案。但在大型应用中，单例的缺点通常超过其优点，所以许多开发者将单例视为一种反模式。

单例是易于理解和学习的，但在使用不当时会引发问题。在这里介绍的多数模式能帮你彼岸依赖于单例。

如果你想轻易访问共享的数据，考虑一个基于ScriptableObject的`Runtime Set`。如果你需要一种在对象之间发送消息的方式，尝试使用基于ScriptableObject的事件频道。从单例中摆脱来重构你的架构，可以提高可伸缩性和可测试性。

## 基于ScriptableObject的事件

正如你已经看到的，ScriptableObject不仅仅用于处理数据，它们还可以像其他脚本一样包含方法，这些方法可以作为对象之间通信的手段。

在观察者模式中，一个主题（subject）向一个或多个松耦合的观察者（observer）广播消息，每一个观察对象都可以独立于主题做出反应，同时不被其他观察者所察觉。主题也可以被称为发布者（pulisher）或广播者（broadcaster）。观察者也被称为订阅者（subscriber）或监听者（listener）。

基于事件的体系结构只在需要时执行，而不是每帧运行。出于这个原因，它通常比在MonoBehaviour的`Update`方法中添加逻辑更优。

![[Pasted image 20240229154633.png]]

你可以用MonoBehaviour或C#对象实现观察者模式。虽然这在Unity开发中已经是一种常见的做法了，但仅使用脚本的方法意味着设计师需要依赖程序团队来完成游戏玩法中需要的每一个事件。

一种替代方法是创建基于ScriptaleObject的事件，ScriptableObject在主题和观察者之间作为一个中间者，同时在编辑器中提供一个图形化的界面。

![[Pasted image 20240229155144.png]]

这么一看，你只是对观察者模式添加了一层额外的开销，但这种架构提供了一些优势。因为ScriptableObject是资产，它们能够访问场景中的所有对象，并且不会在场景加载时消失。

这就是为什么许多开发者在一开始就使用单例：简单，对特定资源的持久访问。ScriptableObject通常可以提供同样的好处，而无需引入许多不必要的依赖。

在基于ScriptableObject的事件中，任何对象都可以作为发布者（广播事件），任何对象都可以作为订阅者（监听事件）。ScriptableObject位于这两者之间，帮助传递信号，充当两者之间的集中中介。

可以把它看作一个”事件频道“，将ScriptableObject想象成一个有许多对象监听它信号的信号塔，对此感兴趣的Monobehaviour可以简单地订阅这个事件频道，然后在某些事情发生时做出回应。

## 举例：事件频道

任何包含以下内容的ScirptableObject都可以作为一个事件频道：

- 一个委托（UnityAction或System.Action）：它通知订阅者并将恰当的数据作为参数传递。`event`关键字可以限制该委托只能被该ScriptableObject（或派生类）所执行。
- 一个事件触发方法：该公共方法用于执行委托。

你可以设定任意数量的事件频道来决定游戏玩法的各个方面。因为ScriptableObject存在于项目级别，所以它可以触发可全局访问的事件，这样可以将场景中不相关的对象连接起来。

> System.Action和UnityAction

System.Action是一个通用的委托类型，其定义在.NET Framework的System命名空间中。它可以被用于Unity项目中，而无需声明自定义的委托。添加`event`关键字可以使委托类型只读；其他对象可以监听委托已注册的方法，但它们不能直接执行那些方法。

UnityAction是Unity引擎专门定义的一个委托类型，你通常将它和UnityEvent类一起使用，这是在Unity中创建事件的另一种方法。UnityEvent和UnityAction会显示在Inspector中，所以它们通常作为一种实现观察者模式，更用户友好的方式。

总的来说，使用System.Action还是UnityAction取决于你的特定需求，你可以在同一项目中有选择地使用其中一个或都使用。

如果你想要一个不与Unity绑定在一起的更通用的委托，就使用System.Action。如果你想要一个专门为UnityEvent设计的委托，就使用UnityAction。

你可以创建一个`VoidEventChannelSO`，来引发一个无需任何参数的事件。

```C#
[CreateAssetMenu(menuName = "Events/Void Event Channel")]
public class VoidEventChannelSO : ScriptableObject
{
	public event UnityAction OnEventRaised;
	
	public void RaiseEvent()
	{
		if (OnEventRaised != null)
			OnEventRaised.Invoke();
	}
}
```

一旦你创建一个`VoidEventChannelSO`类型的ScriptableObject，任何MonoBehaviour都可以监听`OnEventRaised`。例如，创建一个名为StartNewGame，`VoidEventChannelSO`类型的ScriptableObject，另一个对象可以触发`RaiseEvent`方法来触发该事件。

![[Pasted image 20240229163538.png]]

另一个MonoBehaviour可以在Inspector中引用这个事件频道，然后订阅和取消订阅`OnEventRaised`。

```C#
public class StartGame : MonoBehaviour
{
	[SerializeField] private VoidEventChannelSO _onNewGameButton = default;
	
	private void Start()
	{
		_onNewGameButton.OnEventRaised += StartNewGame;
	}
	
	private void OnDestroy()
	{
		_onNewGameButton.OnEventRaised -= StartNewGame;
	}
	
	private void StartNewGame()
	{
		// load level logic here…
	}
}
```

关于对美术或设计师更友好的监听组件，你可以创建一个不需要任何脚本设置的MonoBehaviour。

```C#
public class VoidEventListener : MonoBehaviour
{
	[SerializeField] private VoidEventChannelSO _channel = default;
	
	public UnityEvent OnEventRaised;
	
	private void OnEnable()
	{
		if (_channel != null)
			_channel.OnEventRaised += Respond;
	}
	
	private void OnDisable()
	{
		if (_channel != null)
			_channel.OnEventRaised -= Respond;
	}
	
	private void Respond()
	{
		if (OnEventRaised != null)
			OnEventRaised.Invoke();
	}
}
```

简单地为GameObject添加一个`VoidEventListenr`，然后将事件频道ScriptableObject拖拽到Inspector窗口中。在`OnRaisedEvent`上创建UnityAction，对事件做出反应。

![[Pasted image 20240229164316.png]]

不管你选择哪一个组件去监听事件，事件频道会提供一种运行时各个对象之间通信的方式。事件可以通知场景中的任何需要那些信息的游戏对象。

因为它们是项目级别的资产，基于ScriptableObject的事件可以驱动应用程序的大部分基础结构，这对于在不同系统之间发送消息特别有用。

一些常见的管理系统包括：

- 音频管理：游戏中许多事情可以触发音效。该系统可以播放`AudioClip`，或调整`AduioMixer`对应用程序的事件作出响应。
- 场景管理：该系统处理Unity场景和游戏关卡的加载与卸载。
- UI管理：该系统负责游戏开始前，游戏过程中，和游戏结束后的菜单界面。
- 数据存储管理：该系统处理游戏数据和文件系统设置的保存和加载。

这些系统专注于不同的任务，但它们需要彼此交流，事件可以形成使让它们保持连接的粘合剂。

注意，你可以使用不同的事件负载来为每一个事件发送不同类型的数据。例如，基于ScriptableObject的事件包括`IntEventChannelSO`，`Vector2EventChannelSO`，`VoidEventChannelSO`等等，事件的使用取决于上下文。

应根据游戏玩法而自定义额外的事件类型。例如，一个伤害事件可能需要传递伤害的承受者以及伤害数值。

如何应用这些事件频道只会被你的创造性所限制。除了以上提到的核心系统，事件还可以帮助连接不同的游戏系统，使它们能够互动：

- 摄像机：用于控制玩家从游戏世界中看到的视野以及增加戏剧或电影效果，比如抖动或切换到不同的视角。
- 任务：为了推进游戏或获得奖励，玩家必须完成一些任务或目标。任务通常设计各种游戏玩法元素，比如拾取物品，打败敌人或解决谜题。
- 血量：这是在许多游戏中将玩家，敌人，以及其他任何能够对玩家造成伤害的对象连接在一起的重要元素。
- 成就：就像任务，有一些特殊的奖励是玩家可以通过完成游戏中特定的任务或目标所解锁的。成就可以跨越不同的游戏元素，比如达到一个特定的等级或积累一定量的点数。

这些游戏元素将通过事件的使用，和其他诸如音频、UI、数据存储的管理系统进行交互。这种方法促进了结构中各组件的模块性和独立性，同时仍然允许和其他系统进行通信。

> 调试事件频道

自定义编辑器或属性抽屉可以在Inspector中制造一个"Raise Event"按钮，这能帮助你手动地触发事件以便调试。

下面是一个基础的编辑器脚本，为`VoidEventChannel`创建一个自定义的Inspector按钮。

```C#
[CustomEditor(typeof(VoidEventChannelSO))]
public class VoidEventChannelSOEditor : Editor
{
	public override void OnInspectorGUI()
	{
		DrawDefaultInspector();
		
		VoidEventChannelSO eventChannel = (VoidEventChannelSO)target;
		
		if (GUILayout.Button("Raise Event"))
		{
			eventChannel.RaiseEvent();
		}
	}
}
```

这样可以创建一个按钮，让你可以随意触发事件，使运行时诊断问题更加容易。

![[Pasted image 20240229172827.png]]

更进一步，你还可以为携带数据的事件频道创建这样的按钮。

如果你继续使用事件频道为对象解耦，考虑开发调试工具，比如为每一个事件保持对所有监听者的记录。事件频道类可以包含添加和一处订阅对象的方法，从而在运行时更容易识别那些事件导致了特定的行为。

## 举例：输入读取

监听用户输入的对象需要一个专门的事件频道类型，Unity的输入系统使用InputAction将原始输入数据表示为逻辑概念（例如跳跃，走路等等）。

每一个InputAction，包含它自己的`started`，`performed`和`canceled`事件。

![[Pasted image 20240229173936.png]]

为了破译InputAction的绑定，你可以创建一个特别的`InputReader`ScriptableObject，同样，它作为主题和观察者之间的中间者。然而在这种情况下，MonoBehaviour不会显式地触发事件。

取而代之，输入系统取代了主题或广播者的位置。

![[Pasted image 20240229174323.png]]

这里，我们在输入系统中设置Action和ActionMap。每一个InputAction描述一个单独的输入轴，并绑定到键盘，手柄或其他输入设备。

不直接地订阅InputAction本身，PaddleController监听`OnMoveP1.performed`事件和`OnMoveP2.performd`事件。

![[Pasted image 20240229174655.png]]

由此产生的InputReader标准化了你的游戏对象处理手柄或键盘输入的操作。任何需要输入的游戏对象需要：

- 维持对InputReader ScriptableObject的引用
- 订阅相关的事件并连接其事件处理方法

虽然这种模式用在极简的街机游戏上有点过度了，但我们是为了在一个简单的项目上使这种结构更容易理解。

直到你的项目规模增长，增加了更多组件时，这种模式的好处才会明显。将输入和使用它们的游戏对象解耦，带来了额外的灵活性和可重用性。

如果你不得不在开发过程中修改InputAction，你只需要维护InputReader即可。如果事件不需要改变，那监听事件的对象就不会受影响。因此，维持输入到观察者之间的连接变得更简单了，尤其是当你有大量观察者的时候。

> 静态事件对比非静态事件

你可以选择使用静态事件来减轻在监听对象上定位ScriptableObject的负担。

例如，一个MonoBahaviour可以在它的`OnEnable`方法中订阅InputReader的静态的`MoveP1Event`和`MoveP2Event`事件：

```C#
InputReader.MoveP1Event += OnMoveP1;
InputReader.MoveP2Event += OnMoveP2;
```

当使用静态事件时，管理订阅需要格外谨慎。别忘了在`OnDisable`中取消订阅：

```C#
InputReader.MoveP1Event -= OnMoveP1;
InputReader.MoveP2Event -= OnMoveP2;
```

静态事件总是可访问的，而且如果它们有活动的订阅者就不会作为垃圾被回收。

但是，静态事件是不能序列化的，如果希望在编辑器中进行交互，则使用非静态的事件，并确保引用了正确的ScriptableObject。

