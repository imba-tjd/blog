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
// @license      MIT
// @icon         http://www.example.com/icon.png

// @grant        使用GM API时需要加，可多次声明
// @include      不加则默认是*。会从头开始匹配URL，一般写 *://xxx.com/*，且对末尾不带/的URL也有效。也支持JS的正则。协议其实只支持http和https
// @exclude      优先于include
// @match        与include类似，但对*更严格，见https://open.chrome.360.cn/extension_dev/match_patterns.html
// @require      https://xxx.com/xxx.min.js
// @run-at       默认document-end即在html加载后、其它资源加载前执行；可以改为document-start或document-idle全部加载完后
// @resource     resourceName url/xxx_string
// ==/UserScript==
```

## 主体

* 后缀名为`.user.js`
* 一般会全放到立即执行函数`!function(){...}();`里，但其实早就沙盒化了，顶层是个sandbox对象，不会命名冲突
* 支持'use strict'
* 不清楚是否能顶层await，还是说与浏览器有关。如果需要，可考虑`(async ()=>{...})()`
* setTimeout(f, ms)：第一个参数可用函数对象或匿名函数，但不能是`'f()'`，因为GM脚本执行完后整个环境就消除了，会报未定义
* 可以直接访问和修改document，但默认不能访问window
* 插入的元素受那个页面的CSS影响，如有必要可用iframe

## [GM API](https://wiki.greasespot.net/Greasemonkey_Manual:API)

* 大部分都是promise，也可以await
* GM.info：metadata对象
* GM.setValue getValue listValue deleteValue：跨page和origin储存数据
* GM.getResourceUrl：根据resourceName获得其内容string，没有其他处理
* GM.notification
* GM.openInTab
* GM.registerMenuCommand：4.11新版功能
* GM.setClipboard
* GM.xmlHttpRequest：能跨域，不遵守same-origin
* unsafeWindow：对应BOM的window对象

## 其他API

* 如果只需要储存单域名的数据，可用Web Storage API
* 4.0后完全用WebExtension重写了，所有的GM_开头的都是老的API，新的是`GM.`。GM_log、GM_addStyle没有新的，不过前者console.log就好，后者用其他手段吧。有个gist重新引入但也没必要。好像就与TamperMonkey不兼容了
* https://github.com/sizzlemctwizzle/GM_config 给用户提供图形化设置选项，非官方
* window.focus()：能让浏览器转到运行脚本的页面，不需要grant。普通JS环境调用它无效因为需要service worker权限
* $x()：XPath Helper

## Greasy Fork

* 不能minify
* require的只能是白名单里的网站：https://greasyfork.org/zh-CN/help/external-scripts

## 其他

* jq有可能与页面自带的冲突，考虑用`this.$ = this.jQuery = jQuery.noConflict(true)`
* https://violentmonkey.github.io/guide/using-modern-syntax/ 介绍了现代化的开发方式，提供了简单的JSX Runtime
* 其他脚本站：https://openuserjs.org/ https://www.userscript.zone https://userscripts-mirror.org/ 好像原站挂了
* User Style：https://userstyles.org/ https://github.com/openstyles/stylus

## 参考

* https://wiki.greasespot.net
* TODO: https://github.com/FirefoxBar/userscript 学习其他人是怎么写的
