---
title: 程序集
---

> 文章内容来自于《Illustrated C\# 2012 (4th Edition)》

程序集结构
----------

### 清单

* 程序集名称标识符
* 组成程序集的文件列表
* 一个指示程序集中内容在哪里的地图
* 关于引用的其他程序集的信息

### 类型元数据

* 包含该程序集中定义的所有类型的信息

### CLI代码

* 公共中间语言代码

### 资源

* 可选。包括图形和语言资源

模块
----

* 程序集代码文件称为模块
* 对于有多个模块的程序集，一个文件是主模块（primary module），其他的是次要模块（secondary modules）
* 主模块含有程序集的清单和次要模块的引用
* 次要模块的文件名以扩展名.netmodule结尾
* 多文件程序被视为一个单一模块。它们一起部署并一起定版

程序集标识符
------------

程序集的标识符有4个组成部分，它们一起唯一标识了该程序集：

* 简单名：不带文件扩展名的文件名，也被成为程序集名或友好名称
* 版本号：由4个句点分开的整数字符串组成，形式为MajorVersion.MinorVersion.Build.Revison
* 文化信息：由2\~5个字符组成的字符串，代表一种语言和一个国家或地区
* 公钥：128字节字符串

完全限定名称：由简单名、版本、文化和表示为16字节公钥凭据的公钥组成的列表文本

ToRead：https://www.red-gate.com/simple-talk/dotnet/.net-framework/partitioning-your-code-base-through-.net-assemblies-and-visual-studio-projects/


