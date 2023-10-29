---
title: pve折腾日记
date: 2023-10-29 14:00:00
categories: 
- pve
tags:
- linux
- debian
- pve
---

# pve折腾日记

安装就不介绍了，Proxmox Virtual Environment (PVE) 是基于 Debian 的虚拟化平台。所以安装前建议先格式化为`ext4`或其他格式而非`NTFS`格式，以免安装中无法读盘。

#### 1. 拔插pcie设备，导致进不去后台

如：WiFi网卡、显卡、拆分卡、U2等设备

需要：显示屏、键盘连接至pve设备

连接显示屏，并进入后台输入以下命令

```bash
//查看设备信息及其序号
lspci
//或者查看当前网卡名称及型号
lspci | grep -i ethernet
```

```shell
//输出
root@pve:~# lspci | grep -i ethernet
01:00.0 Ethernet controller: Intel Corporation Ethernet Controller I225-V (rev 03)
```

找到对应网卡，比如我只有一个，然后根据 `01:00.0` (取`01`, 16进制, 例如`0c:00.0就是12`)找到相应的网卡名称 `enp1s0` (`0c`就是`enp12s0`)

然后修改 `/etc/network/interfaces` 的内容

```bash
//你可以使用nano或者vi编辑文件
nano /etc/network/interfaces
```

比如我的是

```
auto lo
iface lo inet loopback

iface **enp2s0** inet manual

auto vmbr0
iface vmbr0 inet static
	address 172.16.0.188/21
	gateway 172.16.0.1
	bridge-ports enp1s0
	bridge-stp off
	bridge-fd 0

iface **wlp2s0** inet manual
```

打 `**` 处修改为 `wlp1s0` 即可。个人大同小异，找到修改即可
