---
tags:
  - Computer
  - ProgramingLanguage
date: 2023-09-03 11:00
---

# Kotlin简介

- 编程语言的分类
	1. 编译型语言：C、C++
	2. 解释型语言：Python、JavaScript、Java

- Java代码的处理过程
  Java代码—>编译—>特殊的class文件—Java虚拟机解释—>二进制数据

- Kotlin语言的优点
	1. 简洁、更加高效
	2. 可以直接调用Java代码，无缝使用Java第三方开源库

## 变量与函数

### 变量

#### 变量的定义

**※** 只允许在变量前声明两种关键字：val（常量）和var（变量）

> 怎么知道是什么类型的变量？答：Kotlin拥有出色的类型推导机制。

**※** 如果要对一个变量延迟赋值，Kotlin就无法推导其类型了，这时就需要显式声明变量类型。

示例：`var a: Int = 10`，此时变量a被声明为Int类型，如果后面再将一个字符串赋值给a，那么编译器将会抛出异常。

#### 什么时候使用val/var

永远优先使用val来声明变量，当val无法满足需求时再使用var。

### 函数

#### 函数定义的语法规则

```Kotlin
fun methodName(param1: Int, param2: Int): Int {
    return 0
}
```

#### Kotlin函数的语法糖

当函数中只有一行代码时，可不必编写函数体，可以直接将唯一的一行代码写在函数定义的尾部，中间用等号连接。

示例：

```Kotlin
fun largerNumber(num1: Int, num2: Int): Int = max(num1, num2)
```

## 程序的逻辑控制

#### if条件语句

▲ if用法在Kotlin中的特别之处：if语句可以有返回值，即每一个条件中最后一行代码的返回值。

示例：

```Kotlin
fun largerNumber(num1: Int, num2: Int) = 
    if(num1 > num2) num1 else num2
```

#### when条件语句

▲ 和if一样，when也可以有返回值，仍然可以用单行代码函数的语法糖。

示例：

```Kotlin
fun getScore(name: String) = when(name) {
    "Tom" -> 86
    "Jim" -> 77
    "Jack" -> 95
    ...
    else -> 0
}
```

when语句允许传入一个任意类型的参数，然后可以when的结构体中定义一系列的条件（精确匹配）。

语法格式：`匹配值 → {执行逻辑}`，当执行逻辑只有一行代码时，{}可省略

除精确匹配外，when语句还允许进行类型匹配。

```Kotlin
fun checkNumber(num: Number) {
    when(num) {
        is Int -> println("number is Int")
        is Double -> println("number is Double")
        else -> println("number not support")
    }
}
```

**is**关键字是类型匹配的核心，相当于Java中的**instanceof**关键字。

**※** when不带参数的用法

```Kotlin
fun getScore(name: String) = when {
    name == "Tom" -> 86  //name.startWith("Tom") -> 86
    name == "Jim" -> 77
    name == "Jack" -> 95
    ...
    else -> 0
}
```

#### for循环语句

**※** for-in循环的用法

区间概念：可以这段Kotlin代码表示一个区间`val range = 0..10`,其中的“**..**”是创建区间的关键字，上述代码表示创建一个0~10的区间，并且两端为闭区间。

有个这个区间，就可以通过for-in循环遍历这个区间：

```Kotlin
fun main() {
    for(i in 0..10) {
        println(i)
    }
}
```

▲ 用**util**代替 **..** 创建一个左闭右开的区间

▲ 用**step**关键字可跳过某些元素

▲ 用**downTo**关键字来创建一个降序的区间

# 面向对象编程

## 类与对象

定义一个类：

```Kotlin
class Person {
    
    var name = " "  //属性
    var age = 0
    
    fun eat() {  //行为
        println("name" + "is eating. He is" + age + "years old.")
    }
    
}
```

实例化:

```Kotlin
val p = Person()
```

## 继承与构造

#### 如何实现继承

1. 使Person类可被继承（在Kotlin中任何一个非抽象类默认都是不可被继承的）

```Kotlin
open class Person {
    ...
}
```

2. 让Student类继承Person类

```Kotlin
class Student: Person() {
    
    var sno = " "
    var grade = 0
    
}
```

#### 主构造函数

```Kotlin
class Student(val sno: String, val grade: Int): Person() {
    init {
        println("Sno is" + sno)
        println("grade is" + grade)
    }
}

//实例化
val student = Student("a123", 5)
```

`val sno: String, val grade: Int`部分是在实例化对象时需传入的参数。

`init`用于书写主构造函数中的逻辑。

`Person()`必须加括号，空括号表示调用无参构造，若父类为有参构造，则括号内要写上对应的参数。

```Kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int): 
        Person(name, age) {
    ...
}
```

`name: String, age: Int`不能加val，否则会导致父类中同名参数产生冲突。

#### 次构造函数

> 任何一个类只能有一个主构造函数，但可以多个次构造函数。

当类中既有主构造函数又有次构造函数时，所有的次构造函数都必须调用主构造函数（包括间接调用）。

示例：

```Kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int): 
        Person(name, age) {
    constructor(name: String, age: Int): this("", 0, name, age) {}
    constructor() : this("", 0) {}
}
```

当类中只有次构造函数，没有主构造函数。

```Kotlin
class Student: Person {  //没有显式定义主构造函数
    constructor(name: String, age: Int): super(name, age){
    
    }
}
```

`Person`不用加括号

`super(name, age)`调用父类构造函数

## 接口

> 接口是实现多态编程的重要组成部分。

#### 有什么用？

在接口中定义一系列的抽象行为，然后由具体的类去实现。

#### 一个类只能继承一个父类，但是可以实现任意多个接口

```Kotlin
interface Study {
    
    fun readBooks()
    fun doHomework()
    //接口中的函数不要求由函数体
}
```

```Kotlin
class Student(name: String, age: Int): Person(name. age): Study {

    override fun readBooks() {
        println(name + "is reading.")
    }
    
    override fun doHomework() {
        println(name + "is doing homework.")
    }
    
}
```

## 函数的可见性修饰符

#### Java（默认Default）

- 对所有类可见public

- 只对当前类内部可见private

- 只对当前类和子类，同一包路径下的类可见protected

- 对同一包路径下的类可见Default

#### Kotlin（默认public）

- 对所有类可见public

- 只对当前类内部可见private

- 只对当前类和子类可见protected

- 只对同一模块的类可见internal

## 数据类与单例类

### 数据类

> 用于将服务器端或数据库中的数据映射到内存中，为编程逻辑提供数据模型的支持。

数据类通常需要重写的方法：equals()、hashcode()、toString()

**在Kotlin中创建数据类：**

```Kotlin
**data** class Cellphone(val brand: String, val price: Double)
```

需要重写的方法根据传入的参数自动生成。

### 单例类

> 用于避免创建重复的对象，希望某个类在全局最多只有一个实例。

**在Kotlin中创建单例类：**

```Kotlin
object Singleton { /*写到这里Singleton就是一个单例类了
                    然后在类中编写需要的函数即可*/    
    object Singleton {
        fun singletonTest() {
            println("singletonTest is called.")
        }
    }

}
```

**如何使用单例类中的函数？**

```Kotlin
Singleton.singletonTest()
```

# Lambda编程

## 集合的创建和遍历

集合的类型：

- List（ArryayList、LinkedList）

- Set（HashSet）

- Map（HashMap）

**现有需求：创建一个包含许多水果名称的集合。**

初始化集合（不可变）

```Kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
```

遍历集合

```Kotlin
for(fruit in list) {
    println(fruit)
}
```

如何创建可变的集合？

```Kotlin
val list = MutableListOf("Apple", "Banana", "Orange", "Pear", "Grape")
list.add("Watermelon")
```

**Map集合的写法**:

```Kotlin
val map = HashMap<string,Int>()
//向map中添加数据
map.put("Apple", 1)
map.put("Banana", 2)
//改进写法
map["Apple"] = 1
map["Banana"] = 2

//从map中读取数据
val number = map["Apple"]
```

**简化写法：**

```Kotlin
val map = mapOf("Apple" to 1, "Banana" to 2 ...)

```

**遍历集合：**

```Kotlin
for((fruit, number) in map) {
    println("fruit is" + fruit + ". number is" + number)
}
```

## Java的函数式API

> Kotlin在调用Java方法时也可以使用函数API，只不过有一个条件限制：**该方法接收一个Java单抽象方法接口参数**。

Java原生API中有一个最常见的单抽象方法接口——Runnable接口，该接口中只有一个待实现的方法run（）。

对于任何一个Java方法，只要它接收Runnable参数，就可以使用函数式API。Runnable接口主要结合线程一起使用。

**Java代码**

```Java
new Thread(new Runnable() {
    @override
    public void run() {
        system.out.println("Thread is running");
    }
}).start();
```

**Kotlin代码**

```Kotlin
Thread(object: Runnable { //object是创建匿名类实例的关键字
    override fun run() {
        println("Thread is running")
    }
}).start()

```

**简化**

1. Runnable（）中只有一个待实现方法，不需显式重写run（）方法。

```Kotlin
Thread(Runnable {
    println("Thread is running")
}).start()
```

2. 若一个Java方法的参数列表不存在一个以上Java单抽象方法接口参数，可以省略接口名。

```Kotlin
Thread({
    println("Thread is running")
}).start()
```

3. 当Lambda表达式是方法的最后一个参数时，可以将Lambda表达式移到方法括号外面。

```Kotlin
Thread {
    println("Thread is running")
}.start()
```

## 集合的函数式API

> 本节重点：函数API的语法结构

#### 代码示例

```Kotlin
//找出集合中单词最长的那个水果
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val maxLengthFruit = list.maxBy{it.length}
println("max length fruit is" + maxLengthFruit)
```

#### 何为Lambda

Lambda是一段可以作为参数传递的代码。

#### Lambda表达式的语法结构

{参数名1：参数类型， 参数名2：参数类型 → 函数体}

**说明：若有参列表则要声明参数列表，函数体内可以编写任意长度的代码，最后一行代码作为表达式的返回值。**

#### 简化过程

▲ 原始代码

```Kotlin
val lambda = {fruit.String -> fruit.length}
val maxLengthFruit = list.maxBy(lambda)
```

1. 不需要专门定义lambda变量，而是直接将Lambda表达式传入maxBy（）函数中。
    
    `val maxLengthFruit = list.maxBy({fruit.String → fruit.length})`
    
2. 当Lambda参数是函数的最后一个参数时，可以将Lambda表达式移到函数括号外面。
    
    `val maxLengthFruit = list.maxBy() {fruit.String → fruit.length}`
    
3. 如果Lambda表达式是函数的唯一一个参数，还可以将函数括号省略。
    
    `val maxLengthFruit = list.maxBy {fruit.String → fruit.length}`
    
4. 由于Kotlin有类型推导机制，所以参数列表大多情况不需要声明参数类型。
    
    `val maxLengthFruit = list.maxBy {fruit → fruit.length}`
    
5. 当Lambda参数列表只有一个参数时，也不必声明参数名，而是可以使用it关键字代替。
    
    `val maxLengthFruit = list.maxBy {it.length}`
    

### 其他常用的函数式API

#### map

> 将集合中的每个元素都映射成另外的值，映射规则在Lambda表达式中指定，最终生成一个新集合。

```Kotlin
val newList = list.map {it.toUpperCase()}
//将水果名称变为大写格式
```

#### filter

> 过滤结合中的数据，可以单独使用，也可以和map一起使用。

```Kotlin
val newList = list.filter {it.length <= 5}
                  .map {it.toUpperCase()}
//只保留5个字母以内的水果

```

注：为保证效率，先使用filter，再使用map。

#### any

> 判断集合中是否至少存在一个元素满足指定条件。

```Kotlin
val anyResult = list.any {it.length <= 5}
```

#### all

> 判断集合中是否所有元素都满足条件。

```Kotlin
val allResult = list.all {it.length <= 5}
```

# 空指针检查

## 解决空指针异常

> 空指针异常是Android系统上崩溃率最高的异常类型。

如何保障代码在执行时的安全？

在调用参数前先进行判空处理：

```Java
public void doStudy(Study: study) {
    if(study != null) {
        study.readBooks();
        study.doHomework();
    }
}
```

## 可空类型系统

> Kotlin将空指针异常的检查提前到了编译时期。

如果我们的业务逻辑就是需要某个参数或变量为空该怎么办？

使用Kotlin提供的可空类型系统，但是在使用时，需要在编译时期将所有潜在空指针异常都处理掉，否则无法编译。

**在类名后加 ？表示该类型可为空**

如Int？表示可为空的整型

String？表示可为空的字符串

```Kotlin
fun doStudy(study: Study?) {
    study.readBooks()
    study.doHomework()
}

```

若调用doStudy（null）就不会提示错误，但是调用参数方法时，会有错误提示：可能有空指针异常的风险。

**解决方法：用if语句做判空处理**

```Kotlin
if(study != null) {
    ...
}
```

## 判空辅助工具

#### ?. 操作符

> 当对象不为空时调用相应的方法，为空时什么都不做。

```Kotlin
if(a != null) {
    a.doSomething()
}

a?.doSomething()
```

#### ?: 操作符

> 若左边表达式结果不为空就返回其结果，若为空则返回右边表达式的结果。

```Kotlin
val c = if(a != null) a else b

val c = a ?: b
```

#### ?. 与 ?: 配合使用

```Kotlin
fun getTextLength(text: String?): Int {
    if(text != null) {
        return text.length
    }
    return 0
}

fun getTextLength(text: String?)
    = text?.length ?: 0

```

#### 非空断言工具 !!

> 若想使用非空断言工具，最好提醒一下自己，有没有更好的解决方法。

```Kotlin
var content: String? = "hello"
fun main() {
    if(context != null) {
        printUpperCase()
    }
}

fun printUpperCase() { //函数不知道外部已经判空
    var upperCase = content!!.toUpperCase() //存在空指针风险无法编译
    println(upperCase)
}
```

#### 辅助工具 let

> let是Kotlin中的一个标准函数，提供函数式API的编程接口，并将原始调用对象作为参数传递到Lambda表达式中。

```Kotlin
obj.let { obj2 ->  //obj和obj2实际上是同一个对象
    //编写具体的逻辑
}
/*解释：调用obj的let函数，Lambda表达式中的代码就会立即执行，
并且这个obj对象本身还会作为参数传递到Lambda表达式中*/
```

```Kotlin
fun doStudy(study: Study?) {
    if(study != null) {
        study.readBooks()
    }
    if(study != null) {
        study.doHomework()
    }   
}

fun doStudy(study: Study?) {
    study?.let{ stu -> //只有一个参数时，可省略不写参数名，替换成it
        stu.readBooks()
        stu.doHomework()
    }
}
```

**let函数可处理全局变量的判空问题。**

# Kotlin中的小魔术

## 字符串内嵌

**语法规则：**

`"hello, ${obj.name}. nice to meet you"`

**当表达式中只有一个变量时，还可以将大括号省略**

`"hello, $name. nice to meet you"`

## 函数的参数默认值

> 我们可以在定义函数时给任意参数设定一个默认值，这样当调用此函数时就不会强制要求调用方为此参数传值，在没有传值的情况下会自动使用参数的默认值。

**如何设置默认值？**

```Kotlin
fun printParams(num: Int, str: String = "hello") {
    println("num is $num, str is $str")
}
```

**在下面的情况下，如果我们想让num参数使用默认值该怎么办？**

```Kotlin
fun printParams(num: Int = 100, str: String) {
    println("num is $num, str is $str")
}
```

如果这样写`printParams("...")`，编译器会认为我们要把字符串赋值给第一个num参数，从而报出类型不匹配的错误。

**！解决方法：通过键值对的方式来传参，从而不用按定义顺序来传参。**

```Kotlin
printParams(str = "world", num = 123)

printlnParam(str = "world")
```

**使用该方式为函数参数设定默认值便不需要使用次构造函数了。**

# 标准函数

> 标准函数指Standard.kt文件中定义的函数，任何Kotlin代码都能调用。

## with函数

接收两个参数：①一个任意类型的对象。②一个Lambda表达式。

with函数会在Lambda表达式中提供第一个参数对象的上下文，并使用Lambda表达式最后一行代码作为返回值。

```Kotlin
val result = with(obj) {
    //obj的上下文
    "value" //with函数的返回值
}
```

**with函数的作用：可以在连续调用同一个对象的多个方法时，让代码变得更加精简。**

- 示例
    
    有一个水果列表，现在想吃完所有水果，并将结果打印，可以这样写：
    

```Kotlin
val list = listOf("Apple", "Banana", "Orange")
val builder = StringBuilder()
builder.append("Start eating fruit.\n")
for(fruit in list) {
    builder.append(fruit).append("\n")
}
builder.append("Ate all fruits.")
val result = builder.toString()
println(result)
```

观察上述代码，会发现调用了很多次builder对象的方法，这时就可以考虑使用with函数：

```Kotlin
val list = listOf("Apple", "Banana", "Orange")
val result = with(StringBuilder ()) {
    append("Starting eating fruit.\n")
    for(fruit in list) {
    builder.append(fruit).append("\n")
    }
    append("Ate all fruits.")
    toString()
}
println(result)
```

## run函数

> run函数不能直接调用，而是调用某个对象run方法；run函数只接收一个Lambda参数，并且会在Lambda表达式中提供调用对象的上下文。

```Kotlin
val result = obj.run {
    //obj的上下文
    "value" //run函数的返回值
}
```

- 用run函数重写吃水果的代码

```Kotlin
val list = listOf("Apple", "Banana", "Orange")
val result = StringBuilder().run {
    append("Starting eating fruit.\n")
    for(fruit in list) {
        append(fruit).append("\n")
    }
    append("Ate all fruit.")
    toString()
}
println(result)
```

## apply函数

> 和run函数类似，也要在某个对象上调用，并且只接收一个Lambda参数，也会在Lambda表达式中提供调用对象的上下文，但是apply函数无法指定返回值，而是自动返回调用对象本身。

```Kotlin
val result = obj.apply{
    //obj的上下文
}
//result == obj
```

- 用apply函数重写吃水果的代码

```Kotlin
val list = listOf("Apple", "Banana", "Orange")
val result = StringBuilder().apply {
    append("Starting eating fruit.\n")
    for(fruit in list) {
        append(fruit).append("\n")
    }
    append("Ate all fruit.")
}
println(result.toString())
```

# 静态方法

> 静态方法在调用时不需要创建实例，非常适合编写一些工具类的功能。

Kotlin提供了比静态方法更好用的语法结构——单例类。

不过使用单例类写法会使整个类中的方法都变为静态方法，如果只希望某一个或几个方法变成静态方法怎么办？

使用`companion object() {}`

```Kotlin
class util() {
    fun doAction1() {
        println("do action1")
    }   
    
    companion object() { //在util内部产生一个伴生类，Kotlin会保证Util类只有一个伴生对象
        fun doAction2() { //其实这并不是静态方法，而是调用伴生对象的方法
            println("do action2")
        }
    }
}
```

> 如何定义真正的静态方法？
> 两种方式：注解、顶层方法

#### 注解

如果给**单例类**或者**compannion object**中的方法加上@JvmStatic注解，那么Kotlin编译器就会将这些方法编译成真正的静态方法。

```Kotlin
class util() {
    fun doAction1() {
        println("do action1")
    }
    
    companion object() { 
        @JvmStatic
        fun doAction2() { //此时doAction2（）就成了真正的静态方法了
            println("do action2")
        } 
    }
}
```

在Kotlin和Java中都可以用Util.doAction2()的写法来调用。

#### 顶层方法

> 顶层方法指那些没有定义在任何类中的方法。Kotlin编译器会将所有的顶层方法全部编译成静态方法。

**如何定义顶层方法？**

1. 新建**Kotlin类**，创建类型选择**File**。
2. 在该类中定义的方法都是顶层方法，编译后也都会变成静态方法。

# 变量延迟初始化

通过一个具体的例子来看：

```Kotlin
class MainActivity: AppCompatActivity(), View.OnClickListener {
    private var adapter: MsgAdapter? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        adapter = MsgAdapter(msgList)
        ...
    }
    
    override fun onClick(v: View?) {
        ...
        adapter?.ontifyItemInserted(msgList.size - 1)
        ...
    }
}
```

- 说明：
    - 这里我们将adapter设置为全局变量，但是它的初始化工作是在onCreate（）方法中进行的，因此不得不将adapter赋值为null，同时把它的类型声明成MsgAdapter？。
    - 虽然我们会在onCreate（）方法中对adapter进行初始化，同时能确保onClick（）方法中调用adapter的任何方法时仍然奥进行判空处理才行，否则编译是无法通过的。
    - 而当你的代码中有了越来越多的全局变量实例时，这个问题就会变得越来越明显，到时候你可能必须编写大量额外的判空处理代码，只是为了满足Kotlin编译器的需求。
    - 这个问题可以通过对全局变量进行延迟初始化来解决。

**延迟初始化使用的是 lateinit 关键字，它可以告诉编译器，我会晚点对这个变量初始化，这样就不用在一开始的时候将它赋值为null了。**

接下来我们就使用延迟初始化的方式对上述代码进行优化：

```Kotlin
class MainActivity: AppCompatActivity(), View.OnClickListener {
    private lateinit var adapter: MsgAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        adapter = MsgAdapter(msgList)
        ...
    }
    
    override fun onClick(v: View?) {
        ...
        adapter.ontifyItemInserted(msgList.size - 1)
        ...
    }
}
```

- 当然，使用lateinit关键字也不是没有任何风险的，如果我们在adapter变量还没初始化的情况下直接使用它，那么程序就会崩溃。
- 另外，我们还可以通过代码来判断一个全局变量是否已经完成初始化，这样在某些时候能够有效地避免重复对某一个变量进行初始化操作，示例代码如下：

	```Kotlin
	if(!::adapter.isInitialized) {
	    adapter = MsgAdapter (msgList)
	}
	```

- 具体语法就是这样，::adapter.isInitialized可用于判断adpater变量是否已经初始化。

# 使用密封类优化代码

> 密封类可以结合RecyclerView适配器中的ViewHolder一起使用，当然，其使用场景不止于此，它可以在很多时候帮助你写出更加规范和安全的代码。

首先来了解一下密封类具体的作用，来看一个简单的例子。新建一个Kotlin文件，在文件中编写如下代码：

接下来在定义一个getResultMsg（）方法，用于获取最终执行结果的信息，代码如下：

```Kotlin
interface Result
class Success(val msg: String): Result
class Failure(val error: Exception): Result

```

接下来在定义一个getResultMsg（）方法，用于获取最终执行结果的信息，代码如下：

```Kotlin
fun getResultMsg(result: Result) = when(result) {
    is Success -> result.msg
    is Failure -> result.error.message
    else -> throw IllegalArgumentException()
}
```

- 令人讨厌的是，这里我们除了编写Success和Failure的条件分支外，还要再编写一个else分支，否则编译无法通过。但实际上Result的执行结果只可能是Success或者Failure，这个else条件是永远走不到的。
- 另外编写else条件还有一个潜在的风险：如果我们现在新增一个Unknow类并实现Result接口，用于表示未知的执行结果，但是忘记在gerResultMsg（）方法中添加相应的条件分支，编译器在这个时候是不会提醒我们的，而是会在运行的时候进入else条件里面，从而抛出异常使程序崩溃。密封类就可以很好地解决这个问题。

密封类使用的关键字是 sealed class ，它的用法也非常简单，我们可以轻松地将Reuslt接口改造成密封类的写法：

```Kotlin
sealed class Result
class Success(val msg: String): Result()
class Failure(val error: Exception): Result()

fun getResultMsg(result: Result) = when(result) {
    is Success -> result.msg
    is Failure -> "error is ${result.error.message}"
   
}

```

为什么这里去掉了else条件仍然能通过编译呢？这是因为当在when语句中传入一个密封类变量作为条件时，编译器会自动检查该密封类有哪些子类，并强制要求你将每一个子类所对应条件全部处理，这样就可以保证即使没有编写else分支，也不可能会出现漏写条件的情况。而如果我们现在新增一个Unknown类，并让它继承自Result。次是getResultMsg（）方法就一定会报错，必须增加一个Unknown的条件分支才能让代码编译通过。

观看MsgAdapter现在的代码，你会发现onBindViewHolder（）方法中就存在一个没有实际作用的else条件，只是抛出了一个异常而已。对于这部分代码，我们就可以借助密封类的特性来优化。新建一个MsgViewHolder.kt文件，加入如下代码：

```Kotlin
sealed class MsgViewHolder(view: View): RecyclerView.ViewHolder(view)
    
class LeftViewHolder(view: View): MsgViewHolder(view) {
    val leftMsg: TextView = view.findViewById(R.id.leftMsg)
}

class RightViewHolder(view: View): MsgViewHolder(view) {
    val rightMsg: TextView = view.findViewById(R.id.rightMsg)
}

```

- 这里我们定义了一个密封类MsgViewHolder，并让它继承自RecyclerViewHolder，然后让LeftViewHolder和RightViewHolder继承自MsgViewHolder。
- 这样就相当于密封类MsgViewHolder只有两个已知子类，因此在when语句中只要处理这两种情况的条件分支即可。
- 现在修改MsgAdapter中的代码：

```Kotlin
class MsgAdapter(val msgList: List<Msg>): RecyclerView.Adapter<MsgViewHolder>() {
    
    override fun onBindViewHolder(holder: MsgViewHolder, position: Int) {
        val msg = msgList[position]
        when(holder) {
            is LeftViewHolder -> holder.leftMsg.text = msg.content
            is RightViewHolder -> holder.rightMsg.text = msg.content
        }
    }
    
}

```

这里我们将RecyclerView.Adapter的泛型指定刚刚定义的密封类MsgViewHolder，这样onBindViewHolder（）方法传入的参数就变成了MsgViewHolder。然后我们只要在when语句中处理LeftViewHolder和RightViewHolder这两种情况即可，那个讨厌的else也就不需要了，这种RecyclerView适配器的写法更加规范也更加推荐。

# 扩展函数与运算符重载

## 大有用途的扩展函数

> 扩展函数表示即使在不修改某个类的源码的情况下，仍然可以打开这个类，向该类中添加新的函数。

为了帮助理解，我们先来思考一个功能：一段字符串中可能包含字母、数字和特殊符号等字符，现在我们希望统计字符串中字母的数量。

- 如果按照一般的编程思维，可能大多数人会很自然地写出如下函数：

```Kotlin
object StringUtil {
    
    fun letterCount(string: String): Int {
        var count = 0
        for(char in string) {
            if(char.isLetter()) {
                count++
            }
        }
    }
    
}
```

- 这里先定义了一个StringUtil单例类，然后再这个单例类中定义了一个lettersCount（）方法。
- 现在当我们需要统计某个字符串中的字母数量时，只需要编写如下的代码即可：

```Kotlin
val str = "ABC123xyz!@#"
val count = StringUtil.letterCount(str)
println(count)
```

这种写法完全可以正常工作，并且也是Java编程中最标准的实现思维。但是有了扩展函数之后就不一样了，我们可以使用一种更加面向对象的思维来实现这个功能，比如说将lettersCount（）函数添加到String类当中。

**下面就是定义扩展函数的语法结构：**

```Kotlin
fun ClassName.methodName(param1: Int, param2: Int): Int {
    return 0
}
```

- 由于我们希望向String类中添加一个扩展函数，因此需要先创建一个String.kt文件。文件名虽然没有固定的要求，但是建议**向哪一个类中添加扩展函数，就定义一个同名的kotlin文件，这样便于以后查找**。当然，扩展函数也是可以定义在任何一个现有类当中的，并不一定非要创建新文件。不过通常来说，**最好将它定义成顶层方法，这样可以让扩展函数拥有全局的访问域**。
- 现在向String.kt文件中编写如下代码：

```Kotlin
fun String.letterCount(): Int {
    var count = 0
    for(char in this) {
        if(char.isLetter()) {
            count++
        }
    }
    return count
}

```

- 注意这里的代码变化，现在我们将lettersCount（）方法定义成了String类的扩展函数，那么函数中就自动拥有了String实例的上下文。因此lettersCount（）函数就不再需要接收一个字符串参数了，而是直接遍历this即可，因为现在this就代表着字符串本身。
- 定义好扩展函数之后，统计某个字符串中的字母数量只需要这样写即可：

```Kotlin
val count = "ABC123xyz!@#".letterCount()
```

## 有趣的运算符重载

> 运算符重载使用的是**operator**关键字，只要再指定函数的前面加上operator关键字，就可以实现运算符重载的功能了。问题就在于这个指定函数是什么？这是运算符重载里面比较复杂的一个问题，因为不同的运算符对应的重载函数也是不同的。比如说加号运算符对应的时plus（）函数，减号运算符对应的是minus（）函数。

- 以加号运算符为例，如果要是实现两个对象相加的功能，那么它的语法结构如下：

```Kotlin
class Obj {

    operator fun plus(obj: Obj) {
        //处理相加的逻辑
    }

}
```

- 在上述语法结构中，关键字operator和函数名plus都是固定不变的，而接收的参数和函数返回值可以根据你的逻辑自行设定。那么上述代码就表示一个Obj对象可以与另一个Obj对象相加，最终返回一个新的Obj对象。对应的调用方式如下：

```Kotlin
val obj1 = Obj()
val obj2 = Obj()
val obj3 = obj1 + obj2
```

- 这种obj1+obj2的语法看上去好像很神奇，但其实这就是Kotlin给我们提供的一种语法糖，它会在编译的时候被转换成obj1.plus（obj2）的调用方式。

**接下来我们就来实现让两个Money对象相加的功能：**

- 首先定义Money类的结构：

```Kotlin
class Money(val value: Int)
```

- 然后使用运算符重载来实现让个Money对象相加的功能：

```Kotlin
class Money(val value: Int) {
    
    operator fun plus(money: Money): Money {
        val sum = value + money.value
        return Money(sum)
    }
    
}
```

- 接下来使用代码对以上功能进行测试：

```Kotlin
val money1 = Money(5)
val money2 = Money(10)
val money3 = money1 + money2
println(money3.value)
```

- 但是现在Money对象只允许和另一个Money对象相加，如果Money对象可以直接和数字相加的话就更好了。这个功能也是可以是实现的，因为Kotlin允许我们对同一个运算符进行多重重载，代码如下：

```Kotlin
class Money(val value: Int) {
    
    operator fun plus(money: Money): Money {
        val sum = value + money.value
        return Money(sum)
    }
    
    operator fun plus(newValue: Int): Money {
        val sum = value + newValue
        return Money(sum)
    }
    
}
```

- 这里我们又重载了一个plus（）函数，不过这次接受的参数是一个整型数字，其他代码是一样的，现在Money对象就拥有了和数字相加的能力：

```Kotlin
val money1 = Money(5)
val money2 = Money(10)
val money3 = money1 + money2
val money4 = money3 + 20
println(money4.value)
```

![](https://secure2.wostatic.cn/static/m13EeNFUiDktmRUBCXAtiB/image.png?auth_key=1693715236-hML6yggfZmaEXCx9MEQP3G-0-9529c15f9ee57a9355a33461740184c7)

## 结合扩展函数和运算符重载

- 在前面我们使用过一个随机生成字符串长度的函数，代码如下所示：

```Kotlin
fun getRandomLengthString(string: String): String {
    val n = (1..20).random()
    val builder = StringBuilder()
    repeat(n) {
        builder.append(string)
    }
    return builder.toString()
}
```

- 其实这个函数的核心思想就是将传入的字符串重复n次，如果我们能够使用str*n这种写法来表示让str字符串重复n次，这种语法体验是不是很棒呢？而在Kotlin中是可以实现的。
- 要让一个字符串可以乘以一个数字，那么肯定是要在String类中重载乘号运算符才行，但是String类是系统提供的类，我们无法修改这个类的代码。这个时候就可以借助扩展函数功能向String类中添加新函数了。
- 既然是向String类中添加扩展函数，那么我们还是打开刚才创建的String.kt文件，然后加入如下代码：

```Kotlin
operator fun String.times(n: Int): String {
    val builder = StringBuilder()
    repeat(n) {
        builder.append(this)
    }
    return buidler.toString()
}
```

- 现在，字符串就拥有了和一个数字相乘的能力，比如执行如下代码：

```Kotlin
val str = "abc" * 3
println(str)
```

- 另外，必须说明的是，其实Kotlin的String类中已经提供了一个用于将字符串重复n遍输出的repeat（）函数，因此times（）函数还可以进一步精简成如下形式：

```Kotlin
operator fun String.times(n: Int): String = repeat(n)
```

- 掌握了上述功能之后，现在我们就可以在getRandomLengtString（）函数中使用这种魔术一般的写法了，代码如下：

```Kotlin
fun getRandomLengthString(string: String): String = string * (1..20).random()
```

# 高阶函数详解

## 定义高阶函数

> 如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数。

**Kotlin中增加了一个函数类型的概念，如果我们将这种函数类型添加到一个函数的参数声明或者返回值声明当中，那么这就是一个高阶函数了。 **

#### 如何定义一个函数类型呢？

- 不同于定义一个普通的字段类型，函数类型的语法规则是有点特殊的，**基本规则：(String, Int) -> Unit**
- 左边部分用于声明该函数接收什么参数，多个参数之间使用逗号隔开，如果不接收任何参数，写一对空括号即可；右边部分用于声明该函数的返回值是什么类型，如果没有返回值就用Unit。

如下函数即为一个高阶函数：

```Kotlin
fun example(func: (String, Int) -> Unit) {
    func("hello",123)
}
```

#### 高阶函数有什么用途呢？

简单概括，那就是高阶函数允许让函数类型的参数来决定函数的执行逻辑。即使是同一个高阶函数，只要传入不同的函数类型参数，那么它的执行逻辑和最终的返回结果就可能使完全不同的。

**具体例子：**

```Kotlin
fun num1AndNum2(num1:Int, num2: Int, operation: (Int, Int) -> Int) {
    val result = operation(num1, num2)
    return result 
}

fun plus(num1: Int, num2: Int): Int {
    return num1 + num2
}

fun minus(num1: Int, num2: Int): Int {
    return num1 - num2
}

fun main() {
    val num1 = 100
    val num2 = 80 
    val result1 = num1AndNum2(num1, num2, ::plus) //::是一种函数引用方式的写法，表示将函数作为参数传给某个函数
    val result2 = num1AndNum2(num1, num2, ::minus)
    println("result1 is $result1")
    println("result2 is $result2")
}
```

**上述代码如果使用Lambda表达式的写法来实现的话，代码如下：**

```Kotlin
fun main() {
    val num1 = 100
    val num2 = 80
    val result1 = num1AndNum2(num1, num2) {n1, n2 ->
        n1 + n2
    }
    val result2 = num1AndNum2(num1,num2) {n1, n2 ->
        n1 - n2
    }
    println("result1 is $result1")
    println("result2 is $result2")
    }
```

---

回顾apply函数，它可以用于给Lambda表达式提供一个指定的上下文，当需要连续调用同一个对象的多个方法时，apply函数可以让代码变得更加精简，比如StringBuilder就是一个典型的例子。下面就使用高阶函数实现一个类似的功能。

```Kotlin
fun StringBuilder.build(block: StringBuilder.() -> Unit): StringBuilder {
    block()
    return this
}
```

这里我们给StringBuilder类定义了一个build扩展函数，这个扩展函数接收一个函数类型参数，并且返回值类型也是StringBuilder。

注意这里函数类型参数的声明方式和前面有所不同：它在函数类型的前面加上了一个StringBuilder.的语法结构。其实这才是定义高阶函数的完整语法规则，在函数类型的前面加上ClassName.就表示这个函数类型是定义在哪一个类当中的。

另外，将函数类型定义到StringBuilder类当中的好处就是当我们调用build函数时传入的Lambda表达式就会自动拥有StringBuilder的上下文，同时这也是apply函数的实现方式。

**现在我们就可以使用自己创建的build函数来简化StringBuilder构建字符串的方式了，这里用吃水果来举例：**

```Kotlin
fun main() {
    val list = listOf("", "", "", "", ...)
    val result = StringBuilder().build {
        append("Start eating fruits.\n")
        for(fruit in list) {
            append(fruit).append("\n")
        }
        append("Ate all fruit")
    }
    println(result.toString())
}
```

## 内联函数

分析高阶函数的实现原理：

- Kotlin代码最终是要编译成Java字节码的，但Java中并没有高阶函数的概念。
- Kotlin编译器会将高阶函数的语法转换成Java支持的语法结构。
- 我们使用的Lambda表达式在底层被转换成了匿名类的实现方式，这就表明我们每调用一次Lambda表达式，都会创建一个新的匿名类实例，当然也会造成额外的内存和性能开销。
- 为了解决这个问题，Kotlin提供了内联函数的功能，它可以将使用Lambda表达式带来的运行时开销完全消除。

内联函数的用法非常简单，只需要在定义高阶函数时加上**inline**关键字的声明即可。

内联函数的工作原理：Kotlin编译器会将内联函数中的代码在编译的时候自动替换到调用它的地方。

## noinline和crossinline

一个高阶函数如果接收了两个或更多函数类型的参数，这是我们给函数加上inline关键字，那么Kotlin编译器会自动将所有引用的Lambda表达式全部进行内联。

但是，如果我们只想内联其中一个Lambda表达式，就可以使用noinline关键字了，如下：

```Kotlin
inline fun inlineTest(block: () -> Unit, noinline block2: () -> Unit) {

}
```

> 为什么Kotlin还要提供一个noinline关键字来排除内联功能呢？

- 这是因为内联的函数类型参数在编译时会被进行代码替换，因此它没有真正的参数属性。非内联的函数类型参数可以自由地传递给其他任何函数，因为它就是一个真实的参数，而**内联的函数类型参数只允许传递给另一个内联函数**，这也是它最大的局限性。
- 另外，内联函数和非内联函数还有一个重要的区别，那就是**内联函数所引用的Lambda表达式中是可以使用return关键字来进行函数返回的，而非内联函数只能进行局部返回**。

将高阶函数声明成内联函数是一种良好的编程习惯，事实上，大多数高阶函数是可以直接声明成内联函数的，但是也有少部分例外的情况。

如果我们在高阶函数中创建了另外的Lambda或者匿名类的实现，此时再将高阶函数声明成内联函数，就一定会提示错误。

错误原因是**内联函数的Lambda表达式中允许使用return关键字，和高阶函数的匿名类实现中不允许使用return关键字产生了冲突。**

借助**crossinline**关键字就可以很好地解决这个问题。

crossinline就像一个契约，它用于保证在内联函数的Lambda表达式中一定不会使用return关键字，这样冲突就不存在了。

## 高阶函数的应用

### 简化SharedPreferences

新建一个SharedPreferences.kt文件，添加以下代码：

```Kotlin
fun SharedPreferences.open(block: SharedPreferences.Editor.() -> Unit) {
    val editor = edit()
    editor.block()
    editor.apply()
}
```

首先通过扩展函数的方式向SharedPreferences的上下文，因此这里可以直接调用edit（）方法来获取SharedPreferences.Editor对象。另外open函数接收的是一个SharedPreferences.Editor的函数类型参数，因此这里需要调用editor.block（）对函数类型参数进行调用，我们就可以在函数类型参数的具体实现中添加数据了。最后还要调用editor.apply（）方法来提交数据，从而完成数据存储操作。

定义好open函数之后，我们在项目中使用SharedPreferences存储数据就更加方便了，写法如下：

```Kotlin
getSharedPreferences("data", MODE_PRIVATE).open {
    putString("name", "Tom")
    putInt("age", 18)
    putBoolean("married", false)
}
```

Google提供的KTX扩展库中已经包含了上述SharedPreferences的简化用法，这个扩展库会在创建项目时自动引入build.gradle的dependencies中。

因此我们可以直接使用如下写法来向SharedPreferences存储数据：

```Kotlin
getSharedPreferences("data", MODE_PRIVATE).edit {
    putString("name", "Tom")
    putInt("age", 18)
    putBoolean("married", false)
}
```

### 简化ContentValues

1. 新建一个ContentValues.kt文件，定义一个cvOf（）方法：

	```Kotlin
	fun cvOf(vararg pairs: Pair<String, Any?>): ContentValues
	```

	cvOf（）方法的实现功能逻辑，核心思路是先创建一个ContentValues对象，然后遍历Pairs参数列表，取出其中的数据并填入ContentValues中，最终将ContentValues对象返回即可。使用when语句一一进行条件判断，让Pair参数的值和ContentValues所支持的数据类型对应起来，覆盖ContentValues所支持的所有数据类型。这里还使用了Kotlin中Smart Cast功能，比如when语句进入Int条件分支后，这个条件下面的value会被自动转换成Int类型，而不再是Any？类型，这样就不要像Java那样再额外进行一次向下转型了。


	```Kotlin
	fun cvOf(vararg pairs: Pair<String, Any?>): ContentValues {
	    val cv = ContentValues()
	    for(pair in pairs) {
	        val key = pair.first
	        val value = pair.second
	        when(value) {
	            is Int -> cv.put(key, value)
	            is Long -> cv.put(key, value)
	            is Short -> cv.put(key, value)
	            is Float -> cv.put(key, value)
	            is Double -> cv.put(key, value)
	            is Boolean -> cv.put(key, value)
	            is String -> cv.put(key, value)
	            is Byte -> cv.put(key, value)
	            is ByteArray -> cv.put(key, value)
	            null -> cv.putNull(key)
	        }
	    }
	    return cv
	}
	```

	Pair类型：由于Pari是一种键值对的数据结构，因此需要通过泛型来指定它的键和值分别对应什么类型的数据。值得庆幸的是，ContentValues的所有键都是字符串类型的，这里可以直接将Pair键的泛型指定成String。但ContentValues的值却可以有多种类型（字符串类型、整型，浮点型，甚至是null），所以我们需要将Pair值的泛型指定成Any？。这是因为Any是Kotlin中所有类的共同基类，相当于Java中的Object，而Any？则表示允许传入空值。

2. 现在就可以使用类似mapOf（）函数的语法结构来构建ContentValues对象了。

	```Kotlin
	val values = cvOf("name" to "Game of Thrones", "author" to "George", "pages" to 720, "price" to 20.85)
	db.insert("Book", null, values)
	```

3. 虽然现在cvOf（）方法已经非常好用了，但是它和高阶函数却没有一点关系。因为cvOf（）方法接收的参数是Pair类型的可变参数列表，返回值是ContentValues对象，完全没有用到函数类型，这和高阶函数的定义不符。从代码实现方面，可以借助高阶函数来进一步优化，比如借助apply函数：

	```Kotlin
	fun cvOf(vararg pairs: Pair<String, Any?>) = ContentValues().apply {
	    for(pair in pairs) {
	        val key = pair.first
	        val value = pair.second
	        when(value) {
	            is Int -> cv.put(key, value)
	            is Long -> cv.put(key, value)
	            is Short -> cv.put(key, value)
	            is Float -> cv.put(key, value)
	            is Double -> cv.put(key, value)
	            is Boolean -> cv.put(key, value)
	            is String -> cv.put(key, value)
	            is Byte -> cv.put(key, value)
	            is ByteArray -> cv.put(key, value)
	            null -> cv.putNull(key)
	        }
	    }
	}
	```

4. KTX库中同样提供了一个具有相同功能的contentValuesOf（）方法：

	```Kotlin
	val values = contentValuesOf("name" to "Game of Thrones", "author" to "George", "pages" to 720, "price" to 20.85)
	db.insert("Book", null, values)
	```

# 泛型和委托

## 泛型的基本用法

> 泛型允许我们在不指定一个具体类型的情况下进行编程，这样编写出来的代码拥有更好的扩展性。

**泛型主要有两种定义方式：一种是定义泛型类，另一种式定义泛型方法，使用的语法结构都是<T>，T可以用任何英文字母或单词代替。**

**定义一个泛型类可以这样写：**

```Kotlin
class MyClass<T> {

    fun method(param: T): T {
        return param
    }

}
```

这样在调用MyClass类和method（）方法时，就可以将泛型指定成具体的类型。

```Kotlin
val myClass = MyClass<Int>()
val result = myClass.method(123)
```

** 定义一个泛型方法可以这样写：**

```Kotlin
fun <T>method(param: T): T{
    return param
}
```

调用方式有所调整：

```Kotlin
val myClass = MyClass()
val result = myClass.method<Int>(123)
```

由于Kotlin拥有出色的类型推导机制，因此这里可以直接省略泛型指定。

还可以对泛型的类型进行限制。通过指定上界的方式来对泛型的类型进行约束，如：

```Kotlin
fun <T: Number> method(param: T): T{
    return param
}
```

另外在默认情况下，所有的泛型都是可以指定成可空类型的，这是因为在不手动只当上界的时候，泛型的上界默认是Any？。如果想让泛型的类型不可为空，只需要将泛型的上界手动指定成Any即可。

## 类委托和委托属性

> 委托是一种设计模式，基本理念：操作对象自己不会去处理某段逻辑，而是会把工作委托给另一个辅助对象去处理。

**委托功能分为两种：类委托和委托属性。**

### 类委托：将一个类的具体实现委托给另一个类去完成。

借助类委托，可以轻松实现一个自己的实现类。比如定义一个MySet，让它实现Set接口，如下：

```Kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> {
    override val size : Int
        get() = helperSet.size
    override fun contains(element: T) = helperSet.contains(element)
    override fun containsAll(elements: Collection<T> = helperSet.containsAll(elements)
    override fun isEmpty() = helperSet.isEmpty()
    override fun iterator() = helperSet.iterator
}
/*数据结构Set与List有点类似，只是它所存储的数据是无序的，并且不能存储重复的数据。
Set是一个接口，如果要使用它，需要使用它具体的实现类，比如HashSet。*/
```

MySet的构造函数中接受了一个HashSet参数，这就相当于一个辅助对象。然后在Set接口所有的方法实现中，我们都没有进行自己的实现，而是调用了辅助对象中相应的方法实现，这就是一种委托模式。

**好处**：如果我们只是让大部分方法实现辅助对象中的方法，少部分方法实现由自己来重写，甚至加入一些自己独有的方法，那么MySet就会成为一个全新的数据结构类。

**弊端**：如果接口中待实现方法非常多的话，要编写的代码量就很多了，在Kotlin中可以通过类委托功能来解决。

委托使用的关键字是by，在接口声明后面使用by关键字，在接上受委托的辅助对象，就可以免去一大堆模板式的代码了，如下：

```Kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> by helperSet {
 
}
```

### 委托属性：将一个属性（字段）的具体实现委托给另一个类去完成。

语法结构：

```Kotlin
class MyClass {
    var p by Delegate()
}
```

这里使用by关键字连接了左边的p属性和右边的Delegate实例，这样代表着将p属性的具体实现委托给Delegata类去完成。当调用p属性时会自动调用Delegate类的getValue方法，当给p属性赋值时会自动调用Delegate类的setValue（）方法。

因此还要对Delegate类进行具体实现才行，如下：

```Kotlin
class Delegate {
    
    var propValue: Any? = null
    
    operator fun getValue(myClass: MyClass, prop: Kproperty<*>): Any? {
        return propValue
    }
    
    operator fun setValue(myClass: MyClass, prop: Kproperty<*>, value: Any?) {
        propValue = value
    }

}
```

这是一种标准的代码实现模板，在Delegate类中我们必须实现getValue（）和setValue（）方法，并且都要使用operator关键字进行声明。

- getValue（）方法接收两个参数：**第一个参数用于声明该Delegate类的委托功能可以在什么类中使用；第二个参数Kproperty<>是Kotlin中的一个属性操作类，可用于获取各种属性相关的值，在当前场景下用不着，但是必须在方法参数上进行声明。**_另外，<_>这种泛型写法表示你不知道或者不关心泛型的具体类型，只是为了通过语法编译而已，返回值可以声明成任意类型。
- setValue（）方法类似，接收三个参数：前两个参数和getValue（）方法相同，最后一个参数表示具体要赋值给委托属性的值，这个参数类型必须和getValue（）方法返回值的类型一致。

如果在MyClass类中p的属性是使用val声明的，那么就没有必要实现setValue（）方法了。

## 实现一个自己的lazy函数

> by lazy的基本语法结构：`val p by lazy {...}`

在lazy函数中会创建并返回一个Delegate对象，当调用p属性时，其实调用的是Delegate对象的getValue（）方法，然后getValue（）方法中又会调用lazy函数传入的Lambda表达式，这样表达式中的代码就可以执行，并且调用p属性后得到的值就是Lambda表达式中最后一行代码的返回值。

1. 新建一个Later.kt文件：

	```Kotlin
	class Later<T>(val block: () -> T) {
	}
	```

	首先定义了一个Later类，并将让它指定成泛型类。Later的构造函数中接收一个函数类型参数，这个函数类型参数不接收任何参数，并且返回值就是Later类指定的泛型。

2. 接着在Later类中实现getValue（）方法，如下：

	```Kotlin
	class Later<T>(val block: () -> T) {
	    
	    var value: Any? = null
	    
	    operator fun getValue(any: Any?, prop: KProperty<*>): T {
	        if(value == null) {
	            value = block()
	        }
	        return value as T
	    }
	    
	}
	```

	这里将getValue（）方法的第一个参数指定成了Any？类型，表示我们希望Later的委托功能在所有类中都可以使用。然后使用了一个value变量对值进行缓存，如果value为空就调用构造函数中传入的函数类型参数去获取值，否则就直接返回。
	
	由于懒加载技术是不会对属性进行赋值的，因此不用实现setValue（）方法。

3. 到这里，委托属性的功能就已经完成了，为了让它的用法更加类似于lazy函数，最好在定义一个顶层函数，这个函数直接写在Later.kt文件中即可，但要在Later类的外面：

	```Kotlin
	fun <T> later(block: () -> T) = Later(block)
	```

	我们将这个顶层函数也定义成了泛型函数，并且它也接收一个函数类型参数。这个顶层函数的作用很简单：创建Later类的实例，并将接收的函数类型参数传给Later类的构造函数。

这样我们自己编写的later懒加载函数就完成了，可以直接用它代替之前的lazy函数。

# infix函数

> 在前面我们多次使用过`A to B`这样的语法结构构建键值对，包括Kotlin自带的mapOf（）函数，这种语法结构的优点是可读性高。

首先，to不是Kotlin语言的关键字，之所以能使用这种语法结构，是因为Kotlin提供了一种高级语法糖特性：infix函数，这样`A to B`就相当于`A.to(B)`。

一个简单的例子：

- String类中有一个startsWith（）函数，用于判断一个字符串是否以某个指定参数开头，以下代码的判断结果一定是true：

```Kotlin
if("Hello Kotlin".startsWith("Hello")) {
    //处理具体的逻辑
}
```

- 若借助infix函数，我们可以使用一种更具可读性的语法来表达这段代码：

```Kotlin
infix fun String.beginsWith(prefix: String) = startsWith(prefix)

```

- 先不看infix，这是一个String类的扩展函数，我们给String类添加了这个函数，作用效果跟startsWith（）函数一致，因为它的内部实现就是调用String类的startsWith（）函数。
- 加了infix关键字之后，beginsWith（）就变成了一个infix函数，这样除了使用传统的调用方式之外，还可以用一种特殊的语法糖格式来调用：

```Kotlin
if("Hello Kotlin" beginsWith ("Hello")) {
    //处理具体的逻辑
}
```

infix函数允许我们将函数调用时的小数点、括号等计算机相关的语法去掉，从而使用一种更接近英语的语法来编写程序，使代码更具可读性。

另外，由于infix函数的特殊性，有两个严格的限制：1. 不能定义成顶层函数，它必须是某个类的成员函数，可以使用扩展函数的方式将它定义到某个类当中；2. 必须接收且只能接受一个参数，参数类型不限制。

再来看一个复杂一些的例子。比如有一个集合，如果想要判断集合中是否包括某个指定元素，一般可以这样写：

- 借助infix函数来使代码更具可读性：

```Kotlin
infix fun <T> Collection<T>.has(element: T) = contains(element)
```

可以看到，我们给Collection接口添加了一个扩展函数，这是因为Collection是Java以及Kotlin所有集合的总接口，因此给Collection添加一个has（）函数，那么所有集合的子类就都可以使用这个函数了。另外还是用了泛型函数的定义方法，使has（）函数可以接收任意具体类型的参数。

- 使用infix函数优化后的代码：

```Kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
if(list has ("Banana")) {
    //处理具体的逻辑
}
```

# 泛型的高级特性

## 对泛型进行实化

- 在JDK1.5之前，Java是没有泛型功能的，那时诸如List之类的数据结构可以存储任意类型的数据，取出数据时也需要手动向下转型才行，这不仅麻烦，而且危险，比如说我们在同一个List中存储了字符串和整型这两种类型，但是取出数据时无法区分具体的数据类型，如果手动强制转成同一种类型，则会抛出类型转换异常。
- 于是在JDK1.5中，Java引入泛型，这不仅让List之类的数据结构变得简单好用，也让代码变得更安全。实际上，Java的泛型功能时通过类型擦除机制来实现的。就是说泛型对于类型的约束只在编译时期存在，运行时仍然按照以往的机制，JVM是识别不出来我们在代码中指定的泛型类型的。所有基于JVM的语言，它们的泛型功能都是通过类型擦除机制来是实现的，其中当然也包括Kotlin。这种机制使得我们不可能使用 a is T 或者 T::class.java 这样的语法，因为T的实际类型以及在运行时被擦除了。
- 然而不同的是，Kotlin提供了一种内联函数的概念。内联函数中的代码会自动替换到调用它的地方，因此就不存在泛型擦除的问题了，因为代码在编译之后会直接使用实际的类型来替代内联函数中的泛型声明，工作原理如图：

![](https://secure2.wostatic.cn/static/mm6ZwY8WcBpk1jDpwA9DxY/image.png?auth_key=1693716091-6F2waLjh6XVK6HyoNfGMdV-0-15ff13f24b4e146521c79c67a5347339)

可以看到，bar（）是一个带有泛型类型的内联函数，foo（）函数调用了bar（）函数，在代码编译之后，bar（）函数中的代码将可以获得泛型的实际类型，这意味着，Kotlin中是可以将内联函数中的泛型进行实化的。

那么具体怎么写才能将泛型实化呢？首先必须是内联函数才行，其次在声明泛型的地方必须加上reified关键字来表示该泛型要进行实化。示例如下：

```Kotlin
inline fun <reified T> getGenericType() {

}
```

借助泛型实化可以实现什么效果？从函数名就看出来，这里准备实现一个获取泛型实际类型的功能，代码如下：

```Kotlin
inline fun <reified T> getGenericType() = T::class.java
```

虽然只有一行代码，但实现了一个Java中不可能实现的功能：GetGenericType（）函数直接返回当前指定泛型的实际类型。T.class这样的语法在Java中是不合法的，而在Kotlin中，借助泛型实化功能就可以使用T::class.java这样的语法了。

## 泛型实化的应用

在Android四大组件中，除了ContentProvider之外，其余三个组件有一个共同点，就是要结合Intent一起使用。比如说启动一个Activity这样写：

```Kotlin
val intent = Intent(context, MainActivity::class.java)
context.startActivity(intent)
```

利用泛型实化功能我们可以实现一种更加方便的写法，新建一个reified.kt文件，编写如下代码：

```Kotlin
inline fun <reified T> startActivity(context: Context) {
    val intent = Intent(context, T::class.java)
    context.startActivity(intent)
}
```

这里定义了一个startActivity（）函数，接收一个Context参数，并同时使用inline和refied关键字让泛型T成为了一个被实化的泛型。接下来就是神奇的地方，Intent接收的第二个参数应该是一个具体Activity的Class类型，但由于现在T已经是一个被实化的泛型了，因此这里我们可以直接传入T::class.java。最后调用Context的startActivity（）方法来完成Activity的启动。

那么现在启动一个Activity就可以这样写：

```Kotlin
startActivity<MainActivity>(context)
```

但是现在这个函数还是有问题的，因为我们有时还可能会使用Intent附带一些参数，如下：

```Kotlin
val intent = Intent(context, MainActivity::class.java)
intent.putExtra("param1", "data")
intent.putExtra("param2", 123)
context.startActivity(intent)
```

只需要借助高阶函数就可以解决这个问题了，在reified.kt文件中，添加一个新的startActivity（）函数的重载，如下：

```Kotlin
inline fun <reified T> startActivity(context: Context, block: Intent.() -> Unit) {
    val intent = Intent(content, T::class.java)
    intent.block()
    context.startActivity(intent)
}
```

可以看到，在函数中增加了一个函数类型参数，并且它的函数类型是定义在Intent类当中的，在创建完Intent的实例后，随即调用该函数类型参数，并把Intent的实例传入。

```Kotlin
startActivity<MainActivity>(context) {
    putExtra("param1", "data")
    putExtra("param2", 123)
}
```

## 泛型的协变

> 约定：一个泛型类或者泛型接口中的方法，它的参数列表是接收数据的地方，因此可以称它为 in 位置，而它的返回值是输出数据的地方，因此可以称它为 out 位置，如图所示：

![](https://secure2.wostatic.cn/static/b5dAwqtket8VZ3KKQ49w9x/fq1unro47EFxcrhLWQTzVh.png?auth_key=1693716191-nfvakhp3W1XLoS1er4gnrw-0-556e781136f7b0fbdcd1a12db0d43ec9)

首先定义三个类：

```Kotlin
open class Person(val name: String, val age: Int)
class Student(name: String, age: Int): Person(name, age)
class Teacher(name: String, age: Int): Person(name, age)

```

![](https://secure2.wostatic.cn/static/tYSSHteDpLB7DHkJCfx53X/d5NbpPUURJfT1qTwSsMDeo.png?auth_key=1693716191-rZCZiYLufcfw3is1EgDkzy-0-988060902269789c896883a6cbd886b5)

如果某个方法接收一个List<Person>类型的参数，而我们传入一个List<Student>的实例，这样是否合法？

这样似乎是正确的，但是在Java中却不允许，因为List<Person>不能成为List<Student>的子类，否则将可能存在类型转换的安全隐患。

为什么会出现安全隐患？

首先定义一个simpleData类：

```Kotlin
class SimpleData<T> {
    private var data: T? = null
    
    fun set(t: T?) {
        data = t
    }
    
    fun get(): T? {
        return data
    }
}
```

接着我们假设，如果问题中的做法是正确的，那么下述代码就是合法的：

```Kotlin
fun main() {
    val student = Student("name", 19)
    val data = SimpleData<Student>()
    data.set(student)
    handleSimpleData(data) //实际上这行代码会报错，这里假设它能编译通过
    val studentData = data.get()
}

fun handleSimpelData(data: SimpleData<Person>) {
    val teacher = Teacher("Jack", 35)
    data.set(teacher)
}
```

问题发生的主要原因在于我们在handlerSimpleData（）方法中向SimpleData<Person>里设置了一个Teacher实例。如果SimpleData在泛型T上是只读的话，肯定就没有类型转换的安全隐患了。

> **协变的定义：假如定义了一个MyClass<T>的泛型类，其中A是B的子类型，同时MyClass<A>又是MyClass<B>的子类型，那么我们就可以称MyClass在T这个泛型上是协变的。**

如何让MyClass<A>成为MyClass<B>的子类型呢？

只需要让MyClass<T>类中的所有方法都不能接收T类型的参数。换句话说，T只能出现在out位置，而不能出现在in位置。

现在修改SimpleData类的代码，如下：

```Kotlin
class SimpleData<out T>(val data: T?) {
    fun get(): T? {
        return data
    }
}
```

在泛型T的前面加了一个out关键字，表明T只能出现在out位置，不能在in位置，同时意味着SimpleData在泛型T上是协变的。

由于泛型T不能出现在in位置，因此我们只能通过构造函数的方式来对data赋值了。

经过修改后，下面的代码就能通过编译且没有安全隐患了：

```Kotlin
fun main() {
    val student = Student("name", 19)
    val data = SimpleData<Student>(student)
    handleSimpleData(data) 
    val studentData = data.get()
}

fun handleSimpelData(data: SimpleData<Person>) {
    val personData = data.get()
}
```

## 泛型的逆变

协变与逆变的区别：

![](https://secure2.wostatic.cn/static/3ZG3sGJmZjMwSN2ug2fuPg/image.png?auth_key=1693716232-giQ9BV4QP8c29dYiSSVVHA-0-94e6454d16eba146eb2d8363955aa38a)

先定义一个Transformer接口，用于执行一些转换操作：

```Kotlin
interface Transformer<T> {
    fun transform(t: T): String
}
```

对该接口进行实现：

```Kotlin
fun main() {
    val trans = object: Transformer<Person> {
        override fun transform(t: Person): String {
            return "${t.name} ${t.age}"
        }
    }
    handleTransformer(trans) //报错
}

fun handleTransformer(trans: Transformer<Student>) {
    val student = Student("Tom", 19)
    val result = trans.transform(student)
}
```

首先在main（）方法中编写了一个Transformer<Person>的匿名类实现，并通过transform（）方法将传入的Person对象转换成了一个“姓名+年龄”的字符串。而handleTransformer（）方法接收的是一个Transformer<Student>类型的参数，这里再handleTransformer（）方法中创建了一个Student对象，并调用参数的transform（）方法将Student对象转换成一个字符串。

这段代码从安全角度来看没有任何问题，因为Student是Person的子类，使用Transformer<Person>匿名类实现将Student转换成一个字符串也是安全的，并不存在类型转换的安全隐患。但是实际上，在调用handleTransformer（）方法时却会提示语法错误，原因是因为Transformer<Person>并不是Transfromer<Student>的子类型。

那么这个时候逆变就可以派上用场了，它就是专门处理这种情况的。修改Transformer接口中的代码：

```Kotlin
interface Transformer<in T> {
    fun transform(t: T): String
}
```

这里在泛型T的声明前面加上了一个in关键字，表明现在T只能出现在in的位置上，而不能出现在out位置上，同时也意味着Transformer在泛型T上是逆变的。

# 编写好用的工具方法

## 求N个数的最大最小值

比较多个数得到其中的最大值或最小值，如果使用max（）方法进行嵌套，那么代码会变得非常复杂，这时候可以对max（）函数进行简化，代码如下：

```Kotlin
fun Max(vararg numss: Int): Int {
    var maxNum = Int.MIN_VALUE
    for(num in nums) {
        maxNum = max(maxNum, num)
    }
    return maxNum
}
```

这里在Max（）函数的参数声明中使用了vararg关键字，这样就可以让这个函数接受任意多个同类型的参数了。接着使用一个maxNum变量来记录最大值，一开始将它赋值为整型范围内的最小值。然后在for循环中遍历参数列表，完成所有的比较之后得到全部数中的最大值。

不过，目前实现的Max（）函数还有一个缺点，那就是它只能求整型数据中的最大值，如果要想求其他类型那么自然会想到定义重载函数，但这样实现起来未免有些繁琐，并且会产生大量重复代码。这里有另一种更巧妙的办法。

Java中规定所有类型的数字都是可比较的，因此必须实现Comparable接口，在Kotlin中也是如此。那么我们就可以借助反省，将max（）函数修改成接收任意多个实现Comparable接口的参数，代码如下：

```Kotlin
fun <T: Comparable<T>> max(vararg nums: T): T {
    if(nums.isEmpty()) throw RuntimeException("params can not be empty.")
    var maxNum = nums[0]
    for(num in nums) {
        if(num > maxNum) {
            maxNum = num
        }
    }
    return maxNum
}
```

## 简化Toast的用法

首先回顾Toast的标准用法：

`Toast.makeText(context, "This is Toast",Toast.LENGTH_SHORT).show()`

由于Toast是非常常用的功能，每次都需要编写这么厂的一段代码确实让人头疼，这个时候就该考虑对Toast的用法进行简化了。

先分析一下：makeText（）方法接收3个参数：第一个参数是上下文环境，必不可少；第二个参数是Toast显示的内容，可以传入字符串和字符串资源id两种类型；第三个参数是Toast的显示时长，只支持Toast.LENGTH_SHORT和Toast.LENGTH_LONG两种值，变化不大。

那么我们就可以给String类和Int类各添加一个扩展函数，并在里面封装弹出Toast的具体逻辑。这样以后每次想要弹出Toast时，只需要调用它们的扩展函数就可以了。

新建一个Toast.kt文件，编写如下代码：

```Kotlin
fun String.showToast(context: Context) {
    Toast.makeText(context, this, Toast.LENGTH_SHORT).show()
}

fun Int.showToast(context: Context) {
    Toast.makeText(context, this, Toast.LENGTH_SHORT).show()
}
```

这里分别给String类和Int类新增了一个showToast（）函数，并且让它们接收一个Context参数。然后在函数内部，仍然使用Toast原生API用法，只是将弹出的内容改成了this，另外将Toast的显示时长固定设置成Toast.LENGTH_SHORT。

经过这样的扩展后，我们要弹出一段文字提醒就可以这么写了：

`"This is Toast".showToast(context)`

当然，这样的写法直接将显示的时长固定了，如果要变换显示时长，最简单的办法就是在showToast（）函数中再声明一个显示时长的参数，但是这样每次调用又要多传入一个参数，无疑增加了使用复杂度。

其实只要借助函数设定参数默认值的功能就可以解决这个问题了，修改原代码：

```Kotlin
fun String.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, this, Toast.LENGTH_SHORT).show()
}

fun Int.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, this, Toast.LENGTH_SHORT).show()
}
```

## 简化Snackbar的用法

先回顾一下Snackbar的常规用法，如下：

```Kotlin
Snackbar.make(view, "This is Snackbar", Snackbar.LENGTH_SHORT)
    .setAction("Action"){
        //处理具体的逻辑
    }
    .show()
```

由于make（）方法接收一个View参数，Snackbar会使用这个View自动查找最外层的布局，用于展示Snackbar。因此我们就可以给View类添加i一个扩展函数，并在里面封装显示Snackbar的具体逻辑。

新建一个Snackbar.kt文件，编写如下代码：

```Kotlin
fun View.showSnackbar(text: String, duration: Int = Snackabr.LENGTH_SHORT) {
    Snackbar.make(this, text, duration).show()
}

fun View.showSnackbar(resId: Int, duration: Int = Snackbar.LENGTH_SHORT) {
    Snackbar.make(this, resId, duration).show()
}
```

接下来利用高阶函数使其支持setAction（）方法：

```Kotlin
fun View.showSnackbar(
    text: String, acitonText: String? = null,
    duration: Int = Snackbar.LENGTH_SHORT, block:(() -> Unit)? = null
) {
    val snackbar = Snackbar.make(this, text, duration)
    if(actionText != null && block != null) {
        snackbar.setAction(actionText) {
            block()
        }
    }
    snackbar.show()
}

fun View.showSnackbar(
    resId: Int, acitonResId: Int? = null,
    duration: Int = Snackbar.LENGTH_SHORT, block:(() -> Unit)? = null
) {
    val snackbar = Snackbar.make(this, resId, duration)
    if(actionResId != null && block != null) {
        snackbar.setAction(actionResId) {
            block()
        }
    }
    snackbar.show()
}

```

这里我们给showSnackbar（）函数增加了一个函数类型参数，并且还增加了一个用于传递给setAction（）方法的字符串或字符串资源id。这里需要将新增的两个参数都设置成可为空的类型，并将默认值都设置成空，然后只有两个参数都不为空的时候，我们才去调用Snackbar的setAction（）方法来设置额外的点击事件，如果触发了点击事件，只需要调用函数类型参数将事件传递给外部的Lamba表达式即可。

这时调用Snackbar的代码就可以这样写了：

```Kotlin
view.showSnackbar("This is Snackbar", "Action"){
    //处理具体的逻辑
}
```

# 使用协程编写高效的并发程序

> 协程和线程其实有点类似，可以简单地将它理解成一种轻量级的线程。之前学习的线程是非常重量级的，它需要依靠操作系统的调度才能实现不同线程之间的切换。而使用协程却可以仅在编程语言的层面就能实现不同协程之间的切换，从而大大提升了并发编程的效率。

 **具体例子**

现在有如下foo（）和bar（）两个方法：

```Kotlin
fun foo() {
	print(1)
	print(2)
	print(3)
}

fun bar() {
	print(4)
	print(5)
	print(6)
}
```

在没有开启线程的情况下，先后调用这两个方法，理论上输出结果一定是123456。而如果使用了协程，在协程A中去调用foo（）方法，协程B中去调用bar（）方法，虽然它们仍然会运行在同一个线程当中，但是在执行foo（）方法时随时都有可能被挂起而转去执行bar（）方法，执行bar（）方法时也随时都有可能被挂起而转去执行foo（）方法，因此最终的输出结果也就不能确定了。

可以看出，协程允许我们在单线程模式下模拟多线程的效果，代码执行时的挂起与恢复完全是由编程语言来控制的，和操作系统无关。这种特性使得高并发程序的运行效率得到了极大的提升。

## 协程的基本用法

> Kotlin没有将协程纳入标准库的API中，而是以依赖库的形式提供。所以如果要使用协程功能，就需要现在app/build.gradle中添加如下依赖库：

```Groovy
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1'
implementation 'org.jetbrains.ktolinx:kotlinx-coroutines-android:1.1.1'
```

#### 使用GlobalScope.launch函数开启一个协程

```·
GlobalScope.launch { 
    println("codes run in coroutine scope.")
}
```

**GlobalScope.launch**函数可以创建一个协程的作用域，这样传递给launch函数的代码块就是在协程运行的了，上面的代码块只是打印一行日志，然而现在运行代码会发现没有任何输出。这是因为GlobalScope.launch函数每次创建的都是一个顶层协程，这种协程随着程序的结束而结束。日志没有被打印出来就是因为代码块中的代码还没来得及运行，程序就结束了。

**只要让程序延迟一段时间之后再结束，就可以解决这个问题了。**

```Kotlin
GlobalScope.launch { 
    println("codes run in coroutine scope.")
}
Thread.sleep(1000)
```

使用**Thread.sleep（）**方法让主线程阻塞一秒钟，发现日志成功打印。

**但是这种写法还存在问题，如果代码块中的代码无法在一秒钟之内完成，那么就会被强制中断。**

```Kotlin
GlobalScope.launch { 
    println("codes run in coroutine scope.")
    delay(1500)
    println("codes run in coroutine scope finished.")
}
Thread.sleep(1000)
```

这里在代码块中加了一个**delay（）**函数，并在之后又打印一行日志。delay（）函数可以让协程延迟指定之间后再运行，但它和Thread.sleep（）方法不同。delay（）函数是一个非阻塞式的挂起函数，它只会挂起当前协程，而不会影响其他协程的运行。而Thread.sleep（）方法会阻塞当前的线程，这样运行在该线程下的所有协程都会被阻塞。**注意：delay（）函数只能在协程的作用域或其他挂起函数中使用。**

#### 借助runBlocking函数让程序在协程中所有代码运行完毕之后再结束

```Kotlin
runBlocking {
    println("codes run in coroutine scope.")
    delay(1500)
    println("codes run in coroutine scope finished.")
}
Thread.sleep(1000)
```

runBlocking函数同样会创建一个作用域，但是它可以保证在协程作用域内的所有代码和子协程没有全部执行完之前一直阻塞当前线程。

**要注意的是，runBlocking函数通常只应该在测试环境下使用，在正式环境下使用会出现一些性能上的问题。**

---

> 现在两行日志都可以被打印出来了，但因为目前所有代码都是运行在同一个协程中的，似乎没有体现出什么好处。而一旦涉及高并发的应用场景，协程相较于线程的优势就显现出来了。

#### 使用launch函数创建多个线程

```Kotlin
runBlocking {
    launch {
        println("launch1")
        delay(100)
        println("launch1 finished.")
    }
    launch {
        println("launch2")
        delay(100)
        println("launch2 finished.")
    }
}
```

注意这里的launch函数和GlobalScope.launch函数不同。首先它必须在协程的作用域中才能调用，其次它会在当前协程的作用域下创建子协程。子协程的特点是如果外层作用域的协程结束了，该作用域下的所有子协程也会一同结束。相比而言，GlobalScope.launch函数创建的永远是顶层协程，这一点和线程比较像，因为线程也没有层级这一说，永远都是顶层的。

运行结果：

```Kotlin
launch1
launch2
launch1 finished.
launch2 finished.
```

可以看到，两个子协程的日志是交替打印的，说明它们确实是像多线程那样并发运行的。然而这两个子协程实际却运行在同一个线程当中，只是由编程语言来决定如何在多个线程之间进行调度，让谁运行，让谁挂起。调度的过程完全不需要操作系统参与，这也就使得协程的并发效率会出奇地高。

**下面做个实验来看看效率具体有多高：**

```Kotlin
val start = System.currentTimeMillis()
runBlocking {
    repeat(100000) {
        launch {
            println(".")
        }
    }
}
val end = System.currentTimeMillis()
println(end - start)
```

这里使用了repeat函数循环创建了10万的协程，不过在协程当中并没有进行说明有意义的操作，只是象征性地打印一个点，然后记录整个过程的耗时。

第一次运行的结果是329，这说明整个过程仅仅用时329毫秒，这足以证明协程有多么高效。试想一下，如果开启10万个线程，程序或许已经出现OOM异常了。

---

不过，随着launch函数中的逻辑越来越复杂，可能需要将部分代码提取到一个单独的函数中，这时候就产生了一个问题：我们在launch函数中编写的代码是拥有协程作用域的，但是提取到一个单独的函数中就没有协程作用域了，那么我们该如何调用像delay（）这样的挂起函数呢？

为此Kotlin提供了一个suspend关键字，使用它可以将任意函数声明成挂起函数，而挂起函数之间都是可以互相调用的。

```Kotlin
suspend fun printDot() {
    println(".")
    delay(1000)
}
```

这样就可以在printDot（）函数中调用delay（）函数了。

但是suspend关键字只能将一个函数声明成挂起函数，是无法为其提供协程作用域的。比如说现在要在printDot（）函数中调用launch函数，一定是无法调用的，因为launch函数只能在协程作用域中被调用。

#### 使用coroutineScope函数为挂起函数提供协程作用域

> coroutineScope也是一个挂起函数，因此可以在任何其他挂起函数中调用。它的特点是会继承外部的协程作用域并创建一个子作用域。

```Kotlin
suspend fun printDot() = coroutineScope {
    launch {
        println(".")
        delay(1000)
    }
}
```

可以看到，现在就能在printDot（）函数中调用launch函数了。

另外，coroutineScope函数和runBlocking函数还有点类似，它可以保证其作用域内的所有代码和子协程在全部执行完之前，会一直阻塞当前协程。

```Kotlin
runBlocking { 
    coroutineScope { 
        launch { 
            for (i in 1..10) {
                println(i)
                delay(1000)
            }
        }
    }
    println("coroutineScope finished.")
}
println("runBlocking finished.")
```

这里先使用runBlocking创建一个协程作用域，然后又使用coroutineScope函数创建一个子协程作用域。在coroutineScope的作用域中，我们调用launch函数创建了一个子协程，并通过for循环一次打印数字1到10，每次打印间隔1秒钟。最后在runBlocking和coroutineScope函数的结尾分别打印一行日志。

运行程序发现控制台先是每间隔1秒打印一个数字，然后打印coroutineScope函数结尾的日志，最后打印runBlocking函数结尾的日志。

由此可见，coroutineScope函数确实是被当前协程阻塞了，只有当它作用域内的所有代码和子协程都执行完毕之后，coroutineScope函数之后的代码才能得到执行。

---

虽然看上去coroutineScope和runBlocking函数的作用是非常相似的，但是coroutineScope函数只会阻塞当前协程，既不影响其他协程也不影响任何线程，因此不会造成任何性能上的问题。而runBlocking函数由于会阻塞当前线程，如果你恰好又在主线程中调用它的话，那么就有可能会导致界面卡死的情况。

## 更多的作用域构建器

> 目前学习过的GlobalScope.launch、runBlocking、launch、coroutineScope这几种作用域构建器都可以用于创建一个新的协程作用域。不过GlobalScope.launch和runBlocking函数可以在任意地方调用，coroutineScope函数可以在协程作用域或挂起函数中调用，而launch函数只能在协程作用域中调用。

前面提过runBlocking由于会阻塞线程，因此只建议在测试环境下使用。而GlobalScope.launch由于每次都是创建顶层协程，一般也不太建议使用，除非明确了要使用顶层协程。

- 为什么说不建议使用顶层协程？
    
    主要是因为管理成本太高了。比如我们在某个Activity中使用协程发起了一条网络请求，由于网络请求是耗时的，用户在服务器还没来得及响应的情况下就关闭了当前Acitvity，此时按理说应该取消这条网络请求，或者至少不应该进行回调，因为Activity已经不存在了，回调了也没有意义。
    

#### 调用Job对象的cancel（）方法取消协程

不管是GlobalScope.launch函数还是launch函数，它们都会返回一个Job对象，只需要调用Job对象的cancel（）方法就可以取消协程了。

```Kotlin
val job = GlobalScope.launch {}
```

但是如果我们每次创建的都是顶层协程，那么但Activity关闭时，就需要逐个调用所有已创建协程的cancel（）方法，试想一下，这样的代码是不是根本无法维护？

因此，GlobalScope.launch这种协程作用域构建器，在实际项目中也是不太常用的。下面是实际项目中比较常用的写法：

```Kotlin
val job = Job()
val scope = CoroutineScope(job)
scope.launch {
    //具体的逻辑
}
job.cancel()
```

可以看到，我们先创建了一个Job对象，然后把它传入CoroutineScope（）函数当中，注意这里的CoroutineScope（）是个函数，虽然它的命名更像是一个类。CoroutineScope（）函数返回的是一个CoroutineScope对象。有了这个对象之后，就可以随时调用它的launch函数来创建一个协程了。

现在所有调用CoroutineScope的launch函数所创建的协程，都会被关联在Job对象的作用域下面。这样只需要调用一次cancel（）方法，就可以将同一作用域内的所有协程全部取消，从而大大降低了协程管理的成本。

---

> launch函数可以创建一个协程，但只能用于执行一段逻辑，不能获取执行的结果，因为它的返回值永远是一个Job对象。

#### 使用async函数创建一个协程并获取执行结果

async函数必须在协程作用域当中才能调用，它会创建一个新的子协程并返回一个Deferred对象，如果我们想要获取async函数代码块的执行结果，只需要调用Deferred对象的await（）方法即可。

```Kotlin
runBlocking {
    val result = async {
        1 + 9
    }.await()
    println(result)
}
```

在调用了async函数之后，代码块中的代码就会立刻开始执行。当调用await（）方法时，如果代码块种中的代码还没执行完，那么await（）方法就会将当前协程阻塞住，直到可以获得async函数的执行结果。

可编写如下代码进行验证：

```Kotlin
runBlocking {
    val start = System.currentTimeMillis()
    val result1 = async {
        delay(1000)
        5 + 5
    }.await()
    val result2 = async {
        delay(1000)
        4 + 8
    }.await()
    println("result1 is $result1, result2 is $result2")
    println("result1 + result2 = ${result1 + result2}")
    val end = System.currentTimeMillis()
    println("cost ${end - start} ms.")
}
```

但是这种写法是非常低效的，因为两个async函数完全可以同时执行从而提高运行效率，现在对上述代码使用如下的写法进行修改。

```Kotlin
runBlocking {
    val start = System.currentTimeMillis()
    val deferred1 = async {
        delay(1000)
        5 + 5
    }
    val deferred2 = async {
        delay(2000)
        4 + 8
    }
    println("result1 is ${deferred1.await()}, result2 is ${deferred2.await()}")
    println("result1 + result2 = ${deferred1.await() + deferred2.await()}")
    val end = System.currentTimeMillis()
    println("cost ${end - start} milliseconds.")
}
```

现在我们不再每次调用async函数之后立刻使用await（）方法获取结果，而是仅在需要用到结果时次啊调用await（）方法来获取，这样两个async函数就变成一种并行关系了。运行之后发现代码的运行耗时缩短了一毫秒不止，效率果然提升了。

---

#### 特殊的作用域构建器——挂起函数withContext（）

示例写法：

```·
runBlocking {
    val result = withContext(Dispatchers.Default) {
        5 + 5
    }
    println(result)
}
```

调用withContext（）函数之后，会立即执行代码块中的代码，同时将当前协程阻塞住。当代码块中的代码全部执行完之后会将最后一行的执行结果作为返回值返回，因此基本上相当于`val result = async{ 5 + 5 }.await()`的写法。唯一不同的是，withContext（）函数强制要求我们指定一个**线程参数**。

---

> 我们已经知道，协程是一种轻量级的线程概念，因此很多传统编程情况下需要开启多线程执行的并发任务，现在只需要在一个线程下开启多个协程来执行就可以了。但是这并不意味着我们就永远不需要开启线程了，比如说Android中要求网络请求必须在子线程中进行，即使你开启了协程去执行网络请求，假如它是主线程当中的协程那么程序仍然会出错。这时我们就应该通过**线程参数**给协程指定一个具体的运行线程。

线程参数又三种值可选：Dispatchers.Default、[Dispatchers.IO](http://Dispatchers.IO)、Dispatchers.Main。

- Dispatchers.Default：表示会使用一种默认低并发的线程策略。当你要执行的代码属于计算密集型任务时，开启过高的并发反而可能会影响任务的运行效率，此时就可以使用Dispatchers.Default。
- [Dispatchers.IO](http://Dispatchers.IO)：表示会使用一种叫高并发的线程策略。当你要执行的代码大多数时间是在阻塞和等待中，比如说执行网络请求时，为了能够支持更高的并发数量，[此时就可以使用Dispatchers.IO](http://xn--Dispatchers-2u0rs6dm82a8g3akx6aekt4f4b.IO)。
- Dispatchers.Main：表示不会开启子线程，而是在Android主线程中执行代码，但是这个值只能在Android项目中使用，纯Kotlin程序使用这种类型的线程参数会出现错误。

事实上，在刚才所学的协程作用域构建器中，除了coroutineScope函数之外，其他所有的函数都是可以指定这样一个线程参数的，只不过withContext（）函数是强制要求指定的，而其他函数则是可选的。

## 使用协程简化回调的写法

> 在【网络请求的回调方式】小节中，我们学习了编程语言的回调机制，并使用这个机制实现了获取一部网络请求数据响应的功能。不过，回调机制基本上是依靠匿名类来实现的，但是匿名类的写法通常比较繁琐，比如下面的代码。

```Kotlin
HttpUtil.sendHttpRequest(address, object: HttpCallbackListener {
    override fun onFinish(response: String) {
        //得到服务器返回的具体内容
    }
    
    override fun onError(e: Exception) {
        //在这里对异常情况进行处理
    }
})
```

在多少个地方发起网络请求，就需要编写多少次这样匿名类实例，这就让我们不得不思考有没有一种更简单的写法。

在过去，可能确实没有什么更简单的写法了。但是现在有了Kotlin的协程，只需要借助suspendCoroutine函数救恩那个将传统回调机制的写法大幅简化。

#### suspendCoroutine函数的用法

> suspendCoroutine函数必须在协程作用域或挂起函数中才能调用，它接收一个Lambda表达式参数，主要作用是将当前协程立即挂起，然后在一个普通的线程中执行Lambda表达式中的代码。Lambda表达式的参数列表上会传入一个Continuation参数，调用它的resume（）方法或resumeWithException（）可以让协程恢复执行。

首先定义一个request（）函数：

```Kotlin
suspend fun request(address: String): String {
    return suspendCoroutine { continuation -> 
        HttpUtil.sendHttpRequest(address, object: HttpCallbackListener {
            override fun onFinish(response: String) {
                continuation.resume(response)
            }
            
            override fun onError(e: Exception) {
                continuation.resumeWithException(e)
            }
        })
    }
}
```

可以看到，request（）函数是一个挂起函数，并且接收一个address参数。在request（）函数内部，我们调用了刚刚介绍的suspendCoroutine函数，这样当前协程就会被立刻挂起，而Lambda表达式中的代码则会在普通线程中执行。接着我们在Lambda表达式中调用HttpUtil.sendHttpRequest（）方法发起网络请求，并通过传统回调的方式监听请求结果。如果请求成功就调用Continuation的resume（）方法恢复被挂起的协程，并传入服务器相应的数据，该值会成为suspendCoroutine函数的返回值。如果请求失败，就调用Continuation的resumeWithException（）恢复被挂起的协程，并传入具体的异常原因。

你可能会说，这里不是仍然使用了传统回调的写法吗？这是因为，不管之后我们要发起多少次网络请求，都不需要再重复进行回调实现了。比如说获取百度首页的响应数据，就可以这样写：

```Kotlin
suspend fun getBaiduResponse() {
    try {
        val response = request("http://www.baidu.com")
        //对服务器相应的数据进行处理
    } catch (e: Exception) {
        //对异常情况进行处理
    }
}
```

由于getBaiduResponse（）函数被声明成了挂起函数，这样它也只能在协程作用域或其他挂起函数中调用了，使用起来是不是非常有局限性？确实如此，因为suspendCoroutine函数本身就是要结合协程一起使用的。不过通过合理的项目架构设计，我们可以轻松地将各种协程的代码应用到一个普通的项目中。

---

**事实上，suspendCoroutine函数几乎可以用于简化任何回调的写法，比如之前使用Retrofit来发起网络请求需要这样写：**

```·
val appService = ServiceCreator.create<AppService>()
appService.getAppData().enqueue(object: Callback<List<App>>) {
    override fun onResponse(call: Call<List<App>>, response: Response<List<App>>) {
        //得到服务器返回的数据
    }
    
    override fun onFailure(call: Call<List<App>>, t: Throwable) {
        //在这里对异常情况进行处理
    }
}
```

下面使用suspendCoroutine函数来简化：

由于不同的Service接口返回的数据类型也不同，所以我们这次不能像刚才那样针对具体的类型进行编程了，而是要使用泛型的方式。定义一个await（）函数，代码如下：

```Kotlin
suspend fun <T> Call<T>.await(): T {
    return suspendCoroutine { continuation -> 
        enqueue(object : Callback<T> {
            override fun onResponse(call: Call<T>, response: Response<T>) {
                val body = response.body()
                if(body != null) continuation.resume(body)
                else continuation.resumeWithException(RuntimeException("response body is null"))
            }
            
            override fun onFailure(call: Call<T>, t: Throwable) {
                continuation.resumeWithException(t)
            }
        })
    }
}

```

这段代码相比于刚才的request（）函数又复杂了一点。首先await（）函数仍然是一个挂起函数，然后我们给它声明了一个泛型T，并将await（）函数定义成了Call<T>的扩展函数，这样所有返回值是Call类型的Retrofit网络请求接口就都可以直接调用await（）函数了。

接着，await（）函数中使用了suspendCoroutine函数来挂起当前协程，并且由于扩展函数的原因，我们拥有了Call对象的上下文，那么这里就可以直接调用enqueue（）方法让Retrofit发起网络请求。接下俩，使用同样的方式对Retrofit响应的数据或者网络请求失败的情况进行处理就可以了。另外还有一点需要注意，在onResponse（）方法的回调中，我们调用body（）方法解析出来的对象是可能为空的。如果为空的话，这里的做法是手动抛出异常，当然也可以根据自己的逻辑来处理。

有了await（）函数之后，我们调用所有Retrofit的Service接口都会变得极其简单，比如刚才同样的功能就可以使用下面的写法来实现：

```Kotlin
suspend fun getAppData() {
    try {
        val appList = ServiceCreator.create<AppService>().getAppData().await()
        //对服务器响应的数据进行处理
    } catch {
        //对异常情况进行处理
    }
}
```

没有了冗长的匿名类实现，只需要简单调用以下await（）函数就可以让Retrofit发起网络请求，并直接获得服务器响应的数据。当然你会觉得每次发起网络请求都要进行以下try catch处理也比较麻烦，其实这里我们也可以选择不处理。在不处理的情况下，如果发生了异常就会一层层向上抛出，一直到被某一层的函数处理了为止。因此，我们也可以在某个统一的入口函数中只进行一次try catch，从而让代码变得更加精简。

# 使用DSL构建专有的语法结构

> DSL的全称是领域特定语言（Domain Specific Language），它是编程语言赋予开发者的一种特殊能力，通过它我们可以编写出一些看似脱离其原始语法结构的代码，从而构建出一种专有的语法结构。在Kotlin中实现DSL的实现方式并不固定，比如使用infix函数构建出特有语法结构就属于DSL。本节主要目标是通过高阶函数的方式来实现DSL。

当我们在项目中添加一些依赖库时其实就是在使用DSL，比如：

```Groovy
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.6.1'
}
```

Gradle是一种基于Groovy语言的构建工具，因此上述语法结构其实就是Groovy提供的DSL功能。

借助Kotlin的DSL，我们也可以实现类似的语法结构。

首先新建一个DSL.kt文件，在里面定义一个Dependency类，如下：

```Kotlin
class Dependency {
    
    val libraries = ArrayList<String>()
    
    fun implementation(lib: String) {
        libraries.add(lib)
    }
    
}
```

接下来定义一个dependencies高阶函数，如下：

```Kotlin
fun dependencies(block: Dependency.() -> Unit): List<String> {
    val dependency = Dependency()
    dependency.block()
    return dependency.libraries
}
```

dependencies函数接受一个函数类型参数，并且该参数是定义到Dependency类中的，因此调用它的时候需要先创建一个Dependency的实例，然后再通过该实例调用函数类型参数，这样出阿努人的Lambda表达式就能得到执行了。最后，将Dependency类中保存的依赖库集合返回。

经过这样的DSL设计之后，我们就可以在项目中使用如下的语法结构了：

```Kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.6.1")
}
```

当然，这种语法结构与build.gradle文件中使用的语法结构并不完全相同，这主要是因为Kotlin和Groovy在语法层面上还是有一定差别的。

另外，我们也可以通过dependencies函数的返回值来获取所有添加的依赖库，代码如下：

```Kotlin
val libraries = dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.6.1")
    implementation("com.squareup.retrofit2:converter-gson:2.6.1")
}
for(lib in libraries) {
    println(lib)
}
```

这种语法结构比起直接调用Dependency对象的implementation（）方法要更直观一些，而且你会发现添加的依赖库越多，使用DSL写法的优势就越明显。

---

接下来我们尝试编写一个复杂一点的DSL。

如果你了解前端开发的话，应该知道网页的展示都是由浏览器解析HTML代码来实现的，HTML中定义了很多标签，其中<table>标签用于创建一个表格，<tr>标签用于创建表格的行，<td>标签用于创建单元格。将这三种标签嵌套使用，就可以定制出包含任意行列的表格了。

这里我们做个实验，首先创建一个test.txt文件，编写如下HTML代码：

```HTML
<table>
    <tr>
        <td>Apple</td>
        <td>Grape</td>
        <td>Orange</td>
    </tr>
    <tr>
        <td>Pear</td>
        <td>Banana</td>
        <td>Watermelon</td>
    </tr>
</table>
```

将文件后缀名改为html，打开文件进行验证：

![](https://secure2.wostatic.cn/static/2CbJxeiaKWPyDNPtFr1UsA/image.png?auth_key=1693716637-qM4CsCvv5BvnxJ8KSp5dqD-0-cf33879583a65d0ce9cfdc7e9386b74d)

这就是一个两行三列表格的效果，只是默认情况下表格边框的宽度为0，所以我们看不见边框。

如果现在有一个需求，要求我们在Kotlin中动态生成表格所对应的HTML代码，你会怎么做？最简单的方式就是字符串拼接了，但是这样显然十分繁琐，而且字符串拼接的代码也难以阅读。

借助DSL我们可以以一种不可思议的语法结构来实现这个功能。

首先定义一个Td类，代码如下：

```Kotlin
class Td {
    var content = ""
    
    fun html() = "\n\t\t<td>$content</td>"
}
```

由于<td>标签表示一个单元格，其中必然是要包含内容的，于是我们使用了一个content字段来存储单元格中显示的内容。另外还提供了一个html（）方法，当调用这个方法时就返回一段<td>标签的HTML代码，并将content中存储的内容拼接进去。注意，为了让最终输出结果更为直观，这里使用了\n和\t转义符来进行换行和缩进，当然可以不加这些转义符，因为浏览器在解析HTML代码时是忽略换行和缩进的。

完成了Td类，我们再定义一个Tr类，代码如下：

```Kotlin
class Tr {
    private val children = ArrayList<Td>()

    fun td(block: Td.() -> String) {
        val td = Td()
        td.content = td.block()
        children.add(td)
    }

    fun html(): String {
        val builder = StringBuilder()
        builder.append("\n\t<tr>")
        for (childTag in children) {
            builder.append(childTag.html())
        }
        builder.append("\n\t</tr>")
        return builder.toString()
    }

}
```

由于<tr>标签表示行，可以包含多个<td>标签，因此我们首先创建了一个children集合，用于存储当前Tr所包含的Td对象。接下来提供了一个td（）函数，它接收一个定义到Td类中并且返回值是String的函数类型参数。当调用td（）函数时，会先创建一个Td对象，接着调用函数类型参数并获取它的返回值，然后赋值到Td类的content字段中，这样就可以将调用td（）函数时传入的Lambda表达式的返回值赋值给content字段了。当然，这里既然创建了一个Td对象，就一定要记得将它添加到children集合中。

另外，Tr类中也定义了一个html（）方法，它的作用和Td类中的html（）方法一致。只是由于每个Tr都可能会包含很多个Td，因此我们需要使用循环来遍历children集合，将所有的子Td都拼接到<tr>标签当中，从而返回一段嵌套的HTML代码。

我们现在就可以使用如下的语法结构来构建表格中的一行数据：

```Kotlin
val tr = Tr()
tr.td { "Apple" }
tr.td { "Grape" }
tr.td { "Orange" }
```

这仍然不是我们追求的最终效果。接下来继续对DSL进行完善，再定义一个Table类，代码如下：

```Kotlin
class Table {
    private val children = ArrayList<Tr>()
    
    fun tr(block: Tr.() -> String) {
        val tr = Tr()
        tr.block()
        children.add(tr)
    }
    
    fun html(): String {
        val builder = StringBuilder()
        builder.append("<table>")
        for (childTag in children) {
            builder.append(childTag.html())
        }
        builder.append("</table>")
        return builder.toString()
    }
    
}
```

现在就可以使用如下语法结构来构建一个表格了：

```Kotlin
val table = Table()
table.tr { 
    td { "Apple" }
    td { "Grape" }
    td { "Orange" } 
}
table.tr {
    td { "Pear" }
    td { "Banana" }
    td { "Watermelon" }
}
```

再进一步精简，定义一个table（）函数，代码如下：

```Kotlin
fun table(block: Table.() -> Unit): String {
    val table = Table()
    table.block()
    return table.html()
}
```

最终版：

```Kotlin
val html = table {
    tr {
        td { "Apple" }
        td { "Grape" }
        td { "Orange" }
    }
    tr {
        td { "Pear" }
        td { "Banana" }
        td { "Watermelon" }
    }
}
println(html)

```

---

另外，在DSL中还可以利用Kotlin的其他语法忒选哪个，比如通过循环来批量生成<tr>和<td>标签：

```Kotlin
val html = table { 
    repeat(2) {
        tr { 
            val fruits = listOf<String>("Apple", "Grape", "Orange")
            for (fruit in fruits) {
                td { fruit }
            }
        }
    }
}
println(html) 
```