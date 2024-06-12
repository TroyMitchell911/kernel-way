## 获取Linux源码

```bash
git clone https://github.com/TroyMitchell911/pi-linux && cd pi-linux && git checkout linux-6.1.15-k1
```

## 编译

查找配置文件:

```bash
find -name k1*config

# ./arch/riscv/configs/k1-x_fpga_1x4_defconfig
# ./arch/riscv/configs/k1-x_fpga_defconfig
# ./arch/riscv/configs/k1_defconfig
# ./arch/riscv/configs/k1-x_fpga_2x2_defconfig
```

编译:

```bash
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- k1_defconfig && make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j16
```

编译生成文件:

```bash
./arch/riscv/boot/Image.itb
./k1-x_deb1.dtb
```

## 放置kernel和dtb

```bash
mount /dev/mmcblk2p2 /mnt
cp k1-x_deb1.dtb /mnt
cp Image.itb /mnt
umount /mnt
```

现在拔掉sd卡开机应该可以看到启动内核现象了。
