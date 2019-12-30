---
title: 一些开发中遇到的问题
tags:
- 笔记
---

# 前言

在开发的过程中，自己总会遇到形形色色的问题。有时候需求比较赶，通过搜索把问题的解决了就算了，没有时间去深挖一些问题的成因。这是一个非常不好的习惯，通过了解问题的成因，能够加深自己对知识的了解。因为，这个文章就应运而生了。这个文章只是记录一些问题的，具体的成因和解决方案会另外写博客阐释。

# Java开发相关

## synchronized和future一起使用的问题

这个问题主要是在 `Quartz` 框架， 使用阿里的 `OpenTSDB SDK` 的问题。

```java
synchronized(LOCK){
    client.put();
}

.......

Future future = this.httpclient.execute(request, (FutureCallback)null);
      
HttpResponse httpResponse = (HttpResponse)future.get();
```

这个代码会报并发错误，是由于future产生的。
