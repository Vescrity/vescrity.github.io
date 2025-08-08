# Wine 与可用性调整

## 桥接

### Wine 插件桥接至 GNU/Linux DAW

- yabridge

### GNU/Linux 插件桥接至 Wine DAW

似乎有 `carl-vst-wine` 这个项目，介绍为：
```
Carla VST for windows applications. This package provides the Carla wine VST plugin. This
allows to load Carla inside a Windows host (running under wine) but loading the
linux-native version of Carla, thus enabling the use of linux-native plugins inside a
Windows host.
```

不过装了之后虽然识别到了插件，但是并没有成功让 FL 加载出来 carla。

## 调整

有些情况下插件的 UI 在 DAW 中会出现鼠标位置与实际控制的位置的偏移。有时是只有将插件UI放至左上角才可用，亦有整体在某一方向偏移了若干像素的情况。  
目前在 wine 10.0~10.2 等版本中试验，较好的方案是在 winecfg 中开启虚拟桌面。

## 问题/注意事项

好像没成功让 32 位的插件正常运行起来过。
