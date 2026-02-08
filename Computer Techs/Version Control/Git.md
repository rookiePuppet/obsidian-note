# 配置Git

在Git安装完成之后，还需要配置一下用户信息，在Git命令行中输入如下命令进行配置。

```GIT
git config --global user.name "name"
git config --global user.email "email"
```

> Git配置文件

每个仓库的配置文都在.git/config文件中，而当前用户的配置文件是系统用户的根目录下的.gitconfig文件。

`git config`命令不加`--global`是配置当前仓库，加了`--global`就是配置当前用户。

# 命令

## restore

```shell
git restore <file>...
```

恢复文件更改，必须是已加入版本管理的文件。

- `--staged`：恢复暂存区更改
- `--worktree`：恢复工作树更改，默认携带该参数

## reset

```shell
git reset <commit> [mode]
```

改变HEAD指针指向为指定的一次提交，用于撤销操作。

可以指定目标文件或目录，也可以附加`--patch`参数进入交互式模式。

- `--mixed`：保持工作树不变，只更新暂存区，默认模式
- `--soft`：保持工作树和暂存区不变，如果暂存区为空，可以使用`--soft HEAD~5`将前5次提交合并
- `--hard`：覆盖所有文件和目录，可能会覆盖未追踪的文件，会清空暂存区
- `--merge`：重置暂存区并更新工作树中的文件，保留暂存区和工作树有差异的文件
- `--keep`：重置暂存区并更新工作树中的文件，若目标提交与HEAD之间有差异的文件存在本地更改，命令会中止

## revert

```shell
git revert <commit>
```

撤销指定提交，并产生一次新的提交记录该操作，需要工作树保持干净（无任何更改）。

## rebase

```shell
git rebase <branch>
```

将一系列提交移动到另一个基点上。

假设当前处于topic分支，`git rebase master`可将topic分支的提交移动到master分支上。

如果不处于topic分支，则可以使用快捷方式`git rebase master topic`，自动切换到topic分支。

```text
	  A---B---C topic
	 /
D---E---F---G master

>>> rebase之后 >>>

			  A'--B'--C' topic
			 /
D---E---F---G master
```

```shell
git rebase --onto <branch> <begin> <end>
```

使用`--onto`参数可指定某个区间的提交进行变基。

## merge

## log

# gitignore文件

有些文件必须放在Git仓库目录中，但是又不能提交它们，比如保存了数据库密码的配置文件等。

这时就可以在Git根目录下创建一个特殊的文件 *.gitignore*，并把要忽略的文件名写进去，这样Git就会忽略这些文件了。

注意，.gitignore文件需要提交到Git仓库。

### 文件内容

[Github已经为我们准备好了各种.gitignore文件，不需要从头开始写。](https://github.com/github/gitignore)

> 忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Unity自动生成的Library 目录；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

> .gitignore的格式规范如下：

- 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。 可以使用标准的 glob 模式匹配。
- 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。

- 星号（*）匹配零个或多个任意字符；
- [abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；
- 问号（?）只匹配一个任意字符；
- 如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。

### 可能遇到的问题

#### 无法将某文件提交到git

```GIT
$ git add Table.unitypackage
The following paths are ignored by one of your .gitignore files: Table.unitypackage
Use -f if you really want to add them.
```

原因是该文件被.gitignore忽略了。

如果确实需要添加该文件，可以用`git add -f`命令强制添加。

```GIT
git add -f Table.unitypackage
```

或者可能是.gitignore中某个规则写错了，可以用'git check-ignore'命令来检查。

```GIT
git check-ignore -v Table.unitypackage
```

# Git LFS

Git LFS（Large File Storage），即Git大文件存储，是可以把音乐、图片、视频等指定的任意文件存在Git仓库之外，在Git仓库中用一个占用空间不到1KB的文本指针来代替的小工具。

这样就可以减小Git仓库的体积，加快克隆仓库的速度，也避免了因为在Git中存储很多大文件而损失性能的问题。

## 为什么要有LFS

在游戏开发中，美术资源占用很大一部分内存空间，像png、psd等文件是二进制的，体积也很庞大。

而git的diff/patch等是基于文件行的，对于二进制文件来说，git需要存储每次commit的改动。

每次修改二进制文件时，都要提交整个文件，从而产生额外的提交量，导致clone和pull的数据量大增，在线仓库的体积也会迅速增长。

![[Pasted image 20231008174829.png]]

LFS就是为了解决这一问题而产生的工具，它将你标记的大文件保存到另外的仓库，而在主仓库仅保留其指针。

在checkout版本时，根据指针的变量情况更新对应的大文件，而不是在本地保存所有版本的大文件。

## 安装LFS

LFS支持Git的最低版本是1.8.5，最新的Git安装包基本已经自带了LFS，可以使用`git lfs version`命令测试是否安装了LFS。

## 使用LFS

1. 在需要使用LFS的版本库中执行`git lfs install`命令，只需要执行一次。
	```GIT
	git lfs install
	```
2. 使用`git lfs track`命令告诉git需要管理哪些大文件。
	```GIT
	git lfs track "*.psd"
	```
	之后在版本库根目录中会生成一个.gitattributes文件，其中记录了需要LFS跟踪的文件，也可以直接手动编辑该文件。
	> 命令添加与手动添加的区别：使用命令时，会将已有的指定类型的大文件转化为文件指针，而手动编辑方式不会影响现有文件，只会影响后续添加的文件。两种方式都不会减小版本库的体积，因为不会改变之前的提交记录。

## 常用命令

- 查看当前使用Git LFS管理的匹配列表。
	```GIT
	git lfs track
	```
- 使用Git LFS管理指定的文件。
	```GIT
	git lfs track "*.psd"
	```
-  取消Git LFS管理指定的文件。
	```GIT
	git lfs untrack "*.psd"
	```
- 查看当前Git LFS对象的状态。
	```GIT
	git lfs status
	```
-  枚举目前所有被Git LFS管理的具体文件。
	```GIT
	git lfs ls-files
	```
- 检查当前使用的Git LFS版本。
	```GIT
	git lfs version
	```

## 常见问题

前面提到中途使用LFS无法改变原有版本库体积，如果已经上传了一些大文件到版本库中，如何减小版本库的体积？

如果是多人协作项目，最好不要做，因为这样会修改很多提交历史，造成版本库混乱。

首先确保所有人提交了修改，然后由一个人来进行LFS迁移，最后其他人再重新克隆完整的版本库。

先使用`git lfs migrate info -everything`命令按文件类型查看各种文件的空间占用情况。默认显示前5项，可以通过`-top=20`查看更多。

知道哪些文件占用空间比较大之后，就可以使用`git lfs migrate import -everything -include="*.FBX, *.png, *.tga, *.jpg"`命令来修改所有的提交记录。注意区分大小写，比如.fbx和.FBX会被识别为两种类型。

因为修改了提交记录，最后需要强制push来将本地所有的提交推送到远程仓库。

# 遇到的问题

### fatal: protocol 'https' is not supported

克隆时粘贴链接可能使用了Ctrl+V，会生成\^C符号。

解决方法是右键粘贴或Shift+Insert。

## fatal: unable to access: SSL certificate problem: unable to get local issuer certificate

原因是自签名证书无法被信任，临时的解决方案是禁用证书验证。

```shell
git config --global http.sslbackend schannel
```