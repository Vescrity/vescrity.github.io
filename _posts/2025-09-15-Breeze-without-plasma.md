---
layout: post
title: 无 KDE 配置 Qt 主题 (breeze)
categories: [GNU/Linux, Appearence]
---

这配置文件碎了一地啊……

## TLDR

~/.config/kdeglobals 链接至配色文件即可

### 顺使总结下这一堆配置文件都在哪

- 配色方案(color-scheme): 总体的在 `$XDG_DATA_HOME/color-scheme/`
- 风格设为 Breeze/Klassy 等时，也会从 `$XDG_CONFIG_HOME/kdeglobals` 里读取部分配色
- 各应用配置：`$XDG_CONFIG_HOME/${APP_NAME}rc`
    - 很多应用也会在这里保存自己的配色方案
- `$XDG_CONFIG_HOME/kdedefaults/`: 大概是和 Plasma 桌面环境相关的配置项
    - 没见到一般程序读取这里的文件。
- Qt 应用还会去读取 `~/.themes` 下的 GTK 主题


## 背景

长期处于一种大量使用 KDE 应用但是不用 Plasma 桌面的状态。
前几天了解到可以用 matugen 生成配色，把 Qt 风格用 qt6ct 设为 Fusion 后效果也还不错。
不过相比 breeze ，fusion 的风格还是相对稍差了一点。
正好了解到了 Klassy 这个 breeze 的 fork，
装上之后却发现部分配色是遵守我的配置，但是更多的配色则是我以前的 Sweet 配色复活了。
这能忍？

- 新用户下直接使用 Klassy 配置深色 be like (i3, lxqt)：
![](https://github.com/user-attachments/assets/31e515c2-3247-45fe-95e4-9de56f91fd65)

于是我把 .local/share/color-scheme 里的 Sweet 配色删掉，又把 .config 里所有和 Sweet 有关的配置项都处理掉。
可它还是两种配色同时存在。
于是试着拿 strace 追踪了一下一个 qt 程序在打开时访问了哪些配置文件

```bash
strace -e trace=file pcmanfm-qt 2>&1 | grep home | less
```

然后发现了 ~/.config/kdeglobals

打开，然后发现这文件怎么长得那么像我用 matugen 生成的 color-scheme 文件？
于是：

```bash
ln -s ../.local/share/color-scheme/Matugen.colors kdeglobals
```

然后就全都正常了……

最多也就需要我另外在这个配色里再指定几个图标之类的。

## 验证

1. 开新的用户，把我的 matugen 配置复制过去，生成
1. 链接好必要的文件
1. qt6ct 设置 Klassy 以及配色
1. 打开 pcmanfm-qt，符合预期

- 示例：
![](https://github.com/user-attachments/assets/eedeff37-d42d-48f3-9c0f-7520ce3d3387)

