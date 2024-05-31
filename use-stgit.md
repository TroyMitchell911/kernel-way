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

## 修改commit的message

`stgit`初始化，在每个仓库第一次使用`stgit`都需要这么做：

```bash
stg init
```

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

