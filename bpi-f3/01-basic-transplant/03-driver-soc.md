## 移植soc驱动

修改Kconfig增加如下内容：

```bash
$ vim drivers/soc/Kconfig
source "drivers/soc/spacemit/Kconfig"
```

修改Makefile增加如下内容：

```bash
$ vim drivers/soc/Makefile
obj-$(CONFIG_SOC_SPACEMIT)	+= spacemit/
```

复制：

```bash
$ cp -r ../pi-linux/driver/soc/spacemit driver/soc
```

报错信息：

```bash
drivers/soc/spacemit/jpu/jpu.c: In function 'jpu_map_to_register':
drivers/soc/spacemit/jpu/jpu.c:1282:22: error: assignment of read-only member 'vm_flags'
 1282 |         vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP;
      |                      ^~
drivers/soc/spacemit/jpu/jpu.c: In function 'jpu_map_to_physical_memory':
drivers/soc/spacemit/jpu/jpu.c:1292:22: error: assignment of read-only member 'vm_flags'
 1292 |         vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP;
      |                      ^~
drivers/soc/spacemit/jpu/jpu.c: In function 'jpu_map_to_instance_pool_memory':
drivers/soc/spacemit/jpu/jpu.c:1308:22: error: assignment of read-only member 'vm_flags'
 1308 |         vm->vm_flags |= VM_DONTEXPAND | VM_DONTDUMP;
      |                      ^~
In file included from ./include/linux/linkage.h:7,
                 from ./include/linux/kernel.h:17,
                 from drivers/soc/spacemit/jpu/jpu.c:3:
drivers/soc/spacemit/jpu/jpu.c: In function 'jpu_probe':
./include/linux/export.h:31:21: error: passing argument 1 of 'class_create' from incompatible pointer type [-Werror=incompatible-pointer-types]
   31 | #define THIS_MODULE ((struct module *)0)
      |                     ^~~~~~~~~~~~~~~~~~~~
      |                     |
      |                     struct module *
drivers/soc/spacemit/jpu/jpu.c:1673:40: note: in expansion of macro 'THIS_MODULE'
 1673 |         jdev->jpu_class = class_create(THIS_MODULE, cdev_name);
      |                                        ^~~~~~~~~~~
In file included from ./include/linux/device.h:31,
                 from ./include/linux/platform_device.h:13,
                 from drivers/soc/spacemit/jpu/jpu.c:7:
./include/linux/device/class.h:230:54: note: expected 'const char *' but argument is of type 'struct module *'
  230 | struct class * __must_check class_create(const char *name);
      |                                          ~~~~~~~~~~~~^~~~
drivers/soc/spacemit/jpu/jpu.c:1673:27: error: too many arguments to function 'class_create'
 1673 |         jdev->jpu_class = class_create(THIS_MODULE, cdev_name);
      |                           ^~~~~~~~~~~~
./include/linux/device/class.h:230:29: note: declared here
  230 | struct class * __must_check class_create(const char *name);
      |                             ^~~~~~~~~~~~
cc1: some warnings being treated as errors
make[6]: *** [scripts/Makefile.build:243：drivers/soc/spacemit/jpu/jpu.o] 错误 1
make[6]: *** 正在等待未完成的任务....
make[5]: *** [scripts/Makefile.build:480：drivers/soc/spacemit/jpu] 错误 2
make[5]: *** 正在等待未完成的任务....
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c: In function 'spacemit_wlan_get_oob_irq_flags':
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:80:26: error: 'IRQF_TRIGGER_HIGH' undeclared (first use in this function)
   80 |         oob_irq_flags = (IRQF_TRIGGER_HIGH | IRQF_SHARED | IRQF_NO_SUSPEND);
      |                          ^~~~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:80:26: note: each undeclared identifier is reported only once for each function it appears in
drivers/soc/spacemit/spacemit-rf/spacemit-bt.c: In function 'spacemit_bt_probe':
drivers/soc/spacemit/spacemit-rf/spacemit-bt.c:112:12: error: implicit declaration of function 'device_property_read_u32' [-Werror=implicit-function-declaration]
  112 |         if(device_property_read_u32(dev, "power-on-delay-ms",
      |            ^~~~~~~~~~~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-bt.c: At top level:
drivers/soc/spacemit/spacemit-rf/spacemit-bt.c:158:34: error: array type has incomplete element type 'struct of_device_id'
  158 | static const struct of_device_id spacemit_bt_ids[] = {
      |                                  ^~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-bt.c:158:34: warning: 'spacemit_bt_ids' defined but not used [-Wunused-variable]
cc1: some warnings being treated as errors
make[6]: *** [scripts/Makefile.build:243：drivers/soc/spacemit/spacemit-rf/spacemit-bt.o] 错误 1
make[6]: *** 正在等待未完成的任务....
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:80:46: error: 'IRQF_SHARED' undeclared (first use in this function)
   80 |         oob_irq_flags = (IRQF_TRIGGER_HIGH | IRQF_SHARED | IRQF_NO_SUSPEND);
      |                                              ^~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:80:60: error: 'IRQF_NO_SUSPEND' undeclared (first use in this function)
   80 |         oob_irq_flags = (IRQF_TRIGGER_HIGH | IRQF_SHARED | IRQF_NO_SUSPEND);
      |                                                            ^~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c: In function 'spacemit_wlan_probe':
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:135:12: error: implicit declaration of function 'device_property_read_u32' [-Werror=implicit-function-declaration]
  135 |         if(device_property_read_u32(dev, "power-on-delay-ms",
      |            ^~~~~~~~~~~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c: At top level:
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:155:34: error: array type has incomplete element type 'struct of_device_id'
  155 | static const struct of_device_id spacemit_wlan_ids[] = {
      |                                  ^~~~~~~~~~~~~~~~~
drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c:155:34: warning: 'spacemit_wlan_ids' defined but not used [-Wunused-variable]
cc1: some warnings being treated as errors
make[6]: *** [scripts/Makefile.build:243：drivers/soc/spacemit/spacemit-rf/spacemit-wlan.o] 错误 1
make[5]: *** [scripts/Makefile.build:480：drivers/soc/spacemit/spacemit-rf] 错误 2
  AR      drivers/soc/spacemit/pm_domain/built-in.a
make[4]: *** [scripts/Makefile.build:480：drivers/soc/spacemit] 错误 2
make[3]: *** [scripts/Makefile.build:480：drivers/soc] 错误 2
make[2]: *** [scripts/Makefile.build:480：drivers] 错误 2
make[1]: *** [/home/troy/bpi-f3/linux-6.6/Makefile:1913：.] 错误 2
make: *** [Makefile:234：__sub-make] 错误 2
```

让我们来整理一下思路：

- IRQF_TRIGGER_HIGH、IRQF_SHARED、IRQF_NO_SUSPEND宏定义没有定义的问题
  - drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
- struct of_device_id结构体无法找到的问题
  - drivers/soc/spacemit/spacemit-rf/spacemit-bt.c
  - drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
- device_property_read_u32没有定义的问题
  - drivers/soc/spacemit/spacemit-rf/spacemit-bt.c
  - drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
- vm结构体变量访问只读成员vm_flags的问题
  - drivers/soc/spacemit/jpu/jpu.c
- class_create调用参数问题
  - drivers/soc/spacemit/jpu/jpu.c

### IRQF_TRIGGER_HIGH宏定义没有定义的问题

搜索该宏定义所在的头文件:

```bash
$ find include -name "*.h" -exec grep -n "IRQF_TRIGGER_HIGH" {} +
# include/linux/interrupt.h:33:#define IRQF_TRIGGER_HIGH	0x00000004
# include/linux/interrupt.h:35:#define IRQF_TRIGGER_MASK	(IRQF_TRIGGER_HIGH | IRQF_TRIGGER_LOW | \
```

查看drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c是否包含该头文件：

```bash
$ grep -n "interrupt.h" drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
```

很显然，没有，将其添加

### struct of_device_id结构体无法找到的问题

搜索该结构体所在的头文件:

```bash
$ find include -name "*.h" -exec grep -n "struct of_device_id {" {} +
# include/linux/mod_devicetable.h:282:struct of_device_id {
```

查看drivers/soc/spacemit/spacemit-rf/spacemit-bt.c和drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c是否包含该头文件：

```bash
$ grep -n "mod_devicetable.h" drivers/soc/spacemit/spacemit-rf/spacemit-bt.c drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
```

很显然，没有，将其添加

### device_property_read_u32没有定义的问题

搜索该函数所在的头文件:

```bash
$ find include -name "*.h" -exec grep -n "device_property_read_u32" {} +
# include/linux/property.h:47:int device_property_read_u32_array(const struct device *dev, const char *propname,
# include/linux/property.h:175:static inline int device_property_read_u32(const struct device *dev,
# include/linux/property.h:178:	return device_property_read_u32_array(dev, propname, val, 1);
# include/linux/property.h:199:	return device_property_read_u32_array(dev, propname, NULL, 0);
```

查看drivers/soc/spacemit/spacemit-rf/spacemit-bt.c和drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c是否包含该头文件：

```bash
$ grep -n "mod_devicetable.h" drivers/soc/spacemit/spacemit-rf/spacemit-bt.c drivers/soc/spacemit/spacemit-rf/spacemit-wlan.c
```

很显然，没有，将其添加

### vm结构体变量访问只读成员vm_flags的问题

查看drivers/soc/spacemit/jpu/jpu.c中vm结构体变量类型：

```c
static int jpu_map_to_register(struct file *fp, struct vm_area_struct *vm)
{
...
```

可以看到该结构体类型为struct vm_area_struct

搜索该成员所在的头文件:

```bash
$ find include -name "*.h" -exec grep -n "struct vm_area_struct {" {} +
# include/linux/mm_types.h:565:struct vm_area_struct {
```

查看include/linux/mm_types.h中关于vm_flags的定义：

```c
struct vm_area_struct {
        /* The first cache line has the info for VMA tree walking. */

        union {
                struct {
                        /* VMA covers [vm_start; vm_end) addresses within mm */
                        unsigned long vm_start;
                        unsigned long vm_end;
                };
#ifdef CONFIG_PER_VMA_LOCK
                struct rcu_head vm_rcu; /* Used for deferred freeing. */
#endif
        };

        struct mm_struct *vm_mm;        /* The address space we belong to. */
        pgprot_t vm_page_prot;          /* Access permissions of this VMA. */

        /*
         * Flags, see mm.h.
         * To modify use vm_flags_{init|reset|set|clear|mod} functions.
         */
        union {
                const vm_flags_t vm_flags;
                vm_flags_t __private __vm_flags;
        };
...
```

经过对比后发现，vm_flags在6.1.15中定义如下：

```c
unsigned long vm_flags;         /* Flags, see mm.h. */
```

但在6.6中vm_flags是包含在一个union中，并且注释注明需要使用vm_flags_{init|reset|set|clear|mod}这些函数修改vm_flags。

查看vm_flags相关的定义:

```bash
find include -name "*.h" -exec grep -n "vm_flags_init" {} +
# include/linux/mm.h:815:static inline void vm_flags_init(struct vm_area_struct *vma,
# include/linux/mm.h:830: vm_flags_init(vma, flags);
# include/linux/mm.h:861: vm_flags_init(vma, (vma->vm_flags | set) & ~clear);
```

发现其定义在mm.h中，打开drivers/soc/spacemit/jpu/jpu.c，增加其头文件并且修改对flags的操作：

```bash
26d25
< #include <linux/mm.h>
1283,1284c1282
< 	/* vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP; */
< 	vm_flags_set(vm, vm->vm_flags | VM_IO | VM_DONTEXPAND | VM_DONTDUMP);	
---
> 	vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP;
1294,1296c1292
< 	/* vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP; */
< 	
< 	vm_flags_set(vm, vm->vm_flags | VM_IO | VM_DONTEXPAND | VM_DONTDUMP);
---
> 	vm->vm_flags |= VM_IO | VM_DONTEXPAND | VM_DONTDUMP;
1312,1313c1308
< 	/* vm->vm_flags |= VM_DONTEXPAND | VM_DONTDUMP; */
< 	vm_flags_set(vm, vm->vm_flags | VM_DONTEXPAND | VM_DONTDUMP);
---
> 	vm->vm_flags |= VM_DONTEXPAND | VM_DONTDUMP;
```

### class_create调用参数问题

搜索该函数所在的头文件:

```bash
$ find include -name "*.h" -exec grep -n "class_create(" {} +
# include/linux/device/class.h:230:struct class * __must_check class_create(const char *name);
```

发现其在class.h中定义且只有一个参数

```c
struct class * __must_check class_create(const char *name);
```

修改drivers/soc/spacemit/jpu/jpu.c文件：

```bash
27d26
< #include <linux/device/class.h>
1679,1680c1678
< 	/* jdev->jpu_class = class_create(THIS_MODULE, cdev_name); */
< 	jdev->jpu_class = class_create(cdev_name);
---
> 	jdev->jpu_class = class_create(THIS_MODULE, cdev_name);
```

Ending.