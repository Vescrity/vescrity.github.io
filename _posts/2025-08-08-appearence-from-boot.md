---
layout: post
title: 从启动开始的外观配置
categories: [Appearence]
---

> 只是一些记录

- **你能在这里看到什么**:
    - 有趣的 grub+plymouth Minecraft 主题
    - Display-Manager

## Grub

[链接](https://github.com/Lxtharia/double-minegrub-menu)

一个很“仿真”的 Minecraft 主题。照着 README 配一下就 OK。

- 第一个界面有两个选项但只有一个有用，却默认在这有着不算小的等待时长。因而我安装后更改了 `/boot/grub/mainmenu.cfg` 的 `set timeout=` 行。
- 更改后重新生成配置即可。

```bash
sudo grub-mkconfig --output=/boot/grub/grub.cfg
```

## Plymouth

也是“仿真”的 Minecraft 主题。这回我用的 AUR 包 `minecraft-plymouth-theme-git`

[链接](https://github.com/nikp123/minecraft-plymouth-theme)

Arch 上启用 plymouth 还得稍稍做些设置，照着 Archwiki 做就够了。大概是要加 modprobe，并调整一下内核参数即可。

原文写只支持 dracut，但看来 modprobe 也是可用的。

启用好 plymouth 后再设置主题即可。

## Display-Manager

用过 SDDM ，LightDM，~~现在在用 greetd + nwg-hello。~~
- [2024年末] 使用包装过的 `tuigreet` -> [tmux-greet](https://github.com/Vescrity/tmux-greet)

- SDDM 在用 KDE 时倒也很好用——不过我不在用。
- [LightDM 有几个不同的前端](https://wiki.archlinux.org/title/LightDM#Greeter)。
    - 默认 `LightDM GTK Greeter` 能用，但算不上多美观。
    - EndeavourOS 和 Linux Mint 默认使用的是 `slick-greeter`，美观上确实不错但我在 Arch 上用的时候碰到了密码框不展示的问题。
    

### greetd

- greetd 我个人认为是个比较符合 UNIX 哲学的组件——只做 login manager daemon，你可以用几乎任何东西做它的前端。`... does not make assumptions about what the user wants to launch, should it be console-based or graphical.` 即你可启动一个登陆屏幕，也可以是文本登陆界面，甚至直接自动登入你的账号。

> **踩坑**
>
> https://wiki.archlinux.org/title/Greetd#Run_local_programs
> 我的解决方法是将 `~/.zprofile` 软链接了一份到 `~/.profile`

#### tmux-greet

轻量、快速、可配置快捷键(例如调节亮度)、可执行命令(以 greeter 帐户)

#### QtGreet

~~看了几个不同的实现最后对它心动了~~*Update: 感觉不如 `nwg-hello`*。[项目主页&截图](https://gitlab.com/marcusbritanicus/QtGreet)
缺点是配置文档似乎是几乎没有。只好照着默认配置乱改。

需要配合一个 Wayland 混成器使用。项目自带的 `greetwl` 似乎不可用，考虑轻量性我便换了 `dwl` 。

`qtgreet` 需要混成器自动将其启动，因而需对 `dwl` 打几个 patch 并做些配置。我共打了这些：
- autostart: 用于自启动 qtgreet
- foreign-toplevel-management: qtgreeet 需要混成器支持它。不支持也可用但启动会有近 10 秒的延迟。
- naturalscrolltrackpad: 自然滚动
- numlock-capslock: 默认开启 Numlock 。

以及一些常规的或不常规的快捷键：
- 窗口关闭
- 启动 qtgreet: 可允许在一段时间内没有登陆需求时关闭 qtgreet ，有需求再打开。
- 亮度调节：可用于关屏等。我依赖了 `light(-git)` 包来实现。
- 甚至启动终端：默认下 greetd 用用户 greeter 启动这些任务。只是用户主目录是 `/`。直接把主目录改到 `/home/greeter`，就和普通用户几乎无异了。做一个临时命令行登陆或是直接当一个新用户来用都可行。

#### nwg-hello

大体还是如上，不过它在官方仓库中，且有较明确的配置文档。将我的 dwl 改成自动启动 nwg-hello 就可以了。


