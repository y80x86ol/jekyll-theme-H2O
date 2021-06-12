---
layout: post
title: 'Please make sure you have the correct access rights and the repository exists'
date: 2021-06-12
author: ken
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: git
---

> 最近在搭建composer私有库的时候，出现这个问题

### 1、首先重新在git设置一下身份的名字和邮箱

    git config --global user.name "yourname"

    git config --global user.email "your@email.com"

注：yourname是你要设置的名字，your@email是你要设置的邮箱。


### 2、删除.ssh文件夹

    Windows C:\Users\Administrator\.ssh
    
    Mac路径 /Users/你的mac用户名/.ssh

### 3、生成新的ssh key

    ssh-keygen -t rsa -C "your@email.com"（请填你设置的邮箱地址）

会出现提示，没有特殊设置，直接回车即可

然后系统会自动在.ssh文件夹下生成两个文件，id_rsa和id_rsa.pub

### 4、复制你的公钥

    打开 id_rsa.pub 文件，复制里面的内容

### 5、配置github ssh
打开 `https://github.com/` ，登陆你的账户，进入ssh设置

或者打开以下地址也行：`https://github.com/settings/keys`

点击`New SSH key`进行添加

### 6、进行验证

    ssh -T git@github.com

你会看到大概以下内容

```
> The authenticity of host 'github.com (IP ADDRESS)' can't be established.
> RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
> Are you sure you want to continue connecting (yes/no)?
```

输入`yes`，回车

### 结束

经过以上步骤就可以正常使用了

### 备注

如果你以前已经设置过了git相关信息或者生成过ssh，可以直接将ssh复制到github即可。

   ls -al ~/.ssh

### 参考文档

https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh