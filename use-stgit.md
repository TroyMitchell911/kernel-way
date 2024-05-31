## 如何安装

```bash
apt install stgit stgit-contrib
```

## 创建一个git仓库

```bash
# 新建仓库
git init repo && cd repo
# 提交一个空文件
touch README
git add -A
git commit -m "create a empty file"
# 修改该空文件后提交
echo "test text" > README
git add -A
git commit -m "change the file"
# 创建另一个空文件
touch a.txt
git add -A
git commit -m "add a file"
```

## 初始化stg

`stgit`初始化，在每个仓库第一次使用`stgit`都需要这么做：

```bash
stg init
```

## 修改commit的message

从`commit`中生成`stg`管理的`patch`：

```bash
stg uncommit -n 2
```

使用以下命令可查询目前`stg`管理的`patch`，其中`>`符号指向当前顶层的`patch`，`+`符号提示已经应用的`patch`:

```bash
stg series

# + change-the-file
# > add-a-file
```

我们要修改这个`add a file`这个message，所以目前的指向是正确的，可以直接使用以下命令进行编辑`commit`:

```bash
stg edit -m "add a text file"
```

在修改完成后，使用`commit`命令将这些`patch`提交到存储库的历史记录中:

```bash
stg commit --all
```

## 修改某个commit中的文件

从`commit`中生成`stg`管理的`patch`：

```bash
stg uncommit -n 2
```

使用以下命令可查询目前`stg`管理的`patch`，其中`>`符号指向当前顶层的`patch`，`+`符号提示已经应用的`patch`:

```bash
stg series

# + change-the-file
# > add-a-file
```

我们想要修改的是`change-the-file`这个`commit`的时候修改的内容，但我们现在顶层指向的是`add-a-file`这个`patch`，所以我们使用`pop`命令弹出`add-a-file`这个`patch`:

```bash
stg pop
stg series

# > change-the-file
```

现在对`README`文件修改，修改后可使用`status`命令查看工作区的改动:

```bash
echo "hello,world." > README
stg status

#  M README
```

很显然，`status`显示我们并没有将工作区的内容合并到当前`patch`，所以我们使用`refresh`命令让其合并:

```bash
stg refresh
stg status

# (status命令应该不会回显任何内容)
```

在完成了`refresh`之后，别忘了，我们还有一个`patch`在`stack`中，所以使用`push`命令让其回到顶层`patch`:

```bash
stg push
```

在修改完成后，使用`commit`命令将这些`patch`提交到存储库的历史记录中:

```bash
stg commit --all
```

## 将2个commit合并为1个commit

在做这一步之前，我们首先创建一个commit

```bash
touch b.txt
git add -A
git commit -m "add b.txt"
```

从`commit`中生成`stg`管理的`patch`：

```bash
stg uncommit -n 2
```

使用以下命令可查询目前`stg`管理的`patch`，其中`>`符号指向当前顶层的`patch`，`+`符号提示已经应用的`patch`:

```bash
stg series

# + add-a-text-file
# > add-b-txt
```

使用squash命令将这两个commit合并为一个，squash命令具体参数详见https://stacked-git.github.io/man/stg-squash/：

```bash
stg squash -m "add a.txt and b.txt" add-a-text-file add-b-txt
stg series

# > add-a-txt-and-b-txt
```

通过series命令的查询，确实两个message变成了一个message，但是两个文件的创建是否都被合并进去了还有待验证，使用git log查看哈希值后，可以通过diff命令查看差异：

```bash
git log

# commit d5f165d0b98d3bae56494ff4c1c42d3bde8c8d00 (HEAD -> main, refs/patches/main/add-a-txt-and-b-txt)
# Author: TroyMitchell911 <andrew998@126.com>
# Date:   Fri May 31 16:56:46 2024 +0800
# 
#     add a.txt and b.txt

# commit b83c2bb563ef9ff1735809ae0b3eed3cd6b13c44
# ...

git diff --name-only d5f165d0b98d3bae56494ff4c1c42d3bde8c8d00 b83c2bb563ef9ff1735809ae0b3e
ed3cd6b13c44

# a.txt
# b.txt
```

通过diff命令确实可以查看到一个commit中改动了这两个文件，在修改完成后，使用`commit`命令将这些`patch`提交到存储库的历史记录中:

```bash
stg commit --all
```

