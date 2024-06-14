## 移植PCI驱动

`Makefile`（ref: `../pi-linux/drivers/pci/controller/dwc/Makefile`）

```bash
$ vim drivers/pci/controller/dwc/Makefile

29,30d28
< obj-$(CONFIG_PCIE_SPACEMIT) += pcie-spacemit.o
< obj-$(CONFIG_PCI_K1X) += pcie-k1x.o

```

`Kconfig`（ref: `../pi-linux/drivers/pci/controller/dwc/Kconfig`）

```bash
$ vim drivers/pci/controller/dwc/Kconfig

236,261d235
< config PCI_K1X
< 	bool
< 
< config PCI_K1X_HOST
< 	bool "Spacemit K1X PCIe Controller - Host Mode"
< 	depends on SOC_SPACEMIT_K1X
< 	depends on PCI && PCI_MSI_IRQ_DOMAIN
< 	depends on OF && HAS_IOMEM
< 	select PCIE_DW_HOST
< 	select PCI_K1X
< 	default y
< 	help
< 	  Enables support for the PCIe controller in the K1X SoC to work in
<           host mode.
< 
< config PCI_K1X_EP
< 	bool "Spacemit K1X PCIE Controller - Endpoint Mode"
< 	depends on SOC_SPACEMIT_K1X
< 	depends on PCI_ENDPOINT
< 	depends on OF && HAS_IOMEM
< 	select PCIE_DW_EP
< 	select PCI_K1X
< 	help
< 	  Enables support for the PCIe controller in the K1X SoC to work in
<           endpoint mode.
< 
443,451d416
< 
< config PCIE_SPACEMIT
< 	bool "Spacemit PCIe controller"
< 	depends on SOC_SPACEMIT || COMPILE_TEST
< 	depends on PCI_MSI_IRQ_DOMAIN
< 	select PCIE_DW_HOST
< 	help
< 	  Say Y here to enable PCIe controller support on Spacemit SoCs. The
< 	  PCIe controller uses the DesignWare core plus Spacemit hardware wrappers.
```

```bash
$ cp ../pi-linux/drivers/gpio/gpio-k1x.c drivers/gpio/
```

修改`drivers/pci/controller/dwc/pcie-designware-host.c`：

```bash
$ vim drivers/pci/controller/dwc/pcie-designware-host.c

381,386c381
< 	/* ret = dma_set_coherent_mask(dev, DMA_BIT_MASK(32)); */
< 	if (IS_ENABLED(CONFIG_SOC_SPACEMIT_K1PRO))
< 	        ret = dma_set_coherent_mask(dev, DMA_BIT_MASK(40));
< 	else
<                 ret = dma_set_coherent_mask(dev, DMA_BIT_MASK(32));
< 	
---
> 	ret = dma_set_coherent_mask(dev, DMA_BIT_MASK(32));

```

拷贝文件:

```bash
$ cp ../pi-linux/drivers/pci/controller/dwc/pcie-k1x.c drivers/pci/controller/dwc/
$ cp ../pi-linux/drivers/pci/controller/dwc/pcie-spacemit.c drivers/pci/controller/dwc/
```

出现问题：

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig
#
# No change to .config
#
```

按道理来说我们配置好了`Kconfig`，初始化配置文件的时候应该被写入才对，但这里没有，就需要分析一下`Kconfig`了。

一般出现这种问题都是依赖项的问题，也就是`depends on`，让我们来列举一下`depends on`，`EP`的`depends on`暂时不用管：

[^Note]: 在PCI架构中，endpoint通常是外围设备（如网卡、显卡、存储设备等）；host（主机）通常是计算机的中央处理器或控制器。

- SOC_SPACEMIT_K1X
- PCI
- PCI_MSI_IRQ_DOMAIN
- OF
- HAS_IOMEM
- SOC_SPACEMIT

让我们来逐个去`.config`里面去验证：

```bash
$ grep -e "CONFIG_SOC_SPACEMIT_K1X=" -e "CONFIG_PCI=" -e "CONFIG_OF=" -e "CONFIG_HAS_IOMEM=" -e "CONFIG_SOC_SPACEMIT=" -e "CONFIG_PCI_MSI_IRQ_DOMAIN=" .config
CONFIG_SOC_SPACEMIT=y
CONFIG_SOC_SPACEMIT_K1X=y
CONFIG_PCI=y
CONFIG_OF=y
CONFIG_HAS_IOMEM=y
```

发现除了`PCI_MSI_IRQ_DOMAIN`都是有的，也就是说名因为这个依赖项没有，所以产生了问题，去`6.1.15`中搜索该依赖项的配置：

```bash
$ grep -nR "config PCI_MSI_IRQ_DOMAIN" ../pi-linux/
../pi-linux/drivers/pci/Kconfig:54:config PCI_MSI_IRQ_DOMAIN
$ grep -nR "config PCI_MSI_IRQ_DOMAIN"
```

发现在`6.6`的目录中没有找到该配置，但是`6.1.15`在`drivers/pci/Kconfig`找到了这个配置，查看该文件：

```bash
 25 config PCI_MSI
 24         bool "Message Signaled Interrupts (MSI and MSI-X)"
 23         select GENERIC_MSI_IRQ
 22         help
 21            This allows device drivers to enable MSI (Message Signaled
 20            Interrupts).  Message Signaled Interrupts enable a device to
 19            generate an interrupt using an inbound Memory Write on its
 18            PCI bus instead of asserting a device IRQ pin.
 17 
 16            Use of PCI MSI interrupts can be disabled at kernel boot time
 15            by using the 'pci=nomsi' option.  This disables MSI for the
 14            entire system.
 13 
 12            If you don't know what to do here, say Y.
 11 
 10 config PCI_MSI_IRQ_DOMAIN
  9         def_bool y
  8         depends on PCI_MSI
  7         select GENERIC_MSI_IRQ_DOMAIN
```

发现这里选择了`GENERIC_MSI_IRQ_DOMAIN`，继续搜索`GENERIC_MSI_IRQ_DOMAIN`：

```bash
$ grep -nR "config GENERIC_MSI_IRQ_DOMAIN" ../pi-linux/
../pi-linux/kernel/irq/Kconfig:94:config GENERIC_MSI_IRQ_DOMAIN
```

在`6.1.15`的`kernel/irq/Kconfig`均找到了该配置,但`6.6`没有，对比两个文件发现：

```bash
6.6:
14 # Generic MSI hierarchical interrupt domain support
 13 config GENERIC_MSI_IRQ
 12         bool
 11         select IRQ_DOMAIN_HIERARCHY

6.1.15:
 19 # Generic MSI interrupt support
 18 config GENERIC_MSI_IRQ
 17         bool
 16 
 15 # Generic MSI hierarchical interrupt domain support
 14 config GENERIC_MSI_IRQ_DOMAIN
 13         bool
 12         select IRQ_DOMAIN_HIERARCHY
 11         select GENERIC_MSI_IRQ
```

看起来`6.6`将`GENERIC_MSI_IRQ_DOMAIN`合并进了`GENERIC_MSI_IRQ`，而`PCI_MSI`是会选择`GENERIC_MSI_IRQ`的，所以我们只需要`depends on PCI_MSI`即可：

```bash
$ vim drivers/pci/controller/dwc/Kconfig
242c242
< 	depends on PCI && PCI_MSI
---
> 	depends on PCI && PCI_MSI_IRQ_DOMAIN
447c447
< 	depends on PCI_MSI
---
> 	depends on PCI_MSI_IRQ_DOMAIN
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig
#
# configuration written to .config
#
$ make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
  SYNC    include/config/auto.conf.cmd
  CALL    scripts/checksyscalls.sh
  CC      drivers/pci/controller/dwc/pcie-designware.o
  CC      drivers/pci/controller/dwc/pcie-designware-host.o
  CC      drivers/pci/controller/dwc/pcie-k1x.o
  ...
```

