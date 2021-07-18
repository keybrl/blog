---
title: Nginx常用配置套路
date: 2018-07-02
updated: 2019-07-20
layout: post
categories:
- 专业相关
tags:
- Nginx
- Linux
- 教程
banner_img: https://blog-assets-1253422097.file.myqcloud.com/images/2018-07-02-nginx-configure/nginx.jpg
index_img: https://blog-assets-1253422097.file.myqcloud.com/images/2018-07-02-nginx-configure/nginx.jpg
---

> 很久没有更新这篇文章了，因为我最初写这个就是为了方便自己找，一般用到什么就写些什么，因为一段时间都很少用到 Nginx 了，所以自然也没有动力把这篇文章继续写下去。但是现在不一样了，我已经将我的博客迁移到腾讯云上，通过 Nginx 部署了。而且为了博客部署方便，我还写了一个接收 GitHub Webhook 请求的 Django 应用，也需要 Nginx 代理。所以这些 Nginx 配置经验都可以补充进来了。

我们都知道， Nginx 是一个强大的 Web 服务器。只需要简单配置，它就能承载你的 Web 应用。轻易实现诸如负载均衡、反向代理、重定向、HTTPS证书配置、静态资源托管、......这一切只需要填写几个配置文件

我第一次使用 Nginx 是用它部署我的 Python Flask 应用，具体可以看看这篇文章 [CentOS中通过Nginx和uWSGI部署Flask项目](/professional-2017-08-01-Nginx-uWSGI-Flask-in-CentOS/) 。其实后来我发现 Nginx 比我想象的还要好用得多，我经常用它重定向或者搭建静态网站什么的。但是配置文件实在是难背，每次都要借助搜索引擎的帮助，而且还不能特别容易直接找到我想要的东西。所以我不如把我常用的配置记下来吧，找自己博客总还是比瞎搜快很多的。

## 初始配置

在 Ubuntu 上， Nginx 可以通过 `apt install nginx` 安装。如果是那样，那么默认配置文件是 `/etc/nginx/nginx.conf` 。初始内容是这样的

```conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}


#mail {
#    # See sample authentication script at:
#    # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#    # auth_http localhost/auth.php;
#    # pop3_capabilities "TOP" "USER";
#    # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#    server {
#        listen     localhost:110;
#        protocol   pop3;
#        proxy      on;
#    }
#
#    server {
#        listen     localhost:143;
#        protocol   imap;
#        proxy      on;
#    }
#}

```

一般来说这个文件是不需要修改的，这个文件中有一行是 `include /etc/nginx/conf.d/*.conf;` ，意思是，它会把 `/etc/nginx/conf.d/` 下面所有 `.conf` 文件引入到 `http {}` 里面，所以一般情况下，在 `/etc/nginx/conf.d/` 下面写配置文件，一个“应用”写一个 `.conf` 文件，就可以达到配置的目的，而且方便插拔应用（加上或者移除相应的 `.conf` 文件就可以实现一个应用的插拔）。以前我没有这方面的注意，所以搞到配置文件乱七八糟，不方便维护。

还有一个值得注意的是 `/etc/nginx/nginx.conf` 中 `http {}` 内有一句 `include /etc/nginx/sites-enabled/*;` 会让你的 Nginx 在成功安装、启动后，被访问时，显示 Nginx 的欢迎页面，像下面这样

![Nginx 的欢迎页面](https://blog-assets-1253422097.file.myqcloud.com/images/2018-07-02-nginx-configure/nginx_welcome_page.png)

而且无论你设置什么别的配置，80 端口都会被这个页面占用，所以需要先加个井号注释掉。

![Nginx 配置文件注释掉某行](https://blog-assets-1253422097.file.myqcloud.com/images/2018-07-02-nginx-configure/nginx_command.png)

然后剩下的配置可以写在 `/etc/nginx/conf.d/` 里

## 静态文件服务器

最简单最基础的 Nginx 使用场景就是配置一个静态文件服务器，它可以托管你的静态网站（比如这个博客），或者为别的什么静态资源提供一个简单的可通过 http 访问的接口（比如文件下载服务）。

我们在 `/etc/nginx/conf.d/static_website.conf` 写下如下配置（当然，文件名可以是任意的，只要路径和文件名后缀一样就行）

```conf
server {
    # 服务器监听的端口号
    listen      80;
    # 服务的ip地址或者域名
    # 一般可以为 0.0.0.0 ，表示接受通过任意网卡任意域名的访问
    server_name 0.0.0.0;

    # 可选的日志文件路径设置，如果不设置则记录到 Nginx 的主日志文件中，其设置见 `/etc/nginx/nginx.conf`
    access_log /home/someone/logs/neinx_static_files.log;
    error_log  /home/someone/logs/nginx_static_files_error.log;

    # 通过从头部开始能完整匹配 '/' 的 url访问的“路由”
    location / {
        # 静态文件根目录
        root       /home/someone/static_files/;

        # 允许自动索引（比如访问 '/whatever/' 返回 '/whatever/index.html' ，一把用于支持这种路径风格的 url ）
        autoindex  on;

        # 添加响应头，指定响应首部字段 'Cache-Control' 的值为 'no-cache' 。
        # 静态资源服务器推荐这样设置，这样表示允许客户端缓存，但是向用户发送缓存之前必须先向源服务器验证缓存是否有效。
        add_header 'Cache-Control' 'no-cache';
    }
}
```

## 重定向

由于多种原因，我们会需要重定向。比如网站永久转移了，网站维护暂时引导用户访问替代页面，引导使用http的用户使用https访问等，这种情况都需要给用户返回 3xx 状态码并且指定新的 url 。这种操作我们可以使用 rewrite 语句来实现。

我们拿上面提到的静态文件服务器的配置来做个示例，可以参考上面的例子来理解下面这段配置

```conf
server {
    listen      80;
    server_name 0.0.0.0;

    location / {
        root       /home/someone/static_files/;
        autoindex  on;
        add_header 'Cache-Control' 'no-cache';

        rewrite ^/feed.xml$ https://$host/atom.xml permanent;
    }
}
```

上面这个例子其实是源自我自己的真实需求的，你可以试着访问 `https://blog.keybrl.com/feed.xml` ，按一下 F12 调出浏览器开发者工具看看这个请求收到了什么响应。你会发现收到了一个重定向到 `https://blog.keybrl.com/atom.xml` 的 301 状态响应。这表明所访问的资源已经永久转移到所重定向的 url 。这除了可以引导用户代理访问新的 url ，这还能更正某些网络机器人、搜索引擎的过期认识。我以前 feed 的 url 就是 `/feed.xml` 如果这个 feed 被人订阅过，那么信息聚合软件会定期来访问这个 url ，这个资源的 url 修改后，我肯定不希望逐一通知我的订阅者修改这个 url ，所以设定一个 301 重定向就可以悄无声息地完成这个变更。

一个 rewrite 语句包含 4 个部分 `rewrite关键字 原url正则 重定向的url flag` 。

其中， `原url正则` ，是一个正则语句，用于匹配需要重定向的 url 并且从中提取匹配的部分，它只处理 url 中路径的部分。比如说 `https://blog.keybrl.com/whatever/hhh.html?arg1=hhh&arg2=value` 这样一个 url ，会被正则引擎处理的部分仅仅是 `/whatever/hhh.html` ，对于 url 中协议、主机名、端口号、请求参数的处理不应在这个语句中实现。

正则的基本原理和相关概念我就不作更多说明了，不熟悉的同学可以求助搜索引擎。我仅仅介绍一下在这个正则表达式中提取出来的 “捕获组” ，可以在语句中通过 `$ + 捕获组序号` 使用。比如下面这个语句

```conf
rewrite ^/a/([^/]*)/b/([^/]*)/$ https://$host/b/$2/a/$1/ permanent;
```

这个语句会把所有形如 `/a/123/b/asd/` 的 url 都重定向为 `/b/asd/a/123/` ，就是把 url 路径的顺序调转一下。至于作用嘛，比如说你的网站文章可以通过 category 和 tag 两个维度进行筛选，比如说要列出 post 类别的所有有 funny 标签的文章可以访问 `/category/post/tag/funny/` 。但是用户可能觉得 category 和 tag 两个维度是并列的啊，用 `/tag/funny/category/post/` 来索引好像也很正常啊，你不想让用户吃 404 也不想改代码（或者代码根本不是你写的，你也不知道去哪里改），而且你还希望引导用户以后都用 `/category/post/tag/funny/` 来访问，那么你就可以参考上面那个 rewrite 语句。（你如果觉得把这个例子中的 a b 去掉好像更方便，那你可能会进入重定向的死循环，不断交换两项直到超过浏览器重定向次数的最大限制）

`重定向的url` 就很好理解了，就是重定向的目标 url 。不过这里要填的是完整 url ，而不只是一个路径。你除了可以使用前面正则产生的捕获组，还可以使用 `$host` 、 `$args` 、 `$request_method` 、 `$server_protocol` 、 `$server_addr` 、 `$server_port` 、... 这些内置变量。

最后一个 `flag` 可以是

- `last` 本条匹配后，仍往下匹配，最终执行最后一条匹配的规则
- `break` 本条若匹配即终止，不再往后匹配
- `redirect` 返回 302 ，临时重定向
- `permanent` 返回 301 ，永久重定向

下面是一些例子

```conf
rewrite ^(.*)$ https://$host$1 permanent;  # 用在 80 端口的 server 中，重定向到 https
rewrite ^/([^/]*)/(.*)$ https://$1.sample.com/$2 permanent;  # 将形如 https://www.sample.com/cn/whatever 的 url 重定向为 https://cn.sample.com/whatever
rewrite ^(.*).html$ https://$host$1/ permanent;  # 将形如 https://www.sample.com/about.html 的文件风格的 url 重定向为形如 https://www.sample.com/about/ 的目录风格的 url
rewrite ^(.*)/([0-9]{4})/([0-9]{2})/([0-9]{2})/([^/]*)$ https://$host$1/$2-$3-$4-$5 permanent;  # 将形如 https://www.sample.com/2019/07/20/post.html 的 url 重定向为 https://www.sample.com/2019-07-20-post.html
# ......
```

## SSL 证书

9102年了，该给你的网站加个小绿锁（某些浏览器会给 https 的网站在地址栏加个小绿锁）了吧。

如何申请 SSL 证书我就不介绍了，我下面以在腾讯云申请的 SSL 证书为例来说明。比如本站，域名是 `blog.keybrl.com` ，对它申请了 SSL 证书之后，下载之。如果是在腾讯云申请的证书，下载下来是一个压缩包，解压之后得到的文件结构大概是这样的

```text
./
 |- Apache/
 |  |- 1_root_bundle.crt
 |  |- 2_blog.keybrl.com.crt
 |  |- 3_blog.keybrl.com.key
 |
 |- IIS
 |  |- blog.keybrl.com.pfx
 |
 |- Nginx
 |  |- 1_blog.keybrl.com_bundle.crt
 |  |- 2_blog.keybrl.com.key
 |
 |- Tomcat
 |  |- blog.keybrl.com.jks
 |
 |- blog.keybrl.com.csr
```

很显然，人家都给你很贴心地准备了好几种服务器所需的证书文件。在这里我们就只需要 Nginx 文件夹里面的两个文件，文件名是任意的，但是后缀是 `.crt` 和 `.key` 。

把它们上传到服务器，通过 scp 、 ftp 、 tftp 、 sftp 、... 等方法都可以，甚至服务器在你身边的话还可以用 U 盘。存储的路径也是任意的，不过我为了方便整理，就放在 `/etc/nginx/cert/` 目录下

然后在 `/etc/nginx/conf.d/` 目录下下一个配置文件，比如 `/etc/nginx/conf.d/blog.keybrl.com.conf` ，我用域名标注配置文件名，这样我比较容易知道这个配置文件是配置了什么服务。内容如下

```conf
server {
    # https 协议默认使用 443 端口，如果要使用其它端口，需要在 url 中注明端口号
    listen               443;
    # server_name 必须与证书认证的域名一致
    server_name          blog.keybrl.com;

    # 下面是 SSL 相关的设置
    ssl                  on;  # 开启 SSL
    ssl_certificate      cert/1_blog.keybrl.com_bundle.crt;  # SSL 证书文件
    ssl_certificate_key  cert/2_blog.keybrl.com.key;  # 用于 SSL 通信的私钥
    ssl_session_timeout  5m;  # SSL 会话超时时间， 5 分钟
    # SSL 证书加密方法(们)
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    # SSL 协议版本
    ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    # 类同上面的例子
    location / {
        root       /path/to/your/project;
        autoindex  on;
        add_header 'Cache-Control' 'no-cache';
    }
}

# 将 http 请求重定向为 https 请求
server {
    listen       80;
    server_name  blog.keybrl.com;
    rewrite      ^(.*)$ https://$host$1 permanent;
}
```

上面这个只是以静态文件服务器作为例子，只要是有上面例子中 `ssl` 开头的配置，这个服务就是需要通过 SSL 加密的。

## 反向代理

“代理” 很好理解，就是通过代理服务器，代理完成通信嘛。我们常用的代理一般是 “正向代理” 。就是客户端与服务端通信，但是将请求发给代理服务器，由代理服务器向服务端发送请求，并由代理服务器接收来自服务端的响应，然后再将响应返回给客户端。 “反向代理” 也是类似的，只不过被代理的是服务端。客户端以为代理服务器是服务端，向代理服务器发送请求，代理服务器替服务端接收请求，转发给服务端，服务端将响应发送给代理服务器，由代理服务器直接向客户端响应。

这样做的目的嘛，有很多，比如说有多个服务端承载同一个服务，它们由同一个反向代理服务器代理，这样在客户端看来就只有一个服务端，然后代理服务器可以在多个服务端之间选择，负载均衡。

还有，比如说一台服务器上面跑多个服务，为了不冲突，使用不同端口，但是希望这些服务对外看起来都在 80 端口，然后内部通过域名、 url 等区分具体是哪个服务，转发到对应端口。 

还有就是，代理服务器也可以做一些简单处理，比如上面提到的使用 SSL 对通信加密，但是内部就不需要加密了嘛，可以使用 http 与服务端通信，这样就把一个内部的 http 服务，转化为对外的 https 服务。

再有就是安全方面的考虑，比如因为各种原因，直接对外暴露服务端不好，比如说因为服务端使用了没有加密的 http 协议，不安全，或者说服务端有一些预留的后台，不希望暴露给外网。所以在服务器所在网络上加了防火墙，这样通过反向代理，可以对外暴露需要暴露的服务，这样反向代理服务器还有了一点网关的作用。

总之，作用很多。那么用 Nginx 如何配置反向代理呢

比如我用 Django 写了一个应用，希望使用 80 端口访问，同时我在 80 端口已经提供了一个博客的服务。那怎么办呢。首先可以分服务，Nginx 是通过 listen 和 server_name 来标志一个服务的，既然端口号和 ip 地址都一样，那可以通过域名区分。比如说下面这个例子，在配置文件 `/etc/nginx/conf.d/api.keybrl.com.conf` 写下如下配置，这个配置文件是可以和上一个例子中的 `blog.keybrl.com.conf` 共存的

```conf
server {
    listen               443;
    server_name          api.keybrl.com;
    ssl                  on;
    ssl_certificate      cert/1_api.keybrl.com_bundle.crt;
    ssl_certificate_key  cert/2_api.keybrl.com.key;
    ssl_session_timeout  5m;
    ssl_ciphers          ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        proxy_pass http://127.0.0.1:4000;
    }
}

server {
    listen       80;
    server_name  api.keybrl.com;
    rewrite      ^(.*)$ https://$host$1 permanent;
}
```

其实这跟上一个例子很类似，只是 `server_name` 和证书、私钥文件换了。这个服务通过语句 `proxy_pass http://127.0.0.1:4000;` 代理了内部的一个 http 服务。它对外可以通过 https://api.keybrl.com 访问。除了避免与 https://blog.keybrl.com 的博客服务冲突，这个服务还通过在代理服务器加 SSL 的方式将一个内部的 http 服务转为了一个对外的 https 服务。

但如果希望将两个服务合并成一个服务呢？也就是说，不希望让用户看起来访问了两个不同的主机。那可以通过路由区分。也就是用户访问的 url 中的路径。

```conf
server {
    listen               80;
    server_name          api.keybrl.com;

    location / {
        proxy_pass http://127.0.0.1:4000;
    }
    location /auth {
        proxy_pass http://127.0.0.1:4001;
    }
    location /board {
        proxy_pass http://127.0.0.1:4002;
    }
}

上面这个例子中，假设我写了三个不同的应用，一个是一个认证服务，一个是留言板应用，还有一个通用应用，它们分别使用 4000 、 4001 、 4002 端口。通过给一个服务加不同的 `location` 可以根据 url 中路径的不同路由到不同的应用。它的工作情况可以看下面一些例子

```text
http://api.keybrl.com/auth/login?username=keybrl&passwd=12345 -> http://127.0.0.1:4001/auth/login?username=keybrl&passwd=12345
http://api.keybrl.com/auth -> http://127.0.0.1:4001/auth

http://api.keybrl.com/board/add/balabala -> http://127.0.0.1:4002/board/add/balabala

http://api.keybrl.com/ -> http://127.0.0.1:4000/
http://api.keybrl.com/whatever -> http://127.0.0.1:4000/whatever
```

就跟 Django 或者别的什么 web 应用框架里面的路由一样，它会选择最长匹配的路由，执行里面的操作。 location 后面也可以写正则形式的路由规则。上面的 `->` ，是转发的意思，意思是，当用户请求 `http://api.keybrl.com/whatever` Nginx 会将请求代理到 `http://127.0.0.1:4000/whatever` ，将由本机上绑定 4000 端口的进程处理这个请求， url 中的路径不会改变，而且用户不会知道这个代理，用户只会以为 ta 向 `http://api.keybrl.com/whatever` 发送请求，然后得到了 `http://api.keybrl.com/whatever` 返回的响应。这一点和重定向是不一样的。

## 地址映射

这一节的标题我其实是不确定的，我不知道这种操作具体叫什么。如果你要去搜索引擎搜索，我建议你搜索 “nginx 重定向” ，但是我不认为这种操作是重定向。因为它不会触发任何 3xx 状态的响应，客户端也不会知道所请求的资源路径发生改变。

如果你去搜索 “nginx 地址映射” 出来的基本都是， `root` 语句，就像静态文件服务器那一节所介绍的， `root` 可以将 url 中一个诸如 `/` 的路径，映射到本地诸如 `/path/to/somewhere` 的路径，而且这些操作对客户端都是透明的。但是你会发现，这个语句只能在路径前面加路径，不能彻底改变路径。

如果我希望得到这样的映射，并且我希望这个映射对客户端是透明的，也就是说客户端不会收到重定向响应，地址栏的地址也不会发生任何改变

```text
http://api.keybrl.com/board/something -> http://127.0.0.1:4002/something
```

这很像上面一节，反向代理中的例子。没错，这需要用到 `proxy_pass` 语句来完成协议、主机名和端口部分的映射。但是对于路径 `/board/something` 到 `/something` 则无能为力。

很幸运，你不需要再知道更多知识了，因为这个操作完全可以通过上面提到的 `rewrite` 语句实现。只需要这样两句

```conf
rewrite ^/board(.*)$ $1 break;
rewrite ^/board$ / break;
```

之所以需要两句是因为，第二种情况 `/board` 后面如果什么都没有，使用第一个 `rewrite` 语句会引发错误。使用方法跟重定向是完全一样的。只不过需要注意两点，flag不能设置为 `redirect` 或 `permanent` ，目标 url 不能以 `http://` 或者 `https://` 这种完整 url 的开头作为开头。这样映射后就不会出现 3xx 响应，客户端也不会知道这种映射。

这和反向代理结合起来可以灵活配置一台主机上的多个应用。

---

Nginx 配置的技巧还非常多，本人才疏学浅，不能一一介绍，但是上面这些方法，几乎已经覆盖了我所有的配置需求了。如果你感兴趣，你还可以通过阅读 [Nginx 的官方文档](https://docs.nginx.com/) 进一步了解。
