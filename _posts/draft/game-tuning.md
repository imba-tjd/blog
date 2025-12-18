# NV App

## 垂直同步

* 如果帧数是刷新率2倍以上，又想要低延迟，用三重缓冲（快速垂直同步）
* 自适应 (Adaptive)：这是 G-SYNC 诞生之前的“穷人版 G-SYNC”。FPS 高时自动开启V-Sync防撕裂，FPS 低时自动关闭V-Sync（防卡顿，但会撕裂）
* 无边框窗口模式：通过了DWM，要调整G-SYNC“为窗口模式和全屏模式启用”，依赖游戏是否允许 DWM 传递 VRR 信息。现代 DX12 游戏的无边框模式延迟已经非常接近独占全屏了（启用"独立翻转模型"）。默认强制启用某一种垂直同步

## 3A设置

锁 刷新率-3 帧 + G-SYNC（需独显直连+全屏独占） + 关闭游戏内垂直同步 + 在NVCP中开启垂直同步

说明：

1. G-SYNC只在不满刷新率时生效，大于等于刷新率时失效。如果帧率大于刷新率会全屏撕裂，开启V-Sync可防止（在锁刷新率-3时不靠它防止）。
2. G-SYNC生效时，只能解决“刷新开始时间”的同步，不能解决“扫描输出（Scanout）”过程中的偏差。结果是屏幕底部可能撕裂。
3. G-SYNC 激活范围内驱动里的 V-Sync 并不会像传统 V-Sync 那样强行让显卡等待（导致延迟），而是仅仅充当一个“守门员”。这消除了最后的底部撕裂，且几乎不增加任何输入延迟。
4. 如果在游戏里开启垂直同步，很多游戏引擎会默认在这个环节加入一个“预渲染帧队列”。

## Smooth Motion插帧

* 对于RPG Maker游戏，配合Magpie可用。移动时产生果冻效应，对话文字会扭曲

## GPU System Processor

启用GPU内部的某些处理器，让原本一些需要CPU处理的改为GPU内部自己处理，能降低延迟。但如果显卡负载满了可能不会提升，适合低U高显的。

```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0001]
"EnableGpuFirmware"=dword:00000001
```

其中0000为核显，0001为独显，最好具体看一下内容。

验证：nvidia-smi -q 显示 GSP Firmware Version 不为N/A。

缺点：占用0.5GB显存（看FB Memory Usage的Reserved）。可能导致G-Sync失效、WebView2程序出问题。影响HDCP（播放受版权保护的视频的技术）、DSC。有网友说与intel核显一起用的时候会损坏GSP单元。

# 相关软件

* https://www.latencymon.com/
