---
title: HTTP简介
tags:
- 简介
---

# 术语

1. HTTP(Hyper Text Transfer Protocal) 超文件传输协议
2. 客户端(Client)、服务器(Server)
3. 资源(Resoure)、MIME(Mutipurpose Internet Mail Extension) 媒体类型
4. URI(Union Resource Identifier)
    * URL(Union Resource Location):统一资源定位符
    * URN(Union Resource Name): 统一资源名称
5. 事务(transaction)
6. 方法(Method)
7. 状态码(Status Code)
8. 报文(Message)
9. 代理(Proxy)
10. 缓存(Cache)
11. 网关(Gate Way)
12. 隧道(Tunnel)
13. Agent代理

# URL（统一资源定位符）

* schema://\<user>:password@\<host>:\<port>/path;param?\<query>#\<frag>
    * 关键组成：方案（schema）、用户（user）、密码（password）、主机（host）、端口（port）、path（路径）、parameter（参数）、查询（query）、片段（fragment）

    例子：http://fanyi.baidu.com/?aldtype=16047#auto/zh/   

    备注：
  	* frag字段不发送给服务器，客户端根据服务器返回的对象解释。
    * % 用作编码字符的转义标志。其他一些限制字符见《HTTP权威指南》

* 基础URL：相对URL、绝对URL
    * 资源中显式地提供
    * 封装资源的技术URL
    * 没有基础URL

* 自动扩展的URL
    * 添加schema
    * 添加/

* URL字符集
    * ~(0x7E)、空格(%20)、%(0x25)
    * 保留字符和受限字符见《HTTP权威指南》P39

* URI的未来：URN

# 报文

* 分类：请求报文、应答报文。
* 请求报文：起始行（start line）、请求头（request header）
	* 请求头: 主机(Host)、连接方式(Connection)、接受的数据类型(Content-type)、用户代理(Agent)、内容的编码(Conetent-Encoding)、接受的语言(Accept-Lanuage)、cookie

>备注：更加多请求头信息可参考《HTTP权威指南》

应答报文：起始行（start line）、响应头（response header）、响应体（response body）  
![](/img/HTTPRequest.png)

 * 起始行：协议版本+状态码+原因短语
 * 响应头：服务器、日期、内容类型、连接方式
 * 响应体：服务器返回内容。

方法(Method)

* GET 获取服务器的一份文档
* PUT 推送一份文档到服务器
* DELETE 删除服务器的一份文档
* POST 向服务器发送需要处理的数据
* TRACE 跟踪报文
* HEAD 只返回首部
* OPTIONS 看看服务器支持那些方法

状态码（staus code）

* 100~199 信息提示
* 200~299 成功
* 300~399 重定向
* 400～499 客户端错误
* 500~599 服务器错误

首部（header）

* 通用首部  
    * 通用信息首部   
    * 通用代理首部
* 请求头部
    * 信息首部
    * Accept首部
    * 条件首部
    * 安全请求首部
    * 代理首部

* 响应头部
    * 信息首部
    * 协商首部
    * 安全响应首部

* 实体头部
    * 内存首部
    * 缓存首部

* 扩展头部

## 连接管理

HTTP使用TCP的过程

1. 获取主机名
1. DNS解释出IP地址
2. 结合通信端口（80）构造Socket，发起socket连接
3. 客户端发送请求报文
4. 服务器回送响应报文
5. 服务器关闭连接

HTTP性能聚焦的区域

* TCP三次握手连接建立的时间延迟。
* TCP的慢启动拥塞算法。
* 数据聚集的nagle算法
* 用于捎带确认的TCP延迟确认算法
* 端口耗尽

连接的三种方式
![](/img/628984136063207975.jpg)


## 关键句

1. 一个页面通常不是单个资源，而是多个资源的集合。
2. 对于一个程序员来说，对非安全字符的编码仍是明智的。
3. TCP流是分段的，由IP分组发送
