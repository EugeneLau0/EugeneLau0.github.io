---
title: 'MacOS通过安装包安装NGINX服务'
categories:
  - Blog
tags:
  - MacOS
---

最近在使用Mac系统安装nginx时，遇到一个错误`Error: nginx: Invalid bottle tag symbol`导致一直安装不成功。根据以往的习惯就是直接去查询各种博客资料。这次有点出乎意料，没有搜到我想要的答案。最后选择去NG官网看看，没想到还真的找到了答案。

<!--more-->

# 背景

使用 `brew install nginx`安装 NGINX，一直报一个错：

```
Error: nginx: Invalid bottle tag symbol
```

一开始以为是brew版本的原因，就去升级了brew的版本。可在我更新了 home brew后，错误依然存在。

# 建议

**在毫无头绪的时候，建议去官网看看，说不定有什么发现。** 

我就发现了，可以使用[安装包的形式](https://unit.nginx.org/installation/#installation-precomp-pkgs)来安装，这样就可以不用依赖 brew 了。为什么这种最原始的方式，在各大博客里都没有推荐呢，都是推荐用程序包管理器来安装，可能也是方便吧！

另外又找了一篇 [2014 年的博客](https://www.dynamsoft.com/codepool/how-to-configure-and-install-nginx-on-mac-os-x.html)，写了如何在 Mac 上通过安装包的形式安装 NGINX 。

# 教程

毕竟参考的是历史悠远的博客，少许内容跟现在有点差异，我这里提供在我的机器上操作的步骤给到大家，避免大家同我一样再踩坑！

## Install Nginx

1. Download the latest stable version – [nginx 1.22.0](http://nginx.org/en/download.html).
2. Unzip the downloaded package by the command “tar xvzf nginx-1.22.0.tar.gz”.
3. “cd nginx-1.22.0”.
4. “sudo ./configure”. There is an error displayed:

在第 4 步会遇到一个报错：

```sh
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

1. To fix the error, visit the [tutorial page](http://nginx.org/en/docs/configure.html) and read “Building nginx from Sources”, in which you can find the link of PCRE library.
2. Go to [PCRE official site](http://www.pcre.org/), and find the latest version of PCRE library on [SourceForge](http://sourceforge.net/projects/pcre/files/pcre/).
3. Download the package and unzip it by the command “tar xvzf pcre2-10.40.tar.gz”.
4. Now, you can run the configure file again with the parameters  ”sudo ./configure -–with-pcre=<path>”.
5. Configuration is done.

PS：第 4 步需要切换到 NGINX 包目录下执行，path 为 PCRE 的安装包目录（使用绝对路径）

the default nginx path prefix is “/usr/local/nginx”.

1. To install nginx, type in “sudo make install”.
2. Find the executable file “cd /usr/local/nginx/sbin”
3. Launch nginx “sudo ./nginx”

PS：第 1 步，需要在 NGINX 安装包路径下执行！

```
The nginx is successfully running now!
```

## test nginx

NGINX 的默认监听端口为 `80`，按以上步骤启动后，直接在浏览器访问 `127.0.0.1` 或者 `curl 127.0.0.1`即可。

请求响应出现 `Welcome to nginx!`即代表 NGINX 安装成功。