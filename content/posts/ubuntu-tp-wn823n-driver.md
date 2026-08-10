---
title: "Ubuntu 安装 TP-WN823N 免驱无线网卡驱动"
date: 2026-08-10T00:00:00+08:00
lastmod: 2026-08-10T00:00:00+08:00
description: "TP-LINK TP-WN823N v2 免驱版在 Ubuntu 20.04 上编译安装 rtl8192fu 驱动的过程记录。"
tags: ["Ubuntu", "驱动", "无线网卡"]
categories: ["Linux"]
---

# Ubuntu 安装 TP-WN823N 免驱无线网卡驱动
> 参考
>  [GitHub - BrightX/rtl8192fu: Realtek 8192FU Linux USB无线网卡驱动](https://github.com/BrightX/rtl8192fu)
> [ubuntu20.04 安装usb无线网卡 腾达u6 （rtl8192fu）过程记录-CSDN博客](https://blog.csdn.net/g81249707/article/details/131999633)
> [usb驱动也加载了，怎么还是没有呢－论坛－深度科技 (deepin.org)](https://bbs.deepin.org/post/232498)


- 免驱不是linux免驱
- 京东、淘宝客服都说linux不能用
- Ubuntu版本20.04
- 内核版本5.15
- 无线网卡版本tp-wn823n v2 免驱
- 官网的版本不能成功编译

## 具体步骤
- 看接口信息，确定设备ID 0bda：a192
```bash
lsusb
```
---
这里我直接搜了设备ID就有很多教程了，大部分说的是使用rtl8192fu对应的驱动（我在这一步识别到的是disk，就切换了usbmode，所以发现设备ID变了……或许先编译驱动再改就不会像我一样没有驱动）
- 切换usbmode
```
sudo usb_modeswitch -KW -v 0bda -p a192 
```
这个时候设备ID已经变成 2357:0135了……

---

- 下载[https://github.com/BrightX/rtl8192fu](https://github.com/BrightX/rtl8192fu)
- 根据链接中的readme操作，其中主要操作：
```bash
make -j$(nproc)

sudo make install

sudo modprobe 8192fu
```
- 看接口信息
```bash
lsusb

usb-devices
```
- 驱动安装成功`Driver=rtl8192fu`
```bash
T:  Bus=03 Lev=01 Prnt=01 Port=01 Cnt=02 Dev#=  5 Spd=480 MxCh= 0
D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
P:  Vendor=0bda ProdID=f192 Rev=02.00
S:  Manufacturer=Realtek
S:  Product=802.11n  WLAN Adapter
S:  SerialNumber=60EE5CBDFDE9
C:  #Ifs= 1 Cfg#= 1 Atr=80 MxPwr=500mA
I:  If#= 0 Alt= 0 #EPs= 8 Cls=ff(vend.) Sub=ff Prot=ff Driver=rtl8192fu
```
- 驱动安装失败`Driver=(none)`
```bash
T:  Bus=01 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  3 Spd=480 MxCh= 0
D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
P:  Vendor=0bda ProdID=f192 Rev=02.00
S:  Manufacturer=Realtek
S:  Product=802.11n  WLAN Adapter
S:  SerialNumber=60EE5CBDFDE9
C:  #Ifs= 1 Cfg#= 1 Atr=80 MxPwr=500mA
I:  If#=0x0 Alt= 0 #EPs= 8 Cls=ff(vend.) Sub=ff Prot=ff Driver=(none)
```
---
- 如果失败，就卸载驱动
```bash
sudo modprobe -r 8192fu
cd rtl8192fu/
sudo make uninstall
```
- 修改`os_dep\linux\usb_intf.c`，大约在250行位置
```c
#ifdef CONFIG_RTL8192F

/ _=== Realtek demoboard ===_ /  
{USB_DEVICE_AND_INTERFACE_INFO(USB_VENDER_ID_REALTEK, 0xF192, 0xff, 0xff, 0xff), .driver_info = RTL8192F}, /* 8192FU 2_2 _/  
{USB_DEVICE_AND_INTERFACE_INFO(0x2357, 0x0135, 0xff, 0xff, 0xff), .driver_info = RTL8192F}, /_ 增加这行 _/  
{USB_DEVICE_AND_INTERFACE_INFO(USB_VENDER_ID_REALTEK, 0xA725, 0xff, 0xff, 0xff), .driver_info = RTL8192F}, /_ 8725AU 2_2 */  
#endif
```
- 重新编译和安装驱动 
```bash
make -j$(nproc)

sudo make install

sudo modprobe 8192fu

# sudo usb_modeswitch -KW -v 0bda -p a192 

usb-devices
```

## 总结
- 可以直接搜设备ID，找到对应的驱动，比搜产品名字好用
- 我是先修改`usb_mode`再编译驱动的，或许直接编译并应用再改模式就不用改代码了，如果直接编译不行的话，就试试先改`usb_mode`，再改代码的方法