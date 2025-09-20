---
layout: post
title: 通过 cgroup 实现桌面进程挂起、恢复、退出
categories: [GNU/Linux, systemd, cgroup]
---

突发奇想。

## 起因

来自 niri 群聊中提到了 Anyrun，提到了将通过它启动的应用都划至同一 cgroup 下进行管理。
刚去看了一下，也只是个普通的 launcher 而已，cgroup 管理应该是那位群友自己做的功能。
由这个我便想到，我也可以通过 cgroup 来统一管理我通过 launcher 手动启动的应用。

## 过程

直接跳过中间试错好了。

### 1. 在自定义的 cgroup 内启动

`ykrun` :
```bash
#!/bin/bash
systemd-run --user --scope --slice=YukiLauncher.slice --unit="$1-$$".scope /bin/sh -c '"$@"' _ "$@"
```

这个`$@`有点坑，不这样写可能会使得传入参数中有空格时命令解析会出问题。

有了这个脚本之后，例如启动 `foot`，执行：
```bash
ykrun foot
```
就会在 `YukiLauncher.slice` 中创建 `foot-XXXX.scope` 。XXXX 为脚本的 PID。

### wmenu 包装

我习惯用 `wmenu` 直接输入 shell 命令来启动应用。类似的还有 X11 上的 `dmenu`。
虽然 `wmenu` 与 `dmenu` 的功能基本完全一致，但实现上还是有些区别的。

对比的话，`dmenu_run` 是一个这样的脚本：
```bash
#!/bin/bash
dmenu_path | dmenu "$@" | ${SHELL:-"/bin/sh"} &
```

而 `wmenu-run` 则是一个二进制文件。

`dmenu` 和 `wmenu` 功能上倒是没什么区别，从 stdin 读入可用命令列表，然后在界面中获取用户的输入或是选择，
随后把这个输入直接输出到 stdout。对于 `dmenu_run`，就是直接传给 `sh` 来执行。

那么我们要做的包装也就很简单了：
`wmenurun`: 
```bash
#!/bin/bash
EXEC=$(dmenu_path | wmenu -i -N 000000 -S 504375 -f 'Comic Mono 10')
result=$?
if [ $result -eq 0 ]; then
    ykrun "$EXEC"
fi
```

这里 `wmenu` 除 -i 参数外都是用于调节外观的。
这样，我们通过 `wmenurun` 启动的应用就直接通过 `ykrun` 在我们自定义的 cgroup 内启动了。

### 管理

之后就是几个为了便于管理而设置的命令了。
设成 alias 或者脚本都比较无所谓，无非是能不能在 launcher 里调起来的区别。

|命令|内容|说明|
|-|-|-|
| `ykls` | `systemctl --user status YukiLauncher.slice` | 使于查看该 cgroup 状态 |
| `yksusped` | `systemctl --user freeze YukiLauncher.slice` | 禁止该 cgroup 内进程占用 CPU，即冻结|
| `ykcont` | `systemctl --user thaw YukiLauncher.slice` | 解冻 |
| `ykexit` | `systemctl --user stop YukiLauncher.slice` | SIGTERM 所有进程，同时不允许新进程创建 |

本来 suspend  continue exit 都是拿 systemctl kill 命令写的，
但是发现用 kill 会使得终端中启动的应用在恢复时被挂起到后台，会有些怪异的行为。
而 exit 也没有用 kill 则是因为它不管退没退掉都不会阻塞，
加了 `--wait` 则又一直阻塞直至该 slice 被其他的操作 stop 。

本来这个行为也没什么问题，但我想把它和我的桌面注销脚本结合起来，
即注销前先给所有进程 SIGTERM，避免例如：未保存、应用端认为是 crash 等等的问题。
于是这个命令阻塞至何时就很重要了。

之后设了两个 alias 用于对于指定的应用进行暂停/恢复：
```bash
alias susp="systemctl --user kill --signal=SIGSTOP"
alias cont="systemctl --user kill --signal=SIGCONT"
```

这里使用 kill 单纯是因为用 freeze 的话 zsh 没有提供自动补全，
好像只是因为 systemd 的包中没有提供关于 freeze/thaw 的命令补全。[信息更新于 2025-09-20]
没有补全来输应用的 scope 就比较难受了。而用补全则可以直接先敲出启动时用的命令，再自动补全即可。


