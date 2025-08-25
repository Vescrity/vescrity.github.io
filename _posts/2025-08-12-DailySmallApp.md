---
layout: post
title: 日常实用小应用
categories: [Application]
---

总体的汇总做过了，但还打算对一些十分实用的东西做一个具体的介绍，
于是就来了 

像是 tmux 这种比较大众的东西就不放在这里了。

### tldr

man page 太长了，我需要一个简单的帮助页面来提供常用的示例

```bash
pacman -Syu tealdeer    # 一个高效的实现
```

示例输出：

```
$ tldr tar

 归档实用程序。
 通常与压缩方法结合使用，例如 `gzip` 或 `bzip2`.
 更多信息：<https://www.gnu.org/software/tar>.

 创建存档并将其写入文件：

     tar cf 目标文件.tar 路径/到/文件1 路径/到/文件2 ...

 创建一个 gzip 压缩文件并将其写入文件：

     tar czf 目标文件.tar.gz 路径/到/文件1 路径/到/文件2 ...

 使用相对路径从目录创建一个 gzip 压缩文件：

     tar czf 目标文件.tar.gz --directory=路径/到/目录 .

 详细地将（压缩的）存档文件提取到当前目录中：

     tar xvf 源文件.tar[.gz|.bz2|.xz]

 将（压缩的）存档文件解压缩到目标目录中：

     tar xf 源文件.tar[.gz|.bz2|.xz] --directory=目标目录

 创建压缩存档并将其写入文件，使用文件扩展名自动确定压缩程序：

     tar caf 目标文件.tar.xz 路径/到/文件1 路径/到/文件2 ...

 详细列出 tar 文件的内容：

     tar tvf 源文件.tar

 从存档文件中提取与模式匹配的文件：

     tar xf 源文件.tar --wildcards "*.html"

```

### wfrc

我自己写的用来在支持 wlroots 相关协议的 wayland 混成器上进行录屏的脚本

https://github.com/vescrity/wfrc

将它绑在快捷键上，触发一次即可进行选区录屏，再次触发便能中止录屏。
录屏后自动将文件复制至剪切板。
同时默认录制桌面音频。

### you-get

> A tiny downloader that scrapes the web

可以用来下载一些网页上的内容。比如从 AcFun 等视频网站下载视频，
从网易云音乐下载音频等。

##### 安装

可以使用 pipx 来安装

```bash
pipx install you-get
```

### perl-rename

使用正则表达式对文件重命名，
示例见 `tldr perl-rename`

### sdcv

stardict 命令行版，安装一个词典后可用于查词
组合上 `notify-send` 可以在 launcher 中调用查词并在通知中显示结果

```bash
notify-send -i dictionary -a sdnt $1 "$(sdcv $1)" 
```
将它作为一个脚本就OK了

### fd

超快且友好的文件查找工具

```bash
fd "$正则表达式" [目录]
```

### ffmpeg

ffmpeg 功能十分强大，就是命令行参数繁多不够友好

#### 简单用例

##### 格式转换

```bash
ffmpeg -i $InputFile $OutputFile
```
根据后缀自动推断目标格式。亦可从视频转音频

##### 视频响度处理

```bash
ffmpeg -i "$INPUT" -af "loudnorm=I=$TARGET" -c:v copy "$OUTPUT"
```

