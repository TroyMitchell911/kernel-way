## 在PC上配置IP

在PC与开发板连接好后，可以使用`ifconfig`查看到网卡信息:

```bash
ifconfig

# enx00e099a751b1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
#         ether 00:e0:99:a7:51:b1  txqueuelen 1000  (以太网)
#         RX packets 0  bytes 0 (0.0 B)
#         RX errors 0  dropped 0  overruns 0  frame 0
#         TX packets 80  bytes 14726 (14.7 KB)
#         TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

这个还没有IP信息的就是我们要找的，打开设置，找到有线网络，点击`齿轮`图标：

![u-boot-net-connecting](./images/u-boot-net-connecting.png)

IPv4方式选择手动，自己配置IP地址，子网掩码写固定`255.255.255.0`就好，其他可以自己配置，但要遵循规则，如若不懂，则按照下图配置：

![uboot-net-manual](./images/uboot-net-manual.png)

## 在开发板上的U-Boot配置IP

如若开发板的`bootdelay`不为0的话，可在`uboot`阶段按下`空格`或`s`打断引导，如果`bootdelay`为0，则复位时狂按`s`：

```bash
=> setenv ipaddr 192.168.5.88
=> setenv gatewayip 192.168.5.1
=> setenv netmask 255.255.255.0
=> saveenv
=> ping 192.168.5.99
# ethernet@cac80000 Waiting for PHY auto negotiation to complete..... done
# emac_adjust_link link:1 speed:100 duplex:full
# Using ethernet@cac80000 device
# host 192.168.5.99 is alive

```