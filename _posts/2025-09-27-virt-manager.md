---
layout: post
title: Virt Manager 初体验
categories: [GNU/Linux, QEMU/KVM, libvirt]
---

反正是能用。

这里只是拿来当 qemu 前端用

## 安装

```bash
pacman -S virt-manager qemu-desktop
```

## 使用

默认尝试连接 LXC，在上方菜单里选择 文件 -> 添加连接 -> QEMU/KVM 用户会话

为了启用 UEFI，在新建虚拟机的最后一步:

```
    勾选在安装前自定义配置，之后点击完成。
    在概况屏幕, 将固件改为：

        UEFI x86_64: /usr/share/edk2/x64/OVMF_CODE.4m.fd：无安全启动支持的 x64 UEFI，
        UEFI x86_64: /usr/share/edk2/x64/OVMF_CODE.secboot.4m.fd：带安全启动支持的 x64 UEFI（无预置证书）。
    点击应用。

    点击开始安装
```

参考：https://wiki.archlinuxcn.org/wiki/Libvirt#UEFI_%E6%94%AF%E6%8C%81

## 挂载 ISO

在虚拟机的设置中添加硬件 > 存储 > 设备类型：CDROM > 管理 
点击对话框下方的本地浏览，选定指定的文件即可。

## 启动

### 从 U 盘启动

倒也不是很难。在虚拟机界面的菜单里 虚拟机 -> 重定向 USB 设备 就可以让虚拟机访问。
但这时候很可能 UEFI 已经尝试从网络启动了，强制重启后按 Esc 进入启动项选择即可。
关机（不是重启）后重定向会失效，所以似乎只能这样操作。

### 从物理磁盘启动

[Wiki](https://wiki.archlinuxcn.org/wiki/Libvirt#%E9%80%9A%E8%BF%87_virt-manager_%E6%B7%BB%E5%8A%A0%E7%A3%81%E7%9B%98)

```
新建虚拟机：选择 "导入现有磁盘"，粘贴磁盘唯一路径
```

用户不在 disk 组时可能会碰到 permission denied，如果不想把用户加入 disk 组就把对应的磁盘给 chown 一下就好了。（不过每次重启都需要重新操作就是了）

启动 Windows 时发现第一次启动会蓝屏，第二回便能正常启动。

### 从 ISO 文件启动

先照前述【挂载 ISO】的方法添加 ISO，随后启动时 Esc 选择启动项即可。

## 分辨率

似乎大多默认 1280x800 的分辨率，Linux 客户机手动在桌面中设置分辨率即可。
Windows 客户机则可能没有预装 virtio 驱动。首先是需要在设置中将显卡设为 virtio，或许还可以把3D加速启用（至少启用了没见到什么毛病）。然后在[这里](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.285-1/)可以获取到 virtio 的安装iso。

## OpenGL

显示协议里将监听类型设定为 无 之后再启用 OpenGL。

