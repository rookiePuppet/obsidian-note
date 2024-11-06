
## C#4 互操作性提升

## 动态类型

动态类型出现于C#4，它是为了互操作性而引入的。

```C#
dynamic text = "hello world"; // 动态类型声明变量
string world = text.Substring(6);
Console.WriteLine(world);

string broken = text.SUBSTR(6); // 抛出异常
Console.WriteLine(broken)
```

这段代码完全可以通过编译，因为变量`text`的类型是`dynamic`，编译器不会查找它是否存在`SUBSTR`方法，也不会查找`Substring`方法，查找方法的过程发生在代码的执行期。

在特定上下文中查找符号含义的过程称为绑定，动态类型就是把绑定过程从编译时转移到执行期。

### 什么是动态类型

`dynamic`只存在于C#中，IL会将其转换成带有`[Dynamic]`属性的`object`。

dynamic的基本规则：

1. 非指针类型到dynamic类型存在隐式转换
2. dynamic类型的表达式到任何非指针类型存在隐式转换
3. 含有dynamic类型的值的表达式，通常都是在执行期完成绑定
4. 大部分含有dynamic类型值得表达式，表达式编译时的类型也是dynamic

```C#
// 反映第一条规则
dynamic text = "hello world";
// 反映第二、三、四条规则
string world = text.Substring(6);
```

### 在多个上下文中应用动态绑定

下面代码展示了在加法运算符上下文中，根据执行期动态值得类型执行3中加法操作。

```C#
static void Add(dynamic d)
{
	Console.WriteLine(d + d);
}

Add("text");
Add(10);
Add(TimeSpan.FromMinutes(45));
```

上面例子中的每个加法操作都是合理的，如果换成静态类型的上下文，写法就大不相同。

```C#
static void SampleMethod(int value)
{
	Console.WriteLine("Method with int parameter");
}

static void SampleMethod(decimal value)
{
	Console.WriteLine("Method with decimal parameter");
}

static void SampleMethod(object value)
{
	Console.WriteLine("Method with object parameter");
}

static void CallMethod(dynamic d)
{
	SimpleMethod(d)
}

// 使用不同类型间接调用SimpleMethod
CallMethod(10); // int
CallMethod(10.5m); // decimal
CallMethod(10L); // decimal
CallMethod("text"); // object
```

执行期的绑定行为会尽可能和编译时的绑定行为一致，其绑定依据是动态值在执行期的类型。

> 只有动态值才会被动态处理

静态类型值使用编译时类型，动态类型值使用执行期类型，以保证执行期能够尽可能获取到准确的信息。

### 在动态绑定的上下文中，编译器做哪些检查

如果在编译时就能获得方法调用的上下文，那么编译器就可以检查能否找到该方法，如果找不到，就会出现编译错误。该规则适用于以下场景：

- 目标不是动态值得实例方法和索引器
- 静态方法
- 构造器

