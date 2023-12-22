+++
title = "archlinux安装教程"
date = 2023-06-23
+++

# 开始安装

**连接互联网**

```
iwctl
```

首先，如果不知道你的网络设备名称，请列出所有 WiFi 设备：

```
[iwd]# device list
```

如果设备或其相应的适配器已关闭，请将其打开。

```
[iwd]# adapter adapter set-property Powered on
```

然后，要开始扫描网络（注意：这个命令不会输出任何内容），执行：

```
[iwd]# station device scan
```

再然后，就可以列出所有可用的网络：

```
[iwd]# station device get-networks
```

最后，要连接到一个网络：

```
[iwd]# station [名称] connect [SSID]
```

**更新系统时间**

```
timedatectl
```

**创建硬盘分区**

```
fdisk -l(此处为小写字母l)
```

我们使用`cfdisk`

分区类型:EFI系统分区
大小:512M

分区类型:Linux swap
大小:根据内存大小

分区类型:根分区
大小:剩余空间


**格式化分区**

根分区：

```
mkfs.ext4 /dev/root_partition
```

交换空间分区：
```
mkswap /dev/swap_partition
```

EFI系统分区：

```
mkfs.fat -F 32 /dev/efi_system_partition
```

**挂载分区**

根分区：

```
mount /dev/root_partition（根分区） /mnt
```

UEFI系统分区：

```
mkdir -p /mnt/boot
```

```
mount --mkdir /dev/（EFI 系统分区） /mnt/boot/efi
```

交换空间：

```
swapon /dev/swap_partition（交换空间分区）
```

**安装**

```
首先，修改镜像源
```

```
vim /etc/pacman.d/mirrorlist
```

删除所有镜像，填写163镜像源。

```
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
```

可以根据自己所在地区的运营商进行ping比对，以上仅供参考。

安装系统
```
pacstrap /mnt base linux linux-firmware base-devel neovim intel-ucode nano
```
**配置**

 分区表

```
genfstab -U /mnt >> /mnt/etc/fstab

```

```
chroot
```

进入新系统：

```
arch-chroot /mnt
```

时区

设置时区：

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

将系统时间写入硬件时钟：

```
hwclock --sys
```

安装引导程序

```
pacman -S dosfstools grub efibootmgr
```

再次执行

```
mkdir -p /mnt/boot
```

```
mount --mkdir /dev/（EFI 系统分区） /mnt/
boot/efi
```

```
grub-install --target=x86_64-efi --efi-directory=/mnt/boot --bootloader-id=GRUB
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

**检查grub设置:**

检查/etc/default/grub中的GRUB_TIMEOUT 是否设置成了0，设置为一个正数来调整GRUB条目加载前的等待时间，按秒计时。另外要检查GRUB_TIMEOUT_STYLE是否为hidden，设置为menu确保菜单显示。重新生成主配置文件后，重启检查设置是否生效。

重启

```
umount -R /mnt
```

```
reboot
```

