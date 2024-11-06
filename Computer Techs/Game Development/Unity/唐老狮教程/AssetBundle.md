
## AB包理论基础

> AB包是特定于平台的资产压缩包，有点类似压缩文件，资产包括模型、贴图、预设体、音效、材质球等。

### AB包的作用

1. AB包相对于Resources能更好地管理资源。Resources下的资源在打包之后就无法修改，而AB包的存储位置可以自定义，也可以自己选择压缩方式，后期可以动态更新。
2. 减小包体大小
   1. 压缩资源
   2. 减少初始包的大小
3. 热更新
   1. 资源热更新
   2. 脚本热更新

## AB包资源打包

使用官方提供的Asset Bundle Browser工具进行打包。

### Asset Bundle Browser参数相关

#### Build构建页签

- Build Target：目标平台。
- Output Path：目标输出路径。
- Clear Folders：是否清空文件夹，重新打包。
- Copy To StreamingAssets：是否拷贝到StreamingAsssets文件夹。
- Compression：压缩方式。
  - No Compression：不压缩，解压快，包较大。
  - LZMA：压缩体积最小，解压慢，用一个资源就要解压所有资源。
  - LZ4：压缩体积相对LZMA大一点，用什么资源就解压什么资源，内存占用少。
- Exclude Type Infomation：在资源包中不包含资源的类型信息。
- Force Rebuild：重新打包时需要重新构建包，和ClearFolders不同，它不会删除不再存在的包。
- Ignore Type Tree Changes：增量构建检查时，忽略类型数的更改。
- Append Hash：将文件哈希值附加到资源包名上。
- Strict Mode：严格模式，如果打包时报错了，则打包直接失败。
- Dry Run Build：运行时构建。

## AB包资源加载

注意：同一名字的AssetBundle不能被重复加载。

### 同步加载

```c#
//加载AB包
AssetBundle assetBundle = AssetBundle.LoadFromFile(Application.streamingAssetsPath + "/" + "model");
//加载资源
//使用Type指定资源类型进行加载
GameObject cubeObj = assetBundle.LoadAsset("Cube", typeof(GameObject)) as GameObject;
Instantiate(cubeObj);
//使用泛型加载
GameObject sphereObj = assetBundle.LoadAsset<GameObject>("Sphere");
Instantiate(sphereObj);
```

### 异步加载

```c#
IEnumerator LoadABRes(string abName, string resName)
{
    AssetBundleCreateRequest abcr = AssetBundle.LoadFromFileAsync(Application.streamingAssetsPath + "/" + abName);
    yield return abcr;
    AssetBundleRequest abr = abcr.assetBundle.LoadAssetAsync<GameObject>(resName);
    yield return abr;
    Instantiate(abr.asset);
}
```

### 卸载AssetBundle

```c#
//卸载所有AB包，参数表示是否要卸载从AB包中加载出来的资源
AssetBundle.UnloadAllAssetBundles(false);
//卸载指定AB包
asssetBundle.Unload(false);
```

## AB包依赖

如果一个AB包用到了其他AB包中的资源，必须手动加载其他AB包，否则会找不到资源。

利用AB包中的主包可以获取各个AB包的依赖信息（其他AB包名），然后就可以根据依赖信息去加载依赖的AB包了。

```c#
AssetBundle assetBundle = AssetBundle.LoadFromFile(Application.streamingAssetsPath + "/" + "model");
//加载主包
AssetBundle abMain = AssetBundle.LoadFromFile(Application.streamingAssetsPath + "/" + "StandaloneWindows");
//加载主包中的固定文件
AssetBundleManifest manifest = abMain.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
//从固定文件中得到依赖信息
string[] bundleNames = manifest.GetAllDependencies("model");
foreach (var name in bundleNames)
{
    //加载依赖的AB包
    AssetBundle.LoadFromFile(Application.streamingAssetsPath + "/" + name);
}
```

## AB包资源管理器

```c#
public class AssetBundleManager : SingletonAutoMono<AssetBundleManager>
{
    //用于存储已加载AB包的字典
    private Dictionary<string, AssetBundle> dic = new();
    //AB包存储路径
    private string BundlePath => Application.streamingAssetsPath + "/";
    //主包名
    private string MainBundleName
    {
        get
        {
#if UNITY_ANDROID
            return "Android";
#elif UNITY_IOS
            return "IOS";
#else
            return "PC";
#endif
        }
    }
    //主包
    private AssetBundle mainBundle;
    //主包配置文件
    private AssetBundleManifest manifest;

    #region 同步加载
    /// <summary>
    /// 加载AssetBundle
    /// </summary>
    /// <param name="name">包名</param>
    public AssetBundle LoadBundle(string name)
    {
        //加载主包
        if (mainBundle == null)
            mainBundle = AssetBundle.LoadFromFile(BundlePath + MainBundleName);
        //加载主包配置文件
        if (manifest == null)
            manifest = mainBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
        //加载依赖包
        AssetBundle assetBundle = null;
        foreach (string item in manifest.GetAllDependencies(name))
        {
            if (!dic.ContainsKey(item))
            {
                assetBundle = AssetBundle.LoadFromFile(BundlePath + item);
                dic.Add(item, assetBundle);
            }
        }
        //加载目标资源包
        if (!dic.ContainsKey(name))
        {
            assetBundle = AssetBundle.LoadFromFile(BundlePath + name);
            dic.Add(name, assetBundle);
        }
        return assetBundle;
    }

    /// <summary>
    /// 加载指定AssetBundle中的某个资源
    /// </summary>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <returns></returns>
    public Object LoadRes(string bundleName, string resName)
    {
        LoadBundle(bundleName);
        Object obj = dic[bundleName].LoadAsset(resName);
        //如果是GameObject类型，为了方便外部使用，直接在这里实例化
        if (obj is GameObject) return Instantiate(obj);
        return obj;
    }

    /// <summary>
    /// 加载指定AssetBundle中的某个资源
    /// </summary>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <param name="type">资源类型</param>
    /// <returns></returns>
    public Object LoadRes(string bundleName, string resName, System.Type type)
    {
        LoadBundle(bundleName);
        Object obj = dic[bundleName].LoadAsset(resName, type);
        //如果是GameObject类型，为了方便外部使用，直接在这里实例化
        if (obj is GameObject) return Instantiate(obj);
        return obj;
    }

    /// <summary>
    /// 加载指定AssetBundle中的某个资源
    /// </summary>
    /// <typeparam name="T">资源类型</typeparam>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <returns></returns>
    public T LoadRes<T>(string bundleName, string resName) where T : Object
    {
        LoadBundle(bundleName);
        T obj = dic[bundleName].LoadAsset<T>(resName);
        //如果是GameObject类型，为了方便外部使用，直接在这里实例化
        if (obj is GameObject) return Instantiate(obj);
        return obj;
    }

    #endregion

    #region 异步加载
    /// <summary>
    /// 异步加载AssetBundle
    /// </summary>
    /// <param name="name">包名</param>
    /// <param name="callback">AssetBundle加载完成回调函数</param>
    public void LoadBundleAsync(string name, UnityAction<AssetBundle> callback)
    {
        StartCoroutine(ReallyLoadBundleAsync(name, callback));
    }

    private IEnumerator ReallyLoadBundleAsync(string name, UnityAction<AssetBundle> callback)
    {
        //加载主包
        if (mainBundle == null)
        {
            AssetBundleCreateRequest request = AssetBundle.LoadFromFileAsync(BundlePath + MainBundleName);
            yield return request;
            mainBundle = request.assetBundle;
        }
        //加载主包配置文件
        if (manifest == null)
        {
            AssetBundleRequest request = mainBundle.LoadAssetAsync<AssetBundleManifest>("AssetBundleManifest");
            yield return request;
            manifest = request.asset as AssetBundleManifest;
        }
        //加载依赖包  
        foreach (string item in manifest.GetAllDependencies(name))
        {
            if (!dic.ContainsKey(item))
            {
                AssetBundleCreateRequest request = AssetBundle.LoadFromFileAsync(BundlePath + item);
                yield return request;
                dic.Add(item, request.assetBundle);
            }
        }
        //加载目标资源包
        if (!dic.ContainsKey(name))
        {
            AssetBundleCreateRequest request = AssetBundle.LoadFromFileAsync(BundlePath + name);
            yield return request;
            dic.Add(name, request.assetBundle);
            callback?.Invoke(request.assetBundle);
        }
    }

    /// <summary>
    /// 异步加载指定AssetBundle中的某个资源
    /// </summary>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <param name="callback">资源加载完成回调函数</param>
    public void LoadResAsync(string bundleName, string resName, UnityAction<Object> callback)
    {
        StartCoroutine(ReallyLoadResAsync(bundleName, resName, callback));
    }

    private IEnumerator ReallyLoadResAsync(string bundleName, string resName, UnityAction<Object> callback)
    {
        LoadBundle(bundleName);
        AssetBundleRequest request = dic[bundleName].LoadAssetAsync(resName);
        yield return request;
        if (request.asset is GameObject) callback?.Invoke(Instantiate(request.asset));
        callback?.Invoke(request.asset);
    }

    /// <summary>
    /// 异步加载指定AssetBundle中的某个资源
    /// </summary>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <param name="type">资源类型</param>
    /// <param name="callback">资源加载完成回调函数</param>
    public void LoadResAsync(string bundleName, string resName, System.Type type, UnityAction<Object> callback)
    {
        StartCoroutine(ReallyLoadResAsync(bundleName, resName, type, callback));
    }

    private IEnumerator ReallyLoadResAsync(string bundleName, string resName, System.Type type, UnityAction<Object> callback)
    {
        LoadBundle(bundleName);
        AssetBundleRequest request = dic[bundleName].LoadAssetAsync(resName, type);
        yield return request;
        if (request.asset is GameObject) callback?.Invoke(Instantiate(request.asset));
        callback?.Invoke(request.asset);
    }

    /// <summary>
    /// 异步加载指定AssetBundle中的某个资源
    /// </summary>
    /// <typeparam name="T">资源类型</typeparam>
    /// <param name="bundleName">包名</param>
    /// <param name="resName">资源名</param>
    /// <param name="callback">资源加载完成回调函数</param>
    public void LoadResAsync<T>(string bundleName, string resName, UnityAction<T> callback) where T : Object
    {
        StartCoroutine(ReallyLoadResAsync<T>(bundleName, resName, callback));
    }

    private IEnumerator ReallyLoadResAsync<T>(string bundleName, string resName, UnityAction<T> callback) where T : Object
    {
        LoadBundle(bundleName);
        AssetBundleRequest request = dic[bundleName].LoadAssetAsync<T>(resName);
        yield return request;
        if (request.asset is GameObject) callback?.Invoke(Instantiate(request.asset) as T);
        callback?.Invoke(request.asset as T);
    }
    #endregion

    #region 资源卸载
    /// <summary>
    /// 卸载指定AssetBundle
    /// </summary>
    /// <param name="name">包名</param>
    public void Unload(string name)
    {
        if (dic.ContainsKey(name))
        {
            dic[name].Unload(false);
            dic.Remove(name);
        }
    }

    /// <summary>
    /// 异步卸载指定AssetBundle
    /// </summary>
    /// <param name="name">包名</param>
    /// <param name="callback">卸载完成回调函数</param>
    public void UnloadAsync(string name, UnityAction callback)
    {
        StartCoroutine(ReallyUnloadAsync(name, callback));
    }

    private IEnumerator ReallyUnloadAsync(string name, UnityAction callback)
    {
        AssetBundleUnloadOperation operation = dic[name].UnloadAsync(false);
        yield return operation;
        callback?.Invoke();
    }

    /// <summary>
    /// 卸载所有AssetBundle
    /// </summary>
    public void Clear()
    {
        AssetBundle.UnloadAllAssetBundles(false);
        dic.Clear();
    }
    #endregion
}
```

## AB包的上传与下载
