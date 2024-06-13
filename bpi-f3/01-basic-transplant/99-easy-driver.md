## 一些简单的驱动移植

### driver/clk

```bash
$ cp -r ../pi-linux/drivers/clk/spacemit/ drivers/clk/
$ vim drivers/clk/Makefile
# 在其中增加：
# obj-y					+= spacemit/
$ vim drivers/clk/Kconfig
# 在其中增加:
# source "drivers/clk/spacemit/Kconfig"
```

### driver/tty/serial

思路与driver/clk大概相同

### driver/reset

首先修改Kconfig以便支持k1的选项：

```bash
$ vim drivers/reset/Kconfig

321,338d320
< config RESET_K1PRO_SPACEMIT
< 	tristate "Reset controller driver for Spacemit K1PRO SoCs"
< 	depends on SOC_SPACEMIT_K1PRO
< 	help
< 	  Support for reset controllers on Spacemit K1PRO SoCs.
< 
< config RESET_K1X_SPACEMIT
< 	tristate "Reset controller driver for Spacemit K1X SoCs"
< 	depends on SOC_SPACEMIT_K1X
< 	help
< 	  Support for reset controllers on Spacemit K1X SoCs.
< 
< config RESET_K1MATRIX_SPACEMIT
< 	tristate "Reset controller driver for Spacemit K1MATRIX SoCs"
< 	default y
< 	help
< 	  Support for reset controllers on Spacemit K1MATRIX SoCs.
< 
```

修改Makefile支持k1所编译的文件:

```bash
$ vim driver/reset/Makefile

44,46d43
< obj-$(CONFIG_RESET_K1PRO_SPACEMIT) += reset-spacemit-k1pro.o
< obj-$(CONFIG_RESET_K1X_SPACEMIT) += reset-spacemit-k1x.o
< obj-$(CONFIG_RESET_K1_MATRIX_SPACEMIT) += reset-spacemit-k1matrix.o
```

复制相关文件：

```bash
$ cp ../pi-linux/drivers/reset/reset-spacemit-k1x.c drivers/reset/
$ cp ../pi-linux/include/dt-bindings/reset/spacemit-k1x-reset.h include/dt-bindings/reset/
```

### driver/clocksource

Kconfig

```bash
137,143d136
< config SPACEMIT_K1X_TIMER
< 	bool "Spacemit k1x timer driver" if COMPILE_TEST
< 	select CLKSRC_MMIO
< 	select TIMER_OF
< 	help
< 	  Enables the support for the spacemit k1x timer driver.
< 
```

Makefile

```bash
92d91
< obj-$(CONFIG_SPACEMIT_K1X_TIMER)	+= timer-k1x.o
```

dw_apb_timer.c

```bash
104c104
< 	raw_spin_lock(&dw_ced->timer_lock);
---
> 
107d106
< 	raw_spin_unlock(&dw_ced->timer_lock);
210,211d208
< 	
< 	raw_spin_lock(&dw_ced->timer_lock);
221,222d217
< 	
< 	raw_spin_unlock(&dw_ced->timer_lock);
256,257d250
< 	raw_spin_lock_init(&dw_ced->timer_lock);
< 
282c275
< 	/* err = request_irq(irq, dw_apb_clockevent_irq,
---
> 	err = request_irq(irq, dw_apb_clockevent_irq,
284,286c277
< 			  dw_ced->ced.name, &dw_ced->ced); */
< 	err = request_irq(irq, dw_apb_clockevent_irq, IRQF_ONESHOT,
< 			dw_ced->ced.name, &dw_ced->ced);
---
> 			  dw_ced->ced.name, &dw_ced->ced);
```

timer-riscv.c

```bash
12a13
> #include <linux/acpi.h>
30a32
> static bool riscv_timer_cannot_wake_cpu;
36,72d37
< 	csr_set(CSR_IE, IE_TIE);
< 
< 	if (static_branch_likely(&riscv_sstc_available)) {
< #if defined(CONFIG_32BIT)
< 		csr_write(CSR_STIMECMP, next_tval & 0xFFFFFFFF);
< 		csr_write(CSR_STIMECMPH, next_tval >> 32);
< #else
< 		csr_write(CSR_STIMECMP, next_tval);
< #endif
< 	} else
< 		sbi_set_timer(next_tval);
< 
< 	return 0;
< }
< 
< static int riscv_set_state_shutdown(struct clock_event_device *ce)
< {
< 	u64 next_tval = 0xffffffffffffffff;
< 
<         csr_clear(CSR_IE, IE_TIE);
< 
< 	if (static_branch_likely(&riscv_sstc_available)) {
< #if defined(CONFIG_32BIT)
< 		csr_write(CSR_STIMECMP, next_tval & 0xFFFFFFFF);
< 		csr_write(CSR_STIMECMPH, next_tval >> 32);
< #else
< 		csr_write(CSR_STIMECMP, next_tval);
< #endif
< 	} else
< 		sbi_set_timer(next_tval);
< 
<         return 0;
< }
< 
< static int riscv_set_state_oneshot(struct clock_event_device *ce)
< {
< 	u64 next_tval = 0xffffffffffffffff;
75d39
< 
92c56
< 	.features               = CLOCK_EVT_FEAT_ONESHOT | CLOCK_EVT_FEAT_C3STOP,
---
> 	.features		= CLOCK_EVT_FEAT_ONESHOT,
95,97d58
<         .set_state_shutdown     = riscv_set_state_shutdown,
<         .set_state_oneshot_stopped = riscv_set_state_shutdown,
< 	.set_state_oneshot = riscv_set_state_oneshot,
117c78
< 	.rating		= 300,
---
> 	.rating		= 400,
120a82,86
> #if IS_ENABLED(CONFIG_GENERIC_GETTIMEOFDAY)
> 	.vdso_clock_mode = VDSO_CLOCKMODE_ARCHTIMER,
> #else
> 	.vdso_clock_mode = VDSO_CLOCKMODE_NONE,
> #endif
128a95,96
> 	if (riscv_timer_cannot_wake_cpu)
> 		ce->features |= CLOCK_EVT_FEAT_C3STOP;
160c128
< static int __init riscv_timer_init_dt(struct device_node *n)
---
> static int __init riscv_timer_init_common(void)
162,164c130
< 	int cpuid, error;
< 	unsigned long hartid;
< 	struct device_node *child;
---
> 	int error;
165a132
> 	struct fwnode_handle *intc_fwnode = riscv_get_intc_hwnode();
167,190c134
< 	error = riscv_of_processor_hartid(n, &hartid);
< 	if (error < 0) {
< 		pr_warn("Not valid hartid for node [%pOF] error = [%lu]\n",
< 			n, hartid);
< 		return error;
< 	}
< 
< 	cpuid = riscv_hartid_to_cpuid(hartid);
< 	if (cpuid < 0) {
< 		pr_warn("Invalid cpuid for hartid [%lu]\n", hartid);
< 		return cpuid;
< 	}
< 
< 	if (cpuid != smp_processor_id())
< 		return 0;
< 
< 	domain = NULL;
< 	child = of_get_compatible_child(n, "riscv,cpu-intc");
< 	if (!child) {
< 		pr_err("Failed to find INTC node [%pOF]\n", n);
< 		return -ENODEV;
< 	}
< 	domain = irq_find_host(child);
< 	of_node_put(child);
---
> 	domain = irq_find_matching_fwnode(intc_fwnode, DOMAIN_BUS_ANY);
192c136,137
< 		pr_err("Failed to find IRQ domain for node [%pOF]\n", n);
---
> 		pr_err("Failed to find irq_domain for INTC node [%pfwP]\n",
> 		       intc_fwnode);
198c143
< 		pr_err("Failed to map timer interrupt for node [%pOF]\n", n);
---
> 		pr_err("Failed to map timer interrupt for node [%pfwP]\n", intc_fwnode);
202,210d146
< #ifdef CONFIG_SOC_SPACEMIT
< 	if (riscv_isa_extension_available(NULL, SSTC)) {
< 		pr_info("Timer interrupt in S-mode is available via sstc extension\n");
< 		static_branch_enable(&riscv_sstc_available);
< 	}
< #endif
< 
< 	pr_info("%s: Registering clocksource cpuid [%d] hartid [%lu]\n",
< 	       __func__, cpuid, hartid);
213,214c149
< 		pr_err("RISCV timer register failed [%d] for cpu = [%d]\n",
< 		       error, cpuid);
---
> 		pr_err("RISCV timer registration failed [%d]\n", error);
227a163,167
> 	if (riscv_isa_extension_available(NULL, SSTC)) {
> 		pr_info("Timer interrupt in S-mode is available via sstc extension\n");
> 		static_branch_enable(&riscv_sstc_available);
> 	}
> 
235,240d174
< #ifndef CONFIG_SOC_SPACEMIT
< 	if (riscv_isa_extension_available(NULL, SSTC)) {
< 		pr_info("Timer interrupt in S-mode is available via sstc extension\n");
< 		static_branch_enable(&riscv_sstc_available);
< 	}
< #endif
243a178,209
> static int __init riscv_timer_init_dt(struct device_node *n)
> {
> 	int cpuid, error;
> 	unsigned long hartid;
> 	struct device_node *child;
> 
> 	error = riscv_of_processor_hartid(n, &hartid);
> 	if (error < 0) {
> 		pr_warn("Invalid hartid for node [%pOF] error = [%lu]\n",
> 			n, hartid);
> 		return error;
> 	}
> 
> 	cpuid = riscv_hartid_to_cpuid(hartid);
> 	if (cpuid < 0) {
> 		pr_warn("Invalid cpuid for hartid [%lu]\n", hartid);
> 		return cpuid;
> 	}
> 
> 	if (cpuid != smp_processor_id())
> 		return 0;
> 
> 	child = of_find_compatible_node(NULL, NULL, "riscv,timer");
> 	if (child) {
> 		riscv_timer_cannot_wake_cpu = of_property_read_bool(child,
> 					"riscv,timer-cannot-wake-cpu");
> 		of_node_put(child);
> 	}
> 
> 	return riscv_timer_init_common();
> }
> 
244a211,220
> 
> #ifdef CONFIG_ACPI
> static int __init riscv_timer_acpi_init(struct acpi_table_header *table)
> {
> 	return riscv_timer_init_common();
> }
> 
> TIMER_ACPI_DECLARE(aclint_mtimer, ACPI_SIG_RHCT, riscv_timer_acpi_init);
> 
> #endif

```

### driver/I2C

首先修改Kconfig以便支持k1的选项：

```bash
$ vim drivers/i2c/busses/Kconfig

303,310d302
< config I2C_SPACEMIT_K1X
< 	tristate "Spacemit k1x I2C adapter"
< 	help
< 	  Say yes if you want to use I2C interface on sapcemit k1x platform.
< 
< 	  This driver can also be built as a module. If so, the module will be
< 	  called i2c-spacemit-k1x.
< 

```

修改Makefile支持k1所编译的文件:

```bash
$ vim drivers/i2c/busses/Makefile

160d159
< obj-$(CONFIG_I2C_SPACEMIT_K1X)      += i2c-k1x.o
```

复制相关文件：

```bash
$ cp ../pi-linux/drivers/i2c/busses/i2c-k1x.* drivers/i2c/busses/
```

这里应该会遇到无法编译i2c文件的问题，这是因为i2c-support没开启，用menuconfig打开一下，然后更新k1_defconfig就行了。

### driver/irqchip

修改drivers/irqchip/irq-sifive-plic.c(ref: ../pi-linux/drivers/irqchip/irq-sifive-plic.c)：

```bash
42c42
< #define PRIORITY_PER_ID		        4
---
> #define     PRIORITY_PER_ID		4
48,52d47
< #ifdef CONFIG_SOC_SPACEMIT
< #define PENDING_BASE			0x1000
< #define CONTEXT_ENABLE_BASE		0x2080
< #define CONTEXT_ENABLE_SIZE		0x100
< #else
54,55c49,50
< #define CONTEXT_ENABLE_SIZE		0x80
< #endif
---
> #define     CONTEXT_ENABLE_SIZE		0x80
> 
61,64d55
< #ifdef CONFIG_SOC_SPACEMIT
< #define CONTEXT_BASE			0x201000
< #define CONTEXT_SIZE			0x2000
< #else
66,67c57,59
< #define CONTEXT_SIZE			0x1000
< #endif
---
> #define     CONTEXT_SIZE		0x1000
> #define     CONTEXT_THRESHOLD		0x00
> #define     CONTEXT_CLAIM		0x04
69,73c61,62
< #define CONTEXT_THRESHOLD		0x00
< #define CONTEXT_CLAIM			0x04
< 
< #define PLIC_DISABLE_THRESHOLD		0x7
< #define PLIC_ENABLE_THRESHOLD		0
---
> #define	PLIC_DISABLE_THRESHOLD		0x7
> #define	PLIC_ENABLE_THRESHOLD		0
118,123d106
< 
< #ifdef CONFIG_SOC_SPACEMIT
< 	if (!enable)
< 		writel(hwirq, handler->hart_base + CONTEXT_CLAIM);
< #endif
< 
125,130d107
< 
< #ifdef CONFIG_SOC_SPACEMIT
< 	if (enable)
< 		writel(hwirq, handler->hart_base + CONTEXT_CLAIM);
< #endif
< 
174,182c151
< 	/* writel(d->hwirq, handler->hart_base + CONTEXT_CLAIM); */
< 
< 	if (unlikely(irqd_irq_disabled(d))) {
< 		plic_toggle(handler, d->hwirq, 1);
< 		writel(d->hwirq, handler->hart_base + CONTEXT_CLAIM);
< 		plic_toggle(handler, d->hwirq, 0);
< 	} else {
< 		writel(d->hwirq, handler->hart_base + CONTEXT_CLAIM);
< 	}
---
> 	writel(d->hwirq, handler->hart_base + CONTEXT_CLAIM);
225,229c194,195
< #ifdef CONFIG_SOC_SPACEMIT
< 	.flags		= IRQCHIP_AFFINITY_PRE_STARTUP | IRQCHIP_ENABLE_WAKEUP_ON_SUSPEND | IRQCHIP_SKIP_SET_WAKE,
< #else
< 	.flags		= IRQCHIP_AFFINITY_PRE_STARTUP,
< #endif
---
> 	.flags		= IRQCHIP_SKIP_SET_WAKE |
> 			  IRQCHIP_AFFINITY_PRE_STARTUP,
243,247c209,210
< #ifdef CONFIG_SOC_SPACEMIT
< 	.flags		= IRQCHIP_AFFINITY_PRE_STARTUP | IRQCHIP_ENABLE_WAKEUP_ON_SUSPEND | IRQCHIP_SKIP_SET_WAKE,
< #else
< 	.flags		= IRQCHIP_AFFINITY_PRE_STARTUP,
< #endif
---
> 	.flags		= IRQCHIP_SKIP_SET_WAKE |
> 			  IRQCHIP_AFFINITY_PRE_STARTUP,
551,554d513
< #ifdef CONFIG_SOC_SPACEMIT
< 		handler->hart_base = priv->regs + CONTEXT_BASE +
< 			hartid * CONTEXT_SIZE;
< #else
557d515
< #endif
559,562d516
< #ifdef CONFIG_SOC_SPACEMIT
< 		handler->enable_base = priv->regs + CONTEXT_ENABLE_BASE +
< 			hartid * CONTEXT_ENABLE_SIZE;
< #else
565d518
< #endif
575,578d527
< 			#ifdef CONFIG_SOC_SPACEMIT
< 			/* clear pending, which maybe triggered by uboot */
< 			writel(0, priv->regs + PENDING_BASE + (hwirq/32)*4);
< 			#endif

```

### driver/watchdog

首先修改Kconfig：

```bash
$ vim drivers/watchdog/Kconfig

2260,2273d2259
< config SPACEMIT_WATCHDOG
< 	tristate "Spacemit-k1x SoC Watchdog"
< 	depends on SOC_SPACEMIT_K1X
< 	select WATCHDOG_CORE
< 	help
< 	  spacemit k1x plat SoC Watchdog timer. This will reboot your system when
< 	  the timeout is reached.
< 
< config K1X_WDT_TEST
< 	bool "Support K1X watchdog test"
< 	depends on SOC_SPACEMIT_K1X && SPACEMIT_WATCHDOG
< 	help
< 	  This will enable K1X watchdog timer test
< 
```

修改Makefile：

```bash
$ vim drivers/watchdog/Makefile
198d197
< obj-$(CONFIG_SPACEMIT_WATCHDOG) += k1x_wdt.o
```

复制文件：

```bash
$ cp ../pi-linux/drivers/watchdog/k1x_wdt.c drivers/watchdog/
```

