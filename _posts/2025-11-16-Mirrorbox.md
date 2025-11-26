---
layout: post
title: 用 systemd-nspawn 启动一个系统副本容器
categories: [GNU/Linux, Systemd, Systemd-nspawn]
---

Linux 有点太灵活了。

## 目标

在当前系统下启动一个软件包和配置与当前系统一致的容器。
同时容器内写入不影响主机文件，并且可不写入磁盘。

## 方案一：手动 overlay

相对更灵活一点，也能明确知道容器目录在哪里。
可以在启动前自己手动清理不需要的服务与文件等。

```bash
#!/bin/bash
MIRROR_PARENT=/run/user/$UID/
MIRROR_NAME=mirrorbox
MIRROR_PATH="$MIRROR_PARENT"/"$MIRROR_NAME"
cd "$MIRROR_PARENT"
if [ -d "$MIRROR_NAME" ]; then
    echo "mirrorbox 存在。"
else
    echo "mirrorbox 不存在。创建之。"
    mkdir -p "$MIRROR_NAME"
    sudo mount -t tmpfs tmpfs -o size=12G  "$MIRROR_NAME"
fi
cd "$MIRROR_PATH"
chmod 755 .

sudo mkdir {etc,usr,var,opt}{,wk}

sudo mount -t overlay overlay -o upperdir=usr,lowerdir=/usr,workdir=usrwk usr
sudo mount -t overlay overlay -o upperdir=etc,lowerdir=/etc,workdir=etcwk etc
sudo mount -t overlay overlay -o upperdir=var,lowerdir=/var,workdir=varwk var
sudo mount -t overlay overlay -o upperdir=opt,lowerdir=/opt,workdir=optwk opt
sudo mkdir home{,wk}
sudo mount -t overlay overlay -o upperdir=home,lowerdir=/home,workdir=homewk home

# 清理可能敏感的文件
rm -rf  "$PWD/home/$USER/.ssh"
# 清理不需要的服务
sudo rm -f $PWD/etc/systemd/system/multi-user.target.wants/sshd.*
sudo rm -f $PWD/etc/systemd/system/multi-user.target.wants/Network*
sudo rm -f $PWD/etc/systemd/system/multi-user.target.wants/tlp*

# 取消注释可以让一些工具开心
#sudo chown root:root .
sudo systemd-nspawn -b -D . --bind=/dev/dri/:/dev/dri/ --property=DeviceAllow='char-drm rw' \
    --bind=/run/user/$UID/$WAYLAND_DISPLAY:/run/host/wayland \
    --bind-ro=/run/user/$UID/pulse:/run/host/pulse \
    $@

sudo umount usr etc var opt
sudo umount home
cd ..
read -p "要清理掉 mirrorbox 的所有文件吗 (y/N) " answer
if [ "$answer" = "y" ] || [ "$answer" = "Y" ]; then
    echo "将移除 mirrorbox 的 tmpfs 并移除挂载点"
    sudo umount "$MIRROR_PATH"
    sudo rmdir "$MIRROR_PATH"
    echo "清理完成"
else
    echo "不进行清理。"
fi
```

## 方案二：完全交给 systemd-nspawn

首先准备一个目录用于存放容器内写入。
若不打算保留这些写入则找一个 tmpfs 的目录。

例如：在当前目录下的 tmppath 文件夹处挂载一个 12G 的 tmpfs
```bash
sudo mount -t tmpfs tmpfs -o size=12G tmppath
```

随后在 tmppath 内建一个新的文件夹作为 upperdir
```bash
mkdir tmppath/u1
```

启动容器：
```bash
systemd-nspawn -bD / --volatile=yes --overlay=/etc/:tmppath/u1:/etc
```

这里也可以像方案一中那样 bind 一些设备与某些 socket 用于共享图形与音频等。
对于 `/var /opt /home` 也用类似的方案 `--overlay` 一下就可以了。

实话讲全都用参数指定感觉也没比方案一容易多少。

关于为什么要在 tmppath 下再新建文件夹，
是因为 `systemd-nspawn` 会自动把 overlayfs 中需要使用的 workdir 直接指定为提供的 upperdir 的上一级目录中的一个随机命名的新目录。
而在这里，tmppath 的上级目录显然与 tmppath 内不在同一文件系统，这并不符合 overlayfs 的要求，于是会报错。

而关于为什么要指定 overlay，则是因为不指定时只有 `/usr` 被自动 overlay，启动时会执行 `systemd-firstboot`，最后得到的是一个 `/etc` 干净到连 `pacman` 都用不了的系统。而 `/var` 也需要 overlay 进来，因为 `pacman` 数据库在这里。

要是用 `--bind` 等参数，那写入就会被写入到宿主系统了，这显然不是我们想要的了。

## 另见

[利用 Aufs 和 LXC 快速建立一个用于测试的系统副本](https://blog.lilydjwg.me/2014/2/19/duplicate-your-system-with-aufs-and-lxc.42928.html)
[lxc-arch2](https://gist.github.com/lilydjwg/7f949263a98892153cc5)
