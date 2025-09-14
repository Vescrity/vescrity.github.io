---
layout: post
title: 把电脑作为“蓝牙耳机”
categories: [GNU/Linux, bluetooth, Audio]
---

还挺实用的。

## 背景

打算试试让电脑播放手机音频

## 步骤

1. 安装 bluez，启动 bluetooth.service
2. 找一个管理蓝牙的前端，我选择了用 blueman
3. 连接手机
4. 诶，它自动就能从电脑播放了！

### 可能需要的前置条件

反正我系统上有这些玩意

- pipewire
- pipewire-pulse
- wireplumber

## MPRIS?

连好了，但是美中不足的是我们还没有 mpris 控制……
打开 ArchWiki ，搜索 mpris，发现有蓝牙这一栏。于是解决方案：
运行 `mpris-proxy` 即可。也可以做一个 systemd 服务。
详见 https://wiki.archlinuxcn.org/zh-cn/MPRIS#%E8%93%9D%E7%89%99

## 效果？

基本没啥问题，可能音量需要调得稍大一些。
再就是偶尔会有一小下卡顿，一般都是手机侧的问题。

