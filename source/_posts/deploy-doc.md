---
title: Ngnix+Tomcat+MySQL项目部署
date: 2018-06-20 18:01:55
tags:
- 后端
---
# 前言
毕业设计做了一个签到系统的服务端，架构使用的Java栈的传统架构，也是最简单的架构。这个架构主要用Nginx做反向代理和负载均衡（我的项目没有使用到这一块），Tomcat作为Servlet的容器，MySQL作为数据库。项目也比较基础，没有太多难点，但是在项目部署过程当中还是遇到了一些问题，所以想着把部署的过程及记录下来，方便自己查看。
# 系统环境
操作系统：CentOS 7.4 x64  
JDK: 1.8.171  
MySQL: 5.7  
Tomcat: 9.0.8  
<!--more--->
# 软件安装
## MySQL数据库
MySQL由于历史原因（就是Oracle之前收购MySQL，社区担心Oracle闭源MySQL数据库，所以更改了源，默认的源是MariaDB)没有办法直接使用`yum`命令安装，所以需要手动下载然后添加。
```
$ wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```
首先下载MySQL 5.7版本的包，然后添加到仓库。
```
$ sudo rpm -ivh mysql57-community-release-el7-9.noarch.rpm
```
然后直接使用`yum`命令安装MySQL。
```
$ sudo yum install mysql-server
```
安装完成后直接启动MySQL。
```
$ sudo mysql -u root
```
如果启动失败，可以打开配置文件，开启免密登录。
```
$ sudo vim /etc/my.cnf

[mysqld]
skip-grant-tables
```
进入MySQL后，就可以初始化配置了。首先可以修改密码和监听的地址。
```
> use mysql;
> ALTER PASSWORD PASSWORD('your password')
> UPDATE user SET host='ip or %';
> SET GOBAL max_connections=10000;
> exit
```
注释掉前面的那一句`skip-grant-tables`，然后重启mysql。
```
$ systemctl restart mysqld
```
然后启动mysql,创建名为`sign_system`的数据库，导入备份的数据。
```
> CREATE DATABASE sign_system;
> exit

$ mysql -u root -p < backup.sql
```
然后数据库部分的部署基本完成。

## Java环境配置
首先下载JDK，从这个[网址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载，下载完成后，解压。
```
$ tar -zxvf file.tar.gz
```
然后配置环境变量。
```
$ vim /etc/profile

JAVA_HOME='your dir';
PATH='$JAVA_HOME/bin:$PATH'
```
如果默认安装有open-jdk，可以使用命令卸载了它。
```
$ sudo yum remove open-jdk
```
## Tomcat安装配置
下载Tomcat。
```
$ wget http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.8/bin/apache-tomcat-9.0.8.tar.gz
```
解压。
```
$ tar -zxvf <your file>
```
配置tomcat环境变量，进入tomcat目录下的bin目录，创建setenv.sh文件，输入内容。
```
$ CATALINA_HOME/bin touch setenv.sh

$ vim setenv.sh

#! /bin/sh

# add tomcat pid
CATALINA_PID="$CATALINA_HOME/tomcat.pid"

JAVA_HOME='your dir'

# add catalina option
CATALINA_OPTS="-Djava.rmi.server.hostname=uzpeng.top
-Dcom.sun.management.jmxremote.port=1099
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.password.file=$CATALINA_HOME/conf/jmxremote.password
-Dcom.sun.management.jmxremote.access.file=$CATALINA_HOME/conf/jmxremote.access
-Dcom.sun.management.jmxremote.ssl=false"
```
配置`CATALINA_HOME`环境变量
```
$ vim /etc/profile

CATALINA_HOME='your dir'
```
最后配置启动脚本。
```
$ vim /lib/systemd/system/tomcat.service

[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking
PIDFile='your dir'/tomcat.pid
ExecStart='your dir'/bin/catalina.sh start
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```
在tomcat目录下创建`tomcat.pid`文件。
```
$ touch tomcat.pid
```
上传war文件（我使用的是Xftp）到tomcat目录下的webapp，重命名为ROOT.war。然后接着编辑`$CATALINA_HOME/conf/server.xml`
```                       
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <Context docBase="ROOT.war" path=""/>

        .......
  </Host>

```
重启tomcat
```
$ sudo systemctl restart tomcat
```
## Nginx配置
首先直接使用yum命令安装nginx。
```
$ sudo yum install nginx
```
然后打开配置文件配置。
```
$ sudo vim /etc/nginx/nginx.cnf
```
以下是我的配置文件。
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    use epoll;
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    proxy_set_header X-Real-IP $remote_addr;

   server {
     if ($host = <host>)
        return 301 https://www.$host$request_uri;
     }

      server_name  <host>;
   } 
   server { 
      server_name  api.host;

      location / {
           proxy_pass http://localhost:8080;
           keepalive_timeout 300s;
           proxy_send_timeout 300s;
           proxy_read_timeout 300s;
           proxy_http_version 1.1;

           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Host $host;
           proxy_set_header X-NginX-Proxy true;

           # prevents 502 bad gateway error
           proxy_buffers 8 32k;
           proxy_buffer_size 64k;

           proxy_redirect off;

           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
      }

    # managed by Certbot

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/<host>/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/<host>/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  }

 server { 
      server_name  www.host;

      location / {
           proxy_pass http://localhost:4000;
           keepalive_timeout 300s;
           proxy_send_timeout 300s;
           proxy_read_timeout 300s;
           proxy_http_version 1.1;

           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Host $host;
           proxy_set_header X-NginX-Proxy true;

           # prevents 502 bad gateway error
           proxy_buffers 8 32k;
           proxy_buffer_size 64k;

           proxy_redirect off;

           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
      }

    # managed by Certbot

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/<host>/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/<host>/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  }
```
## 安装证书
证书使用的是Let's Encrypt的泛子域名证书，安装步骤如下：
```
$ sudo yum install certbot
$ sudo yum install  python2-certbot-nginx
$ sudo certbot --nginx
$ sudo certbot certonly --agree-tos --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory -d host -d "*.host"
```
# 总结
简单地记录了一下系统部署的过程，方便以后查阅，以后如果有遇到新的问题会在本文章继续补充。