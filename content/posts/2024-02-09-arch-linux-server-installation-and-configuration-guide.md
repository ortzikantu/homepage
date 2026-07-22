+++
title = '一份 Arch Linux 服务器的安装与配置极简指南'
date = 2024-02-09T12:37:00+08:00
update = 2026-04-16T20:09:00+08:00
author = "Hao Wu"
description = ""
tags = []
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
toc = true
draft = false
cover = "/assets/photos/2024/IMG_20240907_151752_hu_785f078dbf859161.webp"
+++

安装 Arch Linux，我向来不爱记笔记，忘了就翻收藏夹、查 ArchWiki。直到最近淘了台小主机准备部署，才惊觉收藏夹里的文章不是过时就是消失。正因如此，我决定写下这篇文章，留存一份可随时查阅与更新的安装配置指南。

淘来的是戴尔 OptiPlex 3060MFF 迷你主机，到手时外壳干净却难掩岁月痕迹。CPU 是与主机一同购入的英特尔 i5 8500T，内存与固态硬盘则拆自退役老笔记本——三星+光威混搭的双 8G 内存，搭配一块 256G 致态 PC005。这类商务定位的主机确实不错，体型小巧，功耗还低，更能直接挂装在显示器背面，比商务一体机好调整、易维护。跑轻量级代码编译、服务部署完全够用，用它当专属开发服务器再合适不过。

## 安装系统

Windows 用户可前往 Arch Linux 官网，从分流服务器下载[最新系统镜像](https://archlinux.org/download/#http-downloads)。下载完成后用[Rufus](https://rufus.ie/)将镜像刻录到U盘中，进入 BIOS 将U盘设置为第一启动项，保存重启后即可进入 Arch Linux 临时系统。该临时系统内置多款急救工具，尽管如今 Arch Linux 滚挂的概率很低，但仍建议保留这块安装盘以备不时之需。

> 小提示：Arch Linux 默认不支持安全启动，需要专门去BIOS关闭“安全启动”。

### 连接网络

安装过程中需联网下载软件包，以下分享我常用的两种联网方法：

第一种是有线联网（网线、手机USB共享网络等均可），插上网线或数据线后，在终端执行`dhcpcd`命令即可完成联网，一键操作十分便捷。

第二种方法则是使用英特尔开源的`iwd`工具通过无线网卡连接无线网，具体步骤如下：

1. 输入`iwctl`并回车，进入iwd操作环境：

```bash
iwctl
```

2. 为确保操作准确，先执行`device list`查看无线网卡名称（通常为`wlan0`，下文均以此为例，实际需按列表显示的名称调整）：

```bash
device list
```

1. 连接WiFi（熟悉后可直接执行第三步）：
    - 扫描附近WiFi：`station wlan0 scan`
    - 查看WiFi列表（获取 SSID）：`station wlan0 get-networks`
    - 连接指定WiFi，回车后输入密码再确认即可：`station wlan0 connect WIFI-1234`

> 小提示：建议WiFi名称和密码仅使用英文及常见符号——临时系统默认未配置中文输入法和 `noto-fonts-cjk` 字库，中日韩文字会显示方块乱码。

4. 连接成功后输入`quit`退出`iwd`。

网络确认连通后，需启用网络时间同步并设置时区，避免后续出错：

```bash
timedatectl set-ntp true
timedatectl set-timezone Asia/Shanghai
```

### 硬盘分区

再次确保能够连上网络后就可开始进行硬盘分区，这里先使用`lsblk`命令列出所有硬盘及其分区，硬盘名字多半也是`/dev/sda`、`/dev/sdb`、`/dev/nvme0n1`之类。由于这台设备仅有一块NVME固态硬盘，所以这里使用`/dev/nvme0n1`来进行操作，具体如下所示，用`fdisk`工具选定`/dev/nvme0n1`硬盘来进行分区操作。

```bash
fdisk /dev/nvme0n1
```

回车后便会进入`fdisk`的环境，第一步输入`g`并回车将硬盘分区表格式化成GPT（GUID分区表）格式;第二步输入`n`并回车创建新分区，其中分区序号、分区起点不用管，回车两次到要求输入分区终点时停下，可以输入`+512M`划出512MB空间、`+1G`为划出1G空间、`+8G`划出8G空间、什么都不写直接回车则是划出剩余所有空间，分区完成后一定要输入`w`保存分区改动再退出，否则就得从头再来。下表是我的分区方案，仅供参考：

|硬盘分区|挂载路径|空间|
|:-:|:-:|:-:|
|/dev/nvme0n1p1|/boot|1GB|
|/dev/nvme0n1p2|/|硬盘剩余所有空间|

分区结束后即可对这三个分区分别格式化：

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -f /dev/nvme0n1p2
```

因为主分区使用的是Btrfs文件系统，所以格式化完成后要将刚在硬盘里划出的`/dev/nvme0n1p2`分区挂载到临时系统中的`/mnt`目录作为新系统的根目录，创建并初始化几个Btrfs子卷，创建完成后再将该分区卸载：

```bash
mount /dev/nvme0n1p2 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
chattr +C /mnt/@var
umount /mnt
```

卸载完成后再按顺序重新挂载`/dev/nvme0n1p2`分区到临时系统的`/mnt`目录并附带压缩功能，然后创建boot目录并将`/dev/nvme0n1p1`分区挂在于其上，再将刚才创建的Btrfs子卷挂载到各自的目录：

```bash
mount /dev/nvme0n1p3 /mnt -o subvol=@,compress=zstd
mkdir /mnt/boot
mkdir /mnt/home
mkdir /mnt/var
mount /dev/nvme0n1p1 /mnt/boot
mount /dev/nvme0n1p3 /mnt/home -o subvol=@home,compress=zstd,nosuid,nodev
mount /dev/nvme0n1p3 /mnt/var -o subvol=@var
```

### 安装基础系统和软件包

这里作为演示仅按前文提到的设备硬件来选择软件包，其余情况会略微补充：

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware grub btrfs-progs intel-ucode efibootmgr bash zsh dhcpcd iwd nano vim openssh git
```

- `base` 系统基础组件
- `base-devel` 包含一些编译组件
- `linux` 内核，也可选zen版、lts版等其他内核
- `linux-headers` 配套上面的内核
- `linux-firmware` 内核中未包含的所有驱动固件包
- `grub` 启动系统时需要用到
- `btrfs-progs` 处理Btrfs文件系统所需
- `intel-ucode` 英特尔微码
- `efibootmgr` UEFI引导所需
- `bash`和`zsh` 都是shell工具
- `nano` 文本编辑器，比vi/vim简单很多
- `vim` 文本编辑器，有些地方会用到
- `openssh` 远程连接需要
- `dhcpcd`与`iwd` 皆为网络工具
- `git` 大名鼎鼎的代码管理工具，拿来下载自己的配置仓库

> [2025年6月22日更新] `linux-firmware`包已拆分，可以按需安装所需固件：
> - linux-firmware-amdgpu AMD
> - linux-firmware-atheros 高通
> - linux-firmware-broadcom 博通
> - linux-firmware-cirrus Cirrus Logic
> - linux-firmware-intel 英特尔
> - linux-firmware-mediatek 联发科
> - linux-firmware-nvidia 英伟达
> - linux-firmware-other 其他
> - linux-firmware-radeon AMD
> - linux-firmware-realtek 瑞昱

> [2025年12月28日更新] 由于近日英特尔收紧开源支持，`iwd`项目开发暂停，这个项目后续命运未卜，特此新增候补方案：
> **NetworkManager**（源自Red Hat，现由GNOME管理维护，功能丰富，只是相较于`iwd`不够KISS，因此基本不用）
> 1. 启动服务NetworkManager.service：
> ```bash
> systemctl enable NetworkManager.service
> systemctl start NetworkManager.service
> ```
> 2. 查看WiFi列表
> ```bash
> nmcli device wifi list
> ```
> 3. 连接WiFi：
> ```bash
> nmcli device wifi connect WIFI-1234 password 密码
> ```
> 4. 关闭Wifi：
> ```bash
> nmcli radio wifi off
> ```
> 这里仅用到NetworkManager的`nmcli`连接无线网的基础功能，其他功能均可前往[ArchWiki](https://wiki.archlinuxcn.org/wiki/NetworkManager)了解。

若想使用图形界面建议安装以下图形驱动：
- `mesa` OpenGL驱动
- `vulkan-intel` 英特尔核显的Vulkan驱动

若使用的是AMD处理器则可将`intel-ucode`更换为`amd-ucode`，Vulkan驱动需要由`vulkan-intel`更换为`vulkan-radeon`。

全部安装完成后便可生成分区表`fstab`：

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

至此，基本系统安装完成。

## 配置系统

使用`arch-chroot`命令前往新系统进行更进一步配置：

```bash
arch-chroot /mnt
```

### 时区、区域与主机设置

进入新系统后与前面操作类似，首先设置时区，然后将硬件时间调整为当前的系统时间。

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

hwclock --systohc
```

时间设置完成之后可以使用`timedatectl`命令查看当前时间，然后使用nano或者vim编辑`/etc/locale.gen`，往下翻找到并取消注释`en_US.UTF-8 UTF-8`或`zh_CN.UTF-8 UTF-8`，保存退出并使用`locale-gen`命令生成区域设置。：

```bash
nano /etc/locale.gen
locale-gen
```

设置语言：

```bash
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```
或
```bash
echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```

想要显示中文还需要安装中文字体：
```bash
pacman -Sy noto-fonts-cjk
```

设置主机名，这里我用arch，可按喜好自定义：

```bash
echo 'arch' > /etc/hostname
```

编辑`hosts`文件：

```bash
nano /etc/hosts
```

进入`hosts`文件后输入以下内容：

```bash
127.0.0.1	localhost
::1	    	localhost
127.0.0.1	arch.localdomain	arch
```

若自定义了别的主机名请务必将上面的arch改为自定义的主机名。

### 文件系统、启动与休眠配置

由于使用了Btrfs文件系统，需要配置一些initramfs参数：

```bash
nano /etc/mkinitcpio.conf
```

进入`mkinitcpio.conf`文件后找到`MODULES`那一行，在括号里的最后添加一个`btrfs`，括号里可能会有其他东西，请**不要随意删减**。

```bash
MODULES = ( btrfs )
```

编辑完成后重新生成initramfs：

```bash
mkinitcpio -P
```

initramfs生成后即可生成引导程序，这里使用前面安装的`grub`进行配置：

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux
grub-mkconfig -o /boot/grub/grub.cfg
```

不出意外即可顺利完成配置，至此也已经可以重启拔U盘进入新系统了，但还有一些小细节需要调整。

### 安装完成之前的收尾

还需一些细微配置，首先将两个联网工具设置成开机自启，成功连接网络后重启即可自动联网：

```bash
systemctl enable iwd
systemctl enable dhcpcd
```

还有ssh功能也不要忘了设置开机自启，使用服务器90%的时间都得用ssh服务：

```bash
systemctl enable sshd
```

目前新系统中仅有一个root用户（就是超级管理员），所以需要添加一个普通用户，并将这个普通用户加入wheel组，为了能方便设置sudo命令，这里作为演示用xiaowang，实际可自定义其他用户名：

```bash
useradd -mG wheel -s /bin/zsh xiaowang
```

为刚创建的用户设置密码：

```bash
passwd xiaowang
```

然后使用`visudo`命令为用户设置root权限，进入`visudo`后找到并取消注释`%wheel ALL=(ALL) ALL`，这里的wheel就是刚刚创建用户附带加入的组名。

全部设置完成后千万不要忘了设置root用户的超级管理员密码：

```bash
passwd
```

到这里安装基本已经完成，可以退出、卸载、重启、拔U盘进入新系统了：

```bash
exit
umount -R /mnt
reboot now
```

启动之后输入用户名和密码即可正常使用TTY界面了，如果是作为服务器使用到此已经可以结束。


### 补充

如果前面已经选择了安装图形驱动，下方展示的是平铺窗口管理器Sway的安装命令和相关服务的设置：

```bash
pacman -S sway kitty swaybg
systemctl enable seatd
```

当然，也不要忘记让普通用户xiaowang加入seat组，否则在某些特殊情况下是没有权限启动Sway的：

```bash
usermod -aG seat xiaowang
```

设定完成重启并正常登录到系统后即可输入`sway`命令进入桌面。
