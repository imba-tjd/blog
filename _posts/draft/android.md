# 刷机

* 概念教程：https://www.bilibili.com/video/BV1BY4y1H7Mc
* 底层刷机工具：https://geek.wugov.com/

## rec

* https://wiki.orangefox.tech/
  * 如果已经安装了TWRP，进入，把zip包传进内部存储，选Install安装，装zip，完成。选位置的时候右下角有个InstallImage，是用来装img文件的，好像升级twrp是用这个，需要选分区
* 线刷：fastboot flush recovery twrp.img。临时进入：fastboot boot twrp.img。卡刷更新：Install - Install from image
* 刷系统：三清（第一次要Format），进入Rec，菜单里打开sideload，运行adb sideload rom.zip
* 分区：boot（内核、root）、efs（IMEI）、persist（传感器校准数据）、modem（基带固件，控制蜂窝通信）、super（包括system和vendor）。system是系统app，vendor是HAL驱动适配具体硬件。根据网友所说，efs3 efsc不会变，efs1 efs2 firmware persist每次备份都变，无论刷不刷，且能备份但不能恢复，会卡米。可以备份boot system data（不包含储存），前两者只用备份一次
* 双清：Dalvik Cache、Cache。三清：Data（但不清除/data/media），相当于恢复出厂设置

## adb

* 下载：https://developer.android.com/tools/releases/platform-tools
  * 远程：https://adb.http.gs/
* 图形化
  * 搞机工具箱：https://jamcz.com/gjgjx/
  * https://aya.liriliri.io/zh/
* 命令帮助：https://tangoadb.dev/next/api/ 也是用ts重新实现的adb

### 命令

* shell：相当于进入手机的Linux目录里，并不是用于省略其它命令的adb前缀
  * pm list package; pm disable-user/uninstall *pkg_name*
  * su：如果已经root过了，用su会在手机上触发授权提示
  * 设置授时服务器：settings put global ntp_server ntp1.aliyun.com。网络连接受限：settings put global captive_portal_https_url   https://connect.rom.miui.com/generate_204
* 安装apk：install x.apk。不是pm的子命令，apk在win上
* 复制文件：push a.txt /sdcard/ 反过来也可以，放到CWD
* reboot recovery/bootloader
* forward tcp:pc_port tcp:phone_port，双方监听localhost，pc主动发。adb监听前者，发给后者
  * reverse tcp:phone_port tcp:pc_port
  * 设备端支持localabstract
* 启动程序（Linux原生用shell）：app_process。不能把stdio用作字节流

## magisk

* 用于获取root权限。需要解锁bl
  * 线刷，无需rec：提取ROM的init_boot.img或boot.img放到手机里，安装apk，“安装”选择img文件产生patched文件。fastboot flush boot magisk_patched.img
  * 卡刷：下载apk，重命名为zip。进入twrp，安装。但这种方式废弃了
* 隐藏：Shamiko https://github.com/LSPosed/LSPosed.github.io/releases
  * 排除列表（黑名单）：“配置排除列表”，设置哪些app不启用模块，但不是隐藏，不防检测。配好后要再开“遵守排除列表”开关
* Delta版、Kitsune版：自带隐藏功能
* 爱玩机工具箱：有模块仓库。免root专区有shizuku功能
* 模块
  * Energized Protection：使用hosts屏蔽广告
  * Debloater：禁用预装应用
  * Riru - Clipboard Whitelist
  * LSPosed
  * F-Droid Privileged：允许F-Droid静默安装
  * https://github.com/CRANKV2/ZRAM 爱玩机工具箱也能调整
  * https://github.com/muink/Magisk-Captive-Manager 解决wifi连接性检测时无法访问Google的问题。爱玩机工具箱也能调整：界面显示调节 - 去！和x
  * HyperCeiler 集成了许多功能的MIUI优化
  * https://github.com/yujincheng08/BiliRoaming 解除番剧区域限制
* 翻车自救：adb shell magisk --remove-modules
* https://magiskcn.com/
* Zygisk：让Magisk运行在安卓系统的Zygote进程中，从而支持LSPosed、Shamiko、配置排除列表。有root但不能注入其它app

### 同类软件

* https://kernelsu.org/zh_CN/ 国产，比magisk更不容易被检测。一般需出场安卓12
  * https://kernelsu-next.github.io/webpage/zh_CN/
* https://apatch.dev/zh_CN/ 国产，支持的内核版本比kernelsu多。好像优点是兼容magisk的模块
* https://sukisu.org/zh/

## [shizuku](https://shizuku.rikka.app/zh-hans/)

* 在维护中的Fork：https://github.com/thedjchi/Shizuku
* 用于只需adb功能执行的程序，无需root权限
* 线激活：adb shell sh /storage/emulated/0/Android/data/moe.shizuku.privileged.api/start.sh
* 本机激活：开发者选项里启动无线调试，本程序将作为客户端，进入授权码界面后，在通知栏里输入。只需配对一次，后续再开无线调试，app直接点启动即可。快速开启：开发者选项 - 快捷设置开发者图块，将无线调试加到下拉菜单里
* 对于已经安装了Magisk，优先用 https://github.com/RikkaApps/Sui 模块

### 相关软件

* App Ops：调整app权限，可以禁用传感器。常驻后台时可以防止操作剪贴板。同类：权限狗，不支持安卓11
* 黑域：https://brevent.jianyu.io/ 设定应用退出后超时多久强行停止。同类：绿色守护。感觉现在出场系统自带了，不需要
* 冰箱，用于停用(freeze/disable)应用：https://iceboxdoc.catchingnow.com/ 免费版可以停用15个，需要使用时可以从冰箱app点击，比用adb命令行方便。免费开源替代：https://github.com/aistra0528/Hail 同类：小黑屋
* 隐藏应用：https://deltazefiro.github.io/Amarok-doc/ Hail也可以
* Scene：http://vtools.omarea.com/
* SAI：用于安装split apk，在设置里调模式
* https://github.com/iamr0s/Dhizuku/blob/main/docs/README_zh_rCN.md 提供“设备所有者”权限。会影响系统自带双开
* https://github.com/timschneeb/awesome-shizuku

## LSPatch

* 免root使用LSPosed框架
* https://github.com/JingMatrix/LSPatch https://github.com/HSSkyBoy/NPatch

## ROM

* https://wiki.lineageos.org/
* https://xiaomirom.com/ 原版MIUI
* https://download.pixelexperience.org/ 不更新了
* https://crdroid.net/ 类原生
* https://pixelos.net/
* https://www.pling.com
* https://grapheneos.org/ https://axpos.org/ https://evolution-x.org/ https://derpfest.org/
* https://www.axionos.org/
* https://thecustomrom.com/

### mido

* 非官方lineage https://github.com/zeelog/OTA/releases https://github.com/zeelog/device_mido_twrp/releases https://t.me/LOSRN4 安卓15
* https://evolution-x.org/devices/mido
* https://t.me/s/rn4downloads https://t.me/s/midoid_update 前者不更新了
  * https://sourceforge.net/projects/nranjan-17/files/BETA/
* https://xdaforums.com/t/rom-16-beta-unofficial-mido-axion-aosp-2-0-22-08-25.4755711 16
* https://xdaforums.com/f/xiaomi-redmi-note-4-snapdragon-roms-kernels-re.6145/

### k30 4g / Poco X2 / phoenix

* https://t.me/s/pocox2officialupdates

## firmware

* 不是ROM或OTA，是一套底层驱动。如果用原厂ROM，则不用管，会自动更新
* https://github.com/XiaomiFirmwareUpdater

## 系统预装应用包名

* realme：https://tieba.baidu.com/p/7404872902
* hm4：https://www.bilibili.com/opus/1026398532683169809 https://lstwwa.com/6846e6c5.html
* miui：https://miuiver.com/wp-content/uploads/miui-pre-installed-software.html

# 验机

* APP：全渠道质检、设备信息by流舟、DevCheck
* 面交：确定付钱的人与给设备的人是同一个人

# 软件

* https://github.com/Lin-arm/GKD_subscription
* Google相关：https://nikgapps.com/ https://github.com/microg/GmsCore/wiki
* https://playmods.net/zh/ 破解版下载站

## 互联

* 副屏：https://www.spacedesk.net/ https://superdisplay.app/beta/ 后者有线技术更好，但apk破解不了
* 播放器（但有线连接不太行）：https://liteapks.com/download/audiorelay-204451 https://audiorelay.net/downloads 可以用热点（WiFi Direct），注意要手动输入热点虚拟网卡IP
  * 另一款：https://georgielabs.net/
* USB共享网络：https://www.jianshu.com/p/61762932acbd https://www.lanzoux.com/b08l43p2j 但实测只能将平板作为跳板，电脑访问平板连的wifi，而不能平板访问电脑。电脑上会安装虚拟网卡
* USB反向共享网络：https://github.com/Genymobile/gnirehtet 开启USB调试，下载rust实现，给adb添加进PATH，运行run，手机上会自动收到安装包。但实测用不了audiorelay，甚至能连接上，但收不到音频数据。看起来是它在win上起了一个NAT进程，安卓开启XPN
* https://github.com/Genymobile/scrcpy 将安卓投屏到电脑

## 桌面

* https://github.com/LawnchairLauncher/lawnchair

## 播放器

* https://github.com/anilbeesetti/nextplayer
* https://github.com/namidaco/namida
* oplayer：有广告，不适配平板

## 相机

* 简单相机，无扫码 https://www.openapk.net/zh/simple-camera/com.simplemobiletools.camera/apk/download

## 输入法

* 微信输入法
* 讯飞输入法
