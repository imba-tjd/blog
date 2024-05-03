https://gohugo.io/

## CLI

* hugo_extended版本用于处理SASS/SCSS
* hugo new site .
* git submodule add https://github.com/CaiJimmy/hugo-theme-stack.git themes/stack
* hugo new post/yourBlogName.md
* hugo 不带verb：生成站点到public下，-D处理draft，--minify
* hugo server：启动本地服务器，会自动重新构建，修改config也会，且网页会自动重载，不会生成public；static无效？

## config

```yml
baseURL: https://example.com/ # 本地运行时会忽略
languageCode: zh-cn
title: My New Hugo Site
theme: stack
paginate: 10 #【默】
DefaultContentLanguage: zh-cn # 主题的默认多语言

uglyURLs: true # 默认false，会把/filename.html变成/filename/
permalinks:
    posts: /posts/:slug # 如果content中存在posts文件夹，重新映射
    page: /:slug # 单页面如about
```

## FrontMatters

```yml
slug: xxx # 永久链接，默认等于文件名
```

## 目录结构

* content：存放md内容，new的时候自动存到这里面，第一个子目录被认为是type
* archetypes：new的模板，同时对应content中的目录
* assets：会被hugo pipe处理
* static：原样复制

## 主题

* https://themes.gohugo.io/hugo-theme-stack/ 国产，有侧边栏和分类
* https://github.com/google/docsy
* https://themes.gohugo.io/hugo-theme-terminal/ 暗色
* https://themes.gohugo.io/hugo-papermod/
* https://themes.gohugo.io/hugo-eureka/ 国产，首页有头图，有TOC
* https://themes.gohugo.io/hugo-clarity/ nav锁定，有侧边栏和分类
* https://github.com/wowchemy/wowchemy-hugo-modules


https://github.com/marketplace/actions/hugo-setup
https://github.com/marketplace/actions/hugo-to-gh-pages
https://lab.github.com/githubtraining/github-pages https://help.github.com/en/github/working-with-github-pages/about-github-pages

其他人的搭建经验：https://zhuanlan.zhihu.com/p/105021100 https://zhuanlan.zhihu.com/p/150173525 https://zhuanlan.zhihu.com/p/165271155 https://zhuanlan.zhihu.com/p/27552299 https://zhuanlan.zhihu.com/p/110137145
