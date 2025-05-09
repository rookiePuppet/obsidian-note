# 多线程

## 进程、线程、协程

> 进程

进程是操作系统分配资源的基本单位，是程序运行的一个实例。有独立的内存空间和系统资源，创建和切换的开销较大。

进程间通信比较复杂，通过消息队列、管道和共享内存等机制。

> 线程

线程是CPU调度的基本单位，是进程中的执行单元。共享进程的内存空间和资源，创建和切换的开销较小。

线程间通信通过共享内存实现。

> 协程

协程是用户态的轻量级线程，由程序员显式控制调度。通常在单线程中运行，创建和切换的开销极小。

线程的生命周期：

1. 创建。通过Thread类或线程池初始化。
2. 就绪。调用Start()后等待CPU调度。
3. 运行。正在执行代码。
4. 阻塞。等待I/O、锁或信号时暂停，如Sleep()、Wait()。
5. 终止。执行完毕或被终止。

## 创建线程

1. 直接创建

```CS
Thread thread = new Thread(() => Console.WriteLine("子线程"));
thread.Start();
```

2. 线程池

```CS
ThreadPool.QueueUserWorkItem(state => Console.WriteLine("线程池任务"));
```

## 线程同步机制

### 锁

1. lock关键字，基于Monitor。

```CS
private object _lockObj = new object();
lock (_lockObj) {
    // 临界区代码
}
```

2. Monitor，更灵活，支持短时

```CS
Monitor.Enter(_lockObj);
try { /* 临界区 */ }
finally { Monitor.Exit(_lockObj); }
```

### 信号量

1. Mutex
2. Semaphore

## 异步编程模型

 创建任务:

```CS
Task.Run(() => Console.WriteLine("Task运行"));
```

async/await:

```CS
public async Task<int> GetDataAsync() {
    await Task.Delay(1000);
    return 42;
}
```

配置上下文:

```CS
await Task.Run(() => {}).ConfigureAwait(false); // 避免强制回到原始上下文
```

取消操作：

```CS
var cts = new CancellationTokenSource();
var token = cts.Token;
Task.Run(() => {
    while (!token.IsCancellationRequested) {
        // 轮询检查取消
    }
}, token);
cts.CancelAfter(5000); // 5秒后取消
```

# 内存

#### 值类型和引用类型在变量赋值时的区别

由于值类型变量存储的数据就是其本身，在赋值时直接对数据进行复制，新变量和原变量相互独立，互不影响。

引用类型变量存储的数据是其引用地址，在赋值时复制的是地址，由于新变量和原变量指向同一块内存，修改其中一个变量时，也会影响另一个。

#### 装箱和拆箱

- 装箱：从值类型转换为引用类型，内存从栈上转移到堆上。
- 拆箱：从引用类型转换为值类型，内存从堆上转移到栈上。

由于装箱和拆箱都会涉及到一次内存拷贝，因此频繁进行装拆箱会大幅影响性能。

利用重载和泛型可以避免装箱和拆箱。

## 垃圾回收机制

核心原理：

1. 分代回收
2. 标记-清除-压缩

> 分代回收

将托管堆中的对象分为三代：0代、1代和2代。

0代中的对象比较小，生存期一般较短，回收最频繁。

1代中的对象是从0代回收之后存活下来的对象，回收频率较低，作为2代与0代之间的缓冲器。

2代中的对象生存期最长，回收频率最低。

经过一次GC之后仍然存活的对象，会被移动到下一代。

> 标记-清除-压缩

1. 标记：遍历并标记所有可达对象，未被标记的对象视为垃圾。
2. 清除：释放掉未标记的对象占用的内存。
3. 压缩：将存活对象移动到连续的内存地址上，清除内存碎片。

注意事项：

1. 避免内存泄漏：对于非托管资源，需要实现IDisposable接口和using语句，或手动调用Dispose方法进行释放。
2. 弱引用：弱引用不会阻止GC将其回收，适合用于缓存大型对象。

## 大对象堆

超过85kB的对象会被分配到大对象堆。

大对象堆和普通的托管堆不同，其没有分代回收机制，回收成本高、频率低。

## 内存泄漏

内存泄漏指的是超出生命周期的对象无法被回收释放，始终占用着一块内存，而这块内存无法再被使用。

常见的内存泄漏：

1. 静态引用
2. 不再使用的引用对象没有置空，一直保持引用
3. 使用非托管堆的资源时没有使用using或没有主动调用Dispose方法
4. 委托或事件注册后没有移除注册


# 语法

## \==和Equals

前者是运算符，后者是Object类的一个虚方法。

在没有重载的前提下，两者都可用于比较两个对象是否相等，对于值类型是比较值，对于引用类型是比较地址。

一般会重写Equals方法用来比较两个对象中的内容是否相同。

## 泛型

### 泛型约束

1. 引用类型约束：where T: class
2. 值类型约束：where T: struct
3. 公共无参构造函数约束：where T: new()
4. 类约束：where T: 类名
5. 接口约束：where T: 接口名

## 委托和事件

### 区别

事件是对委托的一个修饰。

事件在类的外部不能使用=运算符直接赋值，只能使用+=和-=。

## 接口

### 一个类如何同时实现两个具有同名方法的接口

利用显式接口实现。

```CS
void IA.Test() {}
void IB.Test() {}
```

## 闭包

闭包是一个函数，它可以记录定义它时的上下文变量，即使超出其作用域仍然可以访问那些变量。

闭包可用作回调函数。

# 数据与容器

## List的底层实现

List内部利用一个数组来存储元素，其默认容量为0，在添加了第一个元素之后，容量变为4。

在添加元素时，首先判断容量是否足够。若容量不足，则创建一个为当前两倍容量的数组，将原数组拷贝到新数组中，最后更新内部维护的数组引用为新数组。

## Dictionary的底层实现

Dictionary基于哈希表实现，使用拉链法解决哈希冲突。

内部有一个Entry结构体，包含key、value、next字段，next表示的是同一个哈希桶内的下一个Entry的索引。

内部用了一个entries数组来存储所有的键值对，另外还有一个buckets数组存储的是每一个Entry的在entries数组中的索引，

> 添加元素（Add方法）

1. 计算键的哈希码，对buckets数组长度取余，得到桶索引。
2. 通过桶索引找到对应的Entry，然后遍历Entry链表，检查是否存在相同的键，如果存在则抛出异常，否则创建新的Entry对象插入到entries数组，并添加到Entry链表的末尾。若buckets在桶索引出的值为-1，则直接创建Entry对象添加到entries数组；

> 查找元素

计算键的哈希码，定位哈希桶，遍历Entry链表找到对应的键。

> 扩容

当元素数量超过阈值（cout >= capacity * 负载因子）时，创建当前两倍容量的buckets和entries数组，对所有元素重新进行哈希，添加到新的数组中。

目的是减少冲突概率，提升访问效率。

## 数组和链表的区别

1. 存储结构：数组是顺序存储结构，在内存上是连续的；链表是链式存储结构，在内存上是不连续的。
2. 访问效率：数组通过索引可以直接访问某一位置元素，而链表需要从头遍历。
3. 插入删除效率：数组需要整体移动元素，效率较低；链表则能够直接将数据插入到某一位置，或删除。
4. 越界问题：数组的大小是固定的，可能发生越界

## 字符串

由于string类型是不可变的，每次对string变量修改、赋值或者使用字符串操作方法都会在堆上创建新的字符串。

对于大量的字符串修改，可以使用StringBuilder来优化。

# 序列化

序列化指的是将对象转换成可以存储或传输的格式（例如二进制、JSON等）。

常见的序列化方式：

1. 二进制
2. JSON
3. XML
4. YAML

通常在对数据进行持久化存储或在网络通讯时，会使用序列化技术。

# .NET

## 跨语言原理

.NET制定了公共语言基础结构，按照这一标准设计的编程语言（例如C#，VB），对应的编译器会编译成通用的IL代码，最后通过CLR（公共语言运行时）将IL编译成对应操作系统的原生代码。

## 跨平台原理

.NET Core和Mono为不同操作系统实现了对应的CLR（运行时/虚拟机），这样不同的操作系统都可以通过CLR将IL代码编译成机器码进行执行。

-------
# C#和C++的区别

#### 概括

C#是一种完全面向对象的语言，而C++不是。C#基于IL中间语言和.NET Framework CLR，在可移植性、可维护性和健壮性相比C++有很大的改进。C#的设计目标是用来开发快速稳定可扩展的应用程序，当然也可以通过Interop和Pinvoke进行一些层操作。

#### 具体对比

10. **继承：** C++支持多继承，而C#是单继承（可以实现多个接口）。
11. **数组：** 声明数组的语法不同。C#声明数组时中括号在数组类型后面，而C++是在变量名后面。
12. 数据类型： C++中布尔类型可以和整型相互转换，而C#不行。C#中的long类型是64位，C++的long类型是32位。
13. 结构体： C#中的类和结构体在语义上不同。struct是值类型，class是引用类型。
14. 委托类型： C#的委托和C++的函数指针基本相似，但是前者类型安全。
15. 异常处理： C#有finally语句，C++没有。
16. 方法参数： C#使用ref和out传递引用参数，C++通过指针传递。
17. 指针： C#默认不能使用指针，除非使用unsafe关键字。
18. 全局方法/变量： C#方法和变量必须包含在类型声明中。
19. 局部变量：C#局部变量在初始化之前不能使用。
20. 析构函数：C#中不能控制析构函数的调用时间，原因是析构函数由GC自动调用。

# 面向对象的三大特性

- 封装：隐藏对象的属性，外界只能通过对外提供的接口进行访问。
- 继承：子类可以复用父类的成员和方法，并且可以在现有代码的基础上进行扩展。使用*sealed* 关键字修饰的类不可被继承。
- 多态：子类对象父类指针，通过父类创建的指针来调用子类的成员和方法，实现接口的重用。分为静态多态和动态多态。

> 重载和重写的区别：

21. 重载是编译时的多态（静态），重写是运行时的多态（动态）。
22. 重载是定义名称相同但参数不同的方法，而重写指的是是子类重写父类的方法。
23. 重载对参数的个数、类型、顺序没有要求，但重写方法的参数必须和父类一致。
24. 重载对访问修饰没有特殊要求，但重写的访问修饰符的限制必须大于被重写方法的访问修饰符限制。

# 面向对象设计原则

25. **单一职责原则**（Single Responsibility Principle, SRP）：一个对象应该只包含单一的职责，并且该职责被完整地封装在一个类中。就一个类而言，应该仅有一个引起它变化的原因。如果一个类承担的职责过多，就等于将这些职责耦合在一起，当其中一个职责变化时，可能会影响其他职责的运作。
26. **开闭原则**（Open-Closed Principle, OCP）：一个软件实体应当对扩展开放，对修改关闭。也就是说在设计一个模块的时候，应当使这个模块可以在不被修改的前提下被扩展，即实现在不修改源代码的情况下改变这个模块的行为。
27. **里氏替换原则**（Liskov Substitution Principle, LSP）：父类能够被他的子类替代。这是实现多态的基本机制。
28. **依赖倒置原则**（Dependency Inversion Principle, DIP）：高层模块不应该依赖于底层模块，两者都应该依赖其抽象。具体来说，依赖倒置原则要求我们在程序设计时，将依赖关系从具体类向抽象类转移，从而实现解耦。
29. **接口隔离原则**（Interface Segregation Principle, ISP）：在设计接口的时候，尽量以小接口出现，不要设计一个复杂接口。客户端不应该被强制依赖于它们不使用的接口。
30. **迪米特法则**（Law of Demeter, LoD）：也称为最小知识原则（Least Knowledge Principle, LKP），一个对象应当对其他对象保持最少的了解。也就是说，每一个软件单位对其他的单位都只有最少的知识，而且局限于那些与本单位密切相关的软件单位。

# 值类型和引用类型

- 值类型：包含所有简单类型如int、float、bool、char，还有enum和struct。
- 引用类型：包括string、object、class、interface、delegate、array。

值类型继承自*System.ValueType* （实际也继承自System.Object），引用类型继承自*System.Object*。

#### 内存区域

值类型数据存储在栈上，一旦超出作用域就会被自动清理。

引用类型数据存储在堆中，引用地址位于栈上，指向的数据位于堆中，堆内存由GC自动释放。

#### 拷贝策略

值类型是拷贝数据，引用类型是拷贝引用地址。

值类型数据传递时，会在栈上新建一个副本，对副本进行修改不会影响原数据。

引用类型数据传递时，会在栈上拷贝数据的地址，因为新的副本和原变量指向同一个对象实例的数据，所以对副本进行修改时，原变量会受到影响。

#### 适用场合

- 值类型在内存管理方面效率更高，不支持多态，适合做存储数据的载体；引用类型支持多态，适合用于定义应用程序的行为。
- 引用类型可以派生出新的类型，但值类型不能，因为值类型是密封的。
- 引用类型可以为null，值类型需要利用可空类型功能使用null赋值（如`int? a = null`）。



# 字符串相关

string是特殊的引用类型。

string虽然是引用类型，但具备值类型的特征。例如：

```CPP
string str1 = "123"; // 在堆中开辟了一块空间存储“123”
string str2 = str1; // str2和str1指向同一块内存即字符串“123”
str2 = "321"; // 给str2赋新的值会在堆中新开空间存储“321”
// 此时str1和str2指向的内存地址不一致了
```

> 可以得出的结论：string在拼接或修改时会在堆中分配新的内存空间，因此频繁修改string变量会产生大量内存垃圾，可以使用StringBuilder进行优化。

#### 函数中多次使用string的+=处理，会产生大量内存垃圾，如何解决

使用StringBuilder的Append方法，可以减少内存垃圾。

#### string和StringBuilder相比的优势

31. 不可变性。string不可变，这意味着一旦创建了一个字符串，就不能更改它；而StringBuilder是可变的，可以多次修改字符串内容。这也使得string在多线程下更安全。
32. 性能：在处理字符串时，string的一些操作比StringBuilder效率更高。因为StringBuilder需要不断重新分配内存以容纳修改后的字符串，而string不需要。
33. 内存使用：由于string不可变，所以可以安全地存储大量的字符串常量，而无需担心修改它们，这对性能和内存管理都有好处。
34. 兼容性：由于string是基本类型，因此在许多API和库中都使用它，这意味着使用string通常可以和更多的代码和库进行交互。

# 堆和栈的区别

#### GC方面

栈保持先进后出的原则，是一片连续的内存区域，由系统自动分配和维护，产生的垃圾由系统自动释放。

堆是无序的，内存区域不连续，用户可以自己释放，如果内存达到一定阈值时，会通过GC来回收内存垃圾。

#### 存储方面

栈通常保存我们代码执行的步骤，如方法、变量等，使用完之后会被自动清理。

堆上存放的一般是对象，数据等，使用完不会被立即清理。

#### 缓存方面

栈使用的是一级缓存，通常都是被调用时处于存储空间中，调用完立即释放。

堆使用的是二级缓存，生命周期由虚拟机的垃圾回收算法来决定，访问速度相对低一些。

# .NET和Mono的关系

.NET是微软开发的跨平台开发框架，主要用于在Windows环境下进行应用程序开发。.NET提供了丰富的类库、语言编译器和运行时环境（CLR），以支持开发者使用多种编程语言进行应用构建和运行。

Mono是一个开源的跨平台实现，它为.NET提供了类似Java虚拟机的功能，使得.NET应用可以跨平台运行。Mono集成了.NET的编译器、CLR和基础类库，使开发者可以使用.NET进行跨平台开发，并在不同操作系统上运行.NET应用。

# GC相关

#### GC（Garbage Collection）的概念

C#的垃圾回收机制（Garbage Collection，GC）是自动管理内存的一种机制，它负责跟踪和回收不再被程序使用的内存资源。

>垃圾回收机制的主要原理：

35. **对象的分配**：当在C#中创建对象时，这些对象会在堆（Heap）上分配内存空间。
36. **引用计数**：垃圾回收器通过跟踪每个对象的引用计数来确定对象是否仍被使用。每当有一个引用指向对象时，引用计数增加；每当移除一个引用时，引用计数减少。如果引用计数为零，垃圾回收器认为该对象不再被使用。
37. **标记和清扫**：垃圾回收器定期执行标记和清扫的过程。首先，它从应用程序的根（Root）开始，遍历所有可达的对象，并标记它们为“活动”或“存活”。然后，它清扫堆内存，寻找未被标记的对象。这些未被标记的对象被认为是垃圾，因为它们不可达，无法被程序继续使用。
38. **回收和资源释放**：一旦垃圾回收器确定了哪些对象是垃圾，它会释放这些对象占用的内存资源。这会将内存返回给可用内存池，以便将来再次分配。

引用计数是一种简单快速但有时不太精确的方法，可能会出现以下两个问题：

39. **循环引用**：两个或更多对象互相引用但不被外部引用时，它们的引用计数永远不会降到零，即使它们实际上是不可达的。
40. **额外开销**：每次引用变更都需要更新计数，这在某些场景下会增加额外的开销。

相比之下，标记清扫速度较慢但更加准确，它能够找到所有真正可到达的对象，即时存在循环引用。

C#的垃圾回收期结合了两种方法的优点，以实现高效且准确的内存管理。

#### GC会带来的问题

41. **GC操作需要消耗大量的时间**。如果堆内存上有大量的变量或者引用需要检查，那么检查的操作会十分缓慢，这就会使得程序运行缓慢。
42. **GC在关键时候运行**。比如在CPU处于游戏的性能运行关键时刻，任何一个额外的操作都可能会带来极大的影响，使得游戏帧率下降。
43. **GC会导致堆内存的碎片化**。当一个内存单元从堆内存上分配出来，其大小取决于其存储的变量的大小。当该内存被回收到堆内存上的时候，有可能使得堆内存被分割成碎片化的单元。这意味着堆内存总体可以使用的内存单元较大，但是单独的内存单元较小，在下次内存分配的时候不能找到合适大小的存储单元，这也会触发GC操作或者堆内存扩展操作。
44. **GC压力过大**。如果GC优化导致GC频率过高或GC运行时间过长，会影响程序的响应时间和吞吐量，从而导致程序不稳定。
45. **内存泄漏**。如果GC优化导致一些对象没有及时回收，可能会导致内存泄漏，进而导致内存溢出和程序崩溃。
46. **CPU占用过高**。如果GC优化导致GC线程占用过高的CPU资源，会导致应用程序的响应时间变慢，甚至导致应用程序崩溃。

#### GC触发的时机

47. 在堆内存上进行内存分配操作，而内存不够时就会触发GC来获得闲置的内存。
48. GC会自动触发，不同平台的触发频率不一样。
49. 可以手动触发GC。

#### 如何避免GC

50. 减少临时变量的使用，多使用公共对象，多利用缓存机制。
51. 减少new对象的次数。
52. 对于大量的字符串拼接操作，使用StringBuilder替换String。
53. 使用扩容的容器例如List、StringBuilder时，定义时尽量根据存储变量的内存大小定义存储空间，减少扩容操作。
54. 在游戏加载页面或者进度条时GC，而不是在游戏运行的关键时刻进行GC。
55. 对于频繁创建和删除的对象利用对象池进行优化。
56. 减少装箱拆箱操作。例如使用协程时，使用`yield return null`替换`yield return 0`（产生装箱拆箱）；

#### 内存泄漏

内存泄漏指的就是对象超过生命周期后而不能被GC回收，一般指不会再使用的引用对象由于某些操作而不能被GC垃圾回收，而一直占用着内存。

> 常见的内存泄漏

57. 静态引用
58. 不使用的引用对象没有置null，一直被引用
59. 文件操作时，没有使用using或者没有进行Dispose()
60. 委托或事件注册后没有解除注册（有加就有减）

# 语言语法相关

#### 结构体和类

**区别**

|                 | 结构体 | 类    |
| --------------- | --- | ---- |
| 类型              | 值类型 | 引用类型 |
| 存储位置            | 栈   | 堆    |
| 声明无参构造          | 不能  | 能    |
| 声明有参构造后，无参构造被顶掉 | 不会  | 会    |
| 声明析构函数          | 不可以 | 可以   |
| 被继承             | 不能  | 能    |
| 在构造函数中初始化所有成员变量 | 必须  | 非必须  |
| 被static修饰       | 不能  | 能    |

**使用场景**

- 结构体：
	1. 存储轻量级的对象时使用结构体，比如点、矩阵、颜色等。因为结构体存储在栈上，而栈的容量小，但存取速度快。
	2. 存储对象是数据集合时优先考虑结构体，如位置、坐标等。
	3. 变量传值时，如果希望传递深拷贝对象，而不是引用地址，可以使用结构体。
- 类：
	1. 存储重量级对象时使用类。因为类存储在堆中，堆的容量足够大。
	2. 当对象需要继承和多态特征时，使用类。

#### C#中的访问修饰符

**属性修饰符**

- Serializable：按值将对象封送到远程服务器。
- STAThread：单线程套间的意思，一种线程模型。
- MATAThread：多线程套间的意思，一种线程模型。

**存取修饰符**

- public：存储无限制。
- private：只有包含该成员的类可以存取。
- internal：只有当前工程可以存储。
- protected：只有包含该成员的类和派生类可以存取。

**类修饰符**

- abstract：抽象类，指示一个类只能作为其他类的基类。
- sealed：密封类，指示一个类不能被继承。

**成员修饰符**

- abstract：指示该方法或属性没有实现。
- sealed：密封方法。防止派生类对该方法重写。
- delegate：委托。用来定义一个函数指针。
- event：事件。
- const：指定该成员的值只读，不允许修改。
- extern：指示方法在外部实现。
- override：重写。
- readonly：指示一个域只能在声明时以及在类的内部赋值。
- static：指示一个成员属于类型本身，而不是属于特定的对象。
- virtual：指示一个方法或存取器的实现可以在继承类中被覆盖。
- new：在派生类中隐藏指定的基类成员，从而实现重写的功能。

#### 静态构造函数

61. 静态构造函数既没有访问修饰符，也没有参数。
62. 在创建第一个类实例或任何静态成员被引用时调用静态构造函数，最多执行一次。
63. 一个类只能有一个静态构造函数，可以和无参构造函数共存。
64. 如果没有写静态构造函数，而类中包含带有初始值的静态成员，那么编译器会自动生成默认的静态构造函数。
65. 静态构造函数不可被继承。
66. 静态构造函数先于构造函数执行。

#### 虚函数的实现原理

虚函数的原理是**动态绑定**，也就是在运行时期确定函数调用的具体实现。在面向对象的程序中，如果有指针或引用调用的虚函数，那么运行时就就会去确定到底调用哪个函数。

具体来说，每一个有虚函数的类（或有虚继承的类）都有一个虚表，这个虚表中存放的就是虚函数的地址。类的实例在调用虚函数时，就会先找到虚表，然后在虚表中找到需要调用的虚函数地址，进而调用虚函数。

 > !虚表是属于类的，不是属于某个对象的。

子类如果重写了虚函数，那么子类虚表中对应的虚函数入口地址会更新为子类重新实现的虚函数地址。

#### 指针和引用的区别

67. **初始化和使用方式**：引用在创建时必须被初始化，并且一旦初始化，就不能再被另一个对象引用。而指针在定义的时候不必初始化，可以在后面的任何地方重新赋值。
68. **NULL引用**：引用不能使用指向空值的引用，它必须指向某个对象；而指针可以是NULL，不需要总是指向某些对象，可以把指针指向任意对象，所以指针更加灵活，也容易出错。
69. **应用上的区别**：如果指向一个对象就不会改变指向，那么应该使用引用。如果指向NULL（不指向任何对象）或在不同的时刻指向不同的对象，应使用指针。

#### 什么是泛型

泛型是C# 2.0版中引入的一个新功能，它允许在类、接口和和方法中使用类型参数。通过使用泛型类型参数，可以创建能够接受不同类型参数的类和方法，从而实现代码的复用和类型安全。

泛型不会强制对值类型进行装箱拆箱，或对引用类型进行向下强制转换，所以**性能高**。

通过知道使用泛型定义的变量的类型限制，编译器可以在一定程度上验证类型假设，所以提高了程序的**类型安全**。

#### 泛型约束的种类

- 基类约束：约束泛型参数必须是某个基类或基类的子类。
- 接口约束：约束泛型参数必须实现某个接口。
- 无参构造约束：约束泛型参数必须有无参构造函数。
- 值类型约束：约束泛型参数必须是值类型。
- 引用类型约束：约束泛型参数必须是引用类型。

#### unsafe关键字

在C#中，指针是禁止直接使用的。但是，如果你需要使用指针来操作内存，例如对内存进行直接的读写操作，那么就需要使用`unsafe`关键字。

#### ref和out的区别

70. ref修饰的参数必须初始化，out修饰的参数必须在方法中对其完成初始化。
71. out适合用在需要多个返回值的地方，ref用在需要在方法中通过引用修改传入参数的值时。
72. ref参数可以在方法中直接使用，而out参数必须在方法中初始化之后才能使用。

#### foreach迭代器遍历和for循环遍历的区别

73. for循环遍历需要知道遍历的次数，而foreach不需要。
74. 在for循环中需要通过索引来访问元素，但是foreach不需要索引，直接访问元素本身。
75. 在多线程环境下，foreach比for循环更安全，因为它不暴露集合的内部结构。
76. foreach语法更简洁，代码可读性比for更强。
77. 在某些情况下，for循环的性能比foreach更好。

> Foreach循环迭代时，若把其中的某个元素删除，程序报错，怎么找到那个元素？以及具体怎么处理这种情况？(注：Try…Catch捕捉异常，发送信息不可行)

foreach不能进行元素的删除，因为迭代器会锁定迭代的集合，解决方法：记录找到索引或者key值，迭代结束后再进行删除。

*对象必须实现IEnumerable以及IEnumerator接口才能使用Foreach遍历。

#### 委托和事件

**委托是什么，有什么用**

**委托**类似于一种安全的指针引用，在使用它时是当做类来看待而不是一个方法，相当于对一组方法的列表的引用，可以便捷的使用委托对这个方法集合进行操作。委托是对函数指针的封装。

使用委托使程序员可以将方法引用封装在委托对象内。然后可以将该委托对象传递给可调用所引用方法的代码，而不必在编译时知道将调用哪个方法。与C或C++中的函数指针不同，委托是面向对象，而且是类型安全的。

**委托和事件的区别**

78. 委托可以用=赋值，事件只能用+=和-=。
79. 委托可以在外部调用，而事件只能在内部调用。
80. 委托是一个类型，事件修饰的是一个对象。
81. 委托可以作为参数传递，而事件不能。
82. 委托可以作为临时变量，而事件不能。

#### 接口和抽象类

**概念**

- 抽象类：
	- 当多个类中有重复的部分时，可以抽象出一个基类，如果希望基类不能被实例化，就可以将它设置为抽象类。
	- 当需要为一些类提供公共的实现代码时，优先考虑抽象类。因为抽象类中的非抽象方法可以被子类继承，使实现功能的代码更加简单。
- 接口：
	- 当注重代码的扩展性和可维护性时，优先采用接口。
	- 接口可与实现它的类之间不存在任何层次关系，比抽象类更方便灵活。
	- 接口只关心对象之间的交互方法，不关心对象所对应的具体类。
	- 接口是程序之间的一个协议，比抽象类的使用更安全、清晰。

**区别**

83. 接口不是类，不能被实例化，抽象类可以间接实例化。
84. 接口只能做方法声明，抽象类中可以做方法声明，也可以实现方法。
85. 抽象类中可以有优先成员，接口只能包含抽象成员。因此接口是完全抽象，抽象类是部分抽象。
86. 抽象类要被子类继承，接口要被类实现。
87. 抽象类中所有的成员修饰符都能使用，接口中的成员不需要修饰符，都是对外的。
88. 接口可以实现多继承，抽象类只能单继承。
89. 抽象方法要被实现，所以不能是静态的，也不能是私有的。

**使用情形**

- 使用抽象类是为了代码复用，而使用接口是为了实现多态。
- 抽象类适合用来定义某个领域的固有属性，也就是本质，接口适合用来定义某个领域的扩展功能。

#### 反射

反射是指可以在加载程序运行时，动态获取和加载程序集，并可以获取到程序集信息（类、对象、方法等）的一种重要手段。

主要使用的类库：System.Reflection

*核心类：*

90. Assembly：描述程序集。
91. Type：描述类的类型。
92. ConstructorInfo：描述构造函数。
93. MethodInfo：描述所有方法。
94. FieldInfo：描述类的字段。
95. PropertyInfo：描述类的属性。

#### using的作用

在 C# 中，`using` 关键字有几种不同的用途：

96. 命名空间引用：`using` 关键字用于在代码中引入命名空间。这样，您就可以在不使用完全限定名称的情况下使用该命名空间中的类型。例如：

```csharp
using System;
using System.Collections.Generic;
```

97. 创建对象：`using` 关键字可以与 `new` 运算符一起使用，用于在创建对象的同时指定对象的作用域。当离开 `using` 块时，对象会自动被释放。这有助于确保资源的正确释放，例如文件句柄、数据库连接等。例如：

```csharp
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    writer.WriteLine("Hello, world!");
}
```

98. 静态类引用：`using static` 关键字用于引入静态类，以便在不使用类名的情况下直接调用其静态成员。例如：

```csharp
using static System.Math;

double result = Sqrt(16); // 直接调用 Sqrt 方法，无需使用类名
```

总结起来，`using` 关键字在 C# 中主要用于引入命名空间、创建对象并自动释放资源以及引入静态类。这些用法有助于提高代码的可读性和编写效率。

#### sealed关键字的作用

`sealed`关键字在C#中有两种主要用途：

99. 在类声明中使用`sealed`关键字可以阻止其他类从该类继承。这意味着没有其他类可以继承`sealed`修饰的类的行为或状态。
100. 在方法声明中使用`sealed`关键字可以阻止在派生类中对该方法的重写。也就是说，如果一个方法在基类中被标记为`sealed`，那么在任何派生类中都不能重写这个方法。

#### \==和Equals的区别

101. **== 操作符**：这是一个静态的操作符，用于比较两个对象是否有相同的值。如果这两个对象是值类型（如int、float等），`==`操作符会比较它们的值。如果这两个对象是引用类型（如类、接口等），`==`操作符默认比较它们的引用（即，它们是否指向内存中的同一位置），除非该类型重载了`==`操作符。
102. **Equals 方法**：这是一个虚方法，可以在所有的对象上使用。对于引用类型，默认的`Equals`方法（在`Object`类中定义）的行为与`==`操作符相同，都是比较引用。但许多类型（如String、List等）重载了`Equals`方法，用于比较值而非引用。对于值类型，`Equals`方法通常被重载以提供有意义的比较。例如，对于`int`类型，`Equals`方法比较两个整数的值是否相等。

#### 浅拷贝和深拷贝的区别

**浅拷贝的定义是**：只会拷贝基本数据类型的值，以及实例对象的引用地址，并不会复制一份引用地址所指向的对象。也就是说，浅拷贝出来的对象，内部的类属性指向的是同一个对象。

**深拷贝的定义是**：既会拷贝基本数据类型的值，也会针对实例对象的引用地址所指向的对象进行复制。深拷贝出来的对象，内部的属性指向的不是同一个对象，与原对象完全独立。

浅拷贝和深拷贝的主要区别在于拷贝的复杂性和对内存的影响。以下是具体的区别：

103. 拷贝复杂性：浅拷贝只复制对象的**第一层**属性，而深拷贝可以对对象的属性进行**递归**复制。这意味着，如果对象内部还有对象，浅拷贝只会复制内部对象的引用，而深拷贝会递归复制内部对象的所有属性和值。
104. 内存影响：浅拷贝创建新的对象，但里面的元素是原对象中各个子对象的引用，因此**新旧对象还是共享同一块内存**。如果其中一个对象改变了子对象的值，那么另一个对象的子对象值也会改变。深拷贝重新分配一块内存，创建一个新的对象，并且将原对象中的元素，以递归的方式，通过创建新的子对象拷贝到新对象中。因此，**新对象和原对象没有任何关联**，修改新对象不会影响到原对象。

#### 闭包

**闭包是指一个函数能够读取并操作函数之外的另一个函数的变量**。换句话说，闭包是一个函数和对应的引用环境组合起来的实体。

```CS
Func<string, Action<string>> outerFunc = outerVar =>  
{  
    return innerVar => { Console.WriteLine($"{outerVar} {innerVar}"); };  
};  
  
var func = outerFunc("hello");  
func("world");
```

即使外部函数已经执行完毕，但是由于内部函数仍然保持了对外部函数变量的引用，因此外部函数的变量并不会被垃圾回收机制回收，这就是闭包的特性。

# 容器相关
#### 常用的容器类及特点

- Array：数组，内存是连续的，可以通过下标访问元素。优点是访问速度快，缺点是不方便在中间插入元素，需要指定长度。
- ArrayList：动态扩容的数组，可以存储任意类型的数据，其本质上是Object类型的数组。缺点是存取元素涉及装箱拆箱操作，效率不高。
- List：支持泛型和动态扩容的数组，在初始化时必须指定存储数据的类型，比ArrayList更加高效。
- HashSet：集合，和List一样只能存储同一种类型的数据，但其中的元素不会重复并且是无序的，所以不能通过下标访问。
- LinkedList：双向链表，通过连接每个节点来存储数据，并通过节点来访问元素，主要优点是插入和删除操作的时间复杂度都是$O(1)$，并且能保持元素的原有顺序。
- Queue：队列，其中的元素遵循先进先出的规则。添加元素只能在末尾添加，只能访问和移除头部元素。
- Stack：栈，其中的元素遵循先进后出的规则。只能从顶部添加、移除或访问元素。
- Dictionary：字典，用于存储键值对。通过键来访问或修改其对应的值，优点是可以快速查找指定的元素。

#### 常规容器和泛型容器有什么区别

常规容器需要装箱拆箱，性能不及泛型容器，而且泛型容器类型更安全。

#### List自动扩容的过程

List内部使用一个数组来存储元素，如果当前容量不足以存放新的元素，就会触发自动扩容。

105. 添加元素时首先检查数组中是否有剩余的空间，如果有剩余空间则直接添加到末尾。
106. 如果没有剩余空间，会创建一个新的容量为原来两倍的数组，然后将原数组的数据复制到新数组中，最后在末尾添加新元素，将对原数组的引用更新为新数组。

#### ArrayList和List的区别

107. 类型安全性：List是强类型，是类型安全的，只能存储一种类型的对象；ArrayList是弱类型，是非类型安全的，可以存储任意类型的对象。
108. 性能：List通常比ArrayList性能更好，可以使用Sort函数进行排序。
109. 容量：ArrayList有Capacity属性，可以用来设置和获取列表的容量，而List没有。
110. 同步：ArrayList是非同步的，List是同步的。这意味着在多线程环境中，List是线程安全的，而ArrayList不是。
111. 扩展方法：List支持使用LINQ查询，而ArrayList不支持。

#### Dictionary的内部实现原理

**C# Dictionary的内部实现原理是基于哈希表（HashTable）的**。Dictionary是一个泛型集合，它提供了键值对的存储方式。其内部使用哈希表来存储数据，通过键的哈希值来快速查找、插入和删除键值对，因此其查找、插入和删除操作的时间复杂度都近似为O(1)。

哈希表的原理是一种通过**哈希函数**将特定的键映射到特定值的一种数据结构。以下是其具体原理：

112. 对对象元素中的关键字（对象中的特有数据）进行哈希算法的运算，并得出一个具体的算法值，这个值称为哈希值。
113. 哈希值就是这个元素的位置。哈希表本质上是一个数组，哈希值计算出来后，一般会对数组长度进行取模运算，得到一个数组下标，这样元素就可以存放在对应下标的数组空间内。
114. 如果哈希值出现冲突（即不同的关键字对应相同的哈希值），则需要判断这个关键字对应的对象是否相同。如果对象相同，就不存储，因为元素重复。如果对象不同，就存储，在原来对象的哈希值基础+1顺延。
115. 存储哈希值的结构称为哈希表。哈希表能根据哈希值快速找到对应的元素，因此提高了查找效率。

# JIT和AOT的区别

- JIT：Just-In-Time，实时编译。执行慢，安装快，占用空间小。
- AOT：Ahead-Of-Time，提前编译。执行快，安装慢，占用空间大。

**AOT和JIT两种编译方式各有优势，但也存在一些劣势**。

AOT的优势主要在于：

116. **启动速度快**：因为所有代码在程序运行前就已经编译完成，所以程序的启动速度会很快。
117. **性能优化**：AOT编译可以做很多性能优化工作，例如在编译时进行函数内联、静态类型检查等优化操作。

AOT的劣势包括：

118. **编译时间长**：AOT编译需要在程序运行前就完成所有的编译工作，所以编译时间会相对较长。
119. **占用存储空间**：由于AOT编译会将所有代码都编译成机器码，所以编译后的程序会占用更多的存储空间。

JIT的优势主要在于：

120. **更好的动态性**：JIT编译可以在程序运行时根据实际情况进行动态编译，这对于一些动态语言来说有很大的优势。
121. **节省存储空间**：JIT编译只需要在程序运行时编译必要的代码，所以相对来说会节省存储空间。

JIT的劣势包括：

122. **启动速度慢**：由于JIT编译是在程序运行时进行的，所以相对来说程序的启动速度会慢一些。
123. **性能不如AOT**：因为JIT编译是在运行时进行的，所以无法进行像AOT那样的深度优化，性能可能会稍差一些。

# 使用new实例化对象背后的逻辑

124. **分配内存空间**：首先，`new` 操作符会在堆内存中为对象分配足够的内存空间。这个内存空间的大小是根据对象的类型确定的，包括对象中的所有成员变量和内部数据结构。
125. **调用构造函数**：分配内存空间后，`new` 操作符会调用对象的构造函数。构造函数是一个特殊的方法，用于初始化对象的状态。构造函数的名称与对象的类型相同，并且没有返回类型。在构造函数中，可以设置对象的初始状态，例如给成员变量赋值。
126. **返回对象引用**：一旦对象被成功创建并初始化，`new` 操作符会返回一个指向该对象的引用。这个引用是一个指向对象内存地址的指针，可以通过它来访问和操作对象的成员。

# 序列化

**序列化是指将数据结构或对象状态转换为可以存储或传输的格式的过程**。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过反序列化在内存中重新创建该对象。

常见的序列化方式包括**二进制序列化、XML序列化、JSON序列化等**。

我们会在以下情况使用到序列化：

127. 当需要将对象的状态保存到存储媒体中时，如文件、数据库或网络传输，需要序列化。
128. 当需要跨平台交互，如RPC（远程过程调用）或进程间通信时，需要序列化。
129. 当需要将数据进行持久化时，如缓存、保存状态等，需要序列化。