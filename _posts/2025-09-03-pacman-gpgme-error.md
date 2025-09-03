---
layout: post
title: pacman 错误：GPGME 错误：无数据
categories: [Arch, pacman]
---

我怎么知道镜像站怎么一下子全炸了。

一觉醒来梯是通的，但是啥都上不了，pacman -Syu 也同步不了，
打开浏览器发现镜像站干脆都似了？。？

Anyway 先把 pacman 这个问题修了。

[Arch Wiki](https://wiki.archlinux.org/title/Pacman#error:_GPGME_error:_No_data)

```bash
sudo rm -r /var/lib/pacman/sync
```

找个还能用的镜像站就ok了
