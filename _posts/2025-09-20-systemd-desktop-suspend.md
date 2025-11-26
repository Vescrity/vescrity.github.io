---
layout: post
title: 通过 cgroup 实现桌面进程挂起、恢复、退出
categories: [GNU/Linux, systemd, cgroup]
---

突发奇想。

## 效果概览

#### 所有在桌面启动的终端、通过 `wmenu` 、终端启动的程序都可以实现挂起、恢复。

例如，在 `wmenu` 中输入 `firefox` 可以启动 `firefox`。
可以把由这个 `firefox` fork 产生的所有进程全部冻结。
在终端输入 `susp firefox` 然后按 `Tab` 补全，回车即可。
恢复则是 `cont firefox` 然后补全。

亦可直接挂起所有这类程序，执行 `yksuspend` 即可。
将恢复的命令 `ykcont` 绑定至 `niri` 快捷键中，使用快捷键即可恢复。

#### 优雅地退出会话

niri 默认退出时直接结束自身进程，不管应用程序。
这里支持了在退出时首先等待所有前述的那一类程序完成退出后再结束 wayland 会话。

2025-11-18 编辑：结合一些脚本我干脆弄了个『开关机』（登入/注销）音效。

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

这个`$@`有点坑，不这样写可能会使得在命令行中传入参数中有空格时命令解析会出问题。

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
    ykrun $EXEC             # 这里加引号反而会出问题
fi
```

这里 `wmenu` 除 -i 参数外都是用于调节外观的。
这样，我们通过 `wmenurun` 启动的应用就直接通过 `ykrun` 在我们自定义的 cgroup 内启动了。
再在 niri 中给启动终端的命令前加上 `ykrun` 就可以了。

### 管理

之后就是几个为了便于管理而设置的命令了。
设成 alias 或者脚本都比较无所谓，无非是能不能在 launcher 里调起来的区别。

|命令|内容|说明|
|-|-|-|
| `ykls` | `systemctl --user status YukiLauncher.slice` | 使于查看该 cgroup 状态 |
| `yksuspend` | `systemctl --user freeze YukiLauncher.slice` | 禁止该 cgroup 内进程占用 CPU，即冻结|
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

~~这里使用 kill 单纯是因为用 freeze 的话 zsh 没有提供自动补全，~~
~~好像只是因为 systemd 的包中没有提供关于 freeze/thaw 的命令补全。~~[2025-09-20]
事已至此，交个 [PR](https://github.com/systemd/systemd/pull/39052) 好了。

---

[2025-09-22] 该 PR 已合并至上游。于是可以改为：
```bash
alias susp="systemctl --user freeze"
alias cont="systemctl --user thaw"
```

不过冻往的界面最好不要动它，不然恢复时会一下传入过多的事件导致崩溃。
也许我们还是用 kill 的方案更好？但是 kill 的方案的缺点在于对终端中的部分进程有一些影响。

---

没有补全来输应用的 scope 就比较难受了。
而用补全则可以直接先敲出启动时用的命令，再自动补全即可。


