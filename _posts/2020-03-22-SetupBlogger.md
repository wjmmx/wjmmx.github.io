---
title: 搭建个人网站 
key: 20200322
tags: blogger, jekyll, website
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

### 8.2 服务器上的搭建

#### 8.2.1 jekyll环境

1. jekyll环境需要 Ruby，RubyGems，NodeJS，Python。需要根据自己的OS，搭建相应的环境。参见jekyll的[官方文档](http://jekyllcn.com/docs/installation/)。
1. Theme也需要安装依赖，具体可以参照[安装 NexT](http://theme-next.simpleyyt.com/getting-started.html) 

```shell
$ sudo bundle install
```

#### 8.2.2 Web服务器

不管是[Jekyll](https://jekyllrb.com)还是[HUGO](https://gohugo.io)，生成的静态网页都最好由Web服务器来Serve。一般而言，就两种选择：[Apache](https://httpd.apache.org/) 或者 [Nginx](https://nginx.org/en/)。在ubuntu上都是可以apt-get一个命令搞定。我选择的是 [Nginx](https://nginx.org/en/)。在开发本地，不需要安装web服务器，因为Jekyll安装就绪以后就已经可以通过以下命令来查看站点的本地效果。

```shell
$ sudo bundle exec jekyll server
```

在ubuntu服务器下，安装好[Nginx](https://nginx.org/en/)后，[Nginx](https://nginx.org/en/)自动运行。此时访问服务器的IP地址或者你的站点（域名已经起作用了的话），就应该可以看到default的index.html。

##### 8.2.2.1 Web服务器配置

[Nginx](https://nginx.org/en/)的配置文件有两类

###### nginx配置文件 /etc/nginx/nginx.conf

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

###### 站点配置文件 \<sitename\>.conf

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

##### 8.2.2.2 https配置

#### 8.2.3 git posthook

## 九、发布文章