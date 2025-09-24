---
layout: post
title: 把 Manjaro 调包为 Arch Linux
categories: [GNU/Linux, Arch Linux]
---

我当初为什么要推荐别人用 Manjaro 啊（）

[写于：2025-09-23]

总之是第二回这么干了。
上一回是自己当年装的第一个 GNU/Linux 系统，迁移时懂得也不多，
于是迁移之后似乎系统有那么一点哪里不对劲，
也可能单纯当时整出来的心理作用。

这里只做大致思路及操作记录。

## 思路

1. 移除无关紧要的 Manjaro 包，顺带清理不需要的
2. 替换镜像源
3. 移除较核心的 Manjaro 包，在出现依赖问题时用 Arch 源的软件替换 (pacman 包)
4. 使用 Arch 源软件替换所有包
5. 追踪所有未被包管理的文件，进行适当的合并、移除

> 全程没有破坏依赖关系

## 具体过程

这回是远程操作，还没搞备份，大概是胆子太大了（）

### 清理

找出 manjaro 特色包:
```bash
pacman -Qs manjaro
```

直接用 `-Rs` 卸载就好了，大部分都是元包。
卸载时可能会连带某些包，自己看需不需要，
一般不涉及特定的硬件都用不到。

有个 `manjaro-system` 的包可能会提示你是保留的，反正我留到后面才删，
不过看功能提早删了应该也不会有问题。

另外还有 `manjaro-keyring` `manjaro-release` 等包我也保留了。
再就是内核包以及相关的驱动包，以防意外关机后无法启动。

清理不用的包大概就用 `pacman -Qet` 列出来，自己看看要不要用，
用 `-Rs` 删掉或者至少先用 `-D --asdeps` 标记成依赖。
反正就是正常的清理流程。

这几步在替换镜像后做应该也没什么问题。

### 替换镜像源并更新

清理差不多了就可以替换镜像源了。
替换前最好把 manjaro 自带的定时更新镜像源的 systemd 服务停掉

```bash
systemctl stop pamac-mirrorlist.timer
systemctl disable pamac-mirrorlist.timer
systemctl disable pamac-cleancache.timer
```

反正停掉就行，前两天看到它自动更新镜像失败干脆留了个空的镜像列表真没蚌住。

然后随便找几个能用的 Arch 镜像源加到 `/etc/pacman.d/mirrorlist`

```
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.jlu.edu.cn/archlinux/$repo/os/$arch
```

当然原来的都要删掉。
更新数据库：
```bash
pacman -Sy
```

然后 `-Qm` 列出来所有的Arch 官方源中不存在的包，按需清理。
在这边因为数据库更新了，余下的那几个 `manjaro-*` 包自然是不可能有用了，
我选择在这时候移除它们。

`pacman-mirrors` 是 manjaro 的一个镜像更新工具，
这时候移除会提示 `pacman` 依赖它。自然这个依赖关系肯定是 manjaro 增加的。
那么只要把 `pacman` 替换为 Arch 源的就好了：
```bash
pacman -S pacman
```
同理，可能会有个别包扯到了比如 `base` 包，同样操作就好。

替换后就可以把那几个包移除了。
可能还会有几个包之前因为种种原因没有处理掉，移除就好。
当然内核包我还继续保留。

清理干净后就可以替换所有包了：
```bash
pacman -Scc         # 清理缓存
pacman -S $(pacman -Qqn)
```

这个操作可以保留安装原因。
然后选择一个 Arch 源的内核安装，以及对应的相关驱动包。

然后手动更新 grub (`grub-mkconfig`)或是你自己的 bootloader。

### 处理未被跟踪的文件

主要要处理的是一些 Manjaro 生成的配置文件，
包升级时产生的 `*.pacnew` 文件等。

用 `lostfiles` 来查看系统中哪些文件未被包管理追踪：
```bash
pacman -S lostfiles
```

需要以 `root` 权限执行 `lostfiles`。

随后**按需**操作，举例如下：
- 对于 `*.pacnew` 文件，使用 `vimdiff` 工具来合并(将新的配置文件内容合并至旧的)
- 对于 `*.pacsave` 文件，如果你不认识这个东西的话移掉也没什么问题。
- 对于 `/usr` 下的未跟踪文件：
    - 二进制文件以及各种 cache 一般可能是运行时或安装时产生的，不用管就好
    - 还有的可能是非包管理安装的文件，如 `luarocks` 安装的，也不用管
    - 其余的看具体情况移除
- 对于 `/etc` 下的：
    - 某些你认得的配置文件自然是不要动的
    -  `/etc/X11/xorg.conf.d/` 里大概会有 mhwd 自动生成的东西，删掉它，自己创建的配置看情况保留。
    - 其余的按需移除，主要应该就是 mhwd 扔的几个文件。
- 对于 `/var` 下的：应该都不需要操作。`lostfiles` 好像也不会列出这里的。

弄差不多了应该就 OK 了。



