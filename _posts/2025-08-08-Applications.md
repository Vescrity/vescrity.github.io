# GNU/Linux 应用推荐

主要是一些个人常用的应用。

## 实用工具

- sdcv: StarDict 命令行版
- tmux: 终端复用器 ~~平铺窗口管理器~~
- cronie: 定时任务
- imagemagick: 用于命令行的图片处理
- perl-rename: 使用正则表达式来批量命名
- convmv: 解决文件名乱码
- bat: 更好看的 cat
- eza: 更好看的 ls
- tealdeer: *TL;DR* 命令用法速查。tldr 包的更好实现。

### 系统工具

- fcitx5: 输入法框架
- htop: 任务管理
- nvtop: 显卡任务状态查看
- pavucontrol-qt: 基本的音频控制
- tlpui: 高级电源管理 tlp 的 GUI。
- filelight: 类似 spacesniffer
- gparted: 分区工具

#### 文件管理

- dolphin: KDE 的文件管理，带终端面板
- pcmanfm-qt: LXQt 的文件管理，附带桌面
- pcmanfm: LXDE 的文件管理，附带桌面

#### 外观配置

- lxappearence: GTK 外观配置
- nwg-look: GTK 外观配置
- kvantum: QT 主题配置
- qt6ct/qt5ct: QT 外观配置

### 下载

- wget
- aria2: 命令行下载器，支持磁链、torrent 等。
- qbittorrent

## 办公

### Office

- libreoffice
- onlyoffice

### 编缉器

- kate: (KDE)
- retext: 支持实时预览的 Markdown 编缉器 (pyqt, qt-Webengine)
- code: (electron)

### PDF 查看

- okular

## 浏览器

- firefox
- falkon: KDE 项目，基于 Qt-Webengine(Chromium)
- zen: 基于 Firefox
- brave: 基于 Chromium，Chrome 替代。


## 图片

- gwenview: KDE 的图片查看
- imagemagick: 用于命令行脚本的图片处理

## 直播工具链

- obs
- obs-vkcapture
- VTuber Studio
- showmethekey

## Wayland

### WM

- niri
- labwc
- sway(swayfx)
- Hyprland
- wayfire: 3D
- dwl: DWM 的 Wayland 实现

> 基于 wlroots 的混成器似乎在 NVIDIA 专有驱动(例如 i+n 卡笔记本电脑的外接屏)上会闪烁  
> 有些混成器并不基于 wlroots 但也实现了大部分的 wlr api，使得很多指定要在 wlroots 上运行的工具也可用，如 niri, Hyprland

### bar/panel/dock

- waybar
- lxqt-panel
- nwg-panel
- nwg-dock
- wf-dock

### 启动器

- wmenu
- nwg-menu
- nwg-drawer
- lxqt-runner

### 壁纸

- mpvpaper
- swww
- pcmanfm-qt
- ...

### 截屏/录屏

- grim+slurp，录屏 wf-recorder+slurp

## X11

- flameshot
- picom
- polybar
- feh
- dmenu

## 影音

- osd-lyrics: 桌面歌词
- kdenlive: 视频剪辑
- QWinFF: ffmpeg+GUI，格式转换利器
- easyeffect: 效果器

### 播放器

- mpv
    - mpv-mpris
- haruna: mpv+GUI
- audacious: 可用于支持 midi 播放
- vlc

### 制谱

- musescore

### pro-audio

详见 [这里](../MusicProduction)

#### DAW

- Ardour
- lmms
- muse
- ……

#### 插件

- carla
- surge-xt
- yabridge: 在 Linux DAW 中使用 WINE 中的 VST
- lsp-plugins
- ……

#### JACK Control

- qjackctl
- qpwgraph

## 游戏

- ppsspp: PSP 模拟
- dolphin-emu: GameCube/Wii 模拟
- steam [专有软件]
- wine

## 玩具

- activate-linux: 显示一个类似 Windows 激活的水印。
- cava: 用于终端的音频可视化
- fastfetch: 展示系统信息


