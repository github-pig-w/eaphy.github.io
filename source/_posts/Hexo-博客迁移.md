---
title: Hexo 博客迁移
tags:
  - hexo
categories:
  - 网站运维
abstract: <font color=red>ʅ（´◔౪◔）ʃ &nbsp&nbsp&nbsp  文章已加密，有缘自会相见 ！！！</font>
message: 执意要看，请输入暗号 ！
date: 2022-09-12 20:33:55
password:
---

最近新入手 Macbook Pro，使用了6年的 windows 电脑将逐渐被替换，因此需要将老电脑中的Hexo博客迁移至新电脑。由于习惯了目前的Next主题样式，不想更换到最新版，特需要将现在的所有环境版本同步到我的 Parallels Desktop 中的windows11中。



<!-- more -->



**1、安装 node.js 12.0.0 版本**

~~~bash
https://nodejs.org/download/release/v12.0.0/
~~~

**2、安装 git，配置环境变量，设置邮箱用户名**

~~~bash
git config --global user.name "eaphy"

git config --global user.email "up_eaphy@foxmail.com"
~~~

**3、安装 hexo-cli  2.0.0 版本**

~~~bash
npm install hexo-cli@2.0.0 -g
~~~

**4、初始化博客根目录文件夹**

~~~bash
hexo init MyBlog
~~~

**5、将已打包好的老文件夹里面所有文件粘贴到新文件夹，除git相关的文件夹除外**

**6、逐一执行清理、打包运行、推送等脚本**

