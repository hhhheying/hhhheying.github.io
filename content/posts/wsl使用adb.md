---
title: "WSL 使用 adb"
date: 2026-08-25T00:00:00+08:00
lastmod: 2026-08-25T00:00:00+08:00
description: "在 WSL 中使用 adb 连接 Android 设备的版本问题与 usbipd 方案"
tags: ["WSL", "adb", "Android"]
categories: ["技术"]
---

# wsl使用adb
## 版本问题
可能是版本问题，将wsl中的adb的版本和windows的adb版本统一

## 使用usbipd
> [连接 USB 设备 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb)
> [usbip windows 使用，挂载远程usb设备 | ZhouZhou Blog](https://zwc365.com/2024/03/06/usbip%E4%B8%8Evirtualhere%E4%BD%BF%E7%94%A8%E6%8C%82%E8%BD%BD%E8%BF%9C%E7%A8%8Busb%E8%AE%BE%E5%A4%87)

- 根据文档下载usbipd
- 注意windows版本和wsl版本
- 管理员模式运行powershell
```powershell
usbipd list
usbipd bind --busid 1-4
# 可能需要重启
# 打开一个wsl窗口
usbipd attach --wsl --busid 1-4
```
- 在wsl
```bash
lsusb
```
- 需要断开，在win
```powershell
usbipd detach --busid <busid>
```

### 问题
- 正常情况下，网络断开的 `usbip` 设备能够自动恢复，例如关闭wifi、关机等操作。然而如果直接拔掉 windows 电源。设备会一直处于一个挂载状态，无法再被开机自动挂载。
- 此时必须执行 `usbip unbind --busid=xxx` 取消绑定，然后再次执行 `usbip bind --busid=xxx` 才行
```powershell
usbipd unbind --busid 1-4
usbipd bind --busid 1-4
```





<!--stackedit_data:
eyJoaXN0b3J5IjpbNTM2NjIyODA1LDIwNjA0NjI4NjIsLTIwMj
Q2NzYzMDIsLTIwNjc4ODEzMTJdfQ==
-->