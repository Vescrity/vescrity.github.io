---
layout: post
title: bwrap overlay
categories: [GNU/Linux, bwrap, Note]
---

弄明白了一点点，做个记录
```
bwrap --bind /usr /usr --overlay-src /usr --overlay-src /path/to/another-usr --tmp-overlay /usr ...
```
这样就能在沙盒中把两个文件树合并

但是还不知道怎么弄出来一个比如说普通用户 bind 的 /usr 可以在沙盒内对其内文件做临时的修改。

