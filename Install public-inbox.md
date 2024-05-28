# 如何安装public-inbox

## 不使用lei

如果您并不需要`lei`命令，您可以使用以下命令进行安装`public-inbox`。这个命令不确保您一定会安装上lei，但一定可以安装上`public-inbox`。

```bash
sudo apt install public-inbox
```

## 使用lei

如果您想要使用`lei`命令，那么您可能需要从源码开始编译了。
> [!IMPORTANT] 
> 您需要在root用户下执行以下命令

### 构建编译环境

```bash
apt install -y vim
apt install -y wget

apt install -y curl
apt install -y git
apt install -y build-essential
apt install -y pkg-config

apt install -y libgit2-dev

apt install -y xapian-tools

apt install -y libinline-c-perl
apt install -y libany-uri-escape-perl
apt install -y libdbd-sqlite3-perl
apt install -y libsearch-xapian-perl
apt install -y libmail-imapclient-perl
apt install -y libplack-middleware-reverseproxy-perl
apt install -y libcrypt-cbc-perl
apt install -y liblinux-inotify2-perl
```

### 获取源码并解压

文档编写时最新版本是`v1.9.0`，如果您想确保您安装的是最新版本，请访问该网址确认：https://public-inbox.org/public-inbox.git/refs/

```bash
wget https://public-inbox.org/public-inbox.git/snapshot/public-inbox-1.9.0.tar.gz
tar zxvf public-inbox-1.9.0.tar.gz && cd public-inbox-1.9.0
```

### 编译

```bash
perl Makefile.PL INSTALLDIRS=vendor
make OPTIMIZE="-O2 -g -Wall" LD_RUN_PATH="" -j8 V=1 VERBOSE=1
make pure_install DESTDIR=/tmp/public-index
```

### 添加环境变量

```bash
export PERL5LIB=${PERL5LIB}:/tmp/public-index/usr/share/perl5
echo 'export PATH="$PATH:/tmp/public-index/usr/bin"' >> ~/.bashrc
source ~/.bashrc
```

## 测试是否安装成功

