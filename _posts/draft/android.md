## rec

* https://orangefox.download/zh-CN
* 线刷：fastboot flush recovery twrp.img。临时进入：fastboot boot twrp.img。卡刷更新：Install - Install from image

## adb

* 下载：https://developer.android.com/tools/releases/platform-tools
  * 远程：https://adb.http.gs/
* 命令
  * shell：相当于进入手机的Linux目录里，并不是用于省略其它命令的adb前缀
    * pm list package; pm disable-user/uninstall *pkg_name*
  * 安装apk：install x.apk
  * 复制文件：push a.txt /sdcard/
  * reboot recovery/bootloader
* 图形化
  * 搞机工具箱：https://jamcz.com/gjgjx/
  * https://aya.liriliri.io/zh/

## magisk

* 用于获取root权限。需要解锁bl
  * 线刷，无需rec：提取ROM的init_boot.img或boot.img放到手机里，安装apk，“安装”选择img文件产生patched文件。fastboot flush boot magisk_patched.img
  * 卡刷：下载apk，重命名为zip。进入twrp，安装。但这种方式废弃了
* 隐藏：https://github.com/K3V1991/Passing-SafetyNet-with-Magisk-Zygisk-and-DenyList https://github.com/Displax/safetynet-fix https://github.com/LSPosed/LSPosed.github.io/releases
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
  * https://github.com/muink/Magisk-Captive-Manager 解决WIFI连接性检测时无法访问Google的问题。爱玩机工具箱也能调整：界面显示调节 - 去！和x
* 翻车自救：adb shell magisk --remove-modules

### 同类软件

* https://kernelsu.org/zh_CN/ 国产，比magisk更不容易被检测。一般需出场安卓12
  * https://kernelsu-next.github.io/webpage/zh_CN/
* https://apatch.dev/zh_CN/ 国产，支持的内核版本比kernelsu多。好像优点是兼容magisk的模块
* https://sukisu.org/zh/

## [shizuku](https://shizuku.rikka.app/zh-hans/)

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
* https://github.com/HSSkyBoy/NPatch

## 其它软件

* https://nikgapps.com/

## ROM

* https://wiki.lineageos.org/
* https://www.droidontime.com/ 官网很卡
* https://download.pixelexperience.org/ 不过更新了
* https://resurrectionremix.com/ https://sourceforge.net/projects/resurrectionremix-ten/files 感觉不太行
* https://www.aospextended.com/ 522错误
* https://xiaomirom.com/ 非官方原版MIUI
* https://crdroid.net/ 类原生
* https://pixelos.net/

### mido

* 非官方lineage https://github.com/zeelog/OTA/releases https://github.com/zeelog/device_mido_twrp/releases https://t.me/LOSRN4 安卓14
  * https://xdaforums.com/t/android-13-lineageos-20-mido-vanilla-by-zeelog-stable-no-bugs-smooth.4516771/
* https://t.me/s/rn4downloads
* https://download.pixelexperience.org/mido 安卓13，维护者是印度的
  * https://sourceforge.net/projects/nranjan-17/files/BETA/
* https://xdaforums.com/t/android-14-custom-roms-list.4639540/ 14收集
* https://www.pling.com/p/1451939 安卓13
* https://sourceforge.net/projects/mokee/files/RELEASE/mido/ 魔趣，不更新了，安卓10
* 论坛
  * https://forum.xda-developers.com/f/xiaomi-redmi-note-4-snapdragon-roms-kernels-re.6145/
  * https://thecustomrom.com/category/redmi-note-4-4x/

## 桌面

* https://github.com/LawnchairLauncher/lawnchair

## 播放器

* https://github.com/anilbeesetti/nextplayer

## 系统自带应用包名

* realme：https://tieba.baidu.com/p/7404872902
* hm4：https://www.bilibili.com/opus/1026398532683169809

## 模块

* HyperCeiler 集成了许多功能的MIUI优化
* https://github.com/JingMatrix/LSPatch

# 验机

* APP：全渠道质检、设备信息by流舟
* 面交：确定付钱的人与给设备的人是同一个人
