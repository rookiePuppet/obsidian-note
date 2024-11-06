
#### 第一个Lua程序

```lua
-- 单行注释
print("Hello, World") --打印Hello，World到控制台
--[[
    多行注释1
]] --[[
    多行注释2
--]] --[[
    多行注释3
]] --

```

# 变量

Lua中的变量类型：

1. nil，number，string，boolean
2. 函数function
3. 表table
4. 数据结构userdata
5. 协同程序thread

**Lua中所有的变量声明都不需要指定变量类型，Lua会自动判断变量类型，通过type函数可以获得变量的类型。**

```lua
a = nil --nil类似C#中的null，代表空类型
a = 1 --所有的数值都是number类型
a = 1.5
a = "as12e" --字符串声明用双引号或单引号包裹，Lua中没有char类型
a = '12sd'
a = true --布尔类型
```

```lua
--type(a)返回的是字符串类型，表示变量的类型
a = 1
print(type(a)) --number
a = false
print(type(a)) --boolean
```

```lua
--Lua中可以直接使用没有初始化的变量，默认为nil
print(b) --nil
```

# 字符串

#### 获取字符串长度

```lua
--在字符串变量名前面加#即可获取长度
--一个英文字符长度为1，一个中文字符长度为3
s = "abc哈哈哈"
print(#s) --12
```

#### 字符串多行打印

```lua
--使用转义字符
print("123\n123")
--使用[[]]
s = [[
我是
puppet
]]
print(s)

```

#### 字符串拼接

```lua
--使用..
print("123".."456")
s1 = 123
s2 = 456
print(s1..s2)
--使用string.format
print(string.format("我今年%d岁了", 18))

```

#### 其他类型转字符串

```lua
a = true
b = tostring(a)
```

#### string中的公共方法

```lua
str = "abCdefgCd"
--小写转大写
string.upper(str)
--大写转小写
string.lower(str)
--反转字符串
string.reverse(str)
--字符串按索引查找
string.find(str, "Cde")
--截取字符串
string.sub(str, 3, 5)
--字符串重复
string.rep(str, 2)
--字符串修改
string.gsub(str, "Cd")
--字符转ASCII码
string.byte("Lua", 1)
--ASCII码转字符
string.char(79)
```

# 运算符

#### 算数运算符

Lua中的算数运算符有：+ - \* / %和^(幂运算)，但没有自增自减以及复合运算符。

字符串在与数值进行运算时会自动转换成数值类型进行计算。

#### 条件运算符

Lua中的条件运算符有：> < ≥ ≤ ==和\~=(不等于)。

#### 逻辑运算符

Lua中的逻辑运算符有：and(与) or(或) not(非)。

#### 位运算符

Lua不支持位运算，需要自己实现。

#### 三目运算符

Lua不支持三目运算符。

# 条件分支语句

```lua
a = 10

if a > 10 then
  print("a > 10")
elseif a < 10 then
  print("a < 10")
else
  print("a = 10")
end
```

# 循环语句

```lua
num = 0

--while语句
while num < 5 do
  print(num)
  num = num + 1
end

--repeat until语句
repeat
  print(num)
  num = num + 1
until num > 5

--for语句
for i = 1,5 do --i默认递增
  print(i)
end

for i = 1,5,2 do --第三个数自定义增量
  print(i)
end
```

# 函数

#### 无参无返回值

```lua
function F1()
  print("F1函数")
end
F1()

F2 = function()
  print("F2函数")
end
F2()
```

#### 有参无返回值

```lua
function F3(a)
  print(a)
end
F3(1)
F3("123")
F3(true)
--如果传入的参数和函数定义的参数个数不匹配，不会报错，只会补空或者丢弃
F3()
F3(1, 2, 3)
```

#### 有参有返回值

```lua
function F4(a)
  return a, "123", true
end
--多返回值时，在前面声明多个变量来接取即可
--变量数量不匹配也不会出错
temp, temp2, temp3 = F4("1")

```

#### 函数类型

```lua
F5 = function
  print("123")
end
print(type(F5)) --function
```

#### 函数重载

Lua中不支持函数重载，默认调用最后一个声明的同名函数。

#### 变长参数

```lua
function F7(...)
  --变长参数用一个表存起来再使用
  arg = {...}
  for i=1, #arg do
    print(arg[i])
  end
end
F7(1, "123", true, 4, 5, 6)
```

#### 函数嵌套

```lua
function F8()
  return function()
    print(123)
  end
end
f9 = F8()
f9()

```

#### 闭包

```lua
--闭包就是在一个函数中返回另一个函数，改变了外部函数变量的生命周期
function F9(x)
  return function(y)
    return x + y
  end
end
f10 = F9(10)
print(f10(5)) --15
```

# 表

## 数组

```lua
--声明数组
a = {1, 2, nil, 4, "1231", true}
--访问数组元素
print(a[1]) --Lua中索引从1开始
--获取数组长度
print(#a) --输出2，数组中的nil会被忽略，并且nil后面的元素都不会被访问到
```

### 数组的遍历

```lua
--因为nil后面的元素都不会被访问到，所以输出结果只有1，2
for i = 1, #a do
  print(a[i])
end
```

### 二维数组

```lua
a = {{1, 2, 3}, {4, 5, 6}}
print(a[1][1]) --1

```

### 二维数组的遍历

```lua
for i = 1, #a do
  b = a[i]
  for j = 1, #b do
    print(b[j])
  end
end
```

### 自定义索引

```lua
aa = {[0]=1, 2, 3, [-1]=4, 5}
print(aa[0]) --1
print(aa[-1]) --4
print(aa[1]) --2
print(aa[2]) --3
print(aa[3]) --5
print(#aa) --3，忽略小于等于0的索引

bb = {[1]=1, [2]=2, [4]=4, [5]=5}
print(#bb) --5, 如果索引之间最大间隔1，长度为最大的索引值
bb = {[1]=1, [2]=2, [5]=4, [6]=5}
print(#bb) --2, 如果索引之间最大间隔大于1，长度为最大间隔为1时的最大索引值

```

### 迭代器遍历

> 主要用来遍历表，一般不使用#获取长度来遍历表，因为不准确。

```lua
a = {[0]=1, 2, 3, [-1]=4, 5, [5] = 6}

--ipairs，从索引值1开始遍历，索引小于等于0以及索引断开处后面的值遍历不到
for i, k in ipairs(a) do
  print("ipairs遍历键值"..i.."_"..k)
end

--pairs，能遍历到所有的值，可以只遍历键
for i, v in pairs(a) do
  print("pairs遍历键值"..i.."_"..v)
end
```

## 字典

```lua
--字典由键值对构成
a = {["name"] = "布偶", ["age"] = 14, ["1"] = 5}

--访问
print(a["name"]) --用中括号填键的方式来访问单个元素
print(a["age"])
print(a.name) --还可以使用类似访问成员变量的形式来访问，但键不能是数字
--print(a.1) --如果键表示的是数字，就不能这样访问

--修改
a["1"] = "哈哈哈"

--添加
a["sex"] = true

--删除
a["sex"] = nil
```

### 遍历字典

```lua
--只能使用pairs遍历
for k, v in pairs(a) do
  print(k, v)
end
```

## 类和结构体

> Lua中是没有面向对象的，需要我们自己实现。

```lua
Student = {
  age = 1,
  sex = true,
  Up = function()
    --print(age) --不能这样写，需要指定是谁的age
    print(Student.age)
    print("up函数")
  end
  Learn = function(s)
    print(s.age) --还可以通过传参数的方式把Student传进来获取成员变量和成员函数
    print("learn函数")
  end
}
--Lua中类的表现，更像是一个类中有很多静态变量和函数
print(Student.age)
Student.Up()

--可以在外部去声明类的成员变量和函数
Student.name = "布偶"
Student.Speak = function()
  print("speak")
end
function Student:Read()
  print(self.name) --使用:声明函数时，self表示传入的第一个参数
  print("read")
end

--Lua中.和:的区别
Student.Learn(Student)
Student:Learn() --使用:调用函数会默认把调用者作为第一个参数传进函数
Student.Read()
```

## 表的公共方法

#### 插入元素

```lua
t1 = {{age = 1, name = "123"}, {age = 2, name = "345"}}
t2 = {name = "puppet", sex = true}

table.insert(t1, t2) --将t2插入到t1中
```

#### 删除元素

```lua
table.remove(t1) --删除t1中最后一个元素
table.remove(t1, 1) --删除t1中索引值为1的元素
```

#### 排序

```lua
t2 = {5, 2, 7, 9, 4}
table.sort(t2) --默认升序排序
table.sort(t2, function(a, b) --第二个参数可以传一个排序规则函数
  if a > b then
    return true
  end
end) --降序排序
```

#### 拼接

```lua
tb = {"123", "455", "356", "12567"}
str = table.concat(tab, ",") --将表中的元素拼接成一个字符串，用,分隔（只支持字符串和数值）
print(str)
```

# 多脚本执行

## 全局变量和局部变量

```lua
--这样声明的变量都是全局变量
a = 1
b = "53asd"

--在循环和函数中声明的变量默认也都是全局变量
for i = 1, 2 do
  name = "puppet"
end
print(name) --在循环外部也能访问到循环内声明的变量
fun = function()
  age = 10
end
print(age) --函数也同理

--使用local关键字声明局部变量
func = function() {
  local word = "elegant"
}
print(word) --nil，因为word是func函数中的局部变量，在这里无法访问
```

## 多脚本执行

> 使用require关键字可以执行指定lua脚本。

```lua
--Test.lua
print("Test脚本执行")
testA = 123
local testLocalA = 451
return testLocal --可以将某个变量return给外部

--MyLuaScript.lua
require("Test") --Test是另一个lua脚本的名字
print(testA) --可以访问到已经执行过的Test脚本中的testA变量(全局变量)
pirnt(testLocalA) --无法访问Test脚本中的局部变量
```

## 脚本卸载

> 使用require加载执行过的脚本，在卸载之前无法再次执行。

```lua
print(package.loaded["Test"]) --package.loaded["脚本名"]可以判断是否执行过某个脚本
package.loaded["Test"] = nil --这句代码就是卸载脚本
require("Test") --Test脚本可以再次被执行
```

## \_G表

> \_G表是一个总表（table），其中存储了我们声明的所有全局变量。

```lua
for k, v in pairs(_G) do
  print(k, v)
end
```

# 特殊用法

## 多变量赋值

```lua
a, b, c = 1, 2, "123"
print(a)
print(b)
print(c)
--如果后面赋值的变量不够，会自动补空
a, b, c = 1, 2
print(a)
print(b)
print(c) --nil
--如果后面复制的变量多了，会自动省略
a, b, c = 1, 2, 3, 4, 5
print(a)
print(b)
print(c)

```

## 多返回值

```lua
function Test()
  return 10, 20, 30, 40
end

a, b, c, d= Test()
print(a)
print(b)
print(c)
print(d) --nil
```

## 利用and和or模拟三目运算

> and和or不仅可以连接两个布尔值，其他任何变量都可以连接。在lua中认为只有nil和false是假，其他变量和数值都是真。

```lua
x = 1
y = 2
local res = (x > y) and x or y
print(res) --y

```

# 协同程序

## 协程的创建

```lua
--方式1：coroutine.create()
fun = function()
  print(123)
end
co = coroutine.create(fun)
pirnt(co) --协程的本质是一个线程对象
print(type(co))

--方式2：coroutine.wrap()
co2 = coroutine.wrap(fun)
print(co2)
print(type(co2))
```

## 协程的运行

```lua
--方式1：对应通过create创建的协程
coroutine.resume(co)

--方式2：对应通过wrap创建的协程
co2()
```

## 协程的挂起

```lua
fun2 = function()
  local i = 1
  while true do
    print(i)
    i = i + 1
    coroutine.yield(i) --挂起协程
  end
end

co3 = coroutine.create(fun2)
--因为lua脚本是从上到下只执行一遍的，没有循环，所以每次挂起之后都要通过resume来恢复执行
isOk, tempi = coroutine.resume(co3) --1
print(isOk, tempi) --true 2 协程挂起返回的第一个参数是一个布尔值代表挂起是否成功
coroutine.resume(co3) --2
coroutine.resume(co3) --3

co4 = coroutine.wrap(fun2)
print("返回值"..co4()) --这种方式执行协程就没有上面那样的第一个返回值
```

## 协程的状态

> 协程的状态有三种：dead(结束), suspended(挂起), running(运行中)。

```lua
print(coroutine.status(co3))
print(coroutine.running()) --可以得到当前正在运行的协程的线程号
```

# 元表

> 任何表变量都可以作为另一个表变量的元表，任何表变量都可以有自己的元表。当我们子表中进行一些特定操作时，会执行元表中的内容。

## 设置元表

```lua
meta = {}
myTable = {}
--设置元表函数：param1->子表，param2->元表
setmetatable(myTable, meta)
```

## 特定操作

### \_\_tostring

```lua
meta2 = {
  --当子表被当作字符串使用时，会默认调用这个元表中的tostirng方法
  __tostring = function(t)
    return t.name
  end
}
myTable2 = {
  name = "布偶"
}
setmetatable(myTable2, meta2)
print(myTable2) --布偶
```

### \_\_call

```lua
meta3 = {
  __tostring = function(t)
    return t.name
  end
  --当子表被当作函数使用时，会默认调用这个__call中的内容
  --当希望传参数时，默认的第一个参数是调用者本身
  __call = function(t, b)
    print(t)
    print(b)
    print("hhhhh")
  end
}
myTable3 = {
  name = "布偶"
}
setmetatable(myTable3, meta3)
myTable3(10) --布偶 10 hhhhh
```

### 运算符重载

```lua
meta4 = {
  __add = function(t1, t2)
    return t1.age + t2.age
  end
  
  --其他运算符: sub, mul, div, mod, eq, pow, concat, lt, le
}
myTable4 = {
  age = 20
}
myTable5 = {
  age = 10
}
setmetatable(myTable4, meta4)
setmetatable(myTable5, meta4)
--使用条件运算符比较两个对象，这两个对象的元表必须一致才能准确比较
print(myTable4 + myTable5)
```

### \_\_**index和**\_\_newIndex

```lua
meta6 = {
  age = 20
}
--__index写在表外面来初始化
meta6.__index = meta6
myTable6 = {}
--当在子表中找不到某个属性时，会到元表__index指定的表中去找
print(myTable6.age) --20
--rawget方法会会略index指定的表，只会找自己身上的变量
print(rawget(myTable6, "age")) --nil
```

```lua
meta7 = {}
meta7.__newIndex = {}
myTable7 = {}
setmetatable(myTable7, meta7)
--当赋值时，如果赋值一个不存在的索引，那么会把这个值赋给__newIndex指定的表，不会修改自己
myTable7.age = 10
print(myTable7.age) --nil
--rawset方法会忽略newIndex的设置，只会改自己的变量
rawset(myTable, "age", 20)
print(myTable7.age) --nil
```

# 面向对象

## 封装

```lua
Object = {}
Object.id = 1

function Object:new()
  --对象就是一个变量，本质上是表对象
  local obj = {}
  self.__index = self
  --将Object设置成obj的元表
  setmetatable(obj, self)
  return obj
end

local myObj = Object:new()
print(myObj)
print(myObj.id) --1
```

## 继承

```lua
function Object:subClass(className)
  _G[className] = {}
  local obj = _G[className]
  self.__index = self
  setmetatable(obj, self)
end

Object:subClass("Person")
print(Person.id) --1

local p1 = Person:new()
print(p1.id) --1
p1.id = 2
print(p1.id) --2
```

## 多态

```lua
function Object:subClass(className)
  _G[className] = {}
  local obj = _G[className]
  self._index = self
  --给子类定义一个base属性表示父类
   obj.base = self 
  setmetatable(obj, self)
end

Object.subClass("GameObject")
GameObject.posX = 0;
GameObject.posY = 0;
function GameObject:Move()
  self.posX = self.posX + 1
  self.posY = self.posY + 1
  print(self.posX, self.posY)
end

GameObject:subClass("Player")
--重写父类函数
function Player:Move()
    --避免把基类表传入基类的Move方法中，否则就相当于只是让基类自己执行了Move，而子类没有执行
    --self.base:Move()
    --通过.调用将自己传入基类Move方法
    self.base.Move(self)
end

local p1 = Player:new()
p1:Move() --1 1
local p2 = Player:new()
p2:Move() --1 1

```

# 自带库

## 时间相关

```lua
--系统时间
os.time()
os.time({year = 2002, month = 6, day = 9})

local time = os.data("*t")
for k,v in pairs(time) do
  print(k,v)
end

```

## 数学运算相关

```lua
--绝对值
math.abs(-1)
--弧度转角度
math.deg(math.pi)
--三角函数
math.cos(math.pi)
--向上取整
math.ceil(2.5)
--向下取整
math.floor(2.9)
--最大最小值
math.max(1, 0)
math.min(-1, 19)
--小数分离
math.modf(1.2)
--幂运算
math.pow(2, 5)
--随机数
math.randomseed(os.time())
math.random(199)

```

## 路径相关

```lua
--lua脚本加载路径
package.path
package.path = package.path .. ";C:\\"

```

# 垃圾回收

> 垃圾回收的关键字：collectgarbage

```lua
--获取当前lua占用内存数/KB
print(collectgarbage("count"))
--让对象置空就变成垃圾
test = nil
--执行垃圾回收
collectgarbage("collect")
```
