---
tags:
  - AppDevelopement
  - Computer
date: 2023-09-04 09:37
---

# Android系统架构

#### Linux内核层

Android系统基于Linux内核，这一层为Android设备的各种硬件提供了底层的驱动，如显示驱动、音频驱动、照相机驱动、蓝牙驱动、Wi-Fi驱动、电源管理等。

#### 系统运行库层

这一层通过一些C/C++库位Android系统提供了主要的特性支持。如SQLite库提供数据库支持，OpenGL|ES库提供3D绘图支持，Webkit库提供浏览器内核支持等。

在这一层还有Android运行时库，主要提供一些核心库，允许开发者使用Java语言来编写Android应用。另外，Android运行时库中还包含了Dalvik虚拟机（5.0系统之后改为ART运行环境）,它使得每一个Android应用都能运行在独立的进程中，并且拥有一个自己的虚拟机实例。相较于Java虚拟机，Dalkit和ART都是专门位移动设备定制的，它正对手机内存、CPU性能有限等情况做了优化处理。

#### 应用框架层

这一层主要提供构建应用程序时可能用到的各种API，Android自带的一些核心应用就是使用这些API完成的，开发者可以使用这些API来构建自己的应用程序。

#### 应用层

所有安装在手机上的应用程序都是属于这一层的，比如系统自带的联系人、短信等程序，或者是从Google Play上下载的小游戏，当然还包括你自己开发的程序。

![](https://secure2.wostatic.cn/static/mduKzXkDT5w4EkrbBECnHA/image.png?auth_key=1693790256-bxHs7jmi4p5aJVAJqVwHoj-0-5f851d57508858d8903156ca67acb15e)

# 第一行Android代码

## 搭建开发环境

### 准备所需要的工具

**JDK。** Java语言的软件开发工具包，包含Java的运行环境、工具集合、基础类库等内容。

**Android SDK。** Google提供的Android开发工具包，在开发Android程序时需要通过引入该工具包来使用Android相关的API。

**Android Studio。** Android开发平台。

### 搭建环境

1. 下载并安装Android Studio。
    
2. 配置系统环境变量。
    新建变量ANDROID_SDK_HOME，值为Android SDK的存储路径。
    在Path变量中添加这几个值：%ANDROID_SDK_HOME%，%ANDROID_SDK_HOME%\tools，%ANDROID_SDK_HOME%\platform-tools。
    
3. 修改项目和gradle的存储路径。

## 创建第一个项目

### 新建项目

新建Empty Activity，弹出对话框，配置项目的基本信息。

- Name。项目名称。
- Package name。项目的包名。Android系统通过包名来区分不同的应用程序，因此包名必须要具有唯一性。
- Sava location。项目代码存放的位置。
- Minimun API level。项目的最低兼容版本。

### 启动模拟器

打开AVD Manager，新建一个虚拟设备，此外还要下载好欲安装的Android系统镜像，其余的配置如没有特殊要求保持默认即可。

### 运行程序

在顶部工具栏点击绿色三角按钮即可运行当前的项目，通常app就是当前的主项目。

### 分析第一个Android程序

![Project模式的项目结构](https://secure2.wostatic.cn/static/rBtTHFXEe9cX6i1HkRoKUa/image.png?auth_key=1693790403-ciqfXaUDJpWHGbNL1X7vPu-0-ff8934ea973fd5144dd29639e3800865)

- gradle和.idea
    这两个目录下放置的都是Android Studio自动生成的一些文件，无需关心，也不要去手动编辑。
    
- app
    项目中的代码、资源等内容都是放置在这个目录下的。
    
- build
    这个目录主要包含一些在编译时自动生成的文件，无需过多关心。
    
- gradle
    这个目录下包含了gradle wrapper的配置文件，使用gradle wrapper的方式不需要提前将gradle下载好，而是自动根据本地的缓存情况决定是否联网下载gradle。Android Studio默认就是启用gradle wrapper方式的，如果需要更改成离线模式，可以点击Android Studio导航栏→File→Settings→Build，Execution，Deployment→Gradle，进行配置更改。
    
- .gitignore
    这个文件是用来将指定的目录或文件排除在版本控制之外的。
    
- build.gradle
    这是项目全局的gradle构建版本，通常这个文件中的内容是不需要修改的。
    
- gradle.properties
    这个文件是全局的gradle
    配置文件，在这里配置的属性将会影响到项目中所有的gradle编译脚本。
    
- gradlew和gradlew.bat
    这两个文件是用来在命令行界面中执行gradle命令的，其中gradle是在Linux或Mac系统中使用，gradle.bat是在Windows系统中使用。
    
- local.properties
    这个文件用于指定本机中的Android SDK路径，通常内容是自动生成的，并不需要修改。除非本机中的Android SDK位置发生变化。
    
- settings.gradle
    这个文件用于指定项目中所有引入的模块。
	![app目录下的结构](https://secure2.wostatic.cn/static/wsUPuBS7EjBAGYMWX8RJ6Q/image.png?auth_key=1693790403-67SxiU7uWuHTqZnXSnNqWL-0-4d3329febd5f62e3d23b3e1530ee4fcc)

- build
    与外层build目录类似。
    
- libs
    如果在项目中使用第三方jar包，则需要将jar包放在这个目录下，它们会被添加到项目的构建目录里。
    
- androidTest
    用来编写AndroidTest测试用例，可以对项目进行一些自动化测试。
    
- java
    放置所有Java代码，包括Kotlin代码。
    
- res
    项目中使用到的所有图片、布局、字符串等资源都要存放到这个目录下。
    
- AndroidManifest.xml
    这是整个项目的配置文件，在程序中定义的所有四大组件都需要在这里注册，另外还可以在这个文件中给应用程序添加权限声明。
    
- test
    用来编写Unit Test测试用例，是对项目进行自动化测试的另一种方式。
    
- .gitignore
    这个文件用于将app模块内指定的目录或文件排除在版本控制之外，作用和外层的.gitignore文件类似。
    
- app.iml
    IntelliJ IDEA项目自动生成的文件，不需要关注。
    
- build.gradle
    app模块的gradle构建脚本，这个文件中会指定很多项目构建相关的配置。
    
- [proguard-rules.pro](http://proguard-rules.pro)
    用于指定项目代码的混淆规则，当代码开发完成后打包成安装包文件，如果不希望代码被别人破解，通常会将代码进行混淆，从而让破解者难以阅读。
    

### 项目是如何运作的

首先打开AndroidManifest.xml文件，查看如下代码：

```XML
 <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity> 
```

这段代码表示对MainActivity进行注册，没有在AndroidManifest.xml中注册的Activity是不能使用的。其中intent-filter里的两行代码非常重要，<action android:name="android.intent.action.MAIN" />和<category android:name="android.intent.category.LAUNCHER" />表示MainActivity是这个项目的主Activity，打开程序时首先启动的就是这个Activity。

接下来再看一下MainActivity的代码：

```Kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
    
}
```

MainActivity是继承自AppCompatActivity的。AppCompatActivity是AndroidX中提供的一种向下兼容的Activity，可以使Activity再不同系统版本中的功能保持一致性。而Activity是Android系统提供的一个基类，我们在项目中所有自定义的Activity都必须继承它或它的子类才能拥有Activity的特性（AppCompatActivity是Activity的子类）。

onCreate（）方法是一个Activity被创建时必定要执行的方法。因为Android程序的设计是讲究逻辑和试图分离的，所以我们需要在布局文件中编写界面，然后在Activity中引入进来。第二行的setContentView（）方法就是给当前Activity引入布局文件的。

### 详解项目中的资源

> 以drawable开头的目录用来存放图片

> 以mipmap开头的目录都是用来存放应用图标

> 以values开头的目录用来存放字符串、样式、颜色等配置

> 以layout开头的目录用来存放布局文件

![](https://secure2.wostatic.cn/static/5VwoM9qxAeVm8gWtPN2sAt/image.png?auth_key=1693790403-aL9aJgbzBbBkrfwk8nfAoF-0-06ab0d6a2b9acdc277bee04c7015e6ff)

之所以有那么多mipmap和drawable开头的目录，主要是为了能够更好地兼容各种设备。对于drawable目录，IDE没有帮我们自动生成，但我们应该自己创建drawable-hdpi、drawable-xhdpi、drawable-xxhdpi等目录。在编写程序时，最好能够给同一张图片提供几个不同分辨率的版本分别放在这些目录中。在程序运行时，系统便可以根据当前设备的分辨率选择加载哪个目录下的资源。

当然这只是理想情况，一般我们把图片都放在drawable-xxhdpi目录下就好了，因为这是最主流的设备分辨率目录。

### 如何使用资源

打开res/values/strings.xml文件，内容如下：

```···
<resources>
    <string name="app_name">My Application</string>
</resources>
```

可以看到这里定义了一个应用程序名的字符串，有两种方式来引用它：

**在代码中通过R.string.app_name来获得该字符串的引用。**

**在XML中通过@string/app_name来获得该字符串的引用。**

其中的string可以替换成drawable、mipmap，以此类推。

下面是一个简单的例子，打开AndroidManifest.xml文件：

```···
    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApplication">
        ...
    </application>
```

项目的应用图标就是通过android：icon属性指定的，应用的名称则是通过android：label属性指定的。

### 详解build.gradle文件

不同于Eclipse，Android Studio是采用Gradle来构建项目的。Gradle是一个非常先进的项目构建工具，它使用了一种基于Groovy的领域特定语言（DSL）来进行项目设置，摒弃了传统基于XML（如Ant和Maven）的各种繁琐配置。在项目中有两个build.gradle文件，一个位于最外层，另一个位于app目录下，下面对两个文件中的内容进行详细分析。

**先来看最外层：**

```Groovy
buildscript {
    ext.kotlin_version = "1.5.21"
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:4.2.1"
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
        jcenter() // Warning: this repository is going to shut down soon
    }
}
```

- repositories
    
    两处repositories的闭包中都声明了google（）和jcenter（）这两行配置，它们分别对应了一个代码仓库，google仓库中包含的主要是Google自家的扩展依赖库，而jcenter仓库中包含的大多是一些第三方开源库。声明了这两行配置之后，我们就可以在项目中轻松引用任何google和jcenter仓库中的依赖库了。
    
- dependencies
    
    dependencies闭包中使用classpath声明了两个插件：一个Gradle插件和一个Kotlin插件。为什么要声明Gradle插件呢？因为Gradle并不是专门为构建Android项目而开发的，Java、C++等多种项目都刻有使用Gradle来构建，后面的4.2.1是Gradle的版本号，与Android Studio的版本是对应的。
    

**再来看app目录下的：**

```
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    compileSdkVersion 30
    buildToolsVersion "31.0.0"
    
    defaultConfig {
        applicationId "com.example.myapplication"
        minSdkVersion 21
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"
        
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {

    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.3.1'
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.google.android.material:material:1.2.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.1'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
}
```

- plugins
	  这里应用了两个插件，一般有两种值可选：com.android.application表示这是一个应用程序模块，com.android.library表示这是一个库模块。两者最大的区别在于，应用程序模块是可以直接运行的，而库模块只能最为代码库依附于别的应用程序模块来运行。
	  
- android
	compileSdkVersion用于指定项目的编译版本
	buildToolsVersion用于指定项目构建工具的版本
	
- defaultConfig
	applicationId是每个应用的唯一标识符，绝对不能重复，默认使用我们在创建项目时指定的包名。
    minSdkVersion用于指定项目最低兼容的Android系统版本。
    tragetSdkVersion指定的值表示你在该目标版本上已经做过充分的测试，系统将会为你的应用程序启动一些最新的功能和特性。
    versionCode用于指定项目的版本号。
    versionName用于指定项目的版本名。
    testInstrumentationRunner用于在当前项目中启用JUnit测试，可以为当前项目编写测试用例，以保证功能的正确性和稳定性。
    
- buildTypes
	用于指定生成安装文件的相关配置。
	
	- debug
		用于指定生成测试版安装文件的配置，可忽略不写。
      
	- release
	      minifyEnabled用于指定是否对项目的代码进行混淆。
	      proguardFiles用于指定混淆时使用的规则文件。'proguard-android-optimize.txt' 是在<Android SDK>/tools/proguard目录下的，里面是所有项目通用的混淆规则； 'proguard-rules.pro' 是当前项目的根目录下的，里面可以编写当前项目特有的混淆规则。
      
- dependencies
	  implementation fileTree是本地依赖声明，表示将libs目录下所有.jar后缀的文件都添加到项目的构建路径中。
	  implementation是远程依赖声明，android.appcompat:appcompat:1.2.0就是一个标准的远程依赖库格式，其中android.appcompat是域名部分，appcompat是工程名部分，1.2.0是版本号。加上声明后，Gradle在构建项目时会首先检查本地是否已有这个库的缓存，如果没有则会自动联网下载。

## 掌握日志工具的使用

Andorid中的日志工具类是**Log**（android.util.Log）。这个类提供了5个方法供打印日志。

- **Log.v（）**。打印那些最为琐碎的、意义最小的日志信息。对应级别verbose，在Android日志中级别最低。
- **Log.d（）**。打印一些调试信息，这些信息对你调试程序和分析问题应该是有帮助的。对应级别debug，比verbose高级。
- **Log.i（）**。打印一些比较重要的数据，这些数据应该是你非常想看到的，可以帮助分析用户行为的数据。对应级别info，比debug高级。
- **Log.w（）**。打印一些警告信息，提示程序在这个地方可能会有潜在的风险，最好去修复一下这些出现警告的地方。对应级别warn，比info高级。
- **Log.e（）**。打印程序中的错误信息，比如程序进入了catch语句中。当有错误信息打印出来的时候，一般代表程序出现严重问题了，必须尽快修复。对应级别error，比warn高级。

方法中传入两个参数：

1. **tag**：一般传入当前类名，主要用于过滤信息。
2. **msg**：要打印的具体内容。

在Logcat中可以看到打印信息，可自定义过滤器。




