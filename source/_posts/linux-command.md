---
title: linux常用命令速查
date: 2018-08-22 15:56:32
tags:
- linux
---
# 前言

记录一些linux的常用操作以便查询，暂时先记录，以后有时间再分门别类。

<!--more-->

# Linux常用命令

| 项目                  | 内容                                                    |
| --------------------- | ------------------------------------------------------- |
| 终端ss代理            | `export ALL_PROXY=socks5://127.0.0.1:1080`              |
| 查看系统版本          | lsb_release -a                                          |
| 修改系统主机名        | 编辑 /etc/hostname 文件                                 |
| 查看端口使用情况      | netstat -apn                                            |
| 输出已安装的包        | rpm -qa  centos/dkkg -l ubutu                           |
| 清空                  | ctrl+u 当前输入 crtl+l 屏幕                             |
| 查看占用空间          | du -h `<file or dir>`                                   |
| 解压tar               | tar -zxf `<target>`                                     |
| 创建软连接            | ln -s `<target> <linkname>`                             |
| 搜索mysql配置文件位置 | mysql --help  grep my.cnf
| 查看远程端口打开情况  | telnet `<host> <port>`                                  |
| 登录日志              | /var/log/secure                                         |
| 邮件日志              | /var/log/mail                                           |
| 安装dns相关命令       | yum install bind-utils                                  |
| certbot安装           | [issue](https://github.com/certbot/certbot/issues/5104) |
| 查看登录情况          | last 查看成功的  lastb 查看失败的                       |
| free | 查看内存使用情况 |
| df | 查看系统分区 |
| top | 任务管理器 |
| cal | 查看日历  |
| date | 查看时间 |
| passwd | 修改用户密码 | 

# 附录

certbot泛子域名配置
> `./certbot-auto certonly --agree-tos --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory -d host -d "*.host" certbot certonly -d *.inforsecszu.net --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory`

headless chrome安装
> [安装过程](https://askubuntu.com/questions/79280/how-to-install-chrome-browser-properly-via-command-line)
> [deb版本地址](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)
> [rpm版本地址](https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm)
