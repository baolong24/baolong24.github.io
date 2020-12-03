---
layout: post
title:  "Razer Phone TWRP 适配心得"
date:   2020-12-03 20:11:23
categories: Android Build TWRP
author: 1404のxiaobao(凛凛凛个鳖)
excerpt: 暑假的时候搞到一台雷蛇手机，因为官方的 TWRP 功能基本上都是残废的，就想自己适配个，不过这也是第一次适配，故把第一次的适配步骤写在博客里
tags: Android Build TWRP
---

* content
{:toc}

# Razer Phone TWRP 适配心得
暑假的时候搞到一台雷蛇手机，因为官方的 TWRP 功能基本上都是残废的，就想自己适配个
不过这也是第一次适配，故把第一次的适配步骤写在博客里
语文不是很好，所以可能说的不是很明白，见谅哈哈哈

## 配置编译环境
TWRP 的编译环境要求跟 Android 的一样
```shell
# 安装必要依赖
sudo apt update
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev repo git

# 同步 TWRP 源码
mkdir twrp && cd twrp
repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni -b twrp-9.0 --depth=1
repo sync
```

## 初始化编译必要文件
不管是编译 Android 还是 TWRP，这些文件都是必要的
``Android.mk``, ``AndroidProduct.mk``, ``BoardConfig.mk``, ``omni_cheryl.mk``
这几个文件可以直接从 Lineage 那边拿，然后进行修改

``Android.mk``、``AndroidProduct.mk``、``omni_cheryl.mk`` 一般情况下无需进行修改

编译 TWRP 需要对 ``BoardConfig.mk`` 等文件进行修改

### 修改 BoardConfig.mk
首先需要添加几个 TWRP 必须要的 Flag
```makefile
ALLOW_MISSING_DEPENDENCIES = true # 由于同步的只有 TWRP 源码，编译时需要打开这个
VENDOR_SECURITY_PATCH := 2025-07-05 # 需要加入补丁更新日期来回避防回滚机制
TW_THEME := portrait_hdpi # TWRP 要使用的主题
TW_BRIGHTNESS_PATH := "/sys/devices/soc/c900000.qcom\x2cmdss_mdp/c900000.qcom\x2cmdss_mdp:qcom\x2cmdss_fb_primary/leds/lcd-backlight/brightness" # 亮度调节的内核节点
TW_MAX_BRIGHTNESS := 255 # 设置最高亮度
TW_DEFAULT_BRIGHTNESS := 155 # 设置最低亮度
TW_EXTRA_LANGUAGES := ture # 设置是否添加亚洲语言
TW_IGNORE_MISC_WIPE_DATA := true # 是否在 wipe data 时忽略 misc
TW_SCREEN_BLANK_ON_BOOT := true # 这个我也不知道什么作用
TW_NO_EXFAT_FUSE := true # 这个我也不知道什么作用
TW_INCLUDE_CRYPTO := true #是否添加加密支持
TARGET_CRYPTFS_HW_PATH := vendor/qcom/opensource/commonsys/cryptfs_hw # 解密所需依赖的源码路径
RECOVERY_SDCARD_ON_DATA := true # 设置内部存储的数据是否在 data 分区
BOARD_HAS_NO_REAL_SDCARD := true # 这个我也不知道什么作用
BOARD_PROVIDES_GPTUTILS := true # 这个我也不知道什么作用
TW_USE_TOOLBOX := true # 是否使用 ToolBox
TWRP_INCLUDE_LOGCAT := true # 是否启用 logcat
TARGET_USES_LOGD := true # 是否启用 logcat

# 如果是 A/B 分区的话还得加入这些
# A/B OTA
AB_OTA_UPDATER := true
BOARD_USES_RECOVERY_AS_BOOT := true
BOARD_BUILD_SYSTEM_ROOT_IMAGE := true
```

### 修改 device.mk
由于雷蛇是 A/B 分区，需要在设置里配置一些 A/B 分区的东西

```makefile
PRODUCT_PLATFORM := msm8998 # 定义平台

# A/B
AB_OTA_UPDATER := true # 是否打开 A/B 支持

AB_OTA_PARTITIONS += \ # 定义使用 A/B 特性的分区
    boot \
    system \
    vendor

AB_OTA_POSTINSTALL_CONFIG += \ 
    RUN_POSTINSTALL_system=true \
    POSTINSTALL_PATH_system=system/bin/otapreopt_script \
    FILESYSTEM_TYPE_system=ext4 \
    POSTINSTALL_OPTIONAL_system=true

PRODUCT_PACKAGES += \
    otapreopt_script \
    cppreopts.sh \
    update_engine \
    update_verifier \
    update_engine_sideload

# Boot control
PRODUCT_PACKAGES += \
    android.hardware.boot@1.0-impl \
    android.hardware.boot@1.0-service \
    bootctrl.msm8998 \

PRODUCT_PACKAGES_DEBUG += \
    bootctl

PRODUCT_STATIC_BOOT_CONTROL_HAL := \
    bootctrl.msm8998 \
    libcutils \
    libgptutils \
    libz
```

并在 ``omni_cheryl.mk`` 里调用
```makefile
$(call inherit-product, device/razer/cheryl/device.mk)
```

### 配置 TWRP 分区表

新建文件夹 ``recovery/root/etc``

新建文件 ``twrp.fstab``

该文件就是分区表，格式如下
```
# mount point   fstype     device                                   device2                       flags
# 定义挂载点  定义文件系统类型 定义挂载原块文件路径                                                       定义一些特性
/boot           emmc       /dev/block/bootdevice/by-name/boot                                     flags=display="Boot";slotselect
/misc           emmc       /dev/block/bootdevice/by-name/misc                                     flags=display="Misc"
/data           ext4       /dev/block/bootdevice/by-name/userdata                                 flags=encryptable=ice:aes-256-cts
/system         ext4       /dev/block/bootdevice/by-name/system                                   flags=slotselect;backup=0
/system_image	emmc	   /dev/block/bootdevice/by-name/system					  flags=slotselect
/vendor		ext4       /dev/block/bootdevice/by-name/vendor					  flags=slotselect;display="Vendor";backup=0;wipeingui
/vendor_image	emmc	   /dev/block/bootdevice/by-name/vendor					  flags=slotselect

/usbstorage     vfat       /dev/block/sdg1                          /dev/block/sdg                flags=fsflags=utf8;display="USB Storage";storage;wipeingui;removable
```

**关于 `flags`**
```
# 如果是 A/B 设备，请给使用 A/B 特性的分区定义 slotselect
# 用 backup 来定义可备份分区
# display 用来自定义分区名
# encryptable 来定义加密类型
# removable 用来定义可否热拔插
```

**关于 `recovery.wipe`**

这个我也不知道有什么用，不过我看 A/B 分区的机子都有，我就照着格式搞了
格式如下

```
# All the partitions to be wiped (in order) under recovery.
/dev/block/bootdevice/by-name/system_a
/dev/block/bootdevice/by-name/system_b
/dev/block/bootdevice/by-name/vendor_a
/dev/block/bootdevice/by-name/vendor_b
/dev/block/bootdevice/by-name/userdata
# Wipe the boot partitions last so that all partitions will be wiped
# correctly even if the wiping process gets interrupted by a force boot.
/dev/block/bootdevice/by-name/boot_a
/dev/block/bootdevice/by-name/boot_b
```

### 预编译内核
这一步可有可无，如果您有现成的内核源码，则可以拿内核源码编译
如果没有内核源码或官方未开源则可以从官方包的 `boot.img` 中提取内核
具体怎么操作可以前往 [boot.mokeedev.com](boot.mokeedev.com)
这里不在阐述

有预编译内核后请在 `BoardConfig.mk` 中定义
```makefile
# Prebuilt Kernel
BOARD_KERNEL_BASE := 0x00000000
BOARD_KERNEL_IMAGE_NAME := Image.gz-dtb
BOARD_KERNEL_OFFSET = 0x00008000
BOARD_KERNEL_PAGESIZE := 4096
BOARD_KERNEL_TAGS_OFFSET := 0x00000100
BOARD_RAMDISK_OFFSET := 0x01000000

TARGET_PREBUILT_KERNEL := device/razer/cheryl/prebuilt/kernel

BOARD_KERNEL_CMDLINE := androidboot.console=ttyMSM0 androidboot.hardware=qcom
BOARD_KERNEL_CMDLINE += user_debug=31 msm_rtb.filter=0x37 ehci-hcd.park=3
BOARD_KERNEL_CMDLINE += service_locator.enable=1
BOARD_KERNEL_CMDLINE += swiotlb=2048
BOARD_KERNEL_CMDLINE += firmware_class.path=/vendor/firmware_mnt/image
BOARD_KERNEL_CMDLINE += androidboot.selinux=permissive
```
### init.rc

不同机型，init 部分也不一样
您可以参考 cheryl 的 init
[https://github.com/baolong24/android_device_razer_cheryl/tree/twrp-9.0/recovery/root](https://github.com/baolong24/android_device_razer_cheryl/tree/twrp-9.0/recovery/root)

### bootctrl 和 gpt-utils
如果你的设备采用 A/B 分区，那必须编译这两个组件
确保 tree 里面有编译 `bootctrl` 和 `gpt-utils`
这两个东西可以从其它机型的 tree 里面拿，通用的
可以参考这两条 commit
`bootctrl`: [https://github.com/baolong24/android_device_razer_cheryl/commit/7691eb8dea85dfcb8c92773ad54fb480f8a5e9f7](https://github.com/baolong24/android_device_razer_cheryl/commit/7691eb8dea85dfcb8c92773ad54fb480f8a5e9f7)
`gpt-utils`: [https://github.com/baolong24/android_device_razer_cheryl/commit/fd3020e57d31ab30473f62d5340b1bc9d2b020a2](https://github.com/baolong24/android_device_razer_cheryl/commit/fd3020e57d31ab30473f62d5340b1bc9d2b020a2)

## 添加 Vendor Blobs

这一步需要添加厂商闭源的组件，以保证 TWRP 的正常运行
可以参考这条 [commit](https://github.com/baolong24/android_device_razer_cheryl/commit/e6391e029f9b91cad92f0bfd7a249def1827c7ca) 来添加

## 开始编译
上面的东西都配置好后就可以开始编译了

```shell
. build/envsetup.sh
lunch omni_cheryl-eng
mka bootimage
# 如果您不是 A/B 分区
mka recoveryimage
```

## 编译完成
编译完成了，接下来干什么呢
当然是发酷安装逼啊，哈哈哈哈

## 一些问题
### 我是 A/B 分区，但编译出来的只有 img 文件，没有 zip 的永久刷入包
你可以参考这个 [commit](https://github.com/TeamWin/android_device_oneplus_guacamole/commit/42e2e80df2c59f1125a425ac5df1b751f5a28d35#diff-9c0d294c05fc1d88d698034609bb81c0c69196327594e4c69d2915c80fd9850c)
然后在 TWRP 源码中加入 [https://gerrit.omnirom.org/#/c/android_build/+/33182/](https://gerrit.omnirom.org/#/c/android_build/+/33182/)
来加入对 zip 包的编译

### data 加密前可以正常使用 TWRP，但是加密后就卡在 TWRP Logo
1. 先抓 log 看看 blobs 有没有不齐
2. 如果补齐后还是不行，请参考这条 [commit](https://github.com/baolong24/android_device_razer_cheryl/commit/88f61658ac858302cfb03104bd85011a6e892bf5) 来修复

### 我源码怎么拉不下来啊
挂梯子

### 看完了，我还是不懂怎么办
那我也没办法
