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

## 从 U 盘启动

倒也不是很难。在虚拟机界面的菜单里 虚拟机 -> 重定向 USB 设备 就可以让虚拟机访问。
但这时候很可能 UEFI 已经尝试从网络启动了，强制重启后按 Esc 进入启动项选择即可。

## 问题（？）

客户机里显示特效很有问题，应该需要调节显示选项，但是还没有弄明白（）

