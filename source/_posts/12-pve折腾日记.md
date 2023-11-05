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

安装就不介绍了，Proxmox Virtual Environment ([PVE](https://www.proxmox.com/)) 是基于 Debian 的虚拟化平台。

> 本机均使用root账户，报错请使用sudo命令

## 1. PVE安装/问题解决

> 安装前建议先格式化为`ext4`或其他格式而非`NTFS`格式，以免安装中无法读盘。

>你可以选用以下软件**格式化**(windows):
>
>1. [Rufus](https://rufus.ie/)支持FAT32/NTFS/UDF/ecFAT/ext2/ext3(开启格式化进阶) 注：ext4不支持
>2. DiskGenius [com](https://www.diskgenius.com/)&[cn](https://www.diskgenius.cn/)

pve安装过程很简单，先下载[Proxmox VE 8.0 ISO Installer](https://www.proxmox.com/en/downloads)，[Rufus](https://rufus.ie/) / [balenaEtcher](https://etcher.io/)(支持Linux) 拷进去，进bios u盘启动(ssd/sata没东西会自动读u盘)，默认第一个图形安装，稍等片刻，出现页面后同意，到`Country`选语言、密码、Email(没啥用)、网络(可让他自动获取，进不去后台请往下看)。

安装好之后会自动重启，记得拔掉u盘。之后页面会显示后台进入地址，一般类似为`https://172.16.0.188:8006/`的地址

### 问题

#### 1. 插拔pcie设备，导致进不去后台

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

找到对应网卡，比如我只有一个，然后根据 `01:00.0` (取`01`, 16进制, 例如`0c:00.0就是12`)找到相应的网卡名称 `enp1s0`  (`0c`就是`enp12s0`)

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



## 2. 创建LXC容器

### 下载

你可以从 [cxthhhhh](https://odc.cxthhhhh.com/SyStem/) 或者 [tuna](https://mirrors.tuna.tsinghua.edu.cn/lxc-images/images/) 下载镜像包

### 创建/安装

设置主机名(随便写)、密码(记住)，**取消勾选 `无特权容器`** ，下一步选模板，自己分配磁盘CPU内存，网络我选DHCP(之后到路由器里看IP地址)。

安装好后不要开机，选择容器-选项**开启嵌套**(创建容器用, 比如docker), NFS(共享文件系统)

> 下面我不是很懂，看视频教程是这样

进入PVE后台，编辑文件 `nano /etc/pve/lxc/100.conf` , `100`改为你的容器ID，加上以下内容让他可以安装Docker：

```conf
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```

然后ctrl x, y, 回车保存。示例文件

```shell
arch: amd64
cores: 4
features: fuse=1,mount=nfs,nesting=1
hostname: debian12
memory: 2048
net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=12:12:12:12:12:12,ip=dhcp,ip6=dhcp,type=veth
ostype: debian
rootfs: local-lvm:vm-100-disk-0,size=40G
swap: 2048
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:
```

启动容器，找到路由器的DHCP，查看设备ipv4地址，进入后台。密码root, 密码为自己设置的。(你也可以使用pve面板的控制台)

> 容器架构可以在选项中查看，我是amd64

#### 修改debian源

> 本机均使用root账户，报错请使用sudo命令

1. 备份

   ```shell
   #移动使用mv 复制使用cp
   cp /etc/apt/sources.list /etc/apt/sources.list.bak
   ```

2. 切换源（**请不要全复制**

   `deb` 用于获取已编译的二进制软件包，而 `deb-src` 用于获取软件包的源代码。

   ```shell
   nano /etc/apt/sources.list
   ```

```shell
# 中科大源 可以改成https
deb http://mirrors.ustc.edu.cn/debian buster main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-updates main contrib non-free
deb http://mirrors.ustc.edu.cn/debian buster-backports main contrib non-free
deb http://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src http://mirrors.ustc.edu.cn/debian buster main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian buster-updates main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian buster-backports main contrib non-free
deb-src http://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free

# 清华大学源
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free

deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free

# 官方源
deb http://deb.debian.org/debian buster main contrib non-free
deb http://deb.debian.org/debian buster-updates main contrib non-free
deb http://deb.debian.org/debian-security/ buster/updates main contrib non-free

deb-src http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian-security/ buster/updates main contrib non-free

# 网易源
deb http://mirrors.163.com/debian/ buster main non-free contrib
deb http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ buster/updates main non-free contrib

deb-src http://mirrors.163.com/debian/ buster main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.163.com/debian-security/ buster/updates main non-free contrib

# 阿里云
deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main

deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian-security buster/updates main
```

更新包管理器的软件包列表

```shell
apt update   # 获取最新的软件包信息
apt upgrade  # 升级已安装的软件包(建议)
```





## 3. 安装docker

~~我也不知道怎么安装成功的，debian源使用的官方源。就是从Docker[官方](https://docs.docker.com/engine/install/debian/)教程来一行一行试，不行(最后`sudo apt-get update`报错)。然后删除了`/etc/apt/sources.list.d/docker.list` , 之后使用一键安装指令就可行了~~(网络环境要*好像?建议官方源)

安装

```shell
# 稳定
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 测试通道
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```
安装完成后测试是否安装完成

```shell
docker -v
# 启动docker(除非docker不在运行中)
systemctl start docker
```
设置开机自启
```shell
systemctl enable docker.service # 或者 systemctl enable docker
# 查看是否设置自启动成功
systemctl list-unit-files | grep enable | grep docker
#回复应该是 docker.service             enabled
```

其他指令

```shell
docker ps -a #查看Docker镜像状态
docker start 885c7e250065 #启动镜像
docker update --restart=always 885c7e250065 # 设置镜像自启动
```

### 安装docker可视化面板

主要是方便修改管理

我安装的是[portainer](https://docs.portainer.io/start/install-ce)面板，如果不熟悉英文，可使用[FAST OS DOCKER](https://hub.docker.com/r/wangbinxingkong/fast)(功能没portainer多)或者[汉化版](https://hub.docker.com/r/6053537/portainer-ce)：

```shell
# 汉化版
docker pull 6053537/portainer-ce
docker run -d --restart=always --name="portainer" -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock 6053537/portainer-ce
```

然后访问9000端口设置密码(12位)进入即可

## 4. CF Tunnel安装使用教程

> 需要绑定银行账户，缺点：必须绕美或者购买 Enterprise($3,000起步/月)

### 1. 安装

#### 本地

1. 下载安装包（访问GitHub快的话使用

方法1：官方获取命令 你可以根据架构自己改 64:amd64 32:386 arm64:arm64 arm32:arm

```shell
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```

方法2：去[Github](https://github.com/cloudflare/cloudflared/releases)或[官方](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)下载对应版本(Debian/ubu是`.deb`, CentOS是`.rpm`，然后上传上去

2. 安装/登录

```shell
dpkg -i cloudflared.deb #通过pwd自己看包名，输一点之后按tab补全就行
```
#### Docker安装

进入cf面板，点开自己`账号-->Zero Trust-->Access-->Tunnels-->Create a tunnel`

啥的自己写，进入后会告诉你指令

```shell
docker pull cloudflare/cloudflared
#然后执行面板上的指令
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <token> #token很长
```

### 2. 使用教程

接下来你可以使用**网页仪表板**（小白建议）或者纯**CLI命令版**

1. 仪表板：使用网页token(代码网页上有)，但只能在网页操作，使用后建议在网页操作，也方便远程控制，可以参考[官方页面](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)

   安装token报错可能是你已经装过, 需要重新配置

   ```shell
   sudo cloudflared service uninstall
   ```

2. CLI版：登录 会给一个cf登录地址。或者参考[官方页面](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-local-tunnel/)

```shell
cloudflared tunnel login
```

设置开机启动

```shell
cloudflared service install
systemctl start cloudflared
systemctl enable cloudflared
systemctl status cloudflared
```

> 使用`cloudflared tunnel login`之后你可以都在cf tunnel网页上编辑
>
> 1. 临时访问（不登陆可以临时使用，但重新部署会换域名，会有一个类似`https://wonderful-hotmail-ah-cassette.trycloudflare.com`域名（靠前的位置）
>
> ```shell
> cloudflared tunnel --url localhost:8000 #自己改端口号，ctrl c停止
> ```
> ```shell
> cloudflared tunnel create <隧道名称(Tunnel name)> #创建新隧道
> cloudflared tunnel list #查看部署了哪些隧道
> #创建新隧道。Tunnel name同上，subdomain是创建的域名/子域名，例如a.qsim.top
> cloudflared tunnel route dns <Tunnel name> <subdomain> 
> cloudflared tunnel --url localhost:8000 run <Tunnel name> #使用自己的子域名穿透
> cloudflared tunnel delete <Tunnel name> #删除隧道
> 
> ```
>
> 其余你可以到[官方文档](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/)查看（上面是备份

### 3. 本地卸载

1. apt 均需要管理(`sudo`)权限。直接删除`~/.cloudflared`文件夹就会删除登录信息

```bash
systemctl list-units | grep cloudflared #寻找是否在后台运行
systemctl stop cloudflared #停止服务
# apt remove cloudflared #仅卸载，不删除配置
apt-get purge cloudflared #连同相关的配置文件一起删除
#如果使用remove，你可以自己删除配置文件，也可以使用上面指令删除
rm -r /etc/cloudflared/
rm -r /var/lib/cloudflared/
#卸载完成后，你可以使用以下命令来清理不再使用的依赖项
apt-get autoremove
```

```shell
systemctl stop cloudflared-update.timer #停止cloudflared-update
systemctl disable cloudflared-update.timer #禁用cloudflared-update
```







