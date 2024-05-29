## 如何编译内核

### 获取源码

为了获取具有完整commit记录和branch分支的linux源码，我们需要使用git工具。

安装git工具：

```bash
apt update && apt upgrade -y
apt install -y git
```

获取linux-kernel源码：

```bash
git clone https://github.com/torvalds/linux
```

检查commit记录和分支：

```bash
git branch -a
git log
```

### 最小构建环境

具体最小构建环境版本要求请参考该文档：https://www.kernel.org/doc/html/v6.9-rc7/process/changes.html#changes

```bash
apt update && apt upgrade -y
apt install -y gcc clang rustc bindgen make flex bash  bison pahole mount jfsutils reiserfsprogs xfsprogs  btrfs-progs pcmciautils quota ppp nfs-common grub2-common udev python3-sphinx global build-essential libncurses-dev bison flex libssl-dev libelf-dev bc gcc-riscv64-linux-gnu
```

### 配置与编译

在配置阶段，作为简单的切入点，可以选择默认配置`defconfig`或者使用当前内核的配置`oldconfig`。

默认配置：根据机器型号选择合适的默认配置。

当前内核的配置：获取当前内核的配置作为编译的配置选项，若内核源码有新的选项，则会要求你一一确认。

其他配置选项参考文档：https://docs.kernel.org/admin-guide/README.html

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- defconfig
```

编译直接使用make选项即可，`-j`选项后面的数字是使用核心数

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j16
```

