---
title: postfix+dovecot搭建自己的邮件服务器
date: 2018-03-06 19:37:31
tags:
- 网络
---
 

## 前言

我对于邮箱理解，在很长一段时间里，都是qq邮箱、163邮箱、126邮箱等等的各种第三方邮箱。这些邮箱都是注册一下，然后从web页面登陆进去，就可以收发邮件了。后来上了计算机网络这门课，学习到了有关于邮件的一些协议比如SMTP、POP3、IMAP。但是，当时也只是知道有这几种协议，也没有去实践过，现在想起来真的觉得计算机各方面知识自己都实践得太少了。而且最近在学习java web开发，想着实现一个邮件发送验证码的功能，同时又想拥有一个自己的域名邮箱，所以就决定配置一下邮件服务器。
<!--more-->

## 技术框架

### 系统环境

命令行输入`lsb_realse -a`，查询结果如下：
```
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.3.1611 (Core)
Release:	7.3.1611
Codename:	Core
```

### 软件
| 软件    | 说明               |
| ------- | ------------------ |
| postfix | 提供SMTP服务       |
| dovecot | 提供POP3、IMAP服务 |

### 协议解释
| 协议                                | 说明                       | 端口 |
| ----------------------------------- | -------------------------- | ---- |
| SMTP  imple Mail Transfer Protocol  | 主要作用是推送邮件         | 25   |
| POP3  Post Office Protocol          | 主要作用是从服务器获取邮件 | 110  |
| IMAP  Internet Mail Access Protocol | 主要作用是从服务器获取邮件 | 143  |

协议的工作流程大致如下：
![](/img/mail.png)

## [](#部署实践 "部署实践")部署实践

好的，理论就说到这里了，下面开始部署实践。

首先，查看系统有没有带有sendmail，有就卸载了，然后安装postfix和dovecot
```
rpm -qa | grep sendmail
yum remove sendmail
yum install postfix
yum install dovecot
```

然后开始配置工作，首先配置postfix。
```
vim /etc/postfix/main.cf
```
将默认配置改为以下配置：
```
myhostname = uzpeng.top
mydomain = uzpeng.top
myorigin = $myhostname
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost
mynetworks_style = host
relay_domains = $mydomain
smtpd_sender_restrictions = permit_mynetworks permit
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, check_recipient_access hash:/etc/postfix/access, defer_unauth_destination
mynetworks = 127.0.0.1/32 172.16.252.225/32 <your client ip>/32
```

接着配置dovecot
```
vim /etc/dovecot/dovecot.conf
```

添加如下配置：
```
login_trusted_networks = <your client ip>
mail_location=maildir:~/Maildir
```

然后分别启动服务
```
systemctl start postfix
systemctl start dovecot
```

接着防火请打开，25，110，143端口。

DNS解析添加MX记录指向自己的服务器。

至此，服务器配置完毕。

默认服务器有一个名为liu的用户，下面开始客户端的配置。搜索下载foxmail,账号管理新增用户填充信息如图
![](/img/foxmail.png)
点击创建就可以创建成功了。

然后到了这里，我可以成功的接收邮件了，但是发送的话一直就是接收不到。查看了一下运行日志，提示如下错误：
```
Mar  3 17:30:36 uzpeng postfix/smtp[24921]: connect to mx3.qq.com[183.57.48.35]:25: Connection timed out
Mar  3 17:30:36 uzpeng postfix/smtp[24921]: connect to mx3.qq.com[240e:ff:f040:28::f]:25: Network is unreachable
Mar  3 17:31:06 uzpeng postfix/smtp[24921]: connect to mx2.qq.com[14.17.41.170]:25: Connection timed out
Mar  3 17:31:36 uzpeng postfix/smtp[24921]: connect to mx2.qq.com[59.37.97.124]:25: Connection timed out

```

纳尼？网络都连不上吗？赶紧运行`telnet mx3.qq.com 25`测试了一下端口，结果如下，一直连不通。


后来我本地运行`telnet mx3.qq.com 25`，妥妥得连接上去了，如图
![](/img/telnet.png)  

所以基本断定是服务器的问题了，但是服务器的防火墙是打开了25端口的，莫非阿里云屏蔽这个端口？搜索引擎一搜，果然是这么一回事。
![](/img/aliyun-port.png)

所以搞了一个下午，最终的结果就是阿里云不允许直接使用stmp发送邮件，必须通过第三方。

##  基于第三方邮箱配置自己的域名邮箱

既然阿里云都屏蔽了端口，所以没办法咯，只能用第三方的了，那就直接用阿里家的呗。

控制台&gt;域名与万网&gt;企业邮箱&gt;管理，然后修改管理员账号密码，登录进去，创建一下用户，配置一下基本信息，这些都是比较傻瓜式的操作了，不再赘述。

最后域名解析按照阿里云的提示来，主要是添加MX记录到阿里云的邮箱服务器，添加CNAME记录到阿里的stmp,pop3,imap服务器，服务器端基本就配置完成啦。

客户端的配置和话，和自己搭建的差不多啦，就是服务器地址换一换。

## 遇到的问题

登录不上去。
```
Mar  3 14:42:35 uzpeng dovecot: imap(liu): Error: Invalid user settings. Refer to server log for more information.
Mar  3 14:42:35 uzpeng dovecot: imap-login: Login: user=<liu>, method=PLAIN, rip=218.17.207.66, lip=172.16.252.225, mpid=15181, secured, session=<PBHwZXxmnwDaEc9C>
```

dovecat配置文件添加
```
mail_location=maildir:~/Maildir
```

## 结语

弄了一个下午，总算有了自己的域名邮箱。虽然没有成功在自己服务器跑邮箱服务，但是捣鼓这么一遭，自己对邮件服务还是有了更深刻的理解。好了，最近捣鼓的内容差不多就这样了，接下来需要把时间投入到java web和春招中去了，希望自己能找到满意的工作！