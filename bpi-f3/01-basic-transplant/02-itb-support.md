## 增加itb的支持

使用`diff`命令对比`arch/riscv/Kconfig`和`../pi-linux/arch/riscv/Kconfig`、`arch/riscv/Makefile`和`../pi-linux/arch/riscv/Makefile`以及`arch/riscv/boot/Makefile`和`../pi-linux/arch/riscv/boot/Makefile`的差异进行修改。

这里不再赘述，具体查看：
https://github.com/TroyMitchell911/bpi-f3-linux-6.6/blob/main/arch/riscv/Makefile
https://github.com/TroyMitchell911/bpi-f3-linux-6.6/blob/main/arch/riscv/boot/Makefile

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

查看`Makefile`中如何生成`itb`文件的：

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

查看`Platform`文件：

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

发现这里添加了一个`Image.its.S`文件，那么应该就是在`6.1.15`这个目录下，拷贝过来：

```bash
$ cp ../pi-linux/arch/riscv/generic/Image.its.S arch/riscv/generic/
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
$ ls arch/riscv/boot/*.itb
# arch/riscv/boot/Image.gz.itb  arch/riscv/boot/Image.itb
```

## 加载内核测试

该小节基于`tftp`服务，`details`查看[05-uboot-net.md](../00-started/05-uboot-net.md)

将`内核文件`和`dtb文件`传入`tftp`目录：

```bash
$ cp arch/riscv/boot/Image.itb  ../tftp
$ cp arch/riscv/boot/dts/spacemit/k1-x_deb1.dtb  ../tftp
```

启动`uboot`后打断，设置环境变量：

```bash
=> setenv loadknl 'echo "Loading kernel..."; tftpboot ${kernel_addr_r} Image.itb'
=> setenv loaddtb 'echo "Loading dtb..."; if tftpboot ${dtb_addr} ${dtb_name}; then else echo "load dtb from bootfs fail, use built-in dtb"; setenv dtb_addr ""; fi;'
=> saveenv
Saving Environment to MMC... Writing to MMC(2)... OK
```

复位开发板，查看输出：

```bash
## Loading kernel from FIT Image at 10000000 ...
   Using 'conf-default' configuration
   Verifying Hash Integrity ... OK
   Trying 'kernel' kernel subimage
     Description:  Linux 6.6.0-rc5+
     Type:         Kernel Image
     Compression:  uncompressed
     Data Start:   0x100000c4
     Data Size:    32154624 Bytes = 30.7 MiB
     Architecture: RISC-V
     OS:           Linux
     Load Address: 0x00000000
     Entry Point:  0x00000000
     Hash algo:    crc32
     Hash value:   ee652057
   Verifying Hash Integrity ... crc32+ OK
## Flattened Device Tree blob at 1f000000
   Booting using the fdt blob at 0x1f000000
   Loading Kernel Image
Unhandled exception: Store/AMO access fault
EPC: 0000000077ed7c7a RA: 0000000077edae84 TVAL: 0000000000000000
EPC: 0000000000201c7a RA: 0000000000204e84 reloc adjusted
```

发现出现了错误，此时观察`Load Address`和`Entry Point`都是`0`，一定是哪里出现了问题。

通过对比，发现`6.1.15`的`Load Address`和`Entry Point`都是`0x14000000`，在`6.1.15`的目录中查找这个参数：

```bash
$ grep -nR 0x14000000 ../pi-linux
# ../pi-linux/arch/riscv/configs/k1_defconfig:48:CONFIG_IMAGE_LOAD_OFFSET=0x1400000
```

对比`6.6`目录下的`.config`，发现没有这个选项，肯定是`Kconfig`没有对应选项：

```bash
$ grep -nR CONFIG_IMAGE_LOAD_OFFSET .config 
```

## 修改Kconfig

使用`diff`命令对比`arch/riscv/Kconfig`和`../pi-linux/arch/riscv/Kconfig`的差异，进行修改。

这里不再赘述，具体查看https://github.com/TroyMitchell911/bpi-f3-linux-6.6/blob/main/arch/riscv/Kconfig。

修改完毕后重新执行:

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16

# riscv64-unknown-linux-gnu-ld: kernel/time/clocksource.o: in function `clocksource_max_adjustment':
# ...
```

由于在`Kconfig`中添加了`select ARCH_CLOCKSOURCE_INIT`，所以找不到`riscv`中这个函数的实现:

```bash
$ vim arch/riscv/kernel/time.c

# 将以下内容添加进入
void clocksource_arch_init(struct clocksource *cs)
{
#ifdef CONFIG_GENERIC_GETTIMEOFDAY
        cs->vdso_clock_mode = VDSO_CLOCKMODE_ARCHTIMER;
#else   
        cs->vdso_clock_mode = VDSO_CLOCKMODE_NONE;
#endif
}
```

修改完毕后重新执行:

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
```

重新加载内核测试，出现如下`info`:

```bash
## Loading kernel from FIT Image at 10000000 ...
   Using 'conf-default' configuration
   Verifying Hash Integrity ... OK
   Trying 'kernel' kernel subimage
     Description:  Linux 6.6.0-rc5+
     Type:         Kernel Image
     Compression:  uncompressed
     Data Start:   0x100000c4
     Data Size:    32147968 Bytes = 30.7 MiB
     Architecture: RISC-V
     OS:           Linux
     Load Address: 0x01400000
     Entry Point:  0x01400000
     Hash algo:    crc32
     Hash value:   702a733e
   Verifying Hash Integrity ... crc32+ OK
## Flattened Device Tree blob at 1f000000
   Booting using the fdt blob at 0x1f000000
   Loading Kernel Image
   Loading Ramdisk to 76867000, end 76ec5130 ... OK
   Loading Device Tree to 0000000076851000, end 000000007686630a ... OK

Starting kernel ...

efi_free_pool: illegal free 0x0000000076dc3040
efi_free_pool: illegal free 0x0000000076dc0040
efi_free_pool: illegal free 0x0000000076dbe040
efi_free_pool: illegal free 0x0000000076dbc040
[    0.000000] Linux version 6.6.0-rc5+ (troy@troy-linux) (riscv64-unknown-linux-gnu-gcc () 13.2.0, GNU ld (GNU Binutils) 2.42) #7 SMP PREEMPT Wed Jun 12 17:01:42 CST 2024
[    0.000000] Machine model: spacemit k1-x deb1 board
[    0.000000] SBI specification v1.0 detected
[    0.000000] SBI implementation ID=0x1 Version=0x10003
[    0.000000] SBI IPI extension detected
[    0.000000] SBI RFENCE extension detected
[    0.000000] earlycon: sbi0 at I/O port 0x0 (options '')
[    0.000000] printk: bootconsole [sbi0] enabled
[    0.000000] efi: UEFI not found.
........
[    1.687918]   with environment:
[    1.691115]     HOME=/
[    1.693532]     TERM=linux
[   11.814156] platform leds: deferred probe pending
```

