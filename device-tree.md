## 设备树如何编译 & 如何添加自己的设备树

我们可以通过这样的命令去编译设备树文件：

```bash
make dtbs
```

但这内部是如何执行的呢？打开`Makefile`一探究竟：

```makefile
PHONY += dtbs dtbs_prepare dtbs_install dtbs_check
dtbs: dtbs_prepare
        $(Q)$(MAKE) $(build)=$(dtstree)
```

可以看到`dtbs`依赖`dtbs_prepare`，查看一下`dtbs_prepare`：

```makefile
dtbs_prepare: include/config/kernel.release scripts_dtc
```

可以看到`dtbs_prepare`依赖于`include/config/kernel.release`和`scripts_dtc`，`include/config/kernel.release`暂时忽略，先看看`scripts_dtc`：

```makefile
scripts_dtc: scripts_basic
        $(Q)$(MAKE) $(build)=scripts/dtc
```

`scripts_dtc`又依赖于`scripts_basic`：

```makefile
scripts_basic:
        $(Q)$(MAKE) $(build)=scripts/basic
```

查看`${Q}`的定义， 可以发现其指示是否静默编译，并无大影响，可忽略：

```makefile
ifeq ($(KBUILD_VERBOSE),1)
  quiet =
  Q =
else
  quiet=quiet_
  Q = @
endif
```

`$(MAKE)`就是`make`和调用`make`时传递的参数：

```bash
if: make -j4
then: $(MAKE) = make -j4
```

`$(build)`定义在`scripts/Kbuild.include`中：

```makefile
###
# Shorthand for $(Q)$(MAKE) -f scripts/Makefile.build obj=
# Usage:
# $(Q)$(MAKE) $(build)=dir
build := -f $(srctree)/scripts/Makefile.build obj
```

`scripts_basic`将`Q`, `MAKE`, `build`展开后如下：

```makefile
[@]make[options] -f ./scripts/Makefile.build obj=scripts/basic
```

`scripts/basic/Makefile`是一个基础脚本，其他脚本运行前都应该执行他，这里不再展开。

回到`scripts_dtc`目标中：

```makefile
scripts_dtc: scripts_basic
        $(Q)$(MAKE) $(build)=scripts/dtc
```

将其展开:

```makefile
[@]make[options] -f ./scripts/Makefile.build obj=scripts/dtc
```

这个`script_dtc`目标会执行`scripts/dtc`下的`Makefile`将主机的`dtc`工具编译出来，以便编译`dts`设备树文件：

```makefile
# *** Also keep .gitignore in sync when changing ***
hostprogs-always-$(CONFIG_DTC)          += dtc fdtoverlay
hostprogs-always-$(CHECK_DT_BINDING)    += dtc
```

`dtc`工具是一个可以将`dts`编译成`dtb`也可以将`dtb`编译成`dts`的工具，如将`dtb`编译成`dts`使用如下：

```bash
./scripts/dtc/dtc -I dtb -O dts <dtb-path>/<dtb-name>.dtb -o <dts-path>/<dts-name>.dts
```

回到`dtbs`这个目标中，源码如下:

```makefile
dtbs: dtbs_prepare
        $(Q)$(MAKE) $(build)=$(dtstree)
```

其中，`$(dtstree)`的定义如下:

```bash
ifneq ($(wildcard $(srctree)/arch/$(SRCARCH)/boot/dts/),)
dtstree := arch/$(SRCARCH)/boot/dts
endif
```

`$(SRCARCH)`定义如下：

```makefile
SRCARCH         := $(ARCH)
```

将`dtbs`展开，如下：

```makefile
[@]make[options] -f ./scripts/Makefile.build obj=arch/riscv/boot/dts
```

最终执行到`arch/riscv/boot/dts/Makefile`。

让我们查看下这个`Makefile`：

```makefile
# SPDX-License-Identifier: GPL-2.0
subdir-y += allwinner
subdir-y += canaan
subdir-y += microchip
subdir-y += renesas
subdir-y += sifive
subdir-y += starfive
subdir-y += thead

# 如果CONFIG_BUILTIN_DTB选项启用，则将这些子目录的路径（每个目录名后加上斜杠）设置为 obj-y 变量的值，表示需要在这些子目录中寻找相应的文件进行构建。
obj-$(CONFIG_BUILTIN_DTB) := $(addsuffix /, $(subdir-y))
```

看来如果我们需要添加某个芯片的设备树支持，就在这个`Makefile`中添加子文件夹名称。

以`sifive`为例，查看`arch/riscv/boot/dts/sifive/Makefile`,该文件描述了当某种`SoC`被选中后，哪些`.dtb`文件会被编译出来：

```makefile
# SPDX-License-Identifier: GPL-2.0
# $(CONFIG_ARCH_SIFIVE) 是一个配置选项，表示是否启用SiFive架构的支持。如果启启用$(CONFIG_ARCH_SIFIVE) 的值为 y。
dtb-$(CONFIG_ARCH_SIFIVE) += hifive-unleashed-a00.dtb \
                             hifive-unmatched-a00.dtb
# $(addsuffix .o, $(dtb-y)) 是一个Makefile函数 addsuffix，用于在 $(dtb-y) 变量的每个条目后面添加 .o 后缀。这样处理后，$(dtb-y) 中的每个DTB文件名都会变成 hifive-unleashed-a00.dtb.o 和 hifive-unmatched-a00.dtb.o。
obj-$(CONFIG_BUILTIN_DTB) += $(addsuffix .o, $(dtb-y))
```

