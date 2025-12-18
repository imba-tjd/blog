---
title: 浏览器
---

## 通用

* F6：聚焦到地址栏

## Firefox

* 正在进行的实验：about:studies
* 网络状况：about:networking
* 搜索的高亮只会显示1000个

### 快捷键

* 地址栏搜索选择非默认搜索引擎时按住shift可直接进行搜索而不触发联想
* 按住alt键就可以开启强制选择模式，这样鼠标就不会发出拖拽等其他功能
* 删除地址栏中的搜索记录：Shift+del，但删不掉历史网址的联想，可打开历史窗口删
* 快进/后退：Alt+滚轮

### 下载

* 历史版本：https://ftp.mozilla.org/pub/firefox/releases/
* 最新版本：https://download.mozilla.org/?product=firefox-latest-ssl&os=win64&lang=zh-CN

### about:config

* browser.tabs.closeWindowWithLastTab 关闭最后一个标签页不关闭浏览器
* browser.tabs.loadBookmarksInTabs 书签用新标签打开
* 搜prefetch有一些预读的选项，但是可能有隐私问题
* browser.cache.disk.enable是否在硬盘上缓存数据，大小受browser.cache.disk.smart_size.enabled自动控制，大概1G
* media.videocontrols.picture-in-picture.enabled视频的画中画图标，除了它还要改xxx.video-toggle.enabled才能生效
* network.captive-portal-service.enabled检测网络是否需要登录
* network.http.http3.enabled：开启quic
* security.enterprise_roots.enabled：使用系统证书列表
* browser.urlbar.trimURLs：地址栏隐藏http
* xpinstall.signatures.required：允许安装未签名扩展
* extensions.pocket.enabled false：禁用pocket扩展
* security.tls.version.min：1为TLS1.0
* network.IDN_show_punycode true：中文域名时显示真正的域名
* network.http.rcwn.enabled：是否启用raced竞速请求。当FF检测到磁盘较慢时，会无视缓存直接发网络请求，理论上会使用先成功的，另一个就取消掉，实际好像不是这样
* devtools.selfxss.count：在console中粘贴时会提示“欺诈警告：粘贴您不了解的东西时请务必小心...如果仍想粘贴，请在下方输入allow pasting（不必按回车键）以允许粘贴”。此项可提高阈值，默认是0
* network.http.max-persistent-connections-per-server：连接同一域名的最大连接数，默认6；另有per-proxy默认32
* network.http.accept-encoding：去掉deflate的压缩
* ui.key.menuAccessKey：是否允许按alt打开菜单栏
* security.dialog_enable_delay：下载框变为可点击的时间，默认1000毫秒
* network.dns.disableIPv6
* security.fileuri.strict_origin_policy：file协议的URI只允许读取同一目录下的文件，默认开启
* accessibility.typeaheadfind.manual：快速查找，按`/`和`'`时出现，后者仅链接，默认开启
* privacy.resistFingerprinting：抵抗数字指纹，不过开启后可能产生问题，默认关闭
* browser.backspace_action：设为2则退格键不会导致网页后退，现在默认就用它了
* javascript.options.warp true
* browser.startup.homepage.abouthome_cache.enabled
* browser.startup.preXulSkeletonUI 好像86默认开启
* dom.webgpu.enabled true
* network.trr.exclude-etc-hosts 默认为true，即DoH时忽略hosts
* network.preload：允许`link rel="preload"`
* browser.urlbar.suggest.engines：false
* browser.proton.enabled; browser.proton.contextmenus.enabled
* browser.newtabpage.activity-stream.improvesearch.handoffToAwesomebar：false 在新标签页的搜索引擎栏不要直接输到标签栏里
* security.tls.enable_0rtt_data
* browser.preferences.moreFromMozilla 设置中对手机端的推广
* ~~extensions.unifiedExtensions.enabled~~ false去掉工具栏上的“扩展”按钮，109版本新增，116去掉了
* network.cookie.sameSite.laxByDefault：true
* media.wmf.hevc.enabled=1 启用HEVC硬件解码支持，至少要120beta

#### 页面翻译

* https://zhuanlan.zhihu.com/p/24180927
* https://bbs.kafan.cn/thread-2165299-1-1.html
* https://wiki.mozilla.org/QA/Translation

### DNS over HTTPS

* network.trr.mode：3为强制，2为允许fallback
* network.trr.uri：https://1.1.1.1/dns-query 或 https://mozilla.cloudflare-dns.com/dns-query 或 https://rubyfish.cn/dns-query
* network.trr.bootstrapAddress：1.1.1.1 避免上一条被劫持
* ttr的reference：https://bagder.github.io/TRRprefs/ https://wiki.mozilla.org/Trusted_Recursive_Resolver
* 检测ECH：https://defo.ie/ech-check.php https://tls-ech.dev/ https://www.cloudflare-cn.com/ssl/encrypted-sni dig HTTPS类型，结果里有ech的

### 图标

#### 许可

* https://www.mozilla.org/en-US/foundation/trademarks/policy/
* https://www.mozilla.org/en-US/foundation/trademarks/faq/

#### 来源

* 我的头像：http://www.tokkoro.com/2811634-mozilla-firefox-space-abstract.html ，原作者未知
* quantum版：https://design.firefox.com/photon/visuals/product-identity-assets.html、https://github.com/FirefoxUX/product-identity ；第一个网址里也有许可信息
* https://wiki.mozilla.org/Category:Logos ；但里面没有quantum和新的
* 新quantum版：https://commons.wikimedia.org/wiki/File:Firefox_logo,_2019.svg?uselang=zh

### 更改语言

1. https://support.mozilla.org/zh-CN/kb/使用语言包改变Firefox界面语言
2. https://addons.mozilla.org/zh-CN/firefox/language-tools/
3. about:config：右键，新建，字符串。`intl.locale.requested: en-US`

### 编辑右键菜单

* https://www.reddit.com/r/firefox/comments/7lsqn2/is_it_possible_to_remove_some_options_from_right/
* https://www.reddit.com/r/firefox/comments/7dvtw0/guide_how_to_edit_your_context_menu/
* https://github.com/stonecrusher/simpleMenuWizard
* https://www.youtube.com/watch?v=J0NDzs9PsIg

### 扩展（Add-ons）

* https://github.com/Noitidart/Chrome-Store-Foxified：让FF使用Chrome的扩展商店
* History Master：统计历史访问的网页的数据
* 云盘万能钥匙
* https://github.com/lqzhgood/wechat-need-web 让微信网页版可用

### 主题

* Quantum：https://addons.mozilla.org/zh-CN/firefox/addon/quantum-launch/
* Rainbow Blur：https://addons.mozilla.org/zh-CN/firefox/addon/rainbow-blur-1

### 在容器中运行，网页端访问

* https://github.com/jlesage/docker-firefox
* https://hub.docker.com/r/linuxserver/firefox

### 不使用扩展禁止访问URL

* https://mozilla.github.io/policy-templates/#websitefilter 查看：about:policies。但实测只能禁止直接访问

## Chrome

### 安装包

* https://static.centbrowser.cn/win_stable/
* 适用于老系统的：https://github.com/win32ss/supermium
* 便携版：https://github.com/henrypp/chrlauncher
* 无谷歌服务版：https://ungoogled-software.github.io/ungoogled-chromium-binaries/
* https://github.com/RobRich999/Chromium_Clang https://thorium.rocks/
* 禁止自动更新：C:\Program Files\Google\Update文件夹 改名，但不会阻止下载新版安装包

### 快捷键

* Ctrl+Shift+A：搜索标签页，包括当前已打开的和历史的
* Ctrl+M：在当前窗口新建隐私页

### about:flags

* 已知Edge如果开了启动增强，命令行参数只会在第一次点的时候生效
* 没有方法找出Default对应哪个选项
* Experimental JavaScript、Future V8 VM features
* GPU（chrome://gpu/）
  * Skia Graphite
  * Choose ANGLE graphics backend：默认D3D11。可用--use-angle=d3d11on12，但有个direct_composition_support报错一个特性不支持，内存占用明显增加。有个D3D11WARP是CPU软件渲染不要用
  * Trees in viz：Edge在flags里无。--enable-features=TreesInViz
  * Zero-copy rasterizer：当前状态看Tile Update Mode
  * Enable Zero-Copy Video Capture
  * GPU rasterization：默认已部分有效，手动启用后在所有页面上生效
  * 查看当前用于渲染的GPU：GL_RENDERER。测试：https://webglreport.com/?v=2
* 安全
  * 关闭“您使用的是不受支持的命令行标记”：--test-type，其实有别的用处
  * Disable site isolation (--disable-site-isolation-trials) 有人说会导致CF挑战不过。查看是否生效：chrome://process-internals
  * 单进程 --single-process 必须启用skia或--disable-gpu才能正常显示，否则老版法使用右键和菜单，新版整个无法显示全白。一种也许受支持的类似方式：--renderer-process-limit=N
  * --no-sandbox
* Parallel downloading
* Smooth Scrolling
* Experimental QUIC protocol：考虑禁用
* Auto picture in picture for video playback：考虑禁用
* 启用MV2：--disable-features=ManifestV2Unsupported,ManifestV2Disabled
* Edge：Fluent overlay scrollbars、Experimental Web Platform features（默认显式禁用了）、New PDF Viewer
* Ungoogled：No default browser check、Hide tab close buttons、Tab Hover Cards、Show avatar/people/profile button、SetIpv6ProbeFalse、Hide Fullscreen Exit UI、Hide Extensions Menu
* https://github.com/GoogleChrome/chrome-launcher/blob/main/docs/chrome-flags-for-tools.md https://peter.sh/experiments/chromium-command-line-switches/

### 忽略HSTS证书错误

之前是badidea，现在变成了thisisunsafe

### 书签

* 没有保存历史版本，仅本地保存了上一个版本，在`User Data\Default\Bookmarks.bak`中

### 其他

* chrome://net-export/ chrome://net-internals

## PAC

* `function FindProxyForURL(url, host)`：其中url现在不包含协议和查询参数。返回一个字符串，如果为空或者为`DIRECT`就直连，`PROXY host:port`就使用HTTP代理，改为SOCKS就是SOCKS代理。支持用分号指定Fallback
* 扩展名为pac，MIME为`application/x-ns-proxy-autoconfig`

## Edge

* Enable history accelerator to open the full page：Ctrl+H显示历史页而非浮窗。现在好像没了
* 设置 - 侧栏(sidebar) - 始终显示边栏。下面的 应用和通知设置 - 特定于应用的设置 - Discover - 显示必应聊天

### 禁止Edge自动更新

```
C:\Program Files (x86)\Microsoft\EdgeUpdate 权限全部禁止

无用的：
https://msedgeblockertoolkit.blob.core.windows.net/blockertoolkit/MicrosoftEdgeChromiumBlockerToolkit.exe
下载后运行，指定解压路径。管理员权限运行EdgeChromium_Blocker.cmd /B。/U恢复。实际上就是执行`REG ADD HKLM\SOFTWARE\Microsoft\EdgeUpdate /v DoNotUpdateToEdgeWithChromium /t REG_DWORD /d 1 /f`

加UpdateDefault为0，在edge://policy/里能看到。2表示允许手动更新，0表示禁用。

删除MicrosoftEdgeUpdate.exe

未测试：
加防火墙规则
```

### 卸载Edge

```
C:\Program Files (x86)\Microsoft\
有Edge、EdgeCore、EdgeWebview三个文件夹，已经使用了硬链接优化。

Application\版本号\Installer
setup.exe --uninstall --system-level --force-uninstall

setup.exe --uninstall --msedgewebview --system-level --force-uninstall

卸载上两者后EdgeCore会自动消失。但EdgeUpdate还在，要手动删。System32里还有一个Microsoft-Edge-WebView
```

### HEVC扩展

* ms-windows-store://pdp/?ProductId=9N4WGH0Z6VHQ
* https://bbs.pcbeta.com/forum.php?mod=redirect&goto=findpost&ptid=2053927&pid=57246099 https://zhuanlan.zhihu.com/p/1972401403990381624
* 实测对于媒体播放器，装制造商的这个扩展无法生效。对于浏览器，好像Chrome现在已经支持了，Edge好像在gpu里也能看搜到，即使没装扩展
