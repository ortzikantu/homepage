+++
title = '在笔记本上折腾Archlinux、Wayland、Nvidia'
date = 2026-04-16T20:50:00+08:00
update = ""
author = "Hao Wu"
description = ""
tags = []
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
toc = false
draft = false
cover = "/assets/photos/2026/IMG_20260402_104914_hu_aaf880e27b1ba1f.webp"
+++

这已经是不知道第几次受够 Windows 了，两个系统刷来刷去也算是不厌其烦了，这次又回到了 Linux。

不过这篇文章不会记录完整的安装过程，只是记录一些安装过程中遇到的问题以及操作方案。

首先，设备是微星的 GP68HX 13VF，已启动独显直连，关闭安全启动，正常进入安装流程。

因为要用到 Wayland 和 Nvidia，就不做 Swap 功能了，Hibernate 后再启动总是卡死崩溃，索性放弃这个功能。

到安装基本系统之前先更正一下镜像仓库源，安装盘自带这个工具，可以直接运行：

```bash
sudo reflector --verbose \
  --country China \
  --protocol https \
  --latest 10
  --fastest 10 \
  --sort rate \
  --save /etc/pacman.d/mirrorlist
```

这段命令的作用是检索中国区域内的协议为 https 的十个最新最快的源并保存到指定目录中，生成的 mirrorlist 也可以直接复制到新系统相同目录中覆盖。

内核选择的始终是 `linux-zen`（包括 `linux-zen-headers`），所以驱动只能选择 `nvidia-open-dkms`，顺带一个 `nvidia-utils` 包就好。

到 `mkinitcpio -P` 的时候又出现了vconsole报错的问题，直接手动创建 `/etc/vconsole.conf` 文件并写入下面内容：

```bash
KEYMAP=us
LOCALE=en_US.UTF-8
```

安装 Nvidia 驱动之后需要做一些配置，打开 `/etc/mkinitcpio.conf`，在 `MODULES` 字段里添加下面内容：

```bash
nvidia nvidia_modeset nvidia_uvm nvidia_drm
```

然后重新执行生成命令：

```bash
mkinitcpio -P
```

然后配置内核启动参数，打开 `/etc/default/grub` 并编辑grub配置文件，在GRUB_CMDLINE_LINUX字段中添加以下参数：

```bash
ibt=off nvidia_drm.modeset=1
```

最后重新生成 grub 配置：
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
