## 为linux6.6添加关于spacemit-k1的设备树

首先假设已经在工作目录，创建一个工作树，details请参见[git-worktree.md](../../git-worktree.md)：

```bash
$ mkdir -p ../linux-6.6
$ git worktree add ../linux-6.6 linux-6.6
$ git worktree list
# /home/troy/bpi-f3/pi-linux   ad8dc1c1ef3e [linux-6.1.15-k1]
# /home/troy/bpi-f3/linux-6.6  2915240eddba [linux-6.6]
```

进入6.6分支的工作树目录后，拷贝6.1.15的设备树文件到6.6中，以下步骤details参见[device-tree.md](../../device-tree.md)：

```bash
$ cp -r ../pi-linux/arch/riscv/boot/dts/spacemit/ ./arch/riscv/boot/dts/
```

修改Makefile，添加对spacemit子文件夹的支持：

```bash
$ vim arch/riscv/boot/dts/Makefile

# ...
# subdir-y += spacemit

# obj-$(CONFIG_BUILTIN_DTB) := $(addsuffix /, $(subdir-y))
```

拷贝config文件：

```bash
$ cp ../pi-linux/arch/riscv/configs/k1_defconfig arch/riscv/configs/
```

尝试编译：

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- dtbs
$ ls arch/riscv/boot/dts/spacemit/*.dtb
```

发现arch/riscv/boot/dts/spacemit/中并没有对应生成的dtb文件，查看spacemit下的Makefile：

```makefile
# SPDX-License-Identifier: GPL-2.0
dtb-$(CONFIG_SOC_SPACEMIT_K1X) += k1-x_evb.dtb k1-x_deb2.dtb k1-x_deb1.dtb k1-x_hs450.dtb k1-x_kx312.dtb \
                                  k1-x_MINI-PC.dtb k1-x_mingo.dtb k1-x_MUSE-N1.dtb k1-x_MUSE-Pi.dtb
obj-$(CONFIG_BUILTIN_DTB) += $(addsuffix .o, $(dtb-y))
```

发现dtb-后面的是一个$(CONFIG_SOC_SPACEMIT_K1X)选项，使用grep命令查看:

```bash
$ grep -n "CONFIG_SOC_SPACEMIT_K1X" arch/riscv/configs/k1_defconfig
# 43:CONFIG_SOC_SPACEMIT_K1X=y
$ grep -n "CONFIG_SOC_SPACEMIT_K1X" .config
# 
```

发现k1_defconfig中定义了这个选项，但是.config中没有，则可能是Kconfig.socs中没有定义，确认想法在../pi-linux/arch/riscv/Kconfig.socs中发现了如下：

```makefile
config SOC_SPACEMIT
	bool "Spacemit SoCs"
	select SIFIVE_PLIC
	help
	  This enables support for Spacemit SoCs platform hardware.

if SOC_SPACEMIT

choice
	prompt "Spacemit SOCs platform"
	help
	  choice Spacemit soc platform

	config SOC_SPACEMIT_K1
		bool "k1"
		help
		  select Spacemit k1 Platform SOCs.

	config SOC_SPACEMIT_K2
		bool "k2"
		help
		  select Spacemit k2 Platform SOCs.

endchoice

if SOC_SPACEMIT_K1

choice
	prompt "Spacemit K1 serial SOCs"
	help
	  choice Spacemit K1 soc platform

	config SOC_SPACEMIT_K1PRO
		bool "k1-pro"
		select DW_APB_TIMER_OF
		help
		  This enables support for Spacemit k1-pro Platform Hardware.

	config SOC_SPACEMIT_K1X
		bool "k1-x"
		help
		  This enables support for Spacemit k1-x Platform Hardware.
endchoice

config SOC_SPACEMIT_K1_FPGA
	bool "Spacemit K1 serial SoC FPGA platform"
	default n
	help
	  This enable FPGA platform for K1 SoCs.

endif

config BIND_THREAD_TO_AICORES
	bool "enable bind ai cores when use AI instruction"
	default y
	help
	  This enable bind ai cores when use AI instruction.

endif
```

将其复制到arch/riscv/Kconfig.socs中，再次尝试编译：

```bash
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- dtbs
# SYNC    include/config/auto.conf.cmd
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_evb.dtb
# In file included from arch/riscv/boot/dts/spacemit/k1-x_evb.dts:6:
# arch/riscv/boot/dts/spacemit/k1-x.dtsi:6:10: fatal error: dt-# bindings/mmc/k1x_sdhci.h: 没有那个文件或目录
#     6 | #include <dt-bindings/mmc/k1x_sdhci.h>
```

看来是一些文件的缺失，我们将其拷贝：

```bash
$ cp ../pi-linux/include/dt-bindings/clock/spacemit-k1x-clock.h ./include/dt-bindings/clock/
$ cp ../pi-linux/include/dt-bindings/display/spacemit-dpu.h ./include/dt-bindings/display/
$ cp ../pi-linux/include/dt-bindings/dma/k1x-dmac.h ./include/dt-bindings/dma/
$ cp -r ../pi-linux/include/dt-bindings/mmc/ ./include/dt-bindings/pinctrl/
$ cp ../pi-linux/include/dt-bindings/pinctrl/k1-x-pinctrl.h ./include/dt-bindings/pinctrl/
$ cp ../pi-linux/include/dt-bindings/pmu/k1x_pmu.h ./include/dt-bindings/pmu/
$ cp ../pi-linux/include/dt-bindings/reset/spacemit-k1x-reset.h ./include/dt-bindings/reset/
$ cp ../pi-linux/include/dt-bindings/usb/k1x_ci_usb.h ./include/dt-bindings/usb/
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- dtbs
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_evb.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_deb2.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_deb1.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_hs450.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_kx312.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_MINI-PC.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_mingo.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_MUSE-N1.dtb
#   DTC     arch/riscv/boot/dts/spacemit/k1-x_MUSE-Pi.dtb
$ ls arch/riscv/boot/dts/spacemit/*.dtb
# arch/riscv/boot/dts/spacemit/k1-x_deb1.dtb
# arch/riscv/boot/dts/spacemit/k1-x_hs450.dtb
# arch/riscv/boot/dts/spacemit/k1-x_MINI-PC.dtb
# arch/riscv/boot/dts/spacemit/k1-x_deb2.dtb  
# arch/riscv/boot/dts/spacemit/k1-x_kx312.dtb
# arch/riscv/boot/dts/spacemit/k1-x_MUSE-N1.dtb
# arch/riscv/boot/dts/spacemit/k1-x_evb.dtb  
# arch/riscv/boot/dts/spacemit/k1-x_mingo.dtb  
# arch/riscv/boot/dts/spacemit/k1-x_MUSE-Pi.dtb
```



