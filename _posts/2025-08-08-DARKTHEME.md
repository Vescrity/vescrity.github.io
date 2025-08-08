---
layout: post
title: 主题配置与踩坑
---

本文用于记录在非传统桌面环境(除 KDE Gnome 之外)上配置(深色)主题。

从[这里](https://www.pling.com/)可以找到大量的社区主题。

常见的可配置主题的桌面应用主要基于 QT 和 GTK。


## QT

推荐使用 qt6ct qt5ct kvantum 来配置主题。

Kvantum 用于加载并配置主题，qt6ct 则用于应用主题。

### Kvantum

安装 `kvantum` `kvantum-qt5` 包，运行 `kvantummanager` 启动 kvantum GUI。

下载下来的主题用 kvantum 安装即可。  

推荐主题：[Sweet](https://www.pling.com/p/1294013/)

### qt6ct/qt5ct

设置 `QT_QPA_PLATFORMTHEME=qt6ct`。  

启动 qt6ct 后设置风格为 kvantum-dark/kvantum 即可。字体等其余设置亦可在此配置。  

### **KDE App 部分组件仍为亮色**

之前配置被重置后折腾了半天才找到这个问题。

例如 Okular, kate 等应用可能会见到。似乎因为部分 KDE 应用使用了些 QML。

总之一般这类应用都有自己设置配色方案的配置。将其设为一个黑色的配色方案即可。

## GTK

~~推荐 `lxappearence` 配置~~ `nwg-look`。  

推荐主题：[Sweet-Dark-v40](https://www.pling.com/p/1253385/)

GTK3/4 可能需要用 `gsettings` 命令来配置一些主题，图标。  

```bash
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'   
gsettings set org.gnome.desktop.interface icon-theme 'kora'   
```
