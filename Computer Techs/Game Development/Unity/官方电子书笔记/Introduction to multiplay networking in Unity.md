# 基本概念

在多人游戏中，网络将玩家连接到一个中央服务器或直接相互连接，使他们可以实时共享数据和游玩。

典型的网络游戏架构由两部分组成，即客户端和服务器。

客户端是运行在玩家设备上的游戏实例，负责渲染游戏图像，播放音频，处理玩家输入，以及根据玩家动作发送更新到服务器。

服务器负责管理游戏状态，执行游戏规则，处理客户端之间的通信。服务器可以是由开发者运行的专用的无头机器，也可以是其中一个玩家的设备。

> [!NOTE] 无头服务器
> 指没有图形用户界面，专注于后端任务的服务器。因为它们不渲染图形，更易于扩展，经常部署在专用硬件或云环境中。

## UDP数据包

客户端和服务器之间使用标准网络协议如UDP（User Datagram Protocol），通过交换数据包进行通信。

UDP数据包由**头部**和**负载**（实际数据）两部分组成，负载包含额外的协议特定部分，每一部分也有它们自己的头部和负载，在网络术语中，这种嵌套叫做**封装**。

IP头部和UDP头部的大小固定，包含重要的元数据，如发送方和接收方的地址。而负载部分的大小、结构和内容不一，取决于特定游戏和上下文。

UDP在低延迟和速度方面有优势，但会缺少可靠性。由于UDP协议不提供任何在可靠性、顺序性、拥塞控制方面的机制，数据包在网络中传输时可能会发生以下错误：

- 数据包丢失：数据包有时可能会丢失，永远无法到达，可能是网络拥塞、硬件出错或其他原因导致。
- 重复：同一个数据包可能被接收方多次接收，可能由配置错误的网络硬件或软件导致。
- 重新排序：数据包可能以和发送时不同的顺序到达接收方，可能是因为数据包的传输路径和延迟不同
- 损坏：数据包在传输过程中可能被修改，导致数据不可用

## UDP对比TCP

网络协议是一套规定数据在网络中如何发送和接收的规则和约定，定义了如何建立连接，格式化信息，处理错误和传输数据。

在游戏开发中，UDP因为其快速和轻量的性质，一般更优于TCP（Transmission Control Protocol）。TCP对于网页浏览来说更加可靠适合，因为它能够重发丢失或失序的数据包，从而确保数据顺序性。

UDP为了考虑实时性能，容忍一定的数据丢失，从而获得更高的帧率。UDP也能够在响应性和偶尔的数据丢失之间取得平衡，是多人联机游戏的理想选择。

以下技术能够缓解UDP的不可靠性，和TCP的内置功能不同的是，这些技术可以在实际应用中进行微调。

| 技术               | 描述                                                                    |
| ---------------- | --------------------------------------------------------------------- |
| 序列号              | 每一个数据包都有一个唯一，递增的序列号，接收方利用它来监测数据包丢失和失序                                 |
| 接收确认（ACK）和接收确认掩码 | 数据包中包含最后一次接收到的数据包的序列号，使发送方知道哪些数据已经被成功接收。确认掩码可以同时追踪多个数据包的状态，更快地检测数据包丢失 |
| 超时重传调整（RTO）      | 类似于TCP用于测量数据往返时间的技术，往返时间可以用来调整客户端侧插值或预测                               |
| 超时设定             | 如果在一个特定时间内没有收到确认，则视为数据包丢失                                             |

## Tick和Update

服务器就像单人游戏处理本地玩家输入一样，接收来自客户端的输入，处理全局游戏状态，包括玩家位置、对象状态、物理计算和游戏进程等。

服务器端的核心是**服务器tick**，一个tick指的是服务器基于接收到的输入更新游戏状态的一个周期。tick以一个固定间隔发生，其频率用Hz或tick每秒描述。tick频率决定了游戏世界更新的频率。

update频率指的是客户端和服务器交换数据的频率，更高的频率能提高响应性，但需要更多带宽和算力，一般受限于客户端网络性能和计算资源。创造流畅和响应快速的多人游戏体验的关键在于找到tick频率和update频率之间的一个平衡点。

实际的tick频率根据游戏需求而定。快节奏的FPS一般需要60Hz或更高的tick频率来反映更快速的玩家移动和瞬时射击；实时策略游戏不依赖快速反应，30Hz的tick频率就足够了；大型策略MMO可能使用一个相对较低的10Hz tick频率，以支持海量玩家同时在线。

## 时延

**时延**是指数据从发送源传输到目的地所花费的时间，**往返时间**（RTT）用于度量数据包传输到目的地并返回响应的时间。

一种经验之谈是，当时延为200ms左右时，玩家会注意到游戏体验下降。不同的游戏类型对时延的容忍度不同。

有时候时延是非网络因素导致的，例如玩家输入的延迟或者渲染管线的卡顿。另一个罪魁祸首可能是Vsync，虽然它能防止画面撕裂，但是也会带来额外时延。

大多数时候，网络本身才是时延的主要来源，延迟的类型有：

- 处理延迟：路由器需要花时间读取数据包头，并转发数据包到目的地。这个延迟虽然小，但是会经过很多节点不断累积。
- 传输延迟：将数据包发送到网络上所需的时间，直接受数据包大小影响，在低带宽的网络上更加明显。
- 队列延迟：数据包由于网络拥塞或受限的接口速率而被保留在队列中的延迟。
- 传播延迟：信号在网络之间传输也要花费时间，主要受服务器与用户之间的物理距离，物理媒介和信号类型所影响。

> [!NOTE] 其他网络术语
> - Ping：发送并收回一个基本数据包来测量网络响应性，可将其视为往返时间的简化
> - Jitter：由于网络情况波动引起往返时间变化，造成数据包乱序到达
> - Bandwidth：在一定时间内能够在网络上传输的数据量

## 网络同步

为了保持同步，客户端和服务器需要持续地交换信息来维持所有玩家一致的游戏状态。

客户端一般以高频率（通常是60Hz）给服务器发送用户命令，这些命令可能是动作或输入，例如鼠标或手柄移动或跳跃和射击的按钮按下。服务器收到客户端命令并处理之后，将游戏世界的更新发送回客户端。

需要记住，服务器tick频率、客户端update频率和客户端帧率各自承担不同的功能，不需要保持一致。服务器和客户端一直处于不稳定状态，通过交换连续的动态数据流来减少它们的差异，从而创造出所有客户端在一起游玩的假象。

**状态同步**指的是服务器周期性地将网络对象的状态传输给客户端，更新频率根据游戏类型的需求而定。

**远程程序调用**（RPC）是指远程地调用在服务器或客户端上的函数，用于客户端到服务器之间的通信，例如发送玩家输入，请求特定操作，或触发游戏事件。

**带宽管理**极其影响性能，同步会消耗带宽，可以通过以下策略减少网络数据传输：

- 数据剔除：通过排除非必要的数据更新，专注于游戏必要的数据，来减少网络流量。
- 差分压缩：让服务器只发送相对于上一次更新发生变化的数据，客户端在本地游戏状态中只应用这些差异信息。
- 兴趣管理：依据多个标准对数据同步的优先级进行排序。空间相关性依据物体与玩家的距离及可见性来决定优先级；数据陈旧性则优先处理近期未传输的对象，直至其完成更新；交互相关度重点关注近期与玩家发生过交互或即将产生交互的物体。

## 网络拓扑结构

简单来说，网络拓扑结构指的是设备在多人环境中相互连接和通信的方式。每一种网络模型有其各自的优缺点，要根据游戏类型，期望对游戏状态的控制力，和服务器基础架构的可用资源来决定使用哪一种模型。

拓扑结构会影响游戏的架构、性能和整体的玩家体验。Netcode for GameObjects主要支持两种拓扑结构：**client-server**和**distributed authority**。

### 客户端-服务器（client-server）结构

客户端-服务器拓扑结构是一种常见的网络模型，在客户端设备和一个中央服务器之间划分职责。

客户端代表一个玩家的游戏实例，处理本地输入、渲染和部分游戏状态的模拟，将本地输入发送给服务器并接收更新。

服务器维护着最终明确的游戏世界状态，处理玩家的输入并执行游戏规则。中央服务器需要解决冲突，验证操作，确保一个持续且公平的游戏体验，这种方式有助于防止作弊。

客户端和服务器可以通过互联网或本地局域网（LAN）进行通信。离线LAN游戏通过本地网络进行连接，这种方式具有最低的时延，高安全性和可靠的连接，适用于局域网派对游戏，电子竞技比赛等。

#### 专用游戏服务器

专用游戏服务器是一个独立实体，只处理数据，不作为玩家参与游戏。在处理所有关键的模拟和玩家交互上，具有最高的性能。

在反作弊必要时，专用服务器是必需的，但相应地也会提高通信时延。

专用服务器尤其适合性能敏感型的竞技游戏，如第一人称射击，对于维持公平和减少扰乱行为非常重要。

#### 客户端托管监听服务器

客户端托管的监听服务器同时扮演着服务器和客户端的角色，这样可以降低成本，并且主机无需网络传输，有一定的时延优势。

由于同一个机器既要运行游戏服务器又要生成画面输出，这种方式常常会造成服务器性能低下。另外，因为托管客户端通过家庭网络连接提供服务器支持，这种网络服务提供商一般更注重下载性能，而不是上传性能，所以这种方式比使用位于远程数据中心的专用服务器来说更慢。

### 分布式权限（distributed authority）结构

分布式权限模型将游戏状态的控制和管理权分散给所有参与游戏的客户端，每一个客户端都负责持有、追踪和管理游戏中对象的一部分状态，拥有自主生成和管理这些对象的能力。中央的轻量级服务会监视对象状态的变化，并管理网络传输的路由，但它不对游戏本身进行模拟。

这种结构能够降低成本和输入时延，因为它不需要中央服务器处理所有的游戏操作，每一个客户端都有权力管理它自己的对象，降低了网络往返时间，但是更容易受到作弊的威胁，较不适合需要精确模拟或高竞技性的游戏。

## 网络协议栈

网络协议栈通常指的是软件实现各种通信协议，使它们能够在网络上传输数据。

![[Pasted image 20260210154555.png]]

- 应用层：大多数Unity开发任务发生的地方，Netcode for GameObejcts这些包抽象掉了底层网络开发的复杂性，让开发者可以专注于实现多人联机功能
- 传输层：负责提供可靠的数据传输，错误检测和修正，流程控制和网络设备之间端到端的通信，促进数据包分段和重组，提供错误恢复和数据完整性方面的机制。
- 网络层：负责在不同网络上的联网设备之间路由数据包，依赖网络基础设施和协议。
- 数据链路和物理层：在网络媒介上直接处理数据包的物理传输，一般由操作系统和网络硬件来负责处理

Unity开发者主要在应用层上实现多人联机功能，不需要担心底层，因此可以将网络协议栈简化成如下这样：

![[Pasted image 20260210155954.png]]

# Unity联机解决方案

Unity为各种类型和规模的多人游戏开发提供了综合性的方案，不同类型的游戏有不同的联机需求，休闲游戏一般优先考虑简洁性和成本效益，而竞技游戏需要精确和稳定的网络管理来保证公平性和响应性。

## Netcode for GameObjects

对于休闲合作类型的多人游戏，推荐使用Netcode for GameObjects，这个包通过抽象联网逻辑来简化多人游戏开发，使管理所有玩家的游戏状态更加容易。

Netcode for GameObjects包含以下关键功能：

- NetworkObject：表示网络中应该被同步的任何对象，处理对象的生成、销毁和所有权
- NetworkBeaviour：拥有联网能力的MonoBehaviour，提供处理网络事件的内置回调，让你可以编写服务器侧和客户端侧的代码
- Remote procedure calls(RPCs)：在GameObject远程实例上发送消息和调用方法
- NetworkVariable：用于在网络上同步状态
- NetworkManager：管理网络状态，处理连接和场景管理等任务的中心组件

## Netcode for Entities

Netcode for Entities构建于DOTS(Data-Oriented Technology Stack)和ECS(Entity Component System)之上，为服务器权威的游戏而设计，包含客户端侧预测、插值和延迟补偿等高级特性。

| 特性        | Netcode for GameObjects                                              | Netcode for Entities                    |
| --------- | -------------------------------------------------------------------- | --------------------------------------- |
| 目标群体      | 入门和中级开发者                                                             | 高级开发者                                   |
| 架构        | 面向对象（基于Mono Behaviour）                                               | 面向数据（ECS和DOTS）                          |
| 性能        | 适合小规模游戏                                                              | 为高性能和扩展性优化                              |
| 可扩展性      | 有限，适合少量玩家的游戏                                                         | 高扩展性，适合大规模游戏                            |
| 联网功能      | NetworkVariables，RPCs，NetworkTransform，有限的客户端侧预测、插值，支持UnityTransport | 完整的客户端侧预测、插值、延迟补偿功能，支持更优的UnityTransport |
| Unity服务继承 | 完全支持Lobby、Relay等                                                     | 完全支持Lobby、Relay等                        |

## Unity Transport

Unity Transport是一个与网络代码无关的包，提供底层网络层，专注于性能和可靠性，通过高级特性扩展了传统UDP。

- 可靠性：在UDP上增加可靠的通信功能，确保重要的消息能成功送达，而无需TCP那样的开销
- 安全性：整合加密和认证机制，防止数据被未经授权者访问
- 性能：针对低时延和高吞吐量进行了优化
- 跨平台兼容性：为跨平台和跨设备无缝运行和协同而设计

Netcode for GameObjects和Netcode for Entities都依赖于Unity Transport，如果开发者需要对网络更精细的控制，也可以将Unity Transport作为一个独立的库使用，在其之上构建自定义的网络代码。

> [!NOTE] 第三方联网库
> - Photon Unity Networkng(PUN)
> - Mirror
> - DarkRift Networking 2
> - Forge Networking Remastered

# 配置Netcode工程

## 安装Netcode for GameObjects

在Package Manager安装以下包：

- Netcode for GameObjects
- Multiplayer Tools Window：Unity6引入的5个改善多人游戏开发工作流的新工具
	- Multiplayer Tools Window：在同一个地方提供对所有多人游戏工具和文档的访问入口
	- Network Simulator：模拟真实的网络环境，例如数据包延迟、丢失和断连
	- Runtime Network Stats Monitor(RNSM)：展示实时的网络数据，提供可配置的网络性能屏上监控
	- Network Scene Visualization：在场景视图中可视化展示网络活动和对象所有权，增强调试能力
	- Hierarchy Network Debug view：在Hierarchy窗口右边显示网络对象标识
- Multiplayer Play Mode：用于模拟最多4个玩家（主编辑器玩家+3个虚拟玩家）

## 添加NetworkManager

每一个工程都需要一个NetworkManager组件来支持多人联网，这个组件负责管理网络状态，处理连接和网络配置。

在NetworkManager组件中，配置Network Transport层为Unity Transport，然后编辑器会自动添加一个Unity Transport组件到同一个GameObject上。

传输层负责底层网络任务，例如连接管理，数据传输和数据包加密。Unity Transport组件可以模拟网络环境，如时延、丢包和抖动。

## NetworkObjects

所有需要联网或在不同客户端之间同步的GameObject都需要NetworkObject组件，当一个GameObject拥有NetworkObject，它就变成“可联网的”了，也就是说它的状态和行为可以在网络中被共享和更新。

每一个NetworkObject都有这几个标识符：

- GlobalObjectIdHash：标识工程中的预制体资产
- NetwrokObjectId：相同预制体的不同实例的唯一标识符
- OwnerClientId：标识“拥有”该对象的客户端

标识符帮助NetworkManager追踪NetworkObject，确保其状态在所有连接的客户端之间保持一致。

NetworkObject可以在游戏中动态生成和销毁，生成的NetworkObject会在所有已连接的客户端上出现，每一个NetworkObject都有一个拥有者，一般是控制其行为和状态的客户端。

## 玩家NetworkObject

每一个玩家可以选择性地拥有一个叫做玩家NetworkObject的预制体，这是一个特殊的NetworkObject，一般包含玩家控制器和玩家在游戏中的可视表示。

玩家NetworkObject一般会存储并同步玩家特定的数据，例如玩家名字、分数、背包等其他相关信息，这些数据会在网络中被同步，确保所有已连接的玩家能看到一致的游戏状态。

当一个客户端连接时，NetworkManager会创建一个被该玩家“持有”的玩家NetworkObject，这意味着该玩家拥有控制其自身的权力。

玩家对象可能需要添加以下组件：

- NetworkObject：包含和生成、销毁和所有权相关的属性和事件，每一个联网对象都需要
- NetworkBehaviours：在MonoBehaviour基础上添加联网行为，包含网络变量，远程调用和网络回调
- NetworkAnimators：同步动画状态和参数
- NetworkTransform：同步位置、旋转和缩放

玩家NetworkObject一般负责处理玩家输入，然后按需传递给其他连接的玩家。

玩家逻辑由和游戏机制直接相关的MonoBehaviour与管理网络状态的NetworkBehaviour组成，非联网的组件在每一个玩家的本地实例中运行，本地使用这些组件不仅能优化性能，还可以降低网络流量。

## 创建一个玩家NetworkObject

首先在玩家预制体上添加一个NetworkObject组件，然后将该预制体注册到NetworkManager的Player Prefab。

进入Play Mode后，在NetworkManager的Inspector上选择Start Host，会自动生成玩家NetworkObejct，退出Play Mode后会自动销毁。

## 多人Play Mode

测试多人需要在分离的进程上运行程序，在之前这意味着要构建应用，然后和编辑器一起运行。

Unity6引入了Multiplay Play Mode（MPPM），它可以同时开启多个Unity编辑器实例，模拟多人环境，简化了多人测试流程。

从Window>Multiplayer Play Mode处打开，勾选额外的虚拟玩家。进入Play Mode后，第二个应用的进程会出现在另外一个窗口中，使用Layout按钮启用Inspector和Hierarchy。

使用Scene视图中的Network Visualization面板可以更清晰地区分服务器和客户端实例，实例会被根据带宽（多少数据正在被传输）和所有权（哪一个客户端拥有该对象的控制权）进行上色。

## 添加NetworkBehaviour

为了管理玩家预制体上的MonoBehaviour，我们可以使用一个NetworkBehaviour。NetworkBehaivour是专为联网逻辑设计的特殊MonoBehaviour，提供在不同游戏客户端之间同步操作和状态的必要框架。

NetworkBehaviour共享MonoBehaviour的生命周期，并加入特定的网络功能：

- **RPC方法**：利用远程调用处理网络通信，这些方法被`Rpc`特性标记，通过使用`[Rpc(SendTo.Server)]`和`[Rpc(SendTo.Client)`来向服务器或客户端发送RPC。
- **NetworkVariable**：专门为管理网络同步状态设计的变量，在服务器上对NetworkVariable的修改会自动传递给所有客户端
- **OnNetworkSpawn**和**OnNetworkDespwan**：当NetworkBehaviour被实例化或销毁时触发的生命周期函数
- **Ownership（所有权）**：NetworkBehaviour允许特定的客户端（或服务器）持有对特定对象的所有权，这种权力指的是客户端或服务器能“持有”一个NetworkObject，确保只有指定的玩家能够控制特定的对象或与其交互

我们可以实现一个叫做ClientPlayerMode的NetworkBehaviour来管理玩家移动，这可以确保从主机和服务器的输入只在它们相对应的玩家对象上有效，例如：

```CS
using Unity.Netcode;
using StarterAssets;
using UnityEngine;
using UnityEngine.InputSystem;

namespace NetcodeDemo
{
	public class ClientPlayerMove: NetworkBehaviour
	{
		[SerializeField]
		CharacterController m_CharacterController;
		[SerializeField]
		ThirdPersonController m_ThirdPersonController;
		[SerializeField]
		PlayerInput m_PlayerInput;
		[SerializeField]
		Transform m_CameraFollow;
		
		private void Awake()
		{
			m_PlayerInput.enabled = false;
			m_ThirdPersonController.enabled = false;
			m_CharacterController.enabled = false;
		}
		
		public override void OnNetworkSpawn()
		{
			base.OnNetworkSpawn();
			enabled = IsClient; // Enable if this is a client.
			if (!IsOwner)
			{
				// Disable if this is not the owner
				enabled = false;
				m_PlayerInput.enabled = false;
				m_CharacterController.enabled = false;
				m_ThirdPersonController.enabled = false;
				return;
			}
			// Enable if this is an owner
			m_PlayerInput.enabled = true;
			m_CharacterController.enabled = true;
			m_ThirdPersonController.enabled = true;
		}
	} 
}
```

> [!NOTE] 权力和所有权属性
> 虽然已连接和被准许的客户端可以使用`SpawnWithOwnership`方法持有NetworkObject，但是在默认情况下是服务器持有。Netcode for GameObjects是服务器权威的，意味着只有服务器有权利生成和销毁NetworkObject。
> NetworkBehaviour包含一些属性来确定实例的权力和所有权：
> - IsClient：表示实例是否在客户端上运行
> - IsServer：表示实例是否在服务器上运行
> - IsHost：表示实例是否在主机上运行，主机既是服务器也是客户端
> - IsLocalPlayer：表示相关的NetworkObject是否为本地玩家对象
> - IsOwner：表示本地玩家是否持有该对象或该对象是否为本地玩家对象
> - IsPlayerObject：表示该GameObject是否表示一个网络玩家（被一个特定的客户端控制）
> - IsSceneObject：表示该GameObject是否默认为场景的一部分，不是在游戏期间动态生成的。场景对象通常由服务器管理。


