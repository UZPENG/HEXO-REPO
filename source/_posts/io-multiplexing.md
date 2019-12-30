---
title: Linux-IO多路复用-SELECT/POLL/EPOLL
date: 2018-09-02 18:37:12
tags:
- linux
---

> 本文翻译自`DEVELPERS AREA`的[LINUX – IO MULTIPLEXING – SELECT VS POLL VS EPOLL](http://devarea.com/linux-io-multiplexing-select-vs-poll-vs-epoll)

# 前言

众所周知，在linux的世界里，一切皆文件。每一个进程都拥有一张文件描述符的表，指向文件、socket、硬件还有一些操作系统对象。  

典型的拥有很多IO源的系统都会有一个初始化的阶段，然后进入待机模式——等待客户端请求并且响应。  

最简单的解决方案就是为每一个客户端创建一个线程（或者进程），一直阻塞直到请求发送或者写入了响应。这种模式在客户端数量很小的时候可以工作，但是我们想要扩展到成千上万的客户端，为每个客户端创建线程（或者进程）是一个很糟糕的主意。

# IO多路复用

问题的解决方案是使用内核机制去轮询一系列的文件描述符。在linux系统下，主要有以下三种选择：  
* select(2) 
* poll(2)
* epoll

以上三种方法的思想都是一致的，新建一系列的文件描述符，告诉内核你对每一个描述符的操作，然后使用线程去阻塞一个函数调用直到至少有一个文件描述符请求的操作是可用的。
<!--more-->

# Select系统调用

`select()`系统调用提供了一种实现同步IO多路复用的方法。
```
int select(int nfds, fd_set* readfs, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout)
```
一个对`select()`的调用会阻塞直到指定的文件描述符已经准备进行IO了，或者指定的超时时间到了。  

监控的文件集合分成三类：
* `readfds`文件描述符集合是设置为监控数据是否可读的。
* `writefds`文件描述符集合是设置为监控写入数据是否完成而没有阻塞。
* `exceptsfds`设置为监控是否有异常发生或者带外（out-of-band）数据可用（一般只用于sockets）。

监控的集合可以为NULL，这种情况下select不会监控对应事件。  

在一个成功的返回，有且只有已经准备好IO的对象会被添加至对应集合.  
样例：
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <wait.h>
#include <signal.h>
#include <errno.h>
#include <sys/select.h>
#include <sys/time.h>
#include <unistd.h>
 
#define MAXBUF 256
 
void child_process(void)
{
  sleep(2);
  char msg[MAXBUF];
  struct sockaddr_in addr = {0};
  int n, sockfd,num=1;
  srandom(getpid());
  /* Create socket and connect to server */
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = inet_addr("127.0.0.1");
 
  connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
 
  printf("child {%d} connected \n", getpid());
  while(1){
        int sl = (random() % 10 ) +  1;
        num++;
     	sleep(sl);
  	sprintf (msg, "Test message %d from client %d", num, getpid());
  	n = write(sockfd, msg, strlen(msg));	/* Send message */
  }
 
}
 
int main()
{
  char buffer[MAXBUF];
  int fds[5];
  struct sockaddr_in addr;
  struct sockaddr_in client;
  int addrlen, n,i,max=0;;
  int sockfd, commfd;
  fd_set rset;
  for(i=0;i<5;i++)
  {
  	if(fork() == 0)
  	{
  		child_process();
  		exit(0);
  	}
  }
 
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  memset(&addr, 0, sizeof (addr));
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = INADDR_ANY;
  bind(sockfd,(struct sockaddr*)&addr ,sizeof(addr));
  listen (sockfd, 5); 
 
  for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    fds[i] = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    if(fds[i] > max)
    	max = fds[i];
  }
  
  while(1){
	FD_ZERO(&rset);
  	for (i = 0; i< 5; i++ ) {
  		FD_SET(fds[i],&rset);
  	}
 
   	puts("round again");
	select(max+1, &rset, NULL, NULL, NULL);
 
	for(i=0;i<5;i++) {
		if (FD_ISSET(fds[i], &rset)){
			memset(buffer,0,MAXBUF);
			read(fds[i], buffer, MAXBUF);
			puts(buffer);
		}
	}	
  }
  return 0;
}
```
开始的时候，我们创建了5个子进程，每个进程都连接了服务器并且向其发送消息。服务器使用`accept(2)`去为每个不同客户端创建不同的文件描述符。`select(2)`的第一个参数应该是在三个集合中最多文件描述符集合的文件描述符数量，增加1去检测最大的fd数量。  

主循环创建了一个所有文件描述符的集合，调用`select`然后检查哪个文件描述符已经准备被读取了。为了简单起见，我们不加任何的错误检测。  

返回后，`select`只改变那些文件描述符已经就绪的集合，因此我们需要在每一次迭代构建当前的集合。  

我们需要告诉`select`所有集合中文件描述符（以下简称为fd）的最高数量的原因是由于`fd_set`的内部实现机制。每一个`fd`是被一个bit声明的，因此`fd_set`是一个32个整型的数组(32*32=1024bit)。这个函数检测所有bit去观察是否其集合已经到达了最大值。这意味着如果我们有五个fd，但是其最高的数值是900，这个函数会检测从0到900的所有bit去找到需要监控的fd。另外select有一个POSIX实现——`pselect`，`pselect`使在等待的时候使用一个掩码。

## Select 总结

* 我们需要在每一次调用之前构造每一个集合。  
* select函数需要检测到最高位往前的任何bit——O(n)。  
* 我们需要遍历整个fd集合去检测指定集合是否有返回。  
* select的主要优势就是可移植性强，几乎所有的类unix系统都支持它。

# Poll 系统调用

和`select`函数的低效的三个位掩码fd集合不一样，`poll`提供了一个单独的n个pollfd的结构体数组，函数声明如下：
```c
int poll (struct pollfd *fd, unsigned int nfds, int timeout());
```
`pollfd`结构体的对于不同的事件和事件的返回有不同的成员，所以我们不需要每一次都构建它。
```c
struct pollfd{
  int fd;
  short events;
  short revents;
}
```
对于每一个fd构造一个`pollfd`的对象然后填充其要求的事件。在poll返回之后检测事件变量的值。  

用poll取改变上面的例子：
```c
  for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    pollfds[i].fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    pollfds[i].events = POLLIN;
  }
  sleep(1);
  while(1){
  	puts("round again");
	  poll(pollfds, 5, 50000);
 
	  for(i=0;i<5;i++) {
      if (pollfds[i].revents & POLLIN){
        pollfds[i].revents = 0;
        memset(buffer,0,MAXBUF);
        read(pollfds[i].fd, buffer, MAXBUF);
        puts(buffer);
      }
	  }
 }
```

像我们用`selcet`做的那样，我们需要检测每一个`pollfd`对象去观察它的fd是否已经准备好了，但是我们不需要每一次迭代都构建集合。

## Poll vs Select 

* `poll()`不要求用户计算fd的最高值+1。
* `poll()`在处理大数值的fd时更高效。想象一下，当你通过`select`只监听一个fd，但它的数值是900的时候，你需要检测每一个集合的从0到900的每一个bit。
* `select`的fd集合是静态的大小。
* 使用`selcect`，fd集合会在返回的时候重新构建，因此随后的调用都需要重新初始化它。而`poll`系统调用将输入和输出分离，允许数组在没有改变的情况下复用。
* `select`的`timeout`参数在返回的时候是为定义的，可移植的代码需要重新初始化它。`pselect`没有这个问题。
* `select`移植性更好，有一些类unix系统不支持`poll`。

# Epoll系统调用

当我们使用`select`和`poll`的时候，我们在用户空间处理所有的事情，然后我们每一个调用都发送fd集合然后等待。为了增加另外的socket，我们需要增加它到集合里并且重新调用select/poll。  

`epoll`系统调用帮助我们创建并管理在内核里的context，我们将任务分成三步。
* 使用`epoll_create`创建一个在内核的context。
* 使用`epoll_ctl`从内核中增加或移除fd。
* 使用`epoll_wait`等待内核中的事件。

让我们把上面的例子改为epoll实现的：
```c
struct epoll_event events[5];
  int epfd = epoll_create(10);
  ...
  ...
  for (i=0;i<5;i++) 
  {
    static struct epoll_event ev;
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    ev.data.fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    ev.events = EPOLLIN;
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev); 
  }
  
  while(1){
  	puts("round again");
  	nfds = epoll_wait(epfd, events, 5, 10000);
	
	for(i=0;i<nfds;i++) {
			memset(buffer,0,MAXBUF);
			read(events[i].data.fd, buffer, MAXBUF);
			puts(buffer);
	}
  }
```
我们首先创建了一个context（参数会被忽略但是必须是正数）。当一个客户端连接时，我们创建一个epoll_event对象然后把它添加到context，然后死循环里我们只等待context。

## Epoll vs select/poll

* 我们能够在等待的时候添加或者移除fd。
* `epoll_wait`只会返回已经就绪的对象。
* epoll有更好的性能——O(1) vs O(n)。
* epoll可以水平触发和边缘触发。
* epoll是linux特有的所以不可移植。