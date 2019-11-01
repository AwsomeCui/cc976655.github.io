---
layout: post
title:  "macOS调整LaunchPad图标大小"
date:   2018-11-20 12:30:30
tags: Tools
---
> macOS的Launchpad默认图标很大，看起来很不美观，下面是调整图标大小的方法。

## 打开终端输入一下命令
```bash
# 调整每列显示图标个数
defaults write com.apple.dock springboard-rows -int 7

# 调整每行显示图表个数
defaults write com.apple.dock springboard-columns -int 8

# 重启
defaults write com.apple.dock ResetLaunchPad -bool TRUE
killall Dock
```

## 调整之后的效果图
![2018-11-20-lunchpad](/assets/img/2018-11-20-lunchpad.jpeg)

