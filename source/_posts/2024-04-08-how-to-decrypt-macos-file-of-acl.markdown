---
layout: post
title:  "解决从macOS时间机器导出文件后的权限问题"
date:   2024- 04-08 20:40:00
tags: 
  - MacOS
  - time machine
  - acl
---

 由于我的 mbp2020 在升级到 sonoma 之后会有卡顿的现象，就决定给我的机器降级处理，在降级之前我对我的机器进行了备份，备份的方式是使用时间机器进行备份，备份完成之后我将备份的文件导出到了我的移动硬盘上，然后将我的机器进行了降级处理，降级完成之后我将我的文件导入到我的机器上，但是导入之后发现我的文件权限有问题，导致我无法对我的文件进行操作，这里记录一下解决的方法。
 ```BASH
sudo xattr -r -d com.apple.timemachine.private.directorycompletiondate TARGET_DIR_PATH
sudo chmod -RN TARGET_DIR_PATH
sudo find TARGET_DIR_PATH -type d -exec chmod 755 {} \;
sudo find TARGET_DIR_PATH -type f -exec chmod 644 {} \;
 ```

把拷贝过来的文件夹使用以上命令操作之后可以解决文件夹没有权限的问题。