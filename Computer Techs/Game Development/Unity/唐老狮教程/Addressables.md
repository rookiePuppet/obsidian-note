# 资源加载基础

## 寻址资源设置

将资源设置为可寻址资源的方法：

1. 选中资源后在Inspector窗口勾选Addressables
2. 将资源拖入Addressables Group窗口中

C#文件无法设置为可寻址资源。

原本Resources文件夹中的资源在设置为可寻址资源后，会被移动到一个新的文件夹。

### Addressables Group 窗口

- Group Name/Addressable Name：分组名/可寻址名，可重复
- Path：资源路径，用于定位资源，不可重复
- Labels：标签，可重复，用于资源分类

资源命名注意事项：

- 资源路径一定不能重复，支持名称相同、后缀不同的资源
- 资源名称可以随意修改
- 可以将名称和标签作为双标识来加载资源

#### Profile

可配置打包的目标平台、本地/远程加载路径等信息。

#### Play Mode Script

设置在编辑器模式下运行游戏时，Addressables访问资源的方式。

- Use Asset Database：使用资源数据库，一般在开发阶段使用。使用这种方式不必打包，Addressables会中编辑器的资源数据库中加载资源。
- Use Existing Build：使用已经构建好的资源包，在使用必须打包。

### 创建分组

右键Create New Group可创建分组。

可选择基于某个模板创建分组：

- Packed Asset：默认模板（附带默认的打包加载相关设置）
- Blank（no schema）：空白模板（无默认设置）

一个分组可以作为一个或多个AB包。

## 使用资源标识对象加载

在Mono Behaviour中声明下面这些类型的成员变量，可以在Inspector窗口中使用可寻址资源进行赋值。

- AssetReference：通用资源标识类
- AssetReferenceAtlasSprite：图集资源标识类
- AssetReferenceGameObject：游戏对象标识类
- AssetReferenceSprite：精灵图片标识类
- AssetReferenceTexture：贴图资源标识类
- AssetReferenceTexture2D：
- AssetReferenceTexture3D：
- AssetReferenceT\<T>：指定类型资源标识类

### 加载一般资源

```cs
[SerializeField] private AssetReference assetRef;

// 加载资源  
var asyncHandle = assetRef.LoadAssetAsync<GameObject>();  
// 监听加载完成，使用资源  
asyncHandle.Completed += handle =>  
{  
    if (handle.Status == AsyncOperationStatus.Succeeded)  
    {        
	    Instantiate(handle.Result);  
    }
};
```

### 加载场景资源

加载场景资源使用的API和普通资源不同。

```CS
[SerializeField] private AssetReference sceneRef;

assetRef.LoadSceneAsync().Completed += handle =>  
{  
    // 初始化场景  
};
```

### 释放资源

```CS
assetRef.ReleaseAsset();
```

对于GameObject类型，释放资源的时机一定是在实例化对象之后。

对于其他类型，所加载出来的资源和真正使用的资源是同一对象，释放资源意味着正在使用的资源会丢失。

## 标签的作用

为资源分配标签后，可以指定名称和标签来加载可寻址资源。

标签用于区分资源的种类，例如某种道具有不同的品质（低级/中级/高级），就可以使用标签来区分它们。

使用AssetReferenceUILabelRestriction特性来约束标识类对象。

## 动态加载

### 加载单个资源

使用Addressables的静态函数LoadAssetAsync。

```CS
Addressables.LoadAssetAsync<GameObject>("Red").Completed += handle =>  
{  
    if (handle.Status == AsyncOperationStatus.Succeeded)  
    {        
	    // 使用资源  
    }  
};
```

参数可以传资源名或标签名

如果存在同名、同标签、同类型的资源，Addressables会返回满足条件的第一个资源对象。

对于同名、同标签，但不同类型的资源，可以使用泛型来加载指定类型的资源对象。

### 释放资源

任何资源在释放后，都会影响之前使用该资源的对象。

```CS
// 在使用完资源之后再释放
// 可以写在OnDestroy函数里
Addressables.ReleaseAsset(handle);
```

### 加载场景

```CS
// 和SceneManager.LoadScene的用法相同
Addressables.LoadSceneAsync("Scene", LoadSceneMode.Single, false);
```

### 加载多个资源

```CS
// 第三个参数是对资源搜索结果的合并模式
// None: 不合并，使用第一组结果
// UseFirst：使用第一组结果
// Unio: 合并所有结果
// Intersection: 使用相交结果
Addressables.LoadAssetsAsync<UnityEngine.Object>(new List<string> { "Cube" },  
    obj => { print(obj.name); },  
    Addressables.MergeMode.Intersection);
```