---
title: 搭建个人网站 
key: 20200322
tags: website
---

## 一、搭建网站的考虑因素

* 不希望有数据库
* 希望能跟github集成
* 速度快
* Simple
* 存放个人作品（文章、诗词、照片等）和相关资料

<!--more-->

## 二、选型 [Jekyll](https://jekyllrb.com) VS [HUGO](https://gohugo.io)

有一篇对比的文章值得一读。我做了一个实验，600篇md文章用jekyll编译大约是67.4秒。HUGO确实要快不少，大约8秒。但是HUGO的主题我选了好久选不好，想想一分多钟的编译速度也还能接受，于是就选择了Jekyll。HUGO最吸引我的地方是它自带的搜索功能，后面还是值得再探索一下HUGO的。

## 三、选择域名服务

我这里选择的是[godaddy](https://godaddy.com)。也有其他的域名服务商，最大的好处是不用域名备案。

## 四、选择云服务器

云服务器提供商也是大大小小，各具特色。我选择的是[linode](https://www.linode.com)。

### 基础设施配置

* Ubuntu 18.04 LTS
* Nanode 1GB
* 1 CPU
* 25GB Storage
* 1GB RAM

### 每个月5美元

## 五、选择Theme

[Jekyll](https://jekyllrb.com)和[HUGO](https://gohugo.io)都有很多的Theme可以选择。但是选择多了，也是负担。我足足花了两天时间颠来复去的看[Jekyll的Theme](http://jekyllthemes.org/)和[HUGO的Theme](https://themes.gohugo.io/)。最终选定的是[NexT](https://github.com/iissnan/hexo-theme-next) ，这其实是从另外一个静态站点生成引擎[Hexo](https://hexo.io/)移植而来的 [Jekyll](https://jekyllrb.com) 主题。

## 六、建立部署流程

## 七、网站相关信息设计

* 网站名
* 网站描述
* 网站图标（favicon）
* 作者头像
* 作者简介
* 版权保护信息

## 八、搭建步骤

### 8.1 关联域名和云服务器

这步很简单，其实就是在域名提供商那边增加一个A记录。

![A Record]({{ site.baseurl }}/images/2020-03-22-A-Record.png)

### 8.2 云服务器上jekyll环境的搭建

1. jekyll环境需要 Ruby，RubyGems，NodeJS，Python。需要根据自己的OS，搭建相应的环境。参见jekyll的[官方文档](http://jekyllcn.com/docs/installation/)。
1. Theme也需要安装依赖，具体可以参照[安装 NexT](http://theme-next.simpleyyt.com/getting-started.html) 

```shell
$ sudo bundle install
```

### 8.3 云服务器上的Web服务器搭建

不管是[Jekyll](https://jekyllrb.com)还是[HUGO](https://gohugo.io)，生成的静态网页都最好由Web服务器来Serve。一般而言，就两种选择：[Apache](https://httpd.apache.org/) 或者 [Nginx](https://nginx.org/en/)。在ubuntu上都是可以apt-get一个命令搞定。我选择的是 [Nginx](https://nginx.org/en/)。在开发本地，不需要安装web服务器，因为Jekyll安装就绪以后就已经可以通过以下命令来查看站点的本地效果。

```shell
$ sudo bundle exec jekyll server
```

在ubuntu服务器下，安装好[Nginx](https://nginx.org/en/)后，[Nginx](https://nginx.org/en/)自动运行。此时访问服务器的IP地址或者你的站点（域名已经起作用了的话），就应该可以看到default的index.html。

### 8.4 云服务器上的Web服务器配置

[Nginx](https://nginx.org/en/)的配置文件有两类

#### 8.4.1 nginx配置文件 /etc/nginx/nginx.conf

将默认的注释去掉来enable以下的gzip配置

```shell
gzip on;
gzip_disable "msie6";

gzip_comp_level 6;
gzip_min_length 1100;
gzip_buffers 16 8k;
gzip_proxied any;
gzip_types
        text/plain
        text/css
        text/js
        text/xml
        text/javascript
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/rss+xml
        image/svg+xml;
```

#### 8.4.2 站点配置文件 \<sitename\>.conf

首先在  `/etc/nginx/sites-available/` 目录下创建这个站点的配置文件

```shell
$ sudo vi /etc/nginx/sites-available/wangqiyi.me
```

主要的配置项就是：

* listen
* server_name
* root
* location

```shell
server {
    listen 80;
    server_name wangqiyi.me www.wangqiyi.me;
    root /var/www/html;

    location / {
            try_files $uri $uri/ /index.html;
    }
}

```

将站点配置文件建立链接到`/etc/nginx/sites-enabled/`

```shell
$ sudo ln -s /etc/nginx/sites-available/wangqiyi.me /etc/nginx/sites-enabled/wangqiyi.me
```

重启Nginx服务
```shell
$ service nginx restart
```

### 8.5 https配置

### 8.6 git post-receive hook

部署流程中，可以采用的一种方式是在租用的云服务器上建立一个cron job。定期去github上拉一下内容，如果有新的内容那么就编译，然后部署（copy）到web服务器中。但是这次我尝试了一下另外一种方式：建立一个git post-receive hook。关于这个hook的解释，可以在[git官网](https://git-scm.com/book/zh/v2/自定义-Git-Git-钩子)中找到。简单来说，就是在云服务器上建立一个git仓库，并且在该仓库中建立一个hook。这个hook实际上是个shell脚本，它可以执行你定义的操作。触发器是你在本地的每一次push。

用具体步骤来实例说明一下。

> 第1步、在云服务器（linode server）上，新建一个bare git仓库。

```shell
$ mkdir deploy.wangqiyi.me
$ cd deploy.wangqiyi.me
$ git init --bare
```

*deploy.wangqiyi.me*是一个空的git仓库，该文件夹里面有个hooks文件夹。

> 第2步、进入hooks目录，建立post-receive钩子

```shell
#!/bin/bash
GIT_REPO=$HOME/deploy.wangqiyi.me
TMP_GIT_CLONE=$HOME/tmp/wangqiyi.me
PUBLIC_WWW=/var/www/html

git clone $GIT_REPO $TMP_GIT_CLONE
cd $TMP_GIT_CLONE
# 最关键的是下面这个命令。执行jekyll编译，并将编译后的静态网页输出到web server伺服的网站文件夹
JEKYLL_ENV=production bundle exec jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit
```

*赋予post-receive钩子可执行权限*, 否则钩子不会执行

```shell
$ chmod 755 post-receive
```

> 第3步、 设定本地git中的remote分支，指向Linode Server

```shell
$ git remote add deploy root@<Linode Server IP>:deploy.wangqiyi.me
```

这样就可以直接push到deploy分支上了。
```shell
$ git push deploy master
```

> 第4步、 还可以通过git remote set-url设定origin既指向github又指向Linode Server。这样做的好处是每次我push到origin的时候，实际上我push到了两个服务器。

```shell
$ git remote set-url --add --push origin https://github.com/wjmmx/wangqiyi.git
$ git remote set-url --add --push origin root@139.162.12.156:deploy.wangqiyi.me
```

以下参考文章，值得一读：

* [webhooks](https://sofiya.io/blog/webhooks)
* [Jekyll官网部署页面](http://jekyllcn.com/docs/deployment-methods/)

## 九、发布文章

发布文章挺简单。只要在站点的*_post*目录下面创建文章即可。值得注意的有两点：

1. 文章的文件名需要是“YYYY-MM-DD-Title.md”格式 
2. 文章的图片在发布到站点的时候要用站点的变量 "{{ site.url }}"来指定路径。比如，我把图片放到 *assets/images/posts/YYYY/MM/*目录下，文章中的引用就是

```shell
{{ site.url }}/assets/images/posts/2019/11/2019-11-13.01.jpg
``` 

同样，参考[Jekyll官网的文章发布页面](http://jekyllcn.com/docs/posts/)