---
layout: post
title: 'Mac下制作Dash中文PHP文档'
date: 2021-06-08
author: ken
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: 效率工具
---

> 使用dashing工具快速制作中文PHP文档

### 安装dashing制作工具
    brew install dashing

### 下载PHP中文文档
打开 https://www.php.net/download-docs.php

选择 Chinese (Simplified)-->Many HTML files

附php中文文档直接下载地址：

https://www.php.net/distributions/manual/php_manual_zh.tar.gz

### 解压
    tar -zxvf php_manual_zh.tar.gz

### 进入目录
    cd php_manual_zh

### 生成dashing.json文件
    dashing create

### 修改dashing.json文件
```
{
    "name": "php-8.0-zh",
    "package": "php8",
    "index": "index.html",
    "selectors": {
        "h1.refname": "Command",
        "h2.title": "Package"
    },
    "ignore": [
        "ABOUT"
    ],
    "icon32x32": "favicon.ico",
    "allowJS": false,
    "ExternalURL": "https://www.php.net/manual/zh/"
}
```

### dashing.json文档说明
- name: 文档名字
- package:标示文档包，在搜索的时候可以输入这个名字进行指定查找
- index:首页html
- selectors：采用css3选择器，告诉文档哪些是命令，哪些是包

我们需要对php中文文档里面的html做分析，简单分析如下：

    命令的css都是<h1 class="refname"></h1>
    
    包的css都是<h2 class="title"></h2>

当然你可以根据自己需要定义规则，就能搜索不同的内容，这里的实例基本只搜索一些常用方法。

- ignore:过滤哪些不需要的文档
- icon32x32:文档图标（注：需要将这个文件放到项目目录下）
- allowJS:是否执行js（注：一般的离线文档都不需要执行js）
- ExternalURL:文档外部地址

### 开始制作
    dashing build

会在此文件夹中生成一个php8.docset文件

### 导入文档
双击php8.docset文件，会自动打开dash。

### 备注
1. 如果selectors规则不合适，可以在dash中删除文档，然后修改dashing.json后，重新制作
2. 采用dashing生成的文档暂时没有发现什么问题
3. 配合alfred使用会非常方便

### 参考资料
https://github.com/technosophos/dashing