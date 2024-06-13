## 移植GPIO驱动

makefile（ref: ../pi-linux/driver/gpio/Makefile）

```bash
198d197
< obj-$(CONFIG_GPIO_K1X)			+= gpio-k1x.o
```

Kconfig（ref: ../pi-linux/driver/gpio/Kconfig）

```bash
diff drivers/gpio/Kconfig kkk
361,369d360
< config GPIO_K1X
< 	bool "SPACEMIT-K1X GPIO support"
< 	depends on SOC_SPACEMIT_K1X
< 	help
< 	  Say yes here to support the K1X GPIO device.
< 	  The K1X GPIO device may have several banks, and each
< 	  bank control at most 32 GPIO pins. The number of banks
< 	  is passed by device tree or platform data.
< 
```

```bash
$ cp ../pi-linux/drivers/gpio/gpio-k1x.c drivers/gpio/
```

报错信息：

```bash
drivers/gpio/gpio-k1x.c:53:26: error: field 'chip' has incomplete type
   53 |         struct gpio_chip chip;
      |                          ^~~~
In file included from ./include/linux/init.h:5,
                 from drivers/gpio/gpio-k1x.c:10:
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_to_irq':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:65:25: note: in expansion of macro 'container_of'
   65 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_direction_input':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:73:25: note: in expansion of macro 'container_of'
   73 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_direction_output':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:87:25: note: in expansion of macro 'container_of'
   87 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_get':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:103:25: note: in expansion of macro 'container_of'
  103 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_set':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:117:25: note: in expansion of macro 'container_of'
  117 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_of_xlate':
././include/linux/compiler_types.h:366:27: error: expression in static assertion is not an integer
  366 | #define __same_type(a, b) __builtin_types_compatible_p(typeof(a), typeof(b))
      |                           ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
./include/linux/build_bug.h:78:56: note: in definition of macro '__static_assert'
   78 | #define __static_assert(expr, msg, ...) _Static_assert(expr, msg)
      |                                                        ^~~~
./include/linux/container_of.h:20:9: note: in expansion of macro 'static_assert'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |         ^~~~~~~~~~~~~
./include/linux/container_of.h:20:23: note: in expansion of macro '__same_type'
   20 |         static_assert(__same_type(*(ptr), ((type *)0)->member) ||       \
      |                       ^~~~~~~~~~~
drivers/gpio/gpio-k1x.c:135:25: note: in expansion of macro 'container_of'
  135 |                         container_of(chip, struct k1x_gpio_chip, chip);
      |                         ^~~~~~~~~~~~
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_probe':
drivers/gpio/gpio-k1x.c:374:34: error: 'gpiochip_generic_request' undeclared (first use in this function)
  374 |         k1x_chip->chip.request = gpiochip_generic_request;
      |                                  ^~~~~~~~~~~~~~~~~~~~~~~~
drivers/gpio/gpio-k1x.c:374:34: note: each undeclared identifier is reported only once for each function it appears in
drivers/gpio/gpio-k1x.c:375:31: error: 'gpiochip_generic_free' undeclared (first use in this function)
  375 |         k1x_chip->chip.free = gpiochip_generic_free;
      |                               ^~~~~~~~~~~~~~~~~~~~~
drivers/gpio/gpio-k1x.c:395:9: error: implicit declaration of function 'gpiochip_add' [-Werror=implicit-function-declaration]
  395 |         gpiochip_add(&k1x_chip->chip);
      |         ^~~~~~~~~~~~
cc1: some warnings being treated as errors
```

首先查看struct gpio_chip的定义：

```bash
$ find include -name "*.h" -exec grep -n "struct gpio_chip {" {} +
# include/linux/gpio/driver.h:415:struct gpio_chip {
```

尝试加上这个头文件，现在就剩下一个报错信息了：

```bash
drivers/gpio/gpio-k1x.c: In function 'k1x_gpio_probe':
drivers/gpio/gpio-k1x.c:383:24: error: 'struct gpio_chip' has no member named 'of_node'; did you mean 'fwnode'?
  383 |         k1x_chip->chip.of_node = np;
      |                        ^~~~~~~
      |                        fwnode
```

注释该行，参见：https://github.com/raspberrypi/linux/issues/5472