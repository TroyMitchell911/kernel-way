## 移植mfd驱动

[^Note]: 欲要移植此驱动请先移植I2C驱动

首先修改`Kconfig`以便支持`k1`的选项：

```bash
$ vim drivers/mfd/Kconfig

2220,2229d2219
< config MFD_SPACEMIT_PMIC
< 	tristate "Spacemit Power Management IC"
< 	depends on I2C && OF
< 	depends on SOC_SPACEMIT
< 	select MFD_CORE
< 	select REGMAP_I2C
< 	select REGMAP_IRQ
< 	help
< 	  Select this option to get support for the SPACEMIT Power Management ICs.
< 
```

修改`Makefile`支持`k1`所编译的文件:

```bash
$ vim drivers/mfd/Makefile

287d286
< obj-$(CONFIG_MFD_SPACEMIT_PMIC) += spacemit-mfd.o
```

复制相关文件：

```bash
$ cp ../pi-linux/drivers/mfd/spacemit-mfd.c drivers/mfd/
$ cp -r ../pi-linux/include/linux/mfd/spacemit/ include/linux/mfd/
```

编译遇到如下错误：

```bash
In file included from ./include/linux/mfd/spacemit/spacemit_pmic.h:315,
                 from drivers/mfd/spacemit-mfd.c:12:
./include/linux/mfd/spacemit/spm8821.h:643:10: error: 'const struct regmap_irq_chip' has no member named 'mask_invert'; did you mean 'ack_invert'?
  643 |         .mask_invert = true,                                    \
      |          ^~~~~~~~~~~
drivers/mfd/spacemit-mfd.c:16:1: note: in expansion of macro 'SPM8821_IRQ_CHIP_DESC'
   16 | SPM8821_IRQ_CHIP_DESC;
      | ^~~~~~~~~~~~~~~~~~~~~
drivers/mfd/spacemit-mfd.c:223:21: error: initialization of 'int (*)(struct i2c_client *)' from incompatible pointer type 'int (*)(struct i2c_client *, const struct i2c_device_id *)' [-Werror=incompatible-pointer-types]
  223 |         .probe    = spacemit_pmic_probe,
      |                     ^~~~~~~~~~~~~~~~~~~
drivers/mfd/spacemit-mfd.c:223:21: note: (near initialization for 'spacemit_pmic_i2c_driver.probe')
```

让我们来整理下问题

- struct regmap_irq_chip结构体没有mask_invert成员
- 不兼容的指针类型spacemit_pmic_probe

#### struct regmap_irq_chip结构体没有mask_invert成员

参照https://lore.kernel.org/all/20230220143343.694A6C433D2@smtp.kernel.org/的这里可以知道`mask_invert`这个变量由于没有使用到所以被删除了：

> ```
> Aidan MacDonald (2):
>       regmap-irq: Remove unused type_invert flag
>       regmap-irq: Remove unused mask_invert flag
> ```

那我们直接删除对其的使用就好了

#### 不兼容的指针类型spacemit_pmic_probe

查看`drivers/mfd/spacemit-mfd.c`发现`probe`一个结构体成员：

```c
  5 static struct i2c_driver spacemit_pmic_i2c_driver = {
  4         .driver = {
  3                 .name = "spacemit-pmic",
  2                 .of_match_table = spacemit_pmic_of_match,
  1         },
223         .probe    = spacemit_pmic_probe,
  1         .remove   = spacemit_pmic_remove,
  2         .shutdown = spacemit_pmic_shutdown,
  3 };
```

查找该结构体：

```bash
$ find include -name "*.h" -exec grep -n "struct i2c_driver {" {} +
include/linux/i2c.h:271:struct i2c_driver {
```

发现`probe`成员的定义如下:

```c
 20 struct i2c_driver {
  19         unsigned int class;
  18 
  17         /* Standard driver model interfaces */
  16         int (*probe)(struct i2c_client *client);
  15         void (*remove)(struct i2c_client *client);
  14 
  13 
  12         /* driver model interfaces that don't relate to enumeration  */
  11         void (*shutdown)(struct i2c_client *client);
...
```

查看`spacemit_pmic_probe`函数定义:

```c
 25 static int spacemit_pmic_probe(struct i2c_client *client,
 24                        const struct i2c_device_id *id)
 23 {
 22         int ret;
 21         int nr_cells;
 20         struct device_node *np;
 19         struct spacemit_pmic *pmic;
 18         const struct mfd_cell *cells;
 17         const struct of_device_id *of_id;
 16 
 15         pmic = devm_kzalloc(&client->dev, sizeof(*pmic), GFP_KERNEL);

```

经过查看该函数发现，并没有对参数`id`的使用，所以可以直接把参数删掉。
