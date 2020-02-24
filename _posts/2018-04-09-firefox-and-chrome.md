---
title: Firefox和Chrome
---

## Firefox

* 正在进行的实验：about:studies
* Flash Player去沙箱版：https://bbs.kafan.cn/thread-1839722-1-1.html

### about:config

* 关闭最后一个标签页不关闭浏览器：browser.tabs.closeWindowWithLastTab
* 新标签打开书签和链接：browser.tabs.loadBookmarksInTabs
* 使用MacType渲染（已经失效，cairo在69移除了）：gfx.content.azure.backends: gfx.content.azure.backends;direct2d1.1,cairo,skia；gfx.direct2d.disabled: true
* network.prefetch有一些预读的选项，但是可能有隐私问题
* browser.cache.disk.enable可以设置是否在硬盘上缓存数据，默认最大大小可能有1G。在about:cache可以看到
* media.videocontrols.picture-in-picture.enabled视频的画中画图标，除了它还要改xxx.video-toggle.enabled才能生效
* network.captive-portal-service.enabled检测网络是否需要登录
* network.http.http3.enabled：开启quic
* security.enterprise_roots.enabled：使用系统证书列表

#### 页面翻译

* https://zhuanlan.zhihu.com/p/24180927
* https://bbs.kafan.cn/thread-2165299-1-1.html
* https://wiki.mozilla.org/QA/Translation

### DNS over HTTPS

* network.trr.mode：3为强制，2为允许fallback
* network.trr.uri：https://1.1.1.1/dns-query 或 https://mozilla.cloudflare-dns.com/dns-query
* network.trr.bootstrapAddress：1.1.1.1
* network.security.esni.enabled：true开启esni
* ttr的reference：https://bagder.github.io/TRRprefs/

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

1.  https://support.mozilla.org/zh-CN/kb/使用语言包改变Firefox界面语言
2.  https://addons.mozilla.org/zh-CN/firefox/language-tools/
3.  about:config：右键，新建，字符串。`intl.locale.requested: en-US`

### 编辑右键菜单

* https://www.reddit.com/r/firefox/comments/7lsqn2/is_it_possible_to_remove_some_options_from_right/
* https://www.reddit.com/r/firefox/comments/7dvtw0/guide_how_to_edit_your_context_menu/

### 扩展（Add-ons）

* Chrome Store Foxified：让火狐运行Chrome的扩展
* Grammarly for Firefox：拼写检查
* History Master：统计历史访问的网页的数据
* Zoom Page WE：缩放网页
* 云盘万能钥匙

### 主题

* Quantum：https://addons.mozilla.org/zh-CN/firefox/addon/quantum-launch/

### 其他

* 按住alt键就可以开启强制选择模式，这样鼠标就不会发出拖拽等其他功能

## Chrome

### about:flags

* https://bbs.kafan.cn/thread-2133336-1-1.html
* Password import

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
