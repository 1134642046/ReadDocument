> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/nmtcttn/article/details/86482211)

1.OpenResty 是以 Nginx 为核心，以其他第三方模块组成。包括：Json、Mysql、Redis 等组件模块, 并且支持 Lua 语言，运行起来非常高效，而且非常有助于程序员的高层次提升。安装 OpenResty 需要有 Linux 的支持，还需要一些其他编译组件库的支持，如：g++,openssl,pcre。  
2. 还有一点需要注意：在安装 OpenResty 时，会解压出一些模块或组件的目录，所以安装前需要规划好安装路径。建议采用下面的路径，结构会更加清晰一些。  
![](https://img-blog.csdnimg.cn/20190114185901725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)

### 新建 openresty 目录

```
>mkdir servers
   >cd servers
   >mkdir openresty
   >cd  openresty
```

### 安装依赖 A

```
>yum install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl  
  如果是ubuntu则使用:apt-get
```

### 安装依赖 B

```
1.安装PCRE库(可在官网下载最新版本)
  >cd /usr/local/nginxTools
  >wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz 
  >tar -zxvf pcre-8.37.tar.gz
  >cd pcre-8.34
  >./configure
  >make && make install
  2.安装openssl库
  yum -y install openssl openssl-devel
```

### 下载 OpenResty 压缩文件

```
>wget https://openresty.org/download/openresty-1.13.6.2.tar.gz
 >tar -xzvf openresty-1.13.6.2.tar.gz
 在解压目录中会看到bundle目录，在该目录中科编译生成luajit和lualib文件，以及nginx核心和其他一些第三方模块。
```

### 编译安装 Luajit

```
>cd bundle/LuaJIT-2.1-20180420
 >make clean && make && make install
 >ln -sf luajit-2.1.0-alpha /usr/local/bin/luajit
```

### 下载 ngx_cache_purge 模块，该模块用于清理 nginx 缓存

```
>cd bundle  
 >wget https://github.com/FRiCKLE/ngx_cache_purge/archive/2.3.tar.gz  
 >tar -xvf 2.3.tar.gz
```

### 下载 nginx_upstream_check_module 模块，该模块用于 ustream 健康检查

```
>cd bundle  
 >wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz  
 >tar -xvf v0.3.0.tar.gz
```

### 安装 OpenResty(重点注意:–prefix 中的路径)

```
>cd /usr/local/servers/openresty/openresty-1.13.6.2
 >./configure --prefix=/usr/local/servers/openresty --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2  
 >make && make install  
 --with***                                   安装一些内置/集成的模块
 --with-http_realip_module        取用户真实ip模块
 --with-pcre                                Perl兼容的达式模块
 --with-luajit                                集成luajit模块
 --add-module                            添加自定义的第三方模块，如此次的ngx_che_purge
 编译完成之后，在/usr/local/servers/openresty目录中会看到LuaJit和Lualib文件夹，已经nginx核心文件夹nginx。这说明以安装成功，进入到nginx目录中可以启动openresty
 >/usr/local/servers/openresty/ngnix/sbin/./nginx
```

![](https://img-blog.csdnimg.cn/20190114192707636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)

重点: 访问出现拒绝的情况：  
![](https://img-blog.csdnimg.cn/20190115183433894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)  
大部分都是防火墙的问题，可以进行以下处理：  
1. 在虚拟机内进行检测，如果正常显示就是防火墙问题  
![](https://img-blog.csdnimg.cn/20190115183714724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)  
2. 防火墙端口关闭，防火墙重启 == => 3032 是端口号

```
>firewall-cmd --permanent --add-port=3032/tcp  
 >firewall-cmd --reload  
 >systemctl stop firewalld.service  
 >systemctl start firewalld.service
```

![](https://img-blog.csdnimg.cn/20190115184350449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2019011419283022.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)  
出现上图则说明已经安装成功，启动成功，继续开启 OpenResty 的神秘世界吧！

![](https://img-blog.csdnimg.cn/20190114193103663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25tdGN0dG4=,size_16,color_FFFFFF,t_70)