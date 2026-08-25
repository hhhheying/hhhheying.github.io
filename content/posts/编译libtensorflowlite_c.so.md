---
title: "编译 libtensorflowlite_c.so"
date: 2026-04-14T00:00:00+08:00
lastmod: 2026-04-14T00:00:00+08:00
description: "在 WSL/Linux 下编译 TensorFlow Lite C 库 libtensorflowlite_c.so 的过程记录"
tags: ["TensorFlow", "Android", "编译"]
categories: ["技术"]
---

# 编译libtensorflowlite_c.so
- 环境：wsl/linux
- 需要充足的内存和硬盘空间

## 安装基础工具
- python版本 >= 3.9
- 可以使用虚拟环境
```bash
sudo apt-get update
sudo apt-get install build-essential git unzip python3 python3-pip
```

## 安装bazel
- 编译最新的tensorflow源码需要版本 >= 7.1.4
- 下载安装脚本，并运行。命令行太慢，建议直接在浏览器里下载[Releases · bazelbuild/bazel · GitHub](https://github.com/bazelbuild/bazel/releases)
```bash
curl -L https://github.com/bazelbuild/bazel/releases/download/7.1.4/bazel-7.1.4-installer-linux-x86_64.sh -o bazel-installer.sh
chmod +x bazel-installer.sh
./bazel-installer.sh --user
```

## 下载ndk
- 版本 >= 25
- [NDK 下载 | Android NDK | Android Developers](https://developer.android.com/ndk/downloads?hl=zh-cn)
```bash
unzip android-ndk-r25b-linux.zip -d ~/
echo 'export ANDROID_NDK_HOME="$HOME/android-ndk-r25b"' >> ~/.bashrc
echo 'export PATH="$ANDROID_NDK_HOME:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 下载android sdk
- 直接下载android studio 或者用命令行
- 先下载java，最好选17
```bash
sudo apt update
sudo apt install openjdk-17-jdk 
```
- 下载sdk，如果下载慢可以直接从浏览器下载[下载 Android Studio 和应用工具 - Android 开发者 | Android Developers](https://developer.android.com/studio?hl=zh-cn#command-tools)
```bash
mkdir -p ~/Android/Sdk/cmdline-tools
cd ~/Android/Sdk/cmdline-tools

# 下载并解压（替换为实际版本号）
wget https://dl.google.com/android/repository/commandlinetools-linux-10406996_latest.zip
unzip commandlinetools-linux-10406996_latest.zip
mv cmdline-tools latest  # 重命名为 `latest`

# 配置环境变量
nano ~/.bashrc
export ANDROID_HOME="$HOME/Android/Sdk"
export PATH="$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"
# ctrl + x 退出保存
source ~/.bashrc

# 安装组件
sdkmanager --update
sdkmanager "platform-tools" "build-tools;34.0.0" "platforms;android-34" "emulator"
```

## 下载tensorflow源码
```bash
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
git checkout r2.14  # 切换到指定版本
```

## 运行交互式配置脚本
```bash
./configure
```
- 按需配置

## 编译
- 通用编译（ARM64 架构）
```bash
bazel build -c opt \
  --config=android_arm64 \
  --cxxopt='--std=c++17' \
  //tensorflow/lite/c:libtensorflowlite_c.so
```

## 生成文件
生成库位于
```text
bazel-bin/tensorflow/lite/c/libtensorflowlite_c.so
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIzMzk0MTI3OSw5MTg0NDM5ODIsLTUwOD
A0MzU5NywtMTAzMTM3Nzc0MSwtMTczODQxMzYxNywtMTAyNzg3
OTM3MF19
-->