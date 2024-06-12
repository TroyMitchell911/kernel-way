## 增加itb的支持

拷贝6.1.15的Makefile到6.6中：

[^Note]: 这里为了简单直接拷贝，实际最好用diff命令对比差异，将6.1.15中新增的部分添加进入

```bash
$ cp ../pi-linux/arch/riscv/Makefile arch/riscv/Makefile
$ cp ../pi-linux/arch/riscv/boot/Makefile arch/riscv/boot/
```

尝试编译：

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
# arch/riscv/Makefile:158: arch/riscv/generic/Platform: 没有那个文件或目录
```

看来缺少这个文件：

```bash
mkdir -p arch/riscv/generic && cp ../pi-linux/arch/riscv/generic/Platform arch/riscv/generic/
```

重新尝试编译：

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
# make[2]: *** 没有规则可制作目标“arch/riscv/generic/Image.its.S”，由“arch/riscv/boot/Image.its.S” 需求。 停止
```

查看Makefile中如何生成itb文件的：

```makefile
  1 include $(srctree)/arch/riscv/generic/Platform
159 bootvars-y      = ITS_INPUTS="$(its-y)"
  1 
  2 all:    $(notdir $(KBUILD_IMAGE)) Image.itb Image.gz.itb
  3 
  4 $(BOOT_TARGETS): vmlinux
  5         $(Q)$(MAKE) $(build)=$(boot) $(boot)/$@
  6         @$(kecho) '  Kernel: $(boot)/$@ is ready'
  7 
  8 Image.%: Image Image.gz
  9         $(Q)$(MAKE) $(build)=$(boot) $(bootvars-y) $(boot)/$@
```

查看Platform文件：

``````
1   #
  1 # SPDX-License-Identifier: GPL-2.0
  2 #
  3 # Copyright (C) 2024 Spacemit
  4 #
  5 # This software is licensed under the terms of the GNU General Public
  6 # License version 2, as published by the Free Software Foundation, and
  7 # may be copied, distributed, and modified under those terms.
  8 #
  9 # This program is distributed in the hope that it will be useful,
 10 # but WITHOUT ANY WARRANTY; without even the implied warranty of
 11 # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 12 # GNU General Public License for more details.
 13 
 14 its-y   := Image.its.S
``````

发现这里添加了一个Image.its.S文件，那么应该就是在6.1.16这个目录下，拷贝过来：

```bash
$ cp ../pi-linux/arch/riscv/generic/Image.its.S arch/riscv/generic/
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
$ ls arch/riscv/boot/*.itb
# arch/riscv/boot/Image.gz.itb  arch/riscv/boot/Image.itb
```