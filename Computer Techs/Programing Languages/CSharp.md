# C#与.NET入门

本章目标：

- 建立开发环境。
- 了解现代.NET、.NET Core、.NET Framework、Mono、Xamarin和.NET Standard之间的异同。

> 现代.NET：.NET 6和前身.NET5（来自.NET Core）。

> 传统.NET：.NET Framework、Mono、Xamarin和.NET标准。

## 设置开发环境

微软提供了一系列的代码编辑器和集成开发环境(IDE)：

- Visual Studio
- Visual Studio Code
- GitHub Codespaces

在学习时，最好使用能够帮助编写代码和配置，但不会隐藏实际使用情况的工具。IDE提供了易用的图形用户界面，它在底层有更接近操作、更基本的代码编辑器同时为编码提供帮助，在学习过程中效果更好。

### Visual Studio

1. 下载Visual Studio安装包并启动。
2. 在Workloads选项卡上，选择以下内容：
    - ASP .NET and web development
    - Azure development
    - .NET desktop development
    - Desktop development with C++
    - Universal Windows Platform development
    - Mobile development with .NET
3. 在Indivisual components选项卡的Code tools部分，选择以下内容：
    - Class Designer
    - Git for Windows
    - PreEmptive Protection - Dotfuscator

### Visual Studio Code

1. 下载并安装Visual Studio Code。
2. 下载并安装.NET SDK 3.1、5.0和6.0。
3. 启动Visual Studio Code。
4. 安装C#和.NET Interactive Notebooks扩展。

其他扩展：MSBuild、REST Client、ILSpy .NET Decompiler、Azure Functions for Visual Studio Code、GitHub Repositorier、SQL Server for Visual Studio Code、Protobuf3。

## 理解.NET

### .NET Framework

.NET Framework包括公共语言运行库（Common Language Runtime，CLR）和基类库（Base Class Library，BCL），前者负责管理代码的执行，后者提供丰富的类库来构建应用程序。

对于.NET Framework 4.0或更新版本，在计算机中，为.NET Framework编写的所有应用程序都共享相同版本的CLR以及存储在全局程序集缓存（Global Assembly Cache，GAC）中的库，如果其中一些应用程序需要特定版本以保证兼容性，就会出问题。

> 实际上，.NET Framework仅适用于Windows，不建议使用它来创建新的应用程序。

### Mono、Xamarin和Unity

Mono是由第三方开发的.NET Framework实现，支持跨平台，但落后于.NET Framework的官方实现。

Mono是Xamarin移动平台以及Unity等跨平台游戏开发平台的基础。

### .NET Core

为了使.NET与Windows分离，以达到跨平台的目的，微软重构并删除了.NET Framework的非核心部分，得到了.NET Core。

.NET Core中包含名为CoreCLR的CLR跨平台实现和名为CoreFX的流畅BCL。

.NET Core的运行速度很快，因为可以与应用程序并行部署，所以.NET Core可以频繁地更改，不会影响到同一台计算机上的其他.NET Core程序。

### .NET的未来版本

.NET Core已重命名为.NET，主版本号跳过了数字4 ，以免和.NET Framework 4.x混淆。

.NET Core 3.1 包含了用于构建Web组件的Blazor服务器。

### .NET支持

.NET版本包含长期支持（LTS）的，以及当前的（Current）。

- LTS版本是稳定的，在其生命周期中很少需要更新。
- Current版本包含可根据反馈进行更改的功能。

.NET在整个生命周期中，都要接受安全性和可靠性方面的关键补丁，必须更新最新的补丁才能获得支持。

#### .NET Runtime和.NET SDK版本

.NET Runtime版本控制遵循语义版本控制，也就是说主版本表示非常大的更改，此版本表示新特性，而补丁版本表示bug的修复。

.NET SDK版本控制不遵循语义版本控制，主版本号和次版本号与匹配的运行时版本绑定。

#### 删除.NET的旧版本

执行以下命令可查看当前安装的SDK和运行时：

- dotnet --list-sdks
- dotnet --list-runtimes

在Windows上可使用App & features部分以删除.NET SDK。

### 现代.NET的区别

与单一的传统.NET Framework相比，现代.NET是模块化、开源的。

1. Windows开发
	  现代.NET的一大特性就是支持使用Windows Desktop Pack运行旧的Windows窗体和WPF应用程序。
2. Web开发
	  旧的Web应用开发和服务技术已经从现代.NET中移除了，取而代之的是ASP.NET Core。
3. 数据库开发
	  Entity Framework 是一种对象-关系映射技术，用于处理存储在关系数据库中的数据。现以被精简，并支持非关系数据库，重命名为Entity Framework Core。

### 现代.NET的主题

微软使用Blazor创建了一个网站，展示[现代.NET的重要主题](https://themesof.net)。

### .NET Standard

2019年.NET的情况是，微软控制着三个.NET平台的分支：

- .NET Core：用于跨平台和新应用程序。
- .NET Framework：用于旧应用程序。
- Xamarin：用于移动应用程序。

以上每种.NET平台各有优缺点，它们都是针对不同场景设计的。这导致了一个问题就是，开发人员必须学习三个.NET平台。

因此微软制定了.NET Standard，一套所有.NET平台都可以实现的API规范，来指示兼容性级别。

在.NET Standard 2.0以及后续版本中，微软已将三个.NET平台融合到现代的最低标准，使开发人员可以更容易地在任何类型的.NET之间共享代码。

### 中间语言

dotnet CLI工具使用的C#编译器（名为Roslyn）会将C#源代码转换成中间语言（Intermediate Language，IL）代码，并将IL存储在程序集（DLL或EXE文件）中。IL代码语句就像汇编语言指令，由.NET的虚拟机CoreCLR执行。

在运行时，CoreCLR从程序集中加载IL代码，再由JIT编译器将IL代码编译成本机CPU指令，最后由机器上的CPU执行。

以上两步编译过程带来的好处是，微软能为Linux、macOS以及Windows创建CLR。在编译过程中，相同的IL代码会到处运行，这将为本地操作系统和CPU指令集生成代码。

### 比较.NET技术

| .NET技术 | 说明 | 驻留的操作系统 |
| :------- | :----: | :-----------: |
| 现代.NET | 现代功能集，完全支持C#8、9、10，支持移植现有应用程序，可用于创建新的桌面、移动和Web应用程序及服务 | Windows、macOS、Linux、Android、iOS |
| .NET Framework | 旧的特性集，提供有限的C#8支持，不支持C#9和10，用于维护现有的应用程序 | 只用于Windows |
| Xamarin | 用于移动和桌面应用程序 | Android、iOS和macOS |

