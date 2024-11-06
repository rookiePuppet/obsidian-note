
# Introducing design pattern

对于你遇到的每一个软件设计问题，很多开发者都已经遇到过了。虽然不能直接去问它们关于解决这些问题的建议，但你可以透过设计模式学习到它们的思想。

**设计模式就是在软件工程中一些常见问题的通用解决方案**，它们并不是你可以直接复制粘贴进你的工程的最终代码，而是应该将它们视为你工具箱中额外的工具。

最核心的是，设计模式仅仅是**思想**，不能适用于任何情况，但如果能够正确使用，它们可以帮助你构建具有良好扩展性的大型应用，以及提高代码可读性、简洁性等等。

## Use the KISS principle

KISS原则就是：**Keep it simple, stupid**。换句话说就是，**减少或避免不必要的复杂性**。

每一种设计模式都要权衡使用，在应用设计模式之前，应该想清楚它带来的好处是否值得去做那些额外的工作。当你不确定某一种设计模式是否能解决你的问题，最好是等待一个自然而然的时机再去使用，而不是觉得它很新颖或者高级，总之就是要在你真正需要的时候才去使用设计模式。

# The SOLID Principles

SOLID是软件设计五个核心基础的首字母缩写词：
- **Single responsibility 单一职责**
- **Open-closed 开闭**
- **Liskov substitution 里氏替换**
- **Interface segregation 接口隔离**
- **Dependency inversion 依赖倒转**

## Single-responsibility principle

**一个类只有一个变化的原因，也就是说它应该只有一个主要职责。换句话说，一个类应该封装单一的功能或行为，如果一个类有多个职责，那它会变得紧密耦合且难以维护。**

避免构建庞大整体的类，而是用很多小的类来组装成你的工程。因为简短的类和方法更容易解释、理解和实现。

例如在Unity中，当你创建一个GameObject，你会发现它附带了多种小的组件（Component），比如Transform、Rigidbody等等。

每一个组件只做并且做好一件事，当你在场景中创建出各种GameObject后，这些GameObject上面挂载的组件之间的交互让游戏得以呈现。

如果你忽略了单一职责原则，你也许会创建出一个像是这样的组件：

![[Pasted image 20240204114712.png]]

```CSharp
public class UnrefactoredPlayer : MonoBehaviour
{
	[SerializeField] private string inputAxisName;
	[SerializeField] private float positionMultiplier;
	private float yPosition;
	private AudioSource bounceSfx;
	
	private void Start()
	{
		bounceSfx = GetComponent<AudioSource>();
	}
	
	private void Update()
	{
		float delta = Input.GetAxis(inputAxisName) * Time.deltaTime;
		yPosition = Mathf.Clamp(yPosition + delta, -1, 1);
		transform.position = new Vector3(transform.position.x, yPosition
		* positionMultiplier, transform.position.z);
	}
	
	private void OnTriggerEnter(Collider other)
	{
		bounceSfx.Play();
	}
}
```

UnrefactoredPlayer类有着一堆杂乱的职责，播放音效、管理输入以及处理移动。即使现在这个类还是相对简短的，但随着项目的迭代，它会变得难以维护。现在，尝试将Player类拆分成一些更小的类。

![[Pasted image 20240204115929.png]]

```CSharp
[RequireComponent(typeof(PlayerAudio), typeof(PlayerInput),
typeof(PlayerMovement))]
public class Player : MonoBehaviour
{
	[SerializeField] private PlayerAudio playerAudio;
	[SerializeField] private PlayerInput playerInput;
	[SerializeField] private PlayerMovement playerMovement;
	
	private void Start()
	{
		playerAudio = GetComponent<PlayerAudio>();
		playerInput = GetComponent<PlayerInput>();
		playerMovement = GetComponent<PlayerMovement>();
	}
}

public class PlayerAudio : MonoBehaviour
{
	…
}

public class PlayerInput : MonoBehaviour
{
	…
}

public class PlayerMovement : MonoBehaviour
{
	…
}
```

一个玩家脚本依然管理着其他的组件，但每一个组件类都只做一件事。利用这种设计理念，可以使代码逻辑更容易修改和维护，尤其是当项目需求随着时间推移在不断变化时。

另一方面，你需要有良好的常识来权衡该原则，避免过度简化以至于创建出许多只有一个方法的类。

在利用单一职责原则时，在心中保持这些目标：

- **可读性**：简短的类更加容易阅读和理解。简短的定义因人而异，可以设定一个阈值，当代码行数超过多少时就考虑重构它们，分解成一些小的部分。
- **扩展性**：小的类会更加容易被继承。修改或替换它们，而不必担心意外地破坏功能。
- **复用性**：设计小的和模块化的类可以方便你在游戏的各个模块中重复使用它们。

### 简单并不等同于容易

简单性是软件设计中经常讨论的话题，它是可靠性的先决条件。

许多设计模式和原则会迫使你实现简单性，这样做的目的是使你的代码更具扩展性、灵活性和可读性，当然与此同时也需要做一些额外的工作和规划，所以说简单并不意味着容易。

尽管不依靠设计模式也可以完成相同的功能，但快速且简单的并不一定能创造出简单的结果（原文：something fast and easy doesn't necessarily result in something simple）。让某件事变得简单意味着让它变得专注，专注于做一件事情，不要和其他任务糅合在一起以变得过于复杂化。

## Open-closed principle

开闭原则说的就是：**对扩展开放，对修改封闭**。将各个类之间变得更有组织性，那样当你需要创建新的行为时，就不必修改原始的代码了。

在解释该原则时，一个经典的例子就是计算图形面积的程序。现在有一个叫做AreaCalculator的类，里面有返回正方形和圆形面积的方法，另外还有正方形和圆形的类。

```CSharp
public class AreaCalculator
{
	public float GetRectangleArea(Rectangle rectangle)
	{
		return rectangle.width * rectangle.height;
	}
	
	public float GetCircleArea(Circle circle)
	{
		return circle.radius * circle.radius * Mathf.PI;
	}
}

public class Rectangle
{
	public float width;
	public float height;
}

public class Circle
{
	public float radius;
}
```

![[Pasted image 20240204120246.png]]

你可以创建一个基类叫做Shape，基类中用一个方法来处理各个图形的面积计算。但是这样做会需要大量的if语句来处理每一种图形，扩展性也不好。

为了使程序对扩展开放，不用修改原始的代码（AreaCalculator的内部）。尝试定义一个抽象的Shape类：

```CSharp
public abstract class Shape
{
	public abstract float CalculateArea();
}
```

然后让Rectangle和Circle类都继承自Shape，并实现各自的面积计算方法：

```CSharp
public class Rectangle : Shape
{
	public float width;
	public float height;
	
	public override float CalculateArea()
	{
		return width * height;
	}
}

public class Circle : Shape
{
	public float radius;
	
	public override float CalculateArea()
	{
		return radius * radius * Mathf.PI;
	}
}
```

AreaCalculator可以简化成下面这样：

```CSharp
public class AreaCalculator
{
	public float GetArea(Shape shape)
	{
		return shape.CalculateArea();
	}
}
```

现在，只要是继承了Shape的图形并正确实现了其面积计算的方法，修改过后的AreaCalculator都可以获得它们的面积。

![[Pasted image 20240204120323.png]]

这种设计让调试变得更简单了，如果新的图形出现了问题，你不需要再去查看AreaCalculator，因为旧的代码并没有被修改，只需要在新的代码中检查逻辑是否错误。

利用好接口和抽象，将帮助你避免后续不便于扩展的笨拙switch或者if语句。一旦你习惯遵循开闭原则来建立你的类，从长远来看，增加新代码将会变得更简单。

## Liskov substitution principle

里氏替换原则指出：**派生类必须能够替代它们的基类**，这个原则告诉我们如何应用继承来提高子类的鲁棒性和灵活性。

在面向对象编程中，继承允许你通过子类去增加新的功能，但如果不当使用就会增加不必要的复杂性。

想象一下你的游戏需要一个叫做Vehicle的类，同时它作为各种车辆的基类，例如你可以需要一个小汽车类Car或一个货车类Truck。在每个能够使用Vehicle基类的地方，你应该都可以使用Car或者Truck等等Vehicle的子类。

![[Pasted image 20240204120355.png]]

```CSharp
public class Vehicle
{
	public float speed = 100;
	public Vector3 direction;
	
	public void GoForward()
	{
		...
	}
	
	public void Reverse()
	{
		...
	}
	
	public void TurnRight()
	{
		...
	}
	
	public void TurnLeft()
	{
		...
	}
}
```

另外还需要一个叫做Navigator的类，用来控制车辆沿着预定的路线行驶。

```CSharp
public class Navigator
{
	public void Move(Vehicle vehicle)
	{
		vehicle.GoForward();
		vehicle.TurnLeft();
		vehicle.GoForward();
		vehicle.TurnRight();
		vehicle.GoForward();
	}
}
```

利用Navigator类，你可以传递任何车辆对象进Move方法中，这对于小汽车类和货车类来说都工作良好。

但如果是火车类？TurnLeft和TurnRight方法对于火车类来说都不起作用，因为火车不能脱离它的轨道，所以火车类并没有实现这两个方法。如果把火车对象传进Navigator的Move方法，就是抛出未实现方法的异常。当你不能用子类来替代父类的时候，你就违背了里氏替换原则。

![[Pasted image 20240204120431.png]]

下面是关于如何更紧密地遵循里氏替换原则的一些提醒：
- **如果在子类化时删除某些特性（功能），你很有可能违背了里氏替换原则：** 如果子类行为表现得和基类不一样，那么说明你没有遵循里氏替换原则。（这里说的表现不一样，确切地来讲是针对于子类从基类继承而来的行为，子类可以基于父类做出扩展，但不能改变父类原有的行为）。
- **保持抽象的简单**：往基类里放越多的逻辑，越有可能破坏里氏替换原则。基类中应该仅仅表达一些通用的、公共的逻辑。
- **子类需要拥有和基类一样的公共成员**：这些成员在调用时还需要具有相同的签名和行为。
- **在建立类层次结构之前，考虑类的AP**I：虽然汽车和火车都是交通工具，但它们的运作方式、属性和行为可能截然不同，所以将它们继承自不同的父类可能会更合理。这告诉我们在设计类的继承关系时要考虑更多因素，而不仅仅是分类。
- **优先使用组合而非继承：** 不要试图通过继承来传递功能，而是创建接口或单独的类来封装特定的行为。然后通过混合和匹配不同的功能来构建一个“组合”。

![[Pasted image 20240204120448.png]]

现在，抛弃原来的Vehicle类型，然后把大部分的功能都转移到接口中：

```CSharp
public interface ITurnable
{
	public void TurnRight();
	public void TurnLeft();
}

public interface IMovable
{
	public void GoForward();
	public void Reverse();
}
```

为了更好地遵循里氏替换原则，再创建一个RoadVehicle类型和RailVehicle类型，让Car和Train类型分别继承这两个基类。

![[Pasted image 20240204120524.png]]

```CSharp
public class RoadVehicle : IMovable, ITurnable
{
	public float speed = 100f;
	public float turnSpeed = 5f;
	
	public virtual void GoForward()
	{
		...
	}
	
	public virtual void Reverse()
	{
		...
	}
	
	public virtual void TurnLeft()
	{
		...
	}
	
	public virtual void TurnRight()
	{
		...
	}
}

public class RailVehicle : IMovable
{
	public float speed = 100;
	
	public virtual void GoForward()
	{
		...
	}
	
	public virtual void Reverse()
	{
		...
	}
}

public class Car : RoadVehicle
{
	...
}

public class Train : RailVehicle
{
	...
}
```

使用这种方式，功能通过接口而不是继承来实现，Car和Train不再共享同一个基类，也就满足了里氏替换原则。

这种方式可能是违反直觉的，因为你对于现实世界有一定的假设，这些假设可能不是完全正确的。在软件设计中，这被称为*圆-椭圆*问题。不是每一种确切的“is a”关系都能转化为继承。记住，你想要的软件设计来驱动你的类层次，而不是你对现实的先验知识。

里氏替换原则的意义就在于此，限制继承的使用来保持代码的可扩展性和灵活性。

## Interface segregation principle

接口隔离原则规定，**客户端不应该被强制依赖于它不使用的方法**。换句话说就是，避免使用庞大臃肿的接口。这和单一职责原则的思想类似，旨在告诉你要保持类和方法尽可能简短，这样可以给你最大的灵活性，让接口保持紧凑和专注。

想象你在制作一个有不同玩家单位（可能指的是玩家可以操控的各种角色）的策略游戏，每个单位有着像生命、速度等不同的属性。你可能会想创建一个接口来保证各个单位实现类似的功能：

```CSharp
public interface IUnitStats
{
	public float Health { get; set; }
	public int Defense { get; set; }
	
	public void Die();
	public void TakeDamage();
	public void RestoreHealth();
	
	public float MoveSpeed { get; set; }
	public float Acceleration { get; set; }
	
	public void GoForward();
	public void Reverse();
	public void TurnLeft();
	public void TurnRight();
	
	public int Strength { get; set; }
	public int Dexterity { get; set; }
	public int Endurance { get; set; }
}
```

游戏中可以还有能够损坏的物品，例如可破坏的木桶或板条箱。尽管这些物品不能自己移动，但也需要有生命值的概念，并且不会像其他玩家单位那样拥有各种技能。

因此，与其让木桶实现那个大接口，获得一堆没用的属性和方法，不如将大的接口拆分成几个小的接口，只让木桶实现真正需要、有用的接口。

![[Pasted image 20240204113410.png]]

```CSharp
public interface IMovable
{
	public float MoveSpeed { get; set; }
	public float Acceleration { get; set; }
	
	public void GoForward();
	public void Reverse();
	public void TurnLeft();
	public void TurnRight();
}

public interface IDamageable
{
	public float Health { get; set; }
	public int Defense { get; set; }
	public void Die();
	public void TakeDamage();
	public void RestoreHealth();
}

public interface IUnitStats
{
	public int Strength { get; set; }
	public int Dexterity { get; set; }
	public int Endurance { get; set; }
}
```

对于可以引爆的木桶，你可以再增加一个IExplodable接口。

```CSharp
public interface IExplodable
{
	public float Mass { get; set; }
	public float ExplosiveForce { get; set; }
	public float FuseDelay { get; set; }
	public void Explode();
}
```

由于一个类可以继承多个接口，你可以利用IDamageable、IMoveable和IUnitStats接口创作出一种敌人单位。而能够爆炸的木桶只需要实现IDamageable和IExplodable接口，而不需要其它接口不必要的开销。

同样，这和里氏替换的例子类似，也是倾向使用组合而不是继承。接口隔离原则能够帮助我们使系统解耦，让它们更容易修改以及重构。

## Dependency inversion principle

依赖倒转原则说的是高级模块不应该直接从低级模块导入任何内容，它们都应该依赖于抽象。

当一个类与另一个类有关联时，它就具有依赖性或耦合性。在软件设计中，每一种依赖都伴随着一些风险。

如果一个类对另一个类的工作方式了解的太多，修改第一个类可能会损坏第二个类，反之亦然。高度耦合被认为是编写不干净的代码导致的结果，程序中的一个小错误可能会滚雪球似的引起许多问题。

理想情况下，应当尽可能减少类之间的依赖。每个类还需要其内部的各个部分协同工作，而不是依赖于外部的连接。当对象在内部或私有逻辑上运行时，它就被认为是内聚的。

在最好的情况下，应该做到高内聚、低耦合。

![[Pasted image 20240204142956.png]]

你的游戏程序应当可修改和可扩展，如果它禁不起修改，你需要弄清楚它现在在结构上有什么问题。

依赖倒置原则能够帮助减少类之间的紧耦合现象。在应用中构建类和系统时，有一些是“高级的”，有一些是“低级的”，高级类依赖于低级类来完成某些工作。SOLID告诉我们要转换一下思维。

假设你在制作一个游戏，玩家在关卡中探索时，可以触发门的开启。你或许会创建一个Switch类和一个Door类。

![[Pasted image 20240204144100.png]]

在高层，Switch负责当角色移动到一个特定位置时触发一些事情。在低层，Door负责门的实际开关效果。

```CSharp
public class Switch : MonoBehaviour
{
	public Door door;
	public bool isActivated;
	
	public void Toggle()
	{
		if (isActivated)
		{
			isActivated = false;
			door.Close();
		}
		else
		{
			isActivated = true;
			door.Open();
		}
	}
}

public class Door : MonoBehaviour
{
	public void Open()
	{
		Debug.Log("The door is open.");
	}
	
	public void Close()
	{
		Debug.Log("The door is closed.");
	}
}
```

Switch可以调用Toggle方法来控制门的开关，这确实能够正常工作，但问题是这里的依赖关系是从Door直接连接到Switch的，如果Switch的逻辑不仅仅用于门的开关，还有灯光或是巨型机器人呢？

你可以在Switch类中增加额外的方法，但你会违反开闭原则。

这时，抽象再一次拯救了我们。你可以在门和开关之间插入一个接口，叫做ISwitchable。

![[Pasted image 20240204145509.png]]

ISwitchable仅仅需要一个公共属性来记录开关状态，以及一对开关的方法。

```CSharp
public interface ISwitchable
{
	public bool IsActive { get; }
	public void Activate();
	public void Deactivate();
}
```

然后Switch就是变成这样，依赖于一个ISwitchable的接口对象client，而不是直接依赖Door类对象。

```CSharp
public class Switch : MonoBehaviour
{
	public ISwitchable client;
	
	public void Toggle()
	{
		if (client.IsActive)
		{
			client.Deactivate();
		}
		else
		{
			client.Activate();
		}
	}
}
```

另一方面，你需要让Door实现ISwitchable接口，使其重新工作。

```CSharp
public class Door : MonoBehaviour, ISwitchable
{
	private bool isActive;
	public bool IsActive => isActive;
	
	public void Activate()
	{
		isActive = true;
		Debug.Log("The door is open.");
	}
	
	public void Deactivate()
	{
		isActive = false;
		Debug.Log("The door is closed.");
	}
}
```

现在你就倒转了依赖，接口在它们之间创建了一个抽象，而不是使门和开关生硬地连接在一起。Switch不在直接依赖于Door特定的Open/Close方法，而是使用ISwitchable的Activate/Deactivate方法。

这个小而关键的改变提高复用性，之前的Switch只能和Door工作，而现在它能和任何实现了ISwitchable接口的东西一起工作。

这使你能够创造更多基于开关驱动的类，无论是陷阱门还是激光束。

![[Pasted image 20240204150903.png]]

像SOLID中的其余几个原则一样，依赖倒置原则要求你检查你通常是如何建立类之间的关系的，通过松耦合来方便地扩展项目。

### 对比接口和抽象类

在说明优先使用组合而非继承的理念时，许多例子都使用了接口。然而，你也可以使用抽象类来遵循这些设计模式和原则。在C#中，接口和抽象类都是实现抽象的有效方式，在选择上可以视实际情况而定。

#### 抽象类

通过abstract关键字来定义一个基类，然后你可以通过继承的方法来将公共的功能传递给子类。

**抽象类不能直接实例化，只能被另一个具体的类所继承。**

在前面的例子中，使用抽象类同样可以实现依赖倒置，只是用一种不同的方式：定义一个叫做Switchable的抽象类，然后让Door类、Light类去继承它。

![[Pasted image 20240204153043.png]]

继承定义的是一种“is a”关系，抽象类的优势在于它们**拥有字段、常量以及静态成员**，还可以使用**更多约束性的访问修饰符**，例如protected和private。和接口不同的是，抽象类允许你**实现逻辑**，然后在具体的类中就可以**共享**一些核心的功能。除非你想让一个派生类具有两种不同基类的特性，这时候使用抽象类就行不通了，因为在C#中一个类是不允许继承两个及以上的类的。

如果你有一个Robot的基类，让NPC继承Robot还是Switchable就很难决定了。

![[Pasted image 20240204154135.png]]

### 接口

正如接口隔离原则所示，当某些东西不完全符合继承范式时，接口提供了更大的灵活性，你可以更容易地选择出一种“has a”关系。

接口中只能包含成员的声明，实际实现接口的类将会充实具体的逻辑。

因此，在接口和抽象类之间做选择并不是非此即彼的。如果希望定义一些可以让子类共享的基本功能，就使用抽象类，如果想定义一些辅助的能力供实现类灵活选择，就使用接口。

在上面的例子中，你可以从Robot基类派生出NPC类来继承核心功能，然后再实现ISwitchable接口来为NPC增加开关的功能。

![[Pasted image 20240204160024.png]]

#### 具体对比

| 抽象类         | 接口                    |
| ----------- | --------------------- |
| 完全或部分地实现方法  | 只能声明方法，不能实现           |
| 声明/使用变量和字段  | 只能声明方法和属性             |
| 可以有静态成员     | 不能声明和使用静态成员           |
| 有构造函数       | 没有构造函数                |
| 可以使用所有访问修饰符 | 不能使用访问修饰符（默认都是public） |
**一个类最多可以继承自一个抽象类，但可以实现多个接口。**

## 理解SOLID

理解SOLID原则是一个日常实践的问题，在编码时，应该牢记这五条基本原则。

下面是对五大原则的一个回顾：

- 单一职责：确保类只做一件事情，并且只有一个改变的原因。
- 开闭：应该能够在不改变类已有功能的情况下，扩展类的功能。
- 里氏替换：子类应该能够替换它们的父类。
- 接口隔离：保持接口的简短，只包含少量的方法，类只实现它们需要的方法。
- 依赖倒置：依赖于抽象。不要让一个具体类直接依赖于另一个具体类。

SOLID原则是帮助你写出易于维护和扩展的干净代码的向导，虽然要做一些额外的前期工作，你可能需要将某些功能重构为抽象或接口，但是长期下来往往是有回报的。

SOLID原则并**不是绝对的**，可以自己决定在项目中应用这些原则的严格程度，而且实现的方法也不是唯一的，**原则背后的思想比任何特定的语法都更重要**。

当不确定是否使用它们时，参考KISS原则。保持简单，不要仅仅为了这样做而将这些原则强制加入脚本中，应该让它们通过必要的方式有机地发挥作用。

# 游戏开发中的设计模式

设计模式是是你能够针对日常软件问题重新使用众所周知的解决方案。然而，模式并不是现成的库或框架，也不是一种算法。

设计模式更像是一张蓝图，它是一个总体的规划，具体的实现取决于你，遵循同一个模式的不同程序会有截然不同的代码。

当开发者在开发中遇到相同的问题时，他们中的许多人将不可避免地提出类似的解决方案。一旦这样的解决方案被重复到一定程度，就会有人发现一种模式，并正式地给它起一个名字。

## 四人帮

今天的许多软件设计模式都源于Erich Gamma、Richard Helm、Ralph Johnson和John Vlissides的开创性著作《设计模式:可重用的面向对象软件的元素》，原作者通常被称为“四人帮”，原始的模式被称作GoF模式。

 自从 Gang of Four 在 1994 年出版了《设计模式》一书以来，开发者们在各个领域发现了更多的面向对象的模式。许多工程专业都有一些成熟的模式，游戏开发也不例外。

## 学习设计模式

虽然不学设计模式也不妨碍你成为一名游戏程序员，但设计模式可以帮助你成为更好的开发者。

软件工程师在平时的开发过程中会不断地重新发现它们，你也可能已经在不知不觉中实现过某些设计模式了。训练自己去发现它们，会对下面这些方面有所帮助：

- 学习面向对象编程：设计模式并不是隐藏在StackOverflow帖子中的秘密，它们是解决开发中常见问题的通用方式，能告诉你有多少开发者也遇到过相同的问题。即使你不使用设计模式，别人也会使用。
- 和其他开发者交流：在团队中沟通交流时，模式可以作为一种速记，当你提及命令模式或对象池时，有经验的开发者就会准确地知道你想要实现什么。
- 探索新的框架：当你从Unity资源商店导入内置的包或其他东西时，你不可避免地会碰到这里讨论过的一些设计模式。认识设计模式可以帮助你快速理解一个新框架的工作原理，以及在创建框架时涉及的思维过程。

和其他任何的工具一样，一个设计模式是否有用取决于当下的情景，在特定的情景下会带来好处，但也会有缺点，在软件开发中的每一个决定都伴随着妥协。

## Unity中的模式

Unity已经实现了许多既定的设计模式，省去你自己编写的麻烦。

- *游戏循环*：所有游戏的核心都是一个无限循环，它必须独立于时钟速度运行，因为驱动游戏应用程序的硬件可能变化很大。为了适应不同速度的电脑，游戏开发者通常需要使用固定的时间步长(即设定帧数每秒)和可变的时间步长（即引擎会测量从上一帧开始经过的时间)。
- *更新*：在你的游戏应用程序中，你经常会一次更新一帧对象的行为。虽然你可以在Unity中手动重建，但MonoBehaviour类会自动完成。只需使用适当的Update, LateUpdate或fixeduupdate方法来修改游戏对象和组件，使其符合游戏时钟的一个刻度。
- *原型*：通常需要在不影响原始对象的情况下复制对象。这种创建模式解决了复制和克隆对象以使其他对象与自身相似的问题。这样你就可以避免定义一个单独的类来生成游戏中的每种类型的对象。Unity的Prefab系统为游戏对象实现了一种原型形式。这允许您复制模板对象及其组件。覆盖特定属性以创建Prefab variant或将Prefab嵌套在其他Prefab中以创建层次结构。使用特殊的Prefab编辑模式单独或在上下文中编辑Prefab。
- *组件*：大多数使用Unity的人都知道这个模式。与其创建具有多个职责的大类，不如构建每个只做一件事的小组件。如果使用组合来挑选组件，则将它们组合起来以获得复杂的行为。为物理添加Rigidbody和Collider组件。为3D几何体添加一个MeshFilter和MeshRenderer。每个GameObject的丰富和独特程度取决于它的组件集合。

# 工厂模式

![[Pasted image 20240204171945.png]]

有时候用一个特殊的对象来创建其他对象是很有帮助的，在游戏过程中可能需要生成各种各样的东西，而且你通常不知道在游戏运行时自己需要什么，直到你真的需要它。

工厂模式为你指定一个特别的对象——工厂，它在内部封装了生成“产品”的各种细节，这样最直接的好处就是可以整理你的代码。

但是，如果每个产品都遵循一个公共接口或基类，则可以更进一步，使其包含更多自己的构造逻辑，将其与工厂本身隔离开来。因此，创建新对象变得更具可扩展性。

你还可以创建工厂的子类，每个子类负责生产特定的产品。

## 举例：一个简单工厂

假设你想创建一个工厂模式来为游戏关卡实例化物品，你可以使用预制体来创建游戏对象，但你或许也想在创建每一个实例时执行一些自定义的行为。

与其使用if或switch语句来写这段逻辑，不如创建一个IProduct接口和一个Factory抽象类。

```CSharp
public interface IProduct
{
	public string ProductName { get; set; }
	
	public void Initialize();
}

public abstract class Factory : MonoBehaviour
{
	public abstract IProduct GetProduct(Vector3 position);
	// shared method with all factories
	…
}
```

![[Pasted image 20240204173645.png]]

IProduct接口定义定义了产品之间的共同之处，这个例子中只有一个ProductName属性和一个初始化的方法。

接着你就可以定义任何你需要的产品（ProductA，ProductB……）了，只要实现IProduct接口即可。

Factory基类有一个返回IProduct的GetProduct方法，而且它是抽象的，不能够直接实例化。你需要从Factory继承两个具体的子类（ConcreteFactoryA和ConcreteFactoryB），从子类中得到不同的产品。GetProduct方法接收一个Vector3的变量，因此你可以方便地将预制体实例化在一个特定的位置。在每一个具体工厂中还有一个保存相关联的预制体的变量。

下面是ProductA和ConcreteFactoryA的举例：

```CSharp
public class ProductA : MonoBehaviour, IProduct
{
	[SerializeField] private string productName = "ProductA";
	public string ProductName { get => productName; set => productName
	= value ; }
	
	private ParticleSystem particleSystem;
	
	public void Initialize()
	{
		// any unique logic to this product
		gameObject.name = productName;
		particleSystem = GetComponentInChildren<ParticleSystem>();
		particleSystem?.Stop();
		particleSystem?.Play();
	}
}

public class ConcreteFactoryA : Factory
{
	[SerializeField] private ProductA productPrefab;
	
	public override IProduct GetProduct(Vector3 position)
	{
		// create a Prefab instance and get the product component
		GameObject instance = Instantiate(productPrefab.gameObject,
		position, Quaternion.identity);
		ProductA newProduct = instance.GetComponent<ProductA>();
		// each product contains its own logic
		newProduct.Initialize();
		return newProduct;
	}
}
```

请注意，每一种产品都有自己特定的Initialize逻辑。这个例子中，ProductA的预制体包含了一个粒子系统，它在ConcreteFactoryA实例化ProductA时会开始播放。工厂本身并不包含任何触发粒子效果的具体逻辑，它只需要执行Initialize方法即可，这对所有商品来说都是一样的。

## 优点和缺点

当你需要建立很多商品时，你能从工厂模式中大大受益，定义新的商品类型并不会改变现有的任何商品和相应的代码。

通过分离每一种商品内部逻辑到各自的类中，使工厂类的代码相对简短。每一个工厂只需要知道在每一种商品上调用Initialize方法，而不用清楚底层的细节。

为了实现这种模式所带来的负面效果就是，你需要创建大量的类和子类。像其他的模式一样，它会带来一定的开销，如果你不需要非常多样的商品，这是不必要的。

## 改进

工厂模式的实现可以和上面展示的有很大不同，当你在构建自己的工厂模式时，考虑进行如下的调整：

- 使用字典搜索商品：可以利用键值对的形式在字典中存储各种商品，使用一个唯一的字符串标识作为键，类型作为值，这样可以更方便地检索商品和对应的工厂。
- 使工厂（或工厂管理者）变为静态的：这样可以让工厂更容易使用，但需要额外的设定。而且静态类不会在Inspector窗口中显示，所以你也需要把商品集合也变成静态的。
- 应用在非GameObject和非MonoBehavior上：不要局限于预制体或其他Unity特定的组件，工厂模式可以用在任意C#对象上。
- 和对象池模式结合：工厂不必需要实例化或创建新对象，他们还可以从Hierarchy中取回现有对象。如果你要一次实例化很多对象，可以使用对象池模式来优化内存管理。

工厂可以根据需要生成任何游戏元素，但注意，创建产品往往不是它们的唯一目的，你可能会使用工厂模式作为另一个更大任务的一部分（例如在游戏关卡部分的对话框中设置UI元素）。

# 对象池

对象池是用于在大量创建和销毁游戏物体时，减轻CPU负担的一种优化技术。

对象池模式使用一组初始化好的对象，这些对象在未激活的情况下会在池子中等待被使用。当你需要某个对象时，程序并不实例化出该对象，而是从对象池中请求获取对象并将其激活。当使用完成，就把对象失活并返还到对象池中，而不是将其销毁。

对象池能够减少由垃圾回收峰值可能带来的卡顿。GC峰值通常伴随大量对象的创建和销毁，这是由大量的内存分配所导致的。你可以在适当的时机预先实例化好对象池，比如在加载页面时，这样玩家就不会注意到卡顿。

## 举例：简单池化系统

考虑一个简单的池化系统，其中包含两个定义好的MonoBehaviour：
- 一个ObjectPool，用于保存在要从池中获取的游戏对象集合。
- 一个PooledObject组件，添加到预制体上，让每一个克隆出的对象持有对池的引用。

在ObjectPool中，用一个字段描述池的大小，一个字段关联要存储的PooledObject对象，以及一个表示池子本身的集合（这里使用一个栈）。

```CSharp
public class ObjectPool : MonoBehaviour
{
	[SerializeField] private uint initPoolSize;
	[SerializeField] private PooledObject objectToPool;
	
	// store the pooled objects in a collection
	private Stack<PooledObject> stack;
	
	private void Start()
	{
		SetupPool();
	}
	
	// creates the pool (invoke when the lag is not noticeable)
	private void SetupPool()
	{
		stack = new Stack<PooledObject>();
		PooledObject instance = null;
		for (int i = 0; i < initPoolSize; i++)
		{
			instance = Instantiate(objectToPool);
			instance.Pool = this;
			instance.gameObject.SetActive(false);
			stack.Push(instance);
		}
	}
}
```

SetupPool方法会创建一个新的Stack，然后根据初始的池子大小实例化出PooledObject对象放进池子，在Start函数中执行SetupPool方法，确保它在游戏过程中之后执行一次。

你还需要两个方法，分别用来获取池子中的对象和归还对象。

```CSharp
	// returns the first active GameObject from the pool
	public PooledObject GetPooledObject()
	{
		// if the pool is not large enough, instantiate a new PooledObjects
		if (stack.Count == 0)
		{
			PooledObject newInstance = Instantiate(objectToPool);
			newInstance.Pool = this;
			return newInstance;
		}
		// otherwise, just grab the next one from the list
		PooledObject nextInstance = stack.Pop();
		nextInstance.gameObject.SetActive(true);
		return nextInstance;
	}
	
	public void ReturnToPool(PooledObject pooledObject)
	{
		stack.Push(pooledObject);
		pooledObject.gameObject.SetActive(false);
	}
}
```

GetPooledObject方法会在池子空的时候创建出新的PooledObject对象，如果池子中有对象，它就会直接返回一个可用的对象。只要池子足够大，大多数时候你都得到一个现有对象的引用。

每个对象池中的元素都会有一个小的 PooledObject 组件，只是为了引用 ObjectPool：

```CSharp
public class PooledObject : MonoBehaviour
{
	private ObjectPool pool;
	public ObjectPool Pool { get => pool; set => pool = value; }
	
	public void Release()
	{
		pool.ReturnToPool(this);
	}
}
```

调用Release方法会禁用掉游戏对象，然后返还到对象池中。

## 改进

上面是一个简单的例子，当实际在项目中应用对象池时，考虑进行如下升级：
- 让它变成静态的或单例的：如果你需要在各种地方生成对象池中的对象，考虑把对象池变成静态的，这样就可以在程序中到处使用，但在Inspector窗口中就无法使用了。或者，将对象池模式和单例模式结合，使其可以全局访问方便使用。
- 使用字典来管理多个池：如果你有大量不同的预制体需要池化，将它们存进不同的池子，用键值对来保存各个池子，方便查询。
- 创造性地移除不使用的游戏对象：高效利用对象池的一个要点是，隐藏不使用的对象并将它们返还对象池。利用每一个时机停用对象，比如当对象飞离屏幕，被爆炸遮挡时等等。
- 检查错误：避免释放掉已经在池子中的对象，否则会在运行时引起错误。
- 最大容量上限：大量池化对象会消耗内存，你可能需要移除超出容量限制的对象，避免对象池占用过多的资源。

## UnityEngine.Pool

> 在Unity2021以上的版本，已经内置了对象池系统，所以不需要自己去创建PooledObject或者ObjectPool类了。

```CSharp
using UnityEngine.Pool;

public class RevisedGun : MonoBehaviour
{
	…
	// stack-based ObjectPool available with Unity 2021 and above
	private IObjectPool<RevisedProjectile> objectPool;
	
	// throw an exception if we try to return an existing item, already in the pool
	[SerializeField] private bool collectionCheck = true;
	// extra options to control the pool capacity and maximum size
	[SerializeField] private int defaultCapacity = 20;
	[SerializeField] private int maxSize = 100;
	
	private void Awake()
	{
		objectPool = new ObjectPool<RevisedProjectile>(CreateProjectile,
		OnGetFromPool, OnReleaseToPool, OnDestroyPooledObject,
		collectionCheck, defaultCapacity, maxSize);
	}
	
	// invoked when creating an item to populate the object pool
	private RevisedProjectile CreateProjectile()
	{
		RevisedProjectile projectileInstance = Instantiate(projectilePrefab);
		projectileInstance.ObjectPool = objectPool;
		return projectileInstance;
	}
	
	// invoked when returning an item to the object pool
	private void OnReleaseToPool(RevisedProjectile pooledObject)
	{
		pooledObject.gameObject.SetActive(false);
	}
	
	// invoked when retrieving the next item from the object pool
	private void OnGetFromPool(RevisedProjectile pooledObject)
	{
		pooledObject.gameObject.SetActive(true);
	}
	
	// invoked when we exceed the maximum number of pooled items (i.e.destroy the pooled object)
	private void OnDestroyPooledObject(RevisedProjectile pooledObject)
	{
		Destroy(pooledObject.gameObject);
	}
	
	private void FixedUpdate()
	{
		…
	}
}
```

```CSharp
public class RevisedProjectile : MonoBehaviour
{
	…
	
	private IObjectPool<RevisedProjectile> objectPool;
	
	// public property to give the projectile a reference to its ObjectPool
	public IObjectPool<RevisedProjectile> ObjectPool { set => objectPool = value; }
	
	…
}
```

# 单例模式

单例模式可能是你最早接触到的设计模式，它也是常常被人诟病的模式之一。

根据原始四人帮，单例模式的特点是：
- 保证一个类自己只有一个实例
- 提供对单个实例轻松的全局访问

如果你需要唯一一个对象来协调整个场景的工作，单例模式会非常有用。例如，你可能希望场景中只有一个游戏管理器来指导主要的游戏循环。你可能还希望一次只需要一个文件管理器写入文件系统。像这样的中心、管理器级对象就非常适合使用单例模式。

![[Pasted image 20240205160505.png]]

在游戏编程模式中，单例模式被认为是反模式，弊大于利。这种坏名声是该模式的易用性使其容易被滥用所导致的，开发者往往将单例应用在不恰当的情景中，引入不必要的全局状态或依赖。

接下来就看看如何在Unity中构建单例，并权衡它的利弊，然后你就可以决定是否在你的项目中使用了。

## 举例：简单单例

下面一种最简单的单例写法：

```CSharp
public class SimpleSingleton : MonoBehaviour
{
	public static SimpleSingleton Instance;
	
	private void Awake()
	{
		if (Instance == null)
		{
			Instance = this;
		}
		else
		{
			Destroy(gameObject);
		}
	}
}
```

## 持久性和延迟加载

上面的写法存在两个问题：
- 在加载新的场景时会销毁游戏物体
- 在使用前需要把单例放置在场景中

因为单例通常被用作无处不在的管理器脚本，可以使用DontDestroyOnLoad。

你还可以使用延迟加载的方法，在第一次使用单例时才自动地构建出单例。你只需要一些创建游戏对象的逻辑，然后添加相应的单例组件。

下面是改进过后的单例：

```CSharp
public class Singleton : MonoBehaviour
{
	private static Singleton instance;
	public static Singleton Instance
	{
		get
		{
			if (instance == null)
			{
				SetupInstance();
			}
			return instance;
		}
	}
	
	private void Awake()
	{
		if (instance == null)
		{
			instance = this;
			DontDestroyOnLoad(this.gameObject);
		}
		else
		{
			Destroy(gameObject);
		}
	}
	
	private static void SetupInstance()
	{
		instance = FindObjectOfType<Singleton>();
		if (instance == null)
		{
			GameObject gameObj = new GameObject();
			gameObj.name = "Singleton";
			instance = gameObj.AddComponent<Singleton>();
			DontDestroyOnLoad(gameObj);
		}
	}
}
```

## 使用泛型

以上两个版本的单例都没有解决在同一个场景中创建不同类型单例的问题。例如，你想要有一个AudioManger的单例和一个GameManager的单例，它们现在还无法共存，除非将单例的代码各自复制一份。

下面是泛型版本的单例脚本：

```CSharp
public class Singleton<T> : MonoBehaviour where T : Component
{
	private static T instance;
	public static T Instance
	{
		get
		{
			if (instance == null)
			{
				SetupInstance();
			}
			return instance;
		}
	}
	
	public virtual void Awake()
	{
		RemoveDuplicates();
	}
	
	private static void SetupInstance()
	{
		instance = (T)FindObjectOfType(typeof(T));
		if (instance == null)
		{
			GameObject gameObj = new GameObject();
			gameObj.name = typeof(T).Name;
			instance = gameObj.AddComponent<T>();
			DontDestroyOnLoad(gameObj);
		}
	}
	
	private void RemoveDuplicates()
	{
		if (instance == null)
		{
			instance = this as T;
			DontDestroyOnLoad(gameObject);
		}
		else
		{
			Destroy(gameObject);
		}
	}
}
```

## 优缺点

单例模式不想其他模式那样，它在各方面都破坏了SOLID原则，许多开发者讨厌单例的原因有很多：
- 单例需要全局访问：因为你将它们用作全局实例，许多依赖都被隐藏了，使排查错误变得更难。
- 单例导致测试困难：单元测试必须相互独立，由于单例会跨场景地改变各个游戏对象的状态，它们可能会影响测试。
- 单例鼓励紧耦合：大多数模式都是试图去解耦的，而单例则相反。紧耦合使重构变得困难，如果更改一个组件，会影响另一个与之关联的组件，这会导致不干净的代码。

反对单例的理由是很多的，如果你要构建一款企业级的游戏，并维护数年，你可能希望避开单例。

但许多游戏并不是企业级的应用程序，你不需要像商业软件一样维护它们。

实际上，当你构建一个不需要扩展性的小游戏时，你会发现单例带来的收益是非常诱人的：
- 单例相对易学：单例模式本身就比较直接。
- 单例是友好的：要在另一个组件使用单例，是需要简单地引用公共或静态的实例，场景中的任何对象需要时，单例都是可用的。
- 单例性能好：因为你对静态的单例实例总是有全局的访问权限，你可避免GetComponent或Find操作带来的开销。

# 命令模式

当你想要跟踪一系列特定的操作，命名模式是非常有用的。

如果你玩过有撤销/重做功能或者将你的操作历史保存在列表中的游戏，你或许已经见识过命名模式的工作了。想象一下在一款策略游戏中，玩家可以在执行实际操作之前，计划好几个回合，这就是命名模式。

命名模式允许你将一个或多个请求封装成一个“命令对象”，而不会直接地直接一个方法。

![[Pasted image 20240206161133.png]]

这些命令对象存在于一个队列或者栈中，让你可以控制它们的执行时机。这样就起到了一个小小缓冲区的作用，你可以延迟执行一系列操作，以便稍后回放或撤销。

为了实现命令模式，你需要一个能够包含操作的通用对象，这个命令对象中会保存要执行的逻辑以及如何撤销该命令。

## 命令对象和命令调用者

实现的方式有很多种，这里是一个使用接口的版本：

```CSharp
public interface ICommand
{
	void Execute();
	void Undo();
}
```

每一个游戏操作都将实现ICommand接口。每一个命名对象都只负责自己的Execute和Undo方法，因此在游戏中添加更多的命令不会影响现有的命令。

你还需要另外一个类来执行和撤销这些命令，就创建一个CommandInvoker类吧。CommandInvoker除了要实现执行命令和撤销命令的方法，它还要有一个撤回栈来保存命令序列。

```CSharp
public class CommandInvoker
{
	private static Stack<ICommand> undoStack = new Stack<ICommand>();
	
	public static void ExecuteCommand(ICommand command)
	{
		command.Execute();
		undoStack.Push(command);
	}
	
	public static void UndoCommand()
	{
		if (undoStack.Count > 0)
		{
			ICommand activeCommand = undoStack.Pop();
			activeCommand.Undo();
		}
	}
}
```

## 举例：可撤销的移动

想象一下你要移动一个在迷宫中的游戏角色，你可以创建一个PlayerMover类来负责改变玩家的位置：

```CSharp
public class PlayerMover : MonoBehaviour
{
	[SerializeField] private LayerMask obstacleLayer;
	private const float boardSpacing = 1f;
	
	public void Move(Vector3 movement)
	{
		transform.position = transform.position + movement;
	}
	
	public bool IsValidMove(Vector3 movement)
	{
		return !Physics.Raycast(transform.position, movement, boardSpacing, obstacleLayer);
	}
}
```

根据命令模式，将PlayerMover的Move方法提取为一个对象，创建一个实现ICommand接口的MoveCommand类：

```CSharp
public class MoveCommand : ICommand
{
	PlayerMover playerMover;
	Vector3 movement;
	
	public MoveCommand(PlayerMover player, Vector3 moveVector)
	{
		this.playerMover = player;
		this.movement = moveVector;
	}
	
	public void Execute()
	{
		playerMover.Move(movement);
	}
	
	public void Undo()
	{
		playerMover.Move(-movement);
	}
}
```

MoveCommand中会保存一些执行命名需要用到的参数，需要在构造函数中初始化它们。

当你创建了命令对象并设置好需要的参数之后，就可以调用CommandInvoker的静态ExecuteCommand方法，并将命令对象传入。

![[Pasted image 20240206164542.png]]

InputManager不会直接执行PlayerMover的Move方法，而是另外写一个RunMoveCommand方法：

```CSharp
private void RunPlayerCommand(PlayerMover playerMover, Vector3 movement)
{
	if (playerMover == null)
	{
		return;
	}
	
	if (playerMover.IsValidMove(movement))
	{
		ICommand command = new MoveCommand(playerMover, movement);
		CommandInvoker.ExecuteCommand(command);
	}
}
```

## 优缺点

使用命令模式之后，实现回放和撤销功能就跟创建一个对象集合一样简单，还可以使用命令缓冲区来回放序列中的操作。

例如，在格斗游戏中，按下一系列按钮之后会触发连击。使用命令模式保存玩家的操作，使这种连击的设置更加简单。

另一方面，和其他设计模式一样，命令模式引入了更多的结构，你必须考虑这些额外的类和接口在你的程序中能否带来足够的好处。

## 改进

当使用命令模式时，考虑做如下改进：
- 创建更多的命令：上面的例子只有一种命令，你可以创建无数实现ICommand的命令对象，然后使用CommandInvoker来跟踪它们。
- 增加重做功能就是添加另外一个栈：当你撤销命令时，把命令放进另一个栈来追踪重做操作。这样做，你就可以快速地遍历撤销历史然后重做这些操作。当玩家执行一个全新的命令时，清空撤销栈。![[Pasted image 20240206170933.png]]
- 使用不同类型的集合作为命令对象的缓冲区。![[Pasted image 20240206171214.png]]
- 限制栈的大小。

# 状态模式

想象一下构建一个可玩的角色，在某一时刻，角色可能站在地面上，然后移动方向键，角色开始跑动或行走，按下跳跃键时，角色会跳到半空中，几帧过后，又会回到地面。

## 状态和状态机

游戏都是交互式的，它们迫使我们追踪许多在运行时发生变化的系统。如果画一个图表来表示游戏角色的不同状态，可能是像下面这样的：

![[Pasted image 20240206172817.png]]

这就像一个流程图，但是有一些不同：
- 图表由一系列状态（Standing，Walking，Jumping）组成，而且在某一时刻只有一个状态是激活的。
- 状态与状态之间会基于特定的条件发生转移。
- 当发生状态转移时，目标状态会变成新的激活状态。

这个图表描述的是一种叫做有限状态机的东西，在游戏开发中的一个典型用例就是追踪游戏角色或者道具的内部状态。

要在代码中描述一个基本的有限状态机，可以使用枚举和switch语句这种简单的方法。

```CSharp
public enum PlayerControllerState
{
	Idle,
	Walk,
	Jump
}

public class UnrefactoredPlayerController : MonoBehaviour
{
	private PlayerControllerState state;
	
	private void Update()
	{
		GetInput();
		switch (state)
		{
			case PlayerControllerState.Idle:
				Idle();
				break;
			case PlayerControllerState.Walk:
				Walk();
				break;
			case PlayerControllerState.Jump:
				Jump();
				break;
		}
	}
	
	private void GetInput()
	{
		// process walk and jump controls
	}
	
	private void Walk()
	{
		// walk logic
	}
	
	private void Idle()
	{
		// idle logic
	}
	
	private void Jump()
	{
		// jump logic
	}
}
```

这种方式可以工作，但是PlayerController脚本会很快变得凌乱，每次增加新的状态和复杂逻辑时，都需要我们重新访问PlayerController脚本内部。

## 举例：简单的状态模式

幸运的是，状态模式可以帮助你重新组织逻辑。根据最初的四人帮，状态模式解决了两个问题：

- 当一个对象内部的状态改变时，其行为也应该发生改变。
- 状态特定的行为需要被独立定义，添加新的状态不会影响已有状态的行为。

上面例子中的UnrefactoredPlayerController类能够追踪状态的改变，也就是解决了第一个问题，但它不能解决第二个问题。可以将每一个状态都封装成一个对象，来最小化添加新状态给已有状态带来的影响。

想象一下像这样构建每一种状态：

![[Pasted image 20240206175128.png]]

现在你进入了某个状态，而且每帧都会循环，直到某个条件满足导致控制流退出。

要实现这个模式，首先创建一个IState接口：

```CSharp
public interface IState
{
	public void Enter()
	{
		// code that runs when we first enter the state
	}
	
	public void Update()
	{
		// per-frame logic, include condition to transition to a new state
	}
	
	public void Exit()
	{
		// code that runs when we exit the state
	}
}
```

游戏中每一个实际的状态都会实现IState接口：
- 一个入口：当第一个进入该状态时执行的逻辑。
- 更新：每帧执行的逻辑（有时候也叫做Execute或者Tick）。你还可以像MonoBehaviour那样将Update方法拆分为不同的片段，如FixedUpdate、LateUpdate等等。在Update方法中的功能会每帧执行，直到满足某个条件触发状态的转移。
- 一个出口：在离开状态/转移到另一个新的状态之前执行的逻辑。

你需要为每一个状态都创建一个类，并且实现IState接口。

StateMachine类会具体管理每个状态的进入、更新和退出。

```CSharp
[Serializable]
public class StateMachine
{
	public IState CurrentState { get; private set; }
	
	public WalkState walkState;
	public JumpState jumpState;
	public IdleState idleState;
	
	public void Initialize(IState startingState)
	{
		CurrentState = startingState;
		startingState.Enter();
	}
	
	public void TransitionTo(IState nextState)
	{
		CurrentState.Exit();
		CurrentState = nextState;
		nextState.Enter();
	}
	
	public void Update()
	{
		if (CurrentState != null)
		{
		CurrentState.Update();
		}
	}
}
```

StateMachine为其管理的每一个状态引用一个公共对象，因为StateMachine不继承自MonoBehaviour，所以使用一个构造函数来创建每一个状态实例。

```CSharp
public StateMachine(PlayerController player)
{
	this.walkState = new WalkState(player);
	this.jumpState = new JumpState(player);
	this.idleState = new IdleState(player);
}
```

关于StateMachine，注意一下几点：
- Serializable特性可以让StateMachine在Inspector窗口中显示，另一个MonoBehaviour脚本可以将它作为一个字段使用。
- CurrentState属性是只读的，StateMachine本身并不会显式地设置该属性，像PlayerController这样的外部对象可以调用Initialize方法来设置默认的状态。
- 每一个状态确定自己的转移条件，调用TransitionTo方法来改变状态机当前激活的状态。你可以在初始化状态机的时候，将任何必须的依赖（包括StateMachine自身）传入每一个状态中。

每一个状态对象会管理其自己内部的逻辑，所以你可以根据需要制作尽可能多的状态来描述你的游戏对象或组件。

下面是IdleState的例子：

```CSharp
public class IdleState : IState
{
	private PlayerController player;
	
	public IdleState(PlayerController player)
	{
		this.player = player;
	}
	
	public void Enter()
	{
		// code that runs when we first enter the state
	}
	
	public void Update()
	{
		// Here we add logic to detect if the conditions exist to
		// transition to another state
		…
	}
	
	public void Exit()
	{
		// code that runs when we exit the state
	}
}
```

## 优缺点

状态模式可以帮助你在设定对象内部逻辑时遵循SOLID原则。每一个状态都是相对较小的，仅仅追踪切换到另一个状态的条件。为了符合开闭原则，你可以在不影响现有状态的情况下增加更多的状态，避免了使用笨重的switch语句。

另一方面，如果你只需要管理少数几个状态，这种额外的结构就有点大材小用了。状态模式可能只在你期望你的状态增长到一定的复杂度时才有意义。

## 改进

示例项目中的胶囊体仅仅是随着内部状态的改变去更新颜色和UI，在实际的项目中，你可以在状态改变中实现更复杂的效果：
- 将动画与状态模式结合：状态模式的一个常见应用就是动画。玩家或怪物角色通常在宏观层面表示为原始模型，你可以在内部状态发生改变时给几何体增加动画表现效果，然后游戏角色就可以表现出跑、跳、游泳等行为。
- 添加事件：为了在状态改变时与外部对象通信，你可能需要添加事件。比如在进入或离开某个状态时，通知监听者做出反应。
- 增加层次结构：当你开始使用状态模式描述更加复杂的实体时，你可能希望实现分层状态机。不可避免地，有些状态会是相似的。例如，当玩家在地面上时，他可以在行走状态或奔跑状态下进行躲避和跳跃。可以实现一个超级状态，将一些通用的行为放在里面，通过继承衍生出子状态。
- 实现简单的AI：在生成基础的敌人AI时，有限状态机也会非常有用。用有限状态机构建一个NPC可能是像这样的：![[Pasted image 20240206185251.png]]
