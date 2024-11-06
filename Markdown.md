# 概述

## 设计理念

Markdown视**可读性**为最高准则，致力于使阅读和创作文档变得容易。Markdown语法灵感的最大来源是**纯文本email**的格式，完全由标点符号组成。

## 内联HTML

Markdown用于创作web文档，但Markdown并不是要取代HTML，而是使阅读、创作和编辑文章变得简单。HTML是一种发布格式，而Markdown是一种**创作格式**，其处理的都是纯文本。

Markdown的语法实现了一部分的HTML标签，对于其他未包含在Markdown中的标签，可以直接使用HTML标签。

例如，将HTML表格添加在Markdown文件中：

```markdown
This is a regular paragragh.

<table>
	<tr>
		<td>Foo</td>
	</tr>
</table>

This is another regular paragragh.
```

需要注意的是，Markdown语法在HTML块级元素中不会被处理，也就是说在HTML中无法使用Markdown的语法。

HTML内联元素例如`<span>`, `<cite>`和`<del>`可以在Markdown段落、列表项、标题中任意使用，甚至可以直接使用HTML标签替代Markdown格式。

## 特殊字符自动转义

在HTML中，对于一些特殊符号，如果要将它们作为字面量，就必须转译为字符实体。例如`<`和`&`，需要使用`&lt;`和`&amp;`表示。

这种写法不仅容易出错，也不便于记忆。

但是在Markdown中可以自由地使用这些特殊字符，特殊字符在生成HTML时会被自动转义。

因此，如果希望得到版权符号，可以写`@copy`，Markdown不会对其转义。

但如果写`AT&T`，Markdown就会将其转译为`AT&amp;T`

总而言之，在Markdown的块级元素和内联元素中，尖括号和英镑符号总是被自动编码，使得用Markdown来写HTML代码很容易。

# 块级元素

## 段落和换行

段落就是连续行上的文本，空行将文本划分为多个段落。**普通段落不应使用缩进**。

段落是由一行或多行连续文本组成的，这条规则使Markdown支持“硬换行”。 

## 标题

Markdown支持两种形式的标题，[Setext]和[Atx]

Setext样式的标题使用等号表示一级标题，使用连字符表示二级标题，例如：

```
This is an H1
=============

This is an H2
-------------
```

Atx样式的标题在行开头使用1~6个井号，分别对应1~6级标题，例如：
```
# This is an H1

## This is an H2

###### This is an H3

```

标题行尾可以也加上井号以匹配行首，但井号数量不需要和行首一致，因为是行首井号数量决定标题的级别。

## 块引用

Markdown使用email样式的`>`字符作为块引用，最好对引用文本采取强制换行，并在每一行行首写一个`>`：

```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, 
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. 
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse 
> id sem consectetuer libero luctus adipiscing.
```

在Markdown中只需要在每一个需要强制换行的段落首行前面加一个`>`即可实现同样的效果：

```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse id sem consectetuer libero luctus adipiscing.
```

块引用可以嵌套：

1. 块引用中可以包含块引用
	```	
	> This is the first level of quoting.
	>  
	> > This is nested blockquote.
	> 
	> Back to the first level.
	```
2. 块引用中可以包含Markdown元素
	```
	> ## This is a header.
	> 
	> 1. This is the first list item. 
	> 2. This is the second list item. 
	> 
	> Here's some example code: 
	> 
	> return shell_exec("echo $input | $markdown_script");
	```

## 列表

Markdown支持[[#^e9fc60|有序列表]]和[[#^b9abfc|无序列表]]。

无序列表可以使用星号，加号和连字符来标记。 ^b9abfc

```
* Red 
+ Green 
- Blue
```

有序列表使用数字加句号来标记。 ^e9fc60

```
1. Bird
2. McHale
3. Parish
```

即使使用无序的列表序号来标记有序列表，最终生成的列表序号也是有序的。

若无意创建有序列表，例如：

```
1986. What a great season.
```

可以通过转义符号来取消：

```
1986\. What a great season.
```

## 代码块

预格式化的代码块用于输出编程语言和标记语言。不同于普通段落，代码块中的行会被以原样呈现。Markdown会用`<pre>`和`<code>`标签包围代码块。

```
This is a normal paragraph: 

	This is a code block.
```

Markdown会生成：

```html
<p>This is a normal paragraph:</p>

<pre><code>
This is a code block. 
</code></pre>
```


## 水平线

如果一行中只有三个以上的连字符，星号，或者下划线，则会在该位置生成一个`<hr />`标签，星号和连字符之间的空格也是允许的，下面的几种写法都会生成一条水平线：

```
* * *

****

******

---

----------
```

# 内联元素

## 链接

Markdown支持两种链接形式：内联和引用。这两种形势下链接文本的定界符都是中括号`[]`。

创建内联链接，只需要在链接文本的右括号后紧接一对圆括号，圆括号里放所需的URL链接，还可以放一个可选的链接标题，标题用引号包围：

```
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
```

如果是引用相同服务器下的本地资源，还可以用相对路径：

```
See my [About](/about/) page for details.
```

引用类型的链接放在第二个中括号里，括号里放链接标签：

```
This is [an example][id] reference-style link.
```

然后，在文档中的任意位置，像下面这样定义链接标签，单独占一行：

```
[id]: http://example.com/ "Optional Title Here"
```

链接的定义只存在于Markdown处理期间，最终的HTML中不会出现。

链接定义的名称可以包含字母，数字，空格和标点符号，并且不是大小写敏感的。

隐含链接名称可以忽略链接名称，使链接文本被用作链接名称，链接名称只用一对空的中括号即可，例如：

```
[Google][]

[Google]: http://google.com/
```

虽然链接名称可以放在Markdown文档中的任何位置，但一般倾向于将它们放在引用位置的下面，也可以像底部注释那样放在文档底部：

```
I get 10 times more traffic from [Google][1] than from [Yahoo][2] or [MSN][3]. 

[1]: http://google.com/ "Google" 
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"
```

引用链接的意义在于使文档易于阅读，这样Markdown文档源码更接近于最终的输出，在书写时也不会打断行文而直接添加链接。

## 强调

Markdown使用星号和下划线作为强调标记。

一对星号或下划线使被包裹的文本变为斜体。

两对星号或下划线使被包裹的文本变为粗体。

```
*single asterisks* 

_single underscores_

**double asterisks**

__double underscores__
```

如果星号和下划线两端都有空格，则将其视为字面量。其他情况若需要使用星号和下划线的字面量，需在前面加上转义字符。

## 代码

使用重音符号`\`标记代码片段。不同于预格式化的代码块，代码片段只是在普通段落中标识出代码，例如：

```
Use this `printf()` function.
```

若要在代码片段中使用重音符号的字面量，可以使用多个重音符号作为起始和结束标记：

```
``There is a literal backtick (`) here.``
```

在代码片段中，尖括号和英镑符号会被转换成相应的字符实体，这使得包含HTML标签很容易：

例如Markdown会将：

``Please don't use any `<blink>` tags.``

转换成：

```html
<p>Please don't use any <code>&lt;blink&gt;</code> tags.</p>
```

## 图片

Markdown提供了内联和引用两种形式来插入图片。

内联图片语法如下：

```
![Alt text](/path/to/img.jpg) 

![Alt text](/path/to/img.jpg "Optional title")
```

引用语法如下：

```
![Alt text][id]
```

"id"是图片引用的名称，图片引用使用链接定义的相同语法：

```
[id]: url/to/image "Optional title attribute"
```

Markdown没有语法指定图片尺寸，但可以使用HTML`<img>`标签来指定图片尺寸。

# 其他

## 自动链接

Markdown支持一种“自动”创建URL和email地址链接的简短形式，只需要用尖括号包括链接即可。
> 尖括号可省略。

```
<http://example.com/>

<address@example.com>
```

## 禁用链接

使用\`\`包裹链接，可将链接禁用。

`http://www.baidu.com`

## 反斜杠转义

在Markdown中使用反斜杠转义Markdown语法符号为字面量。

例如在星号前面加反斜杠，可以显示星号。

```
\*literal asterisks\*
```

## 数学公式

行内公式使用一对美元符号将公式包裹：

```
$F = ma$
```

行间公式使用两对美元符号将公式包裹：

```
$$
	P = UI
$$
```

### 公式中的符号

#### 上标、下标与组合

使用`^`使接下来的一个字符或字符组合变为上标，`_`使接下来的一个字符或字符组合变为下标。

```
$x^4$

$x_1$
```

使用`{}`来包裹一组字符：

```
$x^2 + {10}^2$
```

#### 括号

小括号和方括号使用原始的`()`和`[]`即可。

如`(2+3)[4+4]`可直接表示为`(2+3)[4+4]`。

还可以使用`\left`和`\right`自动匹配括号。

如：$\left(2+\left(3+4\right)*9\right)*5$表示为`$\left(2+\left(3+4\right)*9\right)*5$`

---

[其他数学公式相关语法][markdown math]。

[markdown math]: https://www.jianshu.com/p/383e8149136c

## 表格

使用三个或以上的连字符创建表头，使用竖线分离表格列。

```
| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragragh | Text        |
```

效果如下：

| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragragh | Text        |

> 使用这种方式书写表格比较麻烦，可尝试使用[Markdown Tables Generator][]，通过图形界面建表，再将生成的Markdown源码复制进来。

[Markdown Tables Generator]: https://www.tablesgenerator.com/markdown_tables

### 对齐

可以在标题行内的连字符的左侧，右侧或者两侧加冒号，使列中文本左，右或居中对齐。

```
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```

效果如下：

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

## 脚注

使用脚注可以添加注释和参考，避免文档正文混乱。

在方括号中添加插入符号和标识符来创建脚注参考。标识符可以是数字或文字，但不能包含空格或制表符。标识符仅将脚注参考与脚注本身相关联在输出中，脚注按顺序编号。

使用同样的形式在另一处标记脚注，并加入冒号，后面输入具体内容。

```
Here's a simple footnote[^1], and here's a longer one.[^bignote]

[^1]: This is the first footnote.

[^bignote]: Here's one with multiple paragraphs and code.

    Indent paragraphs to include them in the footnote.

    `{ my code }`

    Add as many paragraphs as you like.
```

## 标题链接

标题链接的语法和普通链接基本一样，都由一对中括号和一对小括号组成。

中括号中填写显示在正文中的文本内容，小括号中填写一个井号再附上标题名称，例如：

```
[跳转到标题链接](#标题链接)
```

效果如下：

[跳转到标题链接](#标题链接)

## 任务清单

通过在连字符之后附加中括号(中括号里面需要保留一个空格)创建代办清单，将中括号中的空格替换为`X`，勾选复选框：

```
- [X] Learn git
- [ ] Recover C#
- [ ] Develope new game
```

- [x] Learn git
- [ ] Recover C#
- [ ] Develope new game