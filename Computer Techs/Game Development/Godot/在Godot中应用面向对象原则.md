## 在Godot中应用面向对象原则

两种创建可复用对象的方式：**脚本和场景**，它们并没有真的在底层定义类。

### 脚本在引擎中如何工作

引擎提供的内置类如Node，它们实际上并不是类，而是告诉引擎基于某个内置类执行一系列初始化操作的资源。

Godot的内部类有一些方法用来将类的数据注册到ClassDB中。

ClassDB是一个提供运行时访问类的信息的数据库，保存了类的属性、方法、常量、信号等信息。

当对象要执行一些操作时，比如访问属性、调用方法等，就会检查ClassDB和基类的记录，判断是否支持这些操作。

将脚本附加到对象上就能扩展ClassDB中可用的方法、属性和信号。

> GDscript
> 即使脚本没有使用extends关键字，它也会隐式地继承RefCounted类，因此可以在代码中实例化它，但不能附加到节点上。

### 场景

场景的行为和类有很多相似性，可以把场景当作类来考虑。

创建一个场景，类似一个脚本创建节点并将它们设置为自己的子节点。

场景可以帮助定义这些内容：

- 脚本可以接触的节点
- 节点是如何组织的
- 节点是如何初始化的
- 节点之间的信号连接

场景的实例就是对象，许多面向对象原则也适用于场景，例如单一职责、封装等等。

场景总是其根节点上所附加脚本的一个扩展，所以你可以将它理解为一个类的一部分。

## 场景组织

主题：如何有效地组织场景？应该使用哪些节点？如何放置它们？

### 如何有效地创建关系

随着场景内容的增加，需要把它拆分为多个独立的场景。然而这样做之后，由于节点路径的改变，直接的节点依赖不再有效，信号连接也会中断。

为了修复这个问题，必须以某种方式实例化子场景，这些子场景不依赖于它们所处的具体环境。

在OOP中，应该保持类的单一职责和松散耦合，这样可以使类的大小较小，便于维护，并提高可重用性。

尽可能将场景设计成无依赖的，也就是场景所需要的东西都在其内部。

如果场景必须和外部上下文交互，可以使用依赖注入，由高级API向低级API提供所需依赖。

为了做到这一点，必须暴露数据然后依赖上级去初始化：

1. 连接信号，是完全安全的，但是只能响应行为，而不能启动行为。
```C#
// Parent
GetNode("Child").Connect("SignalName", Callable.From(ObjectWithMethod.MethodOnTheObject));

// Child
EmitSignal("SignalName"); // Triggers parent-defined behavior.
```

2. 调用方法，用来启动行为。
```C#
// Parent
GetNode("Child").Set("MethodName", "Do");

// Child
Call(MethodName); // Call parent-defined method (which child must own).
```

3. 初始化一个Callable类型的属性，比直接调用方法更安全，因为子节点不需要拥有方法的所有权。
```C#
// Parent
GetNode("Child").Set("FuncProperty", Callable.From(ObjectWithMethod.MethodOnTheObject));

// Child
FuncProperty.Call(); // Call parent-defined method (can come from anywhere).
```

4. 初始化节点或其他对象引用。
```C#
// Parent
GetNode("Child").Set("Target", this);

// Child
GD.Print(Target); // Use parent-defined node.
```

6. 初始化节点路径。
```C#
// Parent
GetNode("Child").Set("TargetPath", NodePath(".."));

// Child
GetNode(TargetPath); // Use parent-defined NodePath.
```

这些方法隐藏了子节点对外部访问的可见性，使它们与环境之间松耦合。

这些方法同样适用于所有对象关系，兄弟节点应该只了解自己的层级，由祖先节点来协调它们之间的通信和引用。

```C#
// Parent
GetNode<Left>("Left").Target = GetNode("Right/Receiver");

public partial class Left : Node
{
    public Node Target = null;
    
    public void Execute()
    {
        // Do something with 'Target'.
    }
}

public partial class Right : Node
{
    public Node Receiver = null;
    
    public Right()
    {
        Receiver = ResourceLoader.Load<Script>("Receiver.cs").New();
        AddChild(Receiver);
    }
}
```

应倾向于将数据保留在内部（场景内部），即使将依赖放在外部是松耦合的，也可能会因为外部环境变化而导致错误。否则将迫使开发者使用文档来在微观层面上追踪对象关系，这就是“开发地狱”。

例如，角色类有一个装备武器的方法`EquipWeapon(weapon)`，如果外部传入一个错误的类型或没有传递任何东西，游戏就会崩溃。

可以将方法改成`EquipWeapon(type)`，type是武器类型，在角色类内部根据不同类型获取对应的武器对象，这样无论何时使用角色场景，都能保证行为是一致的，不会因为外部环境变化而出错。

为了避免创建和维护这种文档，可以将独立的节点转换成一个工具脚本，实现`GetConfigurationWarnings`方法。然后编辑器通过脚本进行自我记录，而不需要通过文档复制内容。

总之，场景应该尽可能独立运作，避免硬依赖和紧耦合，目的是确保类的更改不会对其他依赖类产生负面影响，增强稳定性和可维护性。

脚本和场景作为引擎类的扩展，应该遵循所有OOP原则，例如：

- SOLID
- DRY
- KISS
- YAGNI

### 选择节点树结构

组织节点树的方式有无数种，下面有一个例子。

一个游戏应该始终有一个入口，使用一个`Main`节点来表示它。

然后是一个表示游戏世界的节点和一个管理各种菜单、空间的GUI节点，把它们作为Main的子节点。

- Node "Main" (main.gd)
    - Node2D/Node3D "World" (game_world.gd)
    - Control "GUI" (gui.gd)

当切换关卡时，你可以替换`World`节点为新的场景。

下一步是考虑你的项目需要哪些Gameplay系统，如果有这样一个系统：

- 在内部追踪它所有的数据
- 需要全局访问
- 需要单独存在

那就需要创建一个自动加载的单例节点。

组织场景的关键是思考节点在场景树中的关系性，而不是空间位置。

如果节点不依赖于父节点，那么就可以独立存在于场景树中的其他地方，否则应该存在于父节点下。

在Godot中，节点树是一种聚合关系，而不是组合关系。聚合意味着节点可以独立存在，而组合关系意味着组件是实体不可分隔的一部分。

虽然Godot提供移动节点的灵活性，但是应尽量减少移动节点的情况。



