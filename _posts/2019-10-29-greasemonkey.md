---
title: 油猴
---

## [MetaData](https://wiki.greasespot.net/Metadata_Block)

```js
// ==UserScript==
// @name         Hello World
// @name:zh      你好（多语言信息）
// @namespace    用于区分name相同但作者不同，一般是一个网址
// @version      0.1
// @description  描述，也支持多语言
// @author       You
// @homepageURL  http://example.com
// @icon         http://www.example.com/favicon.ico
// @license      MIT 原生无此项

// @grant        使用GM API时需要加，可多次声明
// @include      不加则默认是*。推荐改用match，除非需要用它支持JS的正则
// @exclude      优先于include
// @match        一般写 *://*.xxx.com/path/*，对xxx.com/path也有效。https://open.chrome.360.cn/extension_dev/match_patterns.html
// @require      https://xxx.com/xxx.min.js
// @run-at       默认document-end即在html加载后、其它资源加载前执行。还可以是document-start或document-idle全部加载完，但只有end是保证可靠的，idle设计上当外部资源长时间加载时会在end处加载脚本。若确实在idle时触发，则不应使用window.onload，因为它已经触发过了
// @resource     resourceName url 在安装时下载一次到本地，之后通过API使用它不再需要下载
// @noframes     无参，默认不启用，启用后在iframe里不会执行
// ==/UserScript==
```

## 主体

* 后缀名为`.user.js`
* 一般会全放到立即执行函数`!function(){...}();`里，但其实早就沙盒化了，顶层是个sandbox对象，不会命名冲突
* 支持'use strict'
* 不支持top level await，可用`(async ()=>{...})()`
* 等待：setTimeout(f, ms)：第一个参数可用函数对象或匿名函数，但不能是 'f()'，因为GM脚本执行完后整个环境就消除了，会报未定义。也不能闭包DOM对象的引用，对象还在，但不是真实DOM树里的了
* 可以直接访问和修改document，但默认不能访问window
* 插入的元素受那个页面的CSS影响，如有必要可用iframe

## [GM API](https://wiki.greasespot.net/Greasemonkey_Manual:API)

* 大部分都是promise，也可以await，不能直接使用
* GM.info：metadata对象
* GM.setValue getValue listValue deleteValue：跨page和origin储存数据
* GM.getResourceUrl(resourceName)：获得缓存的对象的URL供AJAX使用
* GM.notification
* GM.openInTab
* GM.registerMenuCommand
* GM.setClipboard
* GM.xmlHttpRequest：能跨域，不遵守same-origin
* unsafeWindow：对应BOM的window对象

## 其他API

* 如果只需要储存单域名的数据，可用Web Storage API
* 4.0后完全用WebExtension重写了，所有的GM_开头的都是老的API，新的是`GM.`，不清楚是否与TamperMonkey兼容。GM_log GM_addStyle GM_getResourceText没有新的。有个gist重新引入，且无需grant，因为那些都是用原生API就能实现的
* https://github.com/sizzlemctwizzle/GM_config 给用户提供图形化设置选项，非官方
* window.focus()：能让浏览器转到运行脚本的页面，不需要grant。普通JS环境调用它无效因为需要service worker权限
* $x()：XPath Helper

## Greasy Fork

* 不允许minify
* require的只能是白名单里的网站：https://greasyfork.org/zh-CN/help/external-scripts
* 实际上访问任意以 .user.js 结尾的非text/html链接就行。如果文件名不以它结尾，可以加 #.user.js

## 其他

* https://violentmonkey.github.io/guide/using-modern-syntax/ 介绍了现代化的开发方式，提供了简单的JSX Runtime
* 其他脚本站：https://openuserjs.org/ https://www.userscript.zone https://userscripts-mirror.org/ 好像原站挂了
* User Style：https://userstyles.org/ https://github.com/openstyles/stylus https://stylebot.dev/
* 老版本Chrome内置了一部分userscript的支持，但后来越来越严格，尤其是原版Chrome。实测Cent可以添加

## 参考

* https://wiki.greasespot.net 中文参考：https://github.com/examplecode/tampermonkey-api-reference
* TODO: 学习其他人是怎么写的 https://github.com/FirefoxBar/userscript https://github.com/hoothin/UserScripts https://github.com/mengzonefire/rapid-upload-userscript https://github.com/Tsuk1ko/userscript https://www.youxiaohou.com/
* violentmonkey现在仍在更新

# 浏览器扩展

* https://open.chrome.360.cn/extension_dev/overview.html
* https://developer.chrome.com/docs/extensions/
* 如何从零开始写一个 Chrome 扩展？ https://www.zhihu.com/question/20179805
* https://developer.mozilla.org/zh-CN/docs/Mozilla/Add-ons/WebExtensions
* https://dev.to/scleriot/build-a-firefox-extension-step-by-step-5dbl
