---
layout: post
title:  "黑解（ICCID）证书备份"
date:   2021-02-03 18:54:20
categories: iOS iPhone checkra1n
author: 1404のxiaobao(凛凛凛个鳖)
excerpt: 最近 ICCID 和黑解又回来了，这个教程旨在如何备份黑解或 ICCID 激活过后的激活证书，以便下次抹除或黑解（ICCID）失效时使用
tags: iOS iPhone checkra1n
---

* content
{:toc}

# 黑解或 ICCID 证书备份
最近 ICCID 和黑解又回来了，这个教程旨在如何备份黑解或 ICCID 激活过后的激活证书，以便下次抹除或黑解（ICCID）失效时使用

## 需要的条件
1. 一台可以越狱的 iPhone，且已经使用 ICCID（黑解）激活
2. 一台 Mac 或者安装了 Linux 的 PC 或者一台已经安装了 TWRP 的 Android 设备
3. 畅通的国际互联网链接
4. 合适的 USB 数据线

## 步骤
### 给您的 iPhone 越狱
#### Mac 部分
1. 前往 [https://checkra.in](https://checkra.in) 下载适用于 macOS 的版本
2. 打开 checkra1n，并将您的 iPhone 连接到 Mac
3. 根据 checkra1n 的提示给您的 iPhone 进行越狱
4. 越狱完成后您的 iPhone 会自动重启进入系统

#### PC 部分
1. 确保您当前使用的系统为 Linux
2. 前往 [https://checkra.in](https://checkra.in) 下载适用于 Linux 的版本
   >若您的 PC 为 32 位，请下载 i486 版本，64 位请下载 x86_64 版本
3. 打开终端，并按下列命令操作

```shell
# 前往 checkra1n 所在文件夹
cd ~/Download

# 给予 checkra1n 权限
chmod a+x checkra1n

# 运行 checkra1n
./checkra1n
```

4. 根据 checkra1n 的提示给您的 iPhone 进行越狱
5. 越狱完成后您的 iPhone 会自动重启进入系统

#### Android 设备部分
1. 确保您的设备已经安装了 TWRP
2. 前往 [https://checkra.in](https://checkra.in) 下载适用于 Android 的版本
   >若您的 PC 为 32 位，请下载 arm 版本，64 位请下载 arm64 版本
3. 重启您的设备到 TWRP
4. 使用 TWRP 中的文件管理，将 checkra1n 移动到 /cache 或 /data，并给予 755 权限
5. 将您的 iPhone 重启到 DFU 模式，然后连接到您的 Android 设备
6. 使用 TWRP 中的终端功能，并按以下命令操作
```shell
# 前往 checkra1n 所在文件夹
cd /cache

# 运行 checkra1n 进行越狱
./checkra1n -c
```
7. 越狱完成后您的 iPhone 会自动重启进入系统
#### 开始备份证书
**接下来的步骤请确保国际互联网连通正常，如果您的国际互联网连接有问题，则在进行以下步骤时会失败或者非常缓慢。**

1. 当 checkra1n 提示 `All done` 时，在您的手机上操作
2. 解锁 iPhone，打开主屏幕中的 checkra1n
3. 点击 checkra1n 中的 Cydia 进行安装

4. 安装完成后会自动返回到主屏幕，此时打开主屏幕新增的 Cydia 应用
   >打开 Cydia 时若提醒更新，请点击忽略
5. 点击搜索，搜索 Filza File Manager，并安装
6. 安装完成后回到桌面，打开新增的 Filza 应用
7. 打开 Filza 后，前往以下路径
   `/var/wireless/Library/Preferences`
8. 找到以下文件，并打开
   `com.apple.commcenter.device_specific_nobackup.plist`
9. 打开文件后，展开 `Root -> kPostponementTicket ->` 然后点击 `ActivationTicket` 旁的 i 进行编辑
10. 复制值内的所有内容，复制粘贴到一个新的文档，这串值就是您的黑解（ICCID）证书

### 还原证书
**若您对手机进行了还原，且黑解（ICCID）已经失效，您需要手动还原证书**

1. 拔卡，按照正常步骤激活 iPhone
2. 将您的 iPhone 越狱，并在 Cydia 中安装 Filza
3. 打开 Filza 后，前往以下路径
   `/var/wireless/Library/Preferences`
4. 找到以下文件，并打开
   `com.apple.commcenter.device_specific_nobackup.plist`
5. 打开文件后，展开 `Root -> kPostponementTicket ->` 然后点击 `ActivationTicket` 旁的 i 进行编辑
6. 将此处的值全部删除，并将您之前备份的值粘贴进来，然后重启手机
7. 您的手机已经还原为黑解状态，可以直接插卡使用

### 删除越狱环境
**若您不需要保留越狱环境，则可以通过该步骤完全移除越狱环境**
**移除越狱环境不会对还原的证书造成影响**
1. 打开 checkra1n，点击 Restore System 即可
