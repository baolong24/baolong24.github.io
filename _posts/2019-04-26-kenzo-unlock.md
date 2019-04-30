---
layout: post
title:  "红米note3全网通(kenzo)非官方解锁"
date:   2019-4-26 7:07:22
categories: Android
author: 1404のxiaobao(凛凛凛个鳖)
excerpt: 适用于隐藏id锁机子解锁bl
tags: Android
---

* content
{:toc}

本文仅适用于红米note3全网通版(kenzo)

## 准备事项
下载这些文件
1.[魔改后的bl](https://baolong24.github.io/RN3_unlockblonly.rar)
2.[MiFlash](http://bigota.d.miui.com/tools/MiFlash2018-5-28-0.zip)
3.[进入9008的工具](https://baolong24.github.io%E9%AB%98%E9%80%9Afastboot%E4%B8%80%E9%94%AE%E8%BF%9B9008.zip)

## 开始解锁

### 安装驱动
把下载好的MiFlash解压，然后打开MiFlash
点击Driver，再点击安装即可
安装完成后会有提示

### 进入9008模式
把手机关机，然后按住音量下加电源键进入Bootloader
把“高通fastboot一键进9008”解压
双击edl.bat
如果此时手机呼吸灯闪红灯代表已经进入了9008

### 刷入魔改后的Bootloader
把“RN3_unlockblonly.rar”解压
打开MiFlash，点击浏览，选择解压后的路径
然后刷入即可
注意，刷入完成后手机并不会自动重启，需要手动重启
按住音量下加电源键重启进入Bootloader

### 解锁Bootloader
打开adb
输入fastboot oem unlock-go
如果无法解锁的话请启动手机
进入设置-开发者选项-oem解锁 打开这一选项
然后重新解锁
