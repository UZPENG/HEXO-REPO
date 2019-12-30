---
title: Android相关笔记整理
tags:
- 笔记
---

# 前言

以前写Android的时候留下了很多笔记，比较零碎。现在也不写Android了，把它总结归纳一下，就当做是缅怀我的过去吧。


# 构建
gradle提速的十个技巧

gradle3.0的特性
* complie 和 implement的区别   
* abi change 和 non-abi change

# RecylerView
`ListView`：
* 展示持久化内容的结构
* 怎么样去快速地展示成千上万的items,创建view的代价高昂并且设备的内存有限。只将用户看到的
创建并且展示，当用户滚动再创建新的可见的view。

最佳实践
* `ViewHolder` 作为回收和复用的基本单位
* 将`View`的创建和绑定分开
* 使用标准框架的焦点和输入处理
* 更容易维护
* 更细粒度的数据变动监听
* 让回收和动画更加高效

`LayoutManager`:`LinearLayoutManager`、`GridLayoutManager`、`StaggeredGridLayoutManager`
* 定位`View``recyclerview` 交互 然后 `layoutmanager` 负责滚动
在可见范围内的焦点处理时recyclerview处理的，如果要从可视范围以外获取视图，需要layoutmanager

`Adaper`:创建并填充需要的view
* 创建view和viewholder
* 绑定item到viewholder
* 通知recyclerview相关的变化
* 处理item的交互
* 多种view type
* 恢复recycler
* 细粒度的data变化事件监听

* 如果有Gaint and expensive的bitmap的话，只缓存视图结构，不缓存代价高昂的View。

# 进程的优先级

![进程优先级](http://www.mrpeak.cn/images/at1.jpg)

线程调度时间片设计
![](http://www.mrpeak.cn/images/at2.jpg)


handler与Looper绑定、Looper和线程绑定。looper的构造函数：
```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }

# 让代码在UI线程执行
runOnUIThread()
```
# 代码混淆

## 概念

* 将变量名改为无意义的符号。单个字母或者是下划线 __ 。
* 重写逻辑使其更难理解，但是必须等价。
* 打乱格式。
* 构造特殊的指令使得反汇编器出错。

## 工具

* ProGuard  免费开源的。Android Studio 支持的，可以被反编译，但是难以阅读。
* DexGuard 收费的。部分不能反编译。
* 备注：大部分人还是使用ProGuard的。第一，因为它是免费的；第二，很多第三方的库没有DexGuard的配置文档，而ProGuard的文档比较齐全，[详情链接](https://shadowzwy.github.io/2016/12/04/Android%E6%B7%B7%E6%B7%86%E5%B7%A5%E5%85%B7-Proguard%E5%AE%9E%E8%B7%B5.html>)

# Android Studio FAQ

问题：gradle 版本不兼容导致的各种问题  
解决办法：在Project的build.gradle文件下配置gradle为最新版本

备注：注意区分gradle的版本和Android Gradle Plguin 版本。

问题：invalid LOC header  
解决办法：删除项目目录下的gradle或者build文件夹，重新编译项目。

问题：编辑器字体有明显锯齿。  
解决办法：打开如下图的设置，设置抗锯齿。
![](/img/12.png)

问题：没有Logcat信息。  
解决办法：如下图所示刷新logcat。如果是华为手机，在拨号处输入
\*#\*#2846579#\*#\*,开启日志。
详情见[文章](http://devlu.me/2016/07/29/open-android-debug-log-in-device)。


问题：升级android studio 3.0后，terminal中文乱码。  
解决办法：在home目录下修改'.bashrc'文件（没有就新建）添加'export LANG=en_US.UTF-8'

