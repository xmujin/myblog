---
title: "Arch linux安装教程"
date: '2026-03-29T11:52:25+08:00'
# weight: 1
# aliases: ["/first"]
tags: ["linux"]
# categories: ["日常"]
author: "向洵"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "添加描述文本"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
## Arch Linux官方镜像下载
### 国内镜像推荐
[中国境内的镜像站](https://archlinux.org/download/#:~:text=China)

## 使用Ventoy制作启动盘
[Ventoy文档](https://www.ventoy.net/cn/doc_start.html)


## 连接到互联网
确保启用了网络接口
```cpp
ip link
```
### 连接wifi
连接wifi使用iwd
> pacman -S iwd

进入交互式提示符
> iwctl

列出wifi设备
> [iwd]# device list

扫描wifi
> station wlan0 scan 

连接wifi
> [iwd]# station wlan0 connect "wifi名" 

## 更新系统时间
> timedatectl

## 创建硬盘分区
列出分区
> fdisk -l

使用分区工具进行分区
> cfdisk /dev/要被分区的磁盘

分区方案
|挂载点|分区|分区类型|注释|建议大小|
|-|-|-|-|-|
|/boot|/dev/efi系统分区|EFI system|EFI系统分区|至少1GB|
|[SWAP]|/dev/swap系统分区|Linux swap|交换空间|至少4GB|
|/|/dev/root分区|Linux root(x86-64)|根目录|剩余空间（大于50GB够用）|

## 格式化分区
```cpp
# mkfs.ext4 /dev/root_partition（根分区）
# mkfs.fat -F 32 /dev/efi_system_partition（EFI 系统分区）
# mkswap /dev/swap_partition（交换空间分区）
```
## 挂载分区
```cpp
# mount /dev/root_partition（根分区） /mnt
# mount --mkdir /dev/efi_system_partition /mnt/boot    (挂载EFI系统分区)
# swapon /dev/swap_partition（启用交换空间分区）
```

## 安装系统
### 选择镜像
需安装的软件包会从文件`/etc/pacman.d/mirrorlist`中所列的镜像站下载。下载软件包时，列表中越靠前的镜像站会拥有越高的优先级。 

### linux内核和常见硬件的基本安装
> # pacstrap -K /mnt base linux linux-firmware

## 配置系统
### 生成fstab文件
生成 fstab 文件以使需要的文件系统（如启动目录 /boot）在启动时被自动挂载，用 -U 或 -L 选项分别设置 UUID 或卷标： 
> # genfstab -U /mnt > /mnt/etc/fstab

## 切换到新系统

> arch-chroot /mnt

### 设置时区

> ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

### 同步时间
这里和windows不一致，它使用的是UTC时间
> hwclock --systohc

### 区域和本地化设置
程序和库如果需要本地化文本，都依赖区域设置，后者明确规定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

需要设置这两个文件：locale.gen 与 locale.conf。

编辑 /etc/locale.gen，然后取消掉 en_US.UTF-8 UTF-8 和其他需要的 UTF-8 区域设置前的注释（#）。

接着执行 locale-gen 以生成 locale 信息： 

> locale-gen

然后创建 locale.conf(5) 文件，并编辑设定 LANG 变量，比如： 

```cpp
/etc/locale.conf
------------------------
LANG=en_US.UTF-8
```

### 网络配置
创建hostname文件
```cpp
/etc/hostname
------------------------
主机名
```

### 安装网络管理软件
> pacman -S networkmanager

### 设置root密码
> passwd

### 安装引导程序
这里使用了grub作为安装

    pacman -S grub efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

**生成主配置文件（非常重要）**
> grub-mkconfig -o /boot/grub/grub.cfg

### 重新启动电脑
直接重启


## 后续的工作
### 安装KDE桌面
安装plasma桌面:plasma-meta或者plasma
> pacman -S plasma-meta
### 安装kde的应用
> pacman -S konsole dolphin # 命令行和文件管理 必装
### 安装登陆管理器
> pacman -S plasma-login-manage

### 安装显卡驱动
针对gtx1650的显卡驱动
> yay -S nvidia-580xx-dkms nvidia-580xx-settins nvidia-580xx-utils

### 进入系统
应该可以直接进系统了

### 安装字体
> pacman -S wqy-microhei # 文泉驿微米黑 

### 安装输入法
> pacman -S fcitx5-im （直接安装包组）
> pacman -S fcitx5-chinese-addons 中文插件







