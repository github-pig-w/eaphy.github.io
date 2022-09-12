---
title: Hexo+Github+Next 搭建个人博客
date: 2019-06-26 22:53:51
tags: 
- hexo
- github
- next
categories:
- 网站运维
---
**1. 在 github 建立 repository**
**2. 安装 [node.js](https://nodejs.org/en/) 和 [git](https://git-scm.com/)**
**3. 安装 hexo**

从G盘输入cmd，回车，输入以下命令
<!-- more -->
```bash
npm install hexo-cli -g
hexo init Eaphy's-Blog //文件夹名字
cd Eaphy's-Blog
npm install
```
**4. 安装 Next**

```
cd Eaphy's-Blog
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

**5. 配置博客**

编辑站点中 _config.yml ,启用 next 主题，切换中文语言

```bash
theme: next
```

```
language: zh-CN
```

编辑主题中 _config.yml ,选择  Pisces Scheme

```bash
#scheme: Muse
#scheme: Mist
scheme: Pisces
```


**6.拉取仓库代码到本地**


```bash
# 拉取代码

cd Eaphy's-Blog

git clone https://github.com/eaphy/eaphy.github.io.git ./public

hexo g

# 提交代码

cd public 

git add .

git commit -m "first commit"

git push

```
