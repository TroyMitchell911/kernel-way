## 加载initramfs

首先获取[initramfs](./resources/initramfs-generic.img)，该文件取自bianbu系统的bootfs.ext4。

将其加载到bootfs:

```bash
mount /dev/mmcblk2p2 /mnt

cp initramfs-generic.img /mnt

umount /mnt
sync
```

## 加载rootfs

获取armbian镜像：https://docs.banana-pi.org/en/BPI-F3/BananaPi_BPI-F3

提取rootfs:

```bash
xz -d 2024-05-24-Armbian-bpi-SpacemiT_24.5.0-trunk_Bananapif3_mantic_legacy_6.1.15_gnome_desktop.img.xz

lsblk /dev/loop14
NAME       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop14       7:14   0  6.2G  0 loop 
├─loop14p1 259:3    0  256M  0 part 
└─loop14p2 259:4    0  5.9G  0 part

# 那么很明显p2就是rootfs

dd if=/dev/loop14p2 of=./armbian_rootfs.ext4 status=progress
```

将rootfs加载到rootfs分区:

```bash
dd if=armbian_rootfs.ext4 of=/dev/mmcblk2p3 status=progress
```

由于不太清楚这个rootfs的密码，所以更改root密码:

```bash
mount /dev/mmcblk2p3 /mnt

vim /mnt/etc/shadow
# 将root中间的乱码一样的东西删掉，改成类似下面这样的
# root::19867:0:99999:7:::
```

更改bootfs的uuid，否则无法挂载boot目录就会无法进入rootfs:

```bash
blkid /dev/mmcblk2p*

# /dev/mmcblk2p1: PARTLABEL="uboot" PARTUUID="4dde07ab-d62d-43bc-bf3f-9edcf3dd6b20"
# /dev/mmcblk2p2: UUID="4FBD-68D3" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="bootfs" PARTUUID="edc5c81f-6a98-46c1-96b4-5937323de874"
# /dev/mmcblk2p3: LABEL="BPI-ROOT" UUID="065c8df8-937f-454d-9e62-feb1c0efea59" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="rootfs" PARTUUID="acbed7ee-5434-4e43-a7b4-883b1b0399c"

# 可以看到bootfs的UUID是4FBD-68D3 rootfs的UUID是065c8df8-937f-454d-9e62-feb1c0efea59

vim /mnt/etc/fstab
# 将/和/boot的UUID都对应改掉

umount /mnt
sync
```



