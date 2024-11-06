# C#调用Lua

## Lua解析器

```c#
using XLua;

//Lua解析器，能够让我们在Unity中执行Lua脚本，一般情况下要保证它的唯一性
LuaEnv env = new LuaEnv();

//执行Lua语言
env.DoString("print('你好，世界')");

//执行Lua脚本
//默认Lua脚本路径是Resources，估计是通过Resources.Load来加载Lua脚本
//所以Lua脚本后缀必须加上.txt才能被找到
env.DoString("require('Main')");

//清除Lua中没有手动释放的对象，垃圾回收
//帧更新中定时执行或切场景时执行
env.Tick();

//销毁Lua解析器
env.Dispose();
```

## Lua文件加载重定向

```c#
void Start()
{
    LuaEnv env = new LuaEnv();
    //xLua提供一个路径重定向的方法允许我们自定义加载Lua文件的规则
    //当我们执行Lua的require函数时相当于执行一个Lua脚本，它就会执行我们自定义传入的这个函数
    env.AddLoader(MyCustomLoader);
    env.DoString("require('Main')");
}

private byte[] MyCustomLoader(ref string filePath) {
    //通过函数中的逻辑加载Lua文件，传入的参数是require执行的Lua脚本文件名
    string path = Application.dataPath + "/Lua/" + filePath + ".lua";
    if(File.Exists(path)) {
        return File.ReadAllBytes(path);
    } else {
        Debug.Log("MyCustomLoader重定向失败，文件名：" + filePath);
        return null;
    }
}
```

## Lua解析管理器

```c#
/// <summary>
/// Lua解析管理器
/// 构建唯一的Lua解析器，提供操作Lua相关的方法
/// </summary>
public class LuaManager : BaseManager<LuaManager>
{

    private LuaEnv luaEnv;

    /// <summary>
    /// Lua中的_G
    /// </summary>
    public LuaTable Global => luaEnv.Global;

    /// <summary>
    /// 初始化Lua解析器
    /// </summary>
    public void Init()
    {
        if (luaEnv != null) return;
        luaEnv = new LuaEnv();
        luaEnv.AddLoader(MyCustomLoader);
        luaEnv.AddLoader(MyCustomABLoader);
    }

    /// <summary>
    /// 执行Lua代码
    /// </summary>
    /// <param name="code"></param>
    public void DoString(string code)
    {
        if (luaEnv == null) Debug.Log("LuaManager:解析器未初始化");
        luaEnv?.DoString(code);
    }

    /// <summary>
    /// 执行Lua脚本
    /// </summary>
    /// <param name="fileName"></param>
    public void DoLuaFile(string fileName)
    {
        DoString($"require('{fileName}')");
    }

    /// <summary>
    /// 回收Lua垃圾
    /// </summary>
    public void Tick()
    {
        if (luaEnv == null) Debug.Log("LuaManager:解析器未初始化");
        luaEnv?.Tick();
    }

    /// <summary>
    /// 销毁Lua解析器
    /// </summary>
    public void Dispose()
    {
        if (luaEnv == null) Debug.Log("LuaManager:解析器未初始化");
        luaEnv?.Dispose();
        luaEnv = null;
    }

    /// <summary>
    /// Lua文件加载重定向
    /// </summary>
    /// <param name="filePath"></param>
    /// <returns></returns>
    private byte[] MyCustomLoader(ref string filePath)
    {
        //通过函数中的逻辑加载Lua文件，传入的参数是require执行的Lua脚本文件名
        string path = Application.dataPath + "/Lua/" + filePath + ".lua";
        if (File.Exists(path))
        {
            return File.ReadAllBytes(path);
        }
        else
        {
            Debug.Log("MyCustomLoader重定向失败，文件名：" + filePath);
            return null;
        }
    }

    /// <summary>
    /// AB包中Lua文件加载重定向
    /// </summary>
    /// <param name="filePath"></param>
    /// <returns></returns>
    private byte[] MyCustomABLoader(ref string filePath)
    {
        TextAsset text = AssetBundleManager.Instance.LoadRes<TextAsset>("lua", filePath);
        if (text != null) return text.bytes;
        else Debug.Log("MyCustomABLoader重定向失败：" + filePath);
        return null;
    }
}

```

## 全局变量获取

#### 获取Lua全局变量

`Global.Get<int>("testNumber")`

！Lua中的局部变量无法获取。

#### 修改Lua全局变量

`Global.Set("testNumber", 10)`

```c#
LuaManager.Instance.Init();
LuaManager.Instance.DoLuaFile("Main");

LuaManager.Instance.Global.Set("testNumber", 10);
int a = LuaManager.Instance.Global.Get<int>("testNumber");
float b = LuaManager.Instance.Global.Get<float>("testFloat");
bool c = LuaManager.Instance.Global.Get<bool>("testBool");
string d = LuaManager.Instance.Global.Get<string>("testString");
Debug.Log(string.Format("a:{0}, b:{1}, c:{2}, d:{3}", a, b, c, d));

```

## 全局函数获取

#### 通过委托获取

```c#
Action action = LuaManager.Instance.Global.Get<Action>("testFunc1");
action();

//非无参无返回的委托需要加[CSharpCallLua]特性
[CSharpCallLua]
public delegate int CustomCall2(int a);

```

#### 通过xLua提供的方式获取

```c#
LuaFunction luaFunction = LuaManager.Instance.Global.Get<LuaFunction>("testFunc1");
luaFunction.Call();
```

### 不同函数类型的获取

#### 无参无返回

```c#
Action action = LuaManager.Instance.Global.Get<Action>("testFunc1");
action();
```

#### 有参有返回

```c#
Func<int, int> func = LuaManager.Instance.Global.Get<Func<int, int>>("testFunc2");
int res = func(10);
```

#### 多返回值

```c#
//多返回值使用ref和out来接收
[CSharpCallLua]
public delegate void CustomCall(int a, out int b, out float c, out string d, out bool e);
  
CustomCall customCall = LuaManager.Instance.Global.Get<CustomCall>("testFunc3");
int b;
float c;
string d;
bool e;
customCall(5, out b, out c, out d, out e);
```

#### 变长参数

```c#
//变长参数类型是根据实际情况来定的
[CSharpCallLua]
public delegate void CustomCall(string a, params int[] args);

CustomCall customCall = LuaManager.Instance.Global.Get<CustomCall>("testFunc4");
customCall("puppet", 20, 51, 161);

```

## 映射到List和Dictionary

```c#
/*List，一般用来映射没有自定义索引的表*/
List<int> list1 = LuaManager.Instance.Global.Get<List<int>>("testList1");
foreach (var item in list1)
{
    Debug.Log(item);
}
//不确定类型的用object
List<object> list2 = LuaManager.Instance.Global.Get<List<object>>("testList2");
foreach (var item in list2)
{
    Debug.Log(item);
}

/*Dictionay，一般用来映射自定义索引的表*/
Dictionary<int, string> dic1 = LuaManager.Instance.Global.Get<Dictionary<int, string>>("testDic1");
foreach (KeyValuePair<int, string> pair in dic1)
{
    Debug.Log($"{pair.Key}, {pair.Value}");
}
//不确定类型的用object
Dictionary<object, object> dic2 = LuaManager.Instance.Global.Get<Dictionary<object, object>>("testDic2");
foreach (var item in dic2)
{
    Debug.Log($"{item.Key}, {item.Value}");
}
```

## 映射到类

```c#
//自定义类，成员名必须和Lua中一致，且必须是public，但数量可以少或多，可以嵌套类
public class CallLuaTestClass
{
    public int testInt;
    public string testString;
    public bool testBool;
    public Action testFunc;
    public void TestFunc()
    {
        testFunc?.Invoke();
    }
}

CallLuaTestClass testClass = LuaManager.Instance.Global.Get<CallLuaTestClass>("testClass");
Debug.Log(testClass.testInt);
Debug.Log(testClass.testString);
Debug.Log(testClass.testBool);
testClass.TestFunc();

```

## 映射到接口

```c#
//自定义接口，需要加[CSharpCallLua]特性，接口中使用成员属性
[CSharpCallLua]
public interface ICallLuaTestInterface
{
    int testInt { get; set; }
    string testString { get; set; }
    bool testBool { get; set; }
    Action testFunc { get; set; }
}

ICallLuaTestInterface test = LuaManager.Instance.Global.Get<ICallLuaTestInterface>("testClass");
Debug.Log(test.testInt);
Debug.Log(test.testString);
Debug.Log(test.testBool);
test.testFunc();
//接口映射是引用拷贝，在C#中修改接口变量相当于直接修改Lua中的变量
test.testBool = false;
ICallLuaTestInterface test1 = LuaManager.Instance.Global.Get<ICallLuaTestInterface>("testClass");
Debug.Log(test1.testBool);

```

## 映射到LuaTable

```c#
//引用拷贝，在C#中改table里的值会直接修改Lua中的值
LuaTable table = LuaManager.Instance.Global.Get<LuaTable>("testClass");
Debug.Log(table.Get<int>("testInt"));
Debug.Log(table.Get<bool>("testBool"));
Debug.Log(table.Get<string>("testString"));

table.Get<LuaFunction>("testFun").Call();
table.Set("testInt", 55)
//使用完要执行Dispose
table.Dispose();
```

# Lua调用C\#

## 类

```lua
-- 通过CS.NameSpace.ClassName获取C#中的类
-- 因为Lua中没有new，所以直接使用类名加括号的形式实例化对象
-- 无参构造
local obj = CS.UnityEngine.GameObject()
-- 有参构造
local obj2 = CS.UnityEngine.GameObject("puppet")

--为了方便使用并且节约性能，可以定义全局变量存储C#类
GameObject = CS.UnityEngine.GameObject
--类中的静态成员可以直接.出来使用
local puppetObj = GameObject.Find("puppet")

-- 如果要使用对象中的成员方法，一定要加:
Vector3 = CS.UnityEngine.Vector3
puppetObj.transform:Translate(Vector3.right)

-- 继承了Mono的类不能直接实例化，而是通过GameObject的AddComponent方法来添加脚本
-- 因为Lua不支持无参泛型函数，所以要使用AddComponent的另一个重载
-- xLua提供了一个typeof方法可以得到类的Type
puppetObj:AddComponent(typeof(CS.UnityEngine.Rigidbody))

```

## 枚举

```lua
-- 枚举的调用规则和类是一样的
-- CS.NameSpace.EnumType.EnumName
PrimitiveType = CS.UnityEngine.PrimitiveType
GameObject = CS.UnityEngine.GameObject

local obj = GameObject.CreatePrimitive(PremitiveType.Cube)

-- 数值转枚举
local a = PrimitiveType.__CastFrom(1)
-- 字符串转枚举
local b = PrimitiveType.__CastFrom("Cube")

```

## 基本数据结构

### Array

```lua
-- 在Lua中获取的C#数组不能用#来获取长度
local obj = CS.MyClass()
print(obj.array.Length)
print(obj.array[0])

-- 遍历数组：虽然Lua中索引从1开始，但遍历C#数组还是从0开始
for i = 0, obj.array.Length - 1 do
    print(obj.array[i])
end

-- 在Lua中创建一个C#数组，使用Array类中的CreateInstance方法
local array = CS.System.Array.CreateInstance(typeof(CS.System.Int32), 10)
print(array.Length)
```

### List

```lua
obj.list:Add(1)
obj.list:Add(2)
obj.list:Add(3)
--长度
print(obj.list.Count)
--遍历
for i = 0, obj.list.Count-1 do
    print(obj.list[i])
end

-- 在Lua中创建List对象
-- 老版本
local list = CS.System.Collections.Generic["List`1[System.String]"]()
-- 新版本 >2.1.12
local List_String = CS.System.Collections.Generic.List(Cs.System.String)
local list = List_String()

```

### Dictionary

```lua
obj.dic:Add(1, "123")
-- 遍历
for k, v in pairs(obj.dic) do
    print(k, v)
end

-- 在Lua中创建一个字典对象
local Dic_String_Vector3 = CS.System.Collections.Generic.Dictionary(CS.System.String, CS.UnityEngine.Vector3)
local dic = Dic_String_Vector3()
dic:Add("123", CS.UnityEngine.Vector3.right)
for i, v in pairs(dic) do
    print(i, v)
end

-- 在Lua中创建的字典，不能直接通过[]的形式获取值
-- 如果要通过键获取值，要使用这种固定方法
print(dic:get_Item("123"))
dic:set_Item("123", nil)
-- 还可以使用TryGetValue方法，该方法有两个返回值
print(dic:TryGetValue("123"))

```

## 函数

建议要在Lua中使用的类都加上\[LuaCallCSharp]特性，可以提升性能。

### 拓展方法

```c#
//想要在Lua中调用类的拓展方法，一定要为静态类加上特性
[LuaCallCSharp]
public static class Tools
{
    public static void Move(this MyClass obj)
    {
        Debug.Log(obj.name + "移动")
    }
}
```

```lua
MyClass = CS.MyClass

-- 调用静态方法
MyClass.Eat()

-- 调用成员方法，需要先实例化
local obj = MyClass()
obj:Speak("哈哈哈哈")

-- 调用拓展方法和调用成员方法一致
obj:Move()
```

### ref和out

```c#
public int RefFun(int a, ref int b, ref int c, int d)
{
    b = a + d;
    c = a - d;
}

public int OutFun(int a, out int b, out int c, int d)
{
    b = a;
    c = d;
    return 200;
}

public int RefOutFun(int a, out int b, ref int c)
{
    b = a * 10;
    c = a * 20;
    return 300;
}
```

```lua
local obj = MyClass()

--ref参数以多返回值的形式返回给Lua
--如果函数存在返回值，那么第一个值就是该返回值
--之后的返回值，就是ref的结果，从左到右一一对应
--ref参数需要传入一个默认值占位置
--a相当于函数返回值，b对应第一个ref，c对应第二个ref
local a, b, c = obj:RefFun(1, 0, 0, 1)

--out参数会以多返回值的形式返回给Lua
--如果函数存在返回值，那么第一个值就是该返回值
--之后的返回值，就是out的结果，从左到右一一对应
--out参数不需要传入占位置的值
local a, b, c = obj:OutFun(20, 30)

--混合使用时，综合上面的规则
--ref需占位，out不用传
--第一个是函数的返回值，之后从左到右依次对应ref或out参数
local a, b, c = obj:RefOutFun(20, 1)

```

### 重载函数

```c#
public void int Calc(int a, int b);
public void int Calc(int a);
public void CalC(float a);
```

```lua
local obj = MyClass()

--虽然Lua不支持函数重载，但是支持调用C#中的重载函数
pirnt(obj:Calc())
print(obj:Calc(15, 1);

--因为Lua中的数值只有Number类型，对于C#中各种精度的数值类型无法分清
print(obj:Calc(10))
print(obj:Calc(10.2)) --0

--解决方法：通过反射机制
--该方法只作了解，尽量不使用
local m1 = typeof(CS.MyClass):GetMethod("Calc", typeof(CS.System.Int32))
local m2 = typeof(CS.MyClass):GetMethod("Calc", typeof(CS.System.Single))
--通过xLua提供的一个方法，把它转成Lua函数来使用
local f1 = xlua.tofunction(m1)
local f2 = xlua.tofunction(m2)
--成员方法第一个参数传对象，静态方法不用传对象
print(f1(obj, 10))
print(f2(obj, 10.2))
```

## 委托与事件

#### 委托

```lua
local obj = CS.MyClass()
--先声明一个函数
local fun = function()
    print("Lua函数Fun")
end

--第一次往委托里加函数不能直接+，要用=
obj.del = fun
--后面才可以用+
obj.del = obj.del + fun
--不建议使用这样的匿名函数
obj.del = obj.del + function()
    print("临时声明的函数")
end

--委托执行
obj.del()
--减函数
obj.del = obj.del - fun
--清空委托
ojb.del = nil
```

#### 事件

```lua
local fun = function()
    print("事件加的函数")
end

--事件中的函数加减和委托不一样
obj:eventAction("+", fun)
obj:eventAction("-", fun)

--事件执行
obj:DoEvent()

--清空事件不能直接置空
--可以在C#类中写一个清空事件的方法，在这里调用
obj:ClearEvent()
```

## 特殊问题

### 二维数组

```lua
local obj = CS.MyClass()

--获取长度
obj.array:GetLength(0)
obj.array:GetLength(1)

--获取元素
--不能使用[]的方式来访问元素
obj.array:GetValue(0, 0)

--遍历
for i = 0, obj.array:GetLength(0)-1 do
    for j = 0, obj.array:GetLength(1)-1 do
        print(obj.array:GetValue(i, j))
    end
end
```

### nil和null

```lua
local obj = GameObject()
local rig = obj:GetComponent(typeof(Rigidbody))

--判空
--第一种方法
if rig:Equals(nil) then
    rig = obj:AddComponent(typeof(Rigidbody))
end

--第二种方法
--Lua全局函数
function IsNull(obj)
    if obj == nil or obj:Equals(nil) then
        return true
    end
    return false
end

if IsNull(rig) then
    rig = obj:AddComponent(typeof(Rigidbody))
end

```

```c#
//第三种方法
//在C#中为Object写拓展方法
[LuaCallCSharp]
public static class Tools
{
    public static bool IsNull(this Object obj)
    {
        return obj == null;
    }
}

//Lua：
if rig:IsNull() then
    rig = obj:AddComponent(typeof(Rigidbody))
end

```

## 让系统类型和Lua能相互访问

```c#

public static class MyClass
{
    [CSharpCallLua]
    public static List<Type> csharpCallLuaList = new List<Type>(){
        typeof(UnityAction<float>)
    };
     
    [LuaCallCSharp]
    public static List<Type> luaCallCsharpList = new List<Type>(){
        typeof(GameObject)
    };
}
```

## 协程

```lua
--xLua提供了一个工具表util
util = require("xlua.util")

GameObject = CS.UnityEngine.GameObject
WaitForSeconds = CS.UnityEngine.WaitForSeconds
--在场景中新建一个空物体，然后挂载脚本上去
local obj = GameObject("Coroutine")
local mono = obj:AddComponent(typeof(CS.LuaCallCSharp))

--定义协程函数
fun = function()
    local a = 1
    while true do
        --Lua中不能直接使用C#中的yield return
        coroutine.yield(WaitForSeconds(1))
        print(a)
        a =  a + 1
        if a > 10 then
            mono:StopCoroutine(b)
        end
    end
end

--不能直接将Lua中的函数传入StartCoroutine函数
--必须先调用xlua.util中的cs_generator(lua函数)
b = mono:StartCoroutine(util.cs_generator(fun))
```

## 泛型函数

```c#
public class MyClass
{
    public interface ITest {}
    public class TestFather {}
    public class TestChild {}
    public void TestFun1<T>(T a , T b) where T: TestFather;
    public void TestFun2<T>(T a);
    public void TestFun3<T>() Where T: TestFather;
    public void TestFun4<T>(T a) where T: ITest;
}
```

```lua
local obj = CS.MyClass
local child = CS.MyClass.TestChild()
local father = CS.MyClass.TestFather()

--支持有约束有参数的泛型函数
obj:TestFun1(child, father)
obj:TestFun1(father, child)

--不支持没有约束的泛型函数
--obj:TestFun2(child)

--不支持有约束但没有参数的泛型函数
--obj:TestFun3()

--不支持非class约束的泛型函数
--obj:TestFun4(child)

```

```lua
--补充知识：让上面不支持使用的泛型函数可以使用
--该方法有一定的限制：
--使用Mono打包时，完全支持这种方式
--使用IL2CPP打包时，只有泛型参数是引用类型或是已经在C#中调用过的值类型参数才能支持
local testFun2 = xlua.get_generic_method(CS.MyClass, "TestFun2")
local testFun2_R = testFun2(CS.System.Int32)
testFun2_R(obj, 1)
```

[xLua背包实践](xLua背包实践_9pRRFRXHduFCjaVxoxzXMZ.md "xLua背包实践")

