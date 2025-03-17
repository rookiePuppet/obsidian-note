# 迭代器

迭代器是包含迭代器块的方法或属性，迭代器块本质上是包含yield return或yield break语句的代码，只能用于返回类型为IEnumerable或IEnumerator及其泛型版本的方法或属性。

迭代器的生成类型指的是迭代器的返回数据的类型，非泛型接口返回类型的迭代器的生成类型为object，泛型接口返回类型的迭代器是泛型接口的实参类型。

yield return语句用于生成返回序列的每一个值，yield break语句用于终止返回序列。

## 延迟执行

延迟执行的基本思想是：只在需要获取计算结果时执行代码。

IEnumerable是可用于迭代的序列，而IEnumerator则是序列的游标，两者类似于书与书签的关系。

IEnumerable.GetEunmerator方法可以创建一个IEnumerator，然后调用其MoveNext方法移动游标，若MoveNext返回true，表示游标移动到了一个可通过Current属性来访问的元素，若返回false，则表示游标已经移动到了序列的末尾。

在迭代器中遇到以下情况时，代码会终止执行：

- 抛出异常
- 方法执行完毕
- 遇到yield break语句
- 执行到yield return语句，迭代器准备返回值