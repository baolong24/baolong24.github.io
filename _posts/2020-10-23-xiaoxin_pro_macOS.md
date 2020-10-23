---
layout: post
title:  "小新 Pro 13‘ 2020 macOS 安装教程"
date:   2020-10-23 18:33:23
categories: PC macOS Hackintosh
author: 1404のxiaobao(凛凛凛个鳖)
excerpt: 小新 Pro 13‘ 2020 macOS 安装教程
tags: PC macOS Hackintosh
---

* content
{:toc}

# 小新 Pro 13‘ 2020 macOS 安装教程

## 电脑配置

CPU：Intel® Core™ i5-10210U CPU @ 1.60GHz(4C8T)
RAM：板载 16 GB 2666 MHz DDR4
硬盘：忆联 AH530
显卡：Intel UHD Graphics 620 & Nvdia MX350
声卡：ALC257
网卡：Intel AX201

## 安装步骤

### 准备 macOS 安装介质

* 前往[黑果小兵下载](https://blog.daliansky.net) 安装镜像
* 使用[balenaEtcher](https://www.balena.io/etcher/)将安装镜像写入U盘

### 刷新 BIOS
> 注意⚠️：刷写BIOS 有风险，后果自负

* 下载 BIOS [https://github.com/daliansky/XiaoXinPro-13-hackintosh/wiki/files/CLCN32WW.DPC10.rar](https://github.com/daliansky/XiaoXinPro-13-hackintosh/wiki/files/CLCN32WW.DPC10.rar)
* 解压后双击 `exe` 文件进行安装

### 解锁 DVMT 、 CFG

* 按 `Fn+F2` 进入 `BIOS`
* 进入 `Security` 设置 `Secure Boot` 为 `Disabled`
* 将 `Advanced -> System Agent -> Graphics Configura -> DVMT PreAllocated` 设置为 `DVMT = 64M`
* 将 `Advanced -> Power Performanc -> CPU Power Manage -> CPU Lock Configuration` 设置为 `CFG =disable`
* 按 `Fn+F10` 保存设置

### 安装 macOS

* 开机，从U盘引导进入系统
* 选择 `Install from Install macOS Catalina`
* 选择简体中文
* 选择磁盘工具
* 进入磁盘工具，点击窗口左上角，选择显示所有设备
* 选择您设备相对应的磁盘
* 点击抹掉
* 在弹出的窗口中输入：名称：`Macintosh HD`；格式：`APFS`；方案：`GUID分区图`
* 点击抹除，然后等待操作结束
* 点击完成，通过菜单选择退出磁盘工具或者按窗口左上角红色按钮离开磁盘工具
* 选择安装macOS，点击继续
* 选择将要安装的磁盘卷标Macintosh HD，点击安装
>  * 重启后将会继续安装，在安装期间，通常会自动重启2-3遍
> * 系统重启后，`CLOVER` 或 `OC` 引导界面会多出几个卷标，请选择 `Boot macOS Install form Macintosh HD` 卷标继续安装。在系统安装过程中，请总是选择 `Boot macOS Install form Macintosh HD` 卷标继续安装，安装完成后，卷标名称将变更为：`Boot macOS form Macintosh HD`

### 安装后的系统配置

#### 清空 kext 缓存
```shell
sudo kextcache -i /
```

### 添加macOS 引导 (macOS下操作)

#### 挂载 EFI 分区
```shell
# 查看所有磁盘分区
diskutil list

# 挂载磁盘EFI分区
sudo diskutil mount disk0s1

# 挂载U盘 EFI 分区
sudo diskutil mount disk1s1

# 打开访达
open .
```

#### 合并EFI分区
* [下载 EFI](https://github.com/daliansky/XiaoXinPro-13-hackintosh/releases)然后进行合并

**这里有一点需要注意**:如果之前安装过Windows系统的话,会存在EFI的目录,只是EFI的目录下面只有BOOT和Microsoft这两个目录,如果希望添加macOS的Clover引导的话,可以将USB的EFI分区里面的EFI目录下面的CLOVER复制到磁盘里的EFI目录下,也就是执行的是**合并**的操作,让EFI同时支持WINDOWS和macOS的引导.千万不要全部复制,否则有可能造成EFI无法启动Windows.

### 添加macOS 引导 (Windows 下操作)

#### 挂载 EFI 分区
* 按下 `Win + R` ，输入 `cmd` 打开命令指示符，并输入以下命令
```shell
diskpart
list disk           # 磁盘列表
select disk n       # 选择EFI分区所在的磁盘，n为磁盘号
list partition      # 磁盘分区列表
select partition n  # 选择EFI分区，n为EFI分区号
set id="ebd0a0a2-b9e5-4433-87c0-68b6b72699c7"	# 设置为EFI分区
assign letter=X     # x为EFI分区盘符
```

#### 合并EFI分区
* [下载 EFI](https://github.com/daliansky/XiaoXinPro-13-hackintosh/releases)然后进行合并

**这里有一点需要注意**:如果之前安装过Windows系统的话,会存在EFI的目录,只是EFI的目录下面只有BOOT和Microsoft这两个目录,如果希望添加macOS的Clover引导的话,可以将USB的EFI分区里面的EFI目录下面的CLOVER复制到磁盘里的EFI目录下,也就是执行的是**合并**的操作,让EFI同时支持WINDOWS和macOS的引导.千万不要全部复制,否则有可能造成EFI无法启动Windows.

### 添加 UEFI 引导项
1. 下载 `BOOTICE`
2. 打开BOOTICE软件,选择物理磁盘,选择欲操作的目标磁盘,点击分区管理,弹出分区管理的窗口,点击分配盘符,为ESP分区分配一个盘符,点击确定
3. 选择UEFI,点击修改启动序列,点击添加按钮,菜单标题填写:CLOVER,选择启动文件,在打开的窗口里选择ESP分区下的目录\EFI\CLOVER\CLOVERX64.EFI,点击保存当前启动项设置
> OC同理

## 完善驱动

### 声卡
* 型号为ALC257，注入ID：99，使用AppleALC仿冒，顺利加载；目前麦克风不工作

### 网卡和蓝牙
* 型号为 Intel AX201 使用 `OpenIntelWireless` 的开源项目可以驱动，目前隔空投送不可用

#### 蓝牙
> 经过我的测试结果表明，目前版本无法连接 AirPods
1. 下载[IntelBlueTooth.zip](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases/download/1.1.2/IntelBluetooth.zip)
2. 解压并将 `IntelBluetoothInjector.kext` 和 `IntelBluetoothFirmware.kext` 放入 `EFI` 的 `kext` 文件夹，并在 `Clover` 或 `OC` 中配置启用
3. 重启即可

#### Wi-Fi
> 客户端 Wi-Fi 方案 (仅Wi-Fi可用，接力、定位等不可用)
1. 下载 [itlwm.kext](https://github.com/OpenIntelWireless/itlwm/releases/download/v1.1.0/itlwm_v1.1.0_Stable.kext.zip)
2. 下载安装 [HeilPort](https://github.com/OpenIntelWireless/HeliPort/releases/download/v1.0.1/HeliPort.dmg)
3. 将 `itlwm.kext` 复制到 `EFI` 内的 `kext` 文件夹，并在 `OC` 中配置启用
4. 重启
5. 打开 HeilPort，连接Wi-Fi

> 原生 Wi-Fi 方案 (Wi-Fi、接力、定位、智能热点可用，但无法连接到隐藏的 Wi-Fi 网络)
> 未在 `Clover` 中测试，因此只介绍 `OC` 配置方法
1. 下载 [AirportItlwm.kext](https://github.com/OpenIntelWireless/itlwm/releases/tag/v1.1.0)
2. 将 `AirportItlwm.kext` 复制到 `EFI` 内的 `kext` 文件夹，并在 `OC` 中配置启用
3. 使用 `OC` 将 `SecureBootModel` 设为 `Default` 
4. 重启即可

## 常见问题

### 注入 HiDPI 后花屏
将 `AAPL,ig-platform-id` 设为 `0500A63E`
`device-id` 设为 `A63E0000`
或将 `AAPL,ig-platform-id` 设为 `0400A53E`
`device-id` 设为 `A53E0004`

### 安装过程中触控板不可用
安装时替换 EFI或外接 USB 鼠标

### i7-10710U 无法安装
仿冒 CPU ID 为 `0x0806EC` 或 `0x0806EB`
