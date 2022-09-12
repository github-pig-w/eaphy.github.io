---
title: Hexo 实用插件
tags:
  - hexo
categories:
  - 网站运维
date: 2019-06-29 15:30:41
---

# 搜索插件
进入博客根目录，从cmd输入以下命令

```bash
npm install hexo-generator-searchdb --save
```



修改 站点配置 文件，注意 search.xml 的路径
<!-- more -->
```bash
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
修改 主题配置 文件，搜索 local_search

```bash
local_search:
  enable: true
```
# GIT 插件
进入博客根目录，从cmd输入以下命令

```bash
npm install hexo-deployer-git --save
```


修改 站点配置 文件

```
deploy:
  type: git
  repo: https://github.com/eaphy/eaphy.github.io.git
  branch: master
```

# 加密插件

进入博客根目录，从cmd输入以下命令

```bash
npm install --save hexo-blog-encrypt
```


修改 站点配置 文件

```
# Security
encrypt:
    enable: true
```
在文章头部加入：

```bash
---
title: 测试密码
tags:
categories:
date: 2019-06-29 16:54:51
password: 123456
abstract: 加密文章，点击 阅读全文，输入密码之后访问全文 ！
message: 请输入密码，回车查看文章内容 ！
---
```

# 图片懒加载插件
进入博客根目录，从cmd输入以下命令

```bash
npm install hexo-lazyload-image --save
```



修改 站点配置 文件，添加以下内容
```bash
lazyload:
  enable: true
  onlypost: false
  loadingImg: /images/loading.png
```

# 文章置顶插件
进入博客根目录，从cmd输入以下命令

```bash
$ npm uninstall hexo-generator-index --save
$ npm install hexo-generator-index-pin-top --save
```

修改 `/themes/next/layout/_macro/post.swig`文件，搜索 `<div class="post-meta">` ，在其下一行添加以下内容：

```bash
{% if post.top  %}
     <i class="fa fa-thumb-tack"></i>
     <font color=696969>置顶</font>
     <span class="post-meta-divider">|</span>
{% endif %}
```
在文章头部加入：`top: true`

```bash
---
title: 测试置顶
tags:
categories:
top: true
date: 2019-06-29 16:54:51
---
```