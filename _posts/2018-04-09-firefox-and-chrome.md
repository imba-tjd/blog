---
title: Firefox和Chrome
---

## Firefox

* 正在进行的实验：about:studies
* 网络状况：about:networking
* 搜索的高亮只会显示1000个

### 下载

* 历史版本：https://ftp.mozilla.org/pub/firefox/releases/
* 最新版本：https://download.mozilla.org/?product=firefox-latest-ssl&os=win64&lang=zh-CN
* https://download-installer.cdn.mozilla.net/pub/firefox/releases/74.0/win64/zh-CN/Firefox%20Setup%2074.0.exe （Firefox Setup 74.0.exe）

### about:config

* 关闭最后一个标签页不关闭浏览器：browser.tabs.closeWindowWithLastTab
* 新标签打开书签和链接：browser.tabs.loadBookmarksInTabs
* 使用MacType渲染（已经失效，cairo在69移除了）：gfx.content.azure.backends ...
* 搜prefetch有一些预读的选项，但是可能有隐私问题
* browser.cache.disk.enable可以设置是否在硬盘上缓存数据，大小受browser.cache.disk.smart_size.enabled自动控制，大概1G
* media.videocontrols.picture-in-picture.enabled视频的画中画图标，除了它还要改xxx.video-toggle.enabled才能生效
* network.captive-portal-service.enabled检测网络是否需要登录
* network.http.http3.enabled：开启quic
* security.enterprise_roots.enabled：使用系统证书列表
* browser.urlbar.trimURLs：地址栏隐藏http
* xpinstall.signatures.required：允许安装未签名扩展
* extensions.pocket.enabled false：禁用pocket扩展
* security.tls.version.min 3：1为TLS1.0
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
* browser.backspace_action：设为2则退格键不会导致网页后退
* javascript.options.warp true
* browser.startup.homepage.abouthome_cache.enabled
* dom.webgpu.enabled true

#### 页面翻译

* https://zhuanlan.zhihu.com/p/24180927
* https://bbs.kafan.cn/thread-2165299-1-1.html
* https://wiki.mozilla.org/QA/Translation

### DNS over HTTPS

* network.trr.mode：3为强制，2为允许fallback
* network.trr.uri：https://1.1.1.1/dns-query 或 https://mozilla.cloudflare-dns.com/dns-query 或 https://rubyfish.cn/dns-query
* network.trr.bootstrapAddress：1.1.1.1 避免上一条被劫持
* network.security.esni.enabled：true开启esni
* ttr的reference：https://bagder.github.io/TRRprefs/ https://wiki.mozilla.org/Trusted_Recursive_Resolver

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

* Chrome Store Foxified：让火狐运行Chrome的扩展
* Grammarly for Firefox：拼写检查
* History Master：统计历史访问的网页的数据
* 云盘万能钥匙

### 主题

* Quantum：https://addons.mozilla.org/zh-CN/firefox/addon/quantum-launch/
* Rainbow Blur：https://addons.mozilla.org/zh-CN/firefox/addon/rainbow-blur-1

### 其他

* 按住alt键就可以开启强制选择模式，这样鼠标就不会发出拖拽等其他功能

## Chrome

* http://static.centbrowser.cn/installer_64/

### about:flags

* https://bbs.kafan.cn/thread-2133336-1-1.html
* Password import
* Experimental JavaScript
* Future V8 VM features
* sharing-qr-code-generator

#### [overlay scrollbar](https://www.zhihu.com/question/64630817/answer/223528093)

* --enable-features=OverlayScrollbar --enable-prefer-compositing-to-lcd-text

#### www

> ```
> Chrome 78 的 flags 页面中去掉了 omnibox-ui-hide-steady-state-url 相关的选项。
> 但是经测试，可以在 chrome://flags 页面控制台执行
> [
>   'omnibox-ui-hide-steady-state-url-path-query-and-ref',
>   'omnibox-ui-hide-steady-state-url-scheme',
>   'omnibox-ui-hide-steady-state-url-trivial-subdomains'
> ].forEach(function(f) {
>   chrome.send('enableExperimentalFeature', [f + '@2', 'true']);
> })
> 来禁用这几个选项。代码中 @2 对应的是 Disabled。 @0 和 @1 分别对应 Default 和 Enabled。
> ```

然而我实测无效。

### 忽略HSTS证书错误

之前是badidea，现在变成了thisisunsafe
