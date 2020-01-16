---
title: 油猴
---

## [MetaData](https://wiki.greasespot.net/Metadata_Block)

```js
// ==UserScript==
// @name:XX-YY   Hello World // 冒号后是多国语言代码，可不加
// @namespace    用来区分name相同但作者不同的脚本，一般是一个网页链接
// @version      0.1
// @description:XX-YY  描述
// @author       You
// @grant        none // https://wiki.greasespot.net/Greasemonkey_Manual:API
// @include      * // 不加则默认就是*
// @exclude      http://diveintogreasemonkey.org/*
// @match        与include类似，但对*更严格，见http://open.chrome.360.cn/extension_dev/match_patterns.html
// @require      https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js
// 只能是白名单里的网站：https://greasyfork.org/zh-CN/help/external-scripts
// @license      MIT
// @icon         http://www.example.net/icon.png
// @run-at       默认是document-end即在html加载后、其它资源加载前执行；可以改为document-start或document-idle（全部加载完后）
// @resource     resourceName url
// 其它：https://greasyfork.org/zh-CN/help/meta-keys
// ==/UserScript==
```

## 主体

```js
// 其实已经可以直接写js了，不过如果要用strict：
!function(){
    'use strict';
    ....
}();
// 也可以用(function(){...})()或者(()=>{...})()或者(async ()=>{...})()
// 一般都会放到立即执行函数里，以防止和外部的产生命名冲突
```

## 延迟调用函数

* 不能直接用`setTimeout("helloworld()", 60);`，因为GM脚本执行完后helloworld就没了，会报未定义
* 可以在定义时把函数手动附到window上：`window.helloworld = function() { ... }`？
* 也可以用匿名函数闭包

## unsafeWindow

* 其实就是访问window。要尽量避免以防止滥用
* 可以不用它完成的一些操作：https://wiki.greasespot.net/Category:Coding_Tips:Interacting_With_The_Page
* 如果实在要使用，用`// @grant unsafeWindow`

## 变化

* 4.0后完全用WebExtension重写了，所有的GM_开头的都是老的API
* GM_log、GM_addStyle都没了。但有人以js脚本的形式提供了：`@require https://gist.githubusercontent.com/arantius/3123124/raw/grant-none-shim.js`
* 储存单域名的数据可以用Web Storage API

## 其它（编程技巧）

* jQuery非冲突模式：this.$ = this.jQuery = jQuery.noConflict(true);
* https://wiki.greasespot.net/XPath_Helper
* https://wiki.greasespot.net/Conditional_Logging
* https://wiki.greasespot.net/CSS_Independent_Content
* https://wiki.greasespot.net/Capturing_and_Bubbling

## 参考

* https://cologler.gitbooks.io/greasemonkey/content/
* https://www.52pojie.cn/thread-614101-1-1.html
* https://wiki.greasespot.net
* https://jixunmoe.github.io/gmDevBook/#/doc/intro/gmScript
