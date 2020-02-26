<!-- TOC -->

- [临时文档](#临时文档)
    - [锁、synchronized、ReetrantLock](#锁synchronizedreetrantlock)
    - [JVM内存模型](#jvm内存模型)
    - [面试题](#面试题)
- [Mybatis面试题](#mybatis面试题)
- [Mysql](#mysql)
    - [索引原理](#索引原理)
    - [聚集索引](#聚集索引)
- [LRU算法](#lru算法)
- [面试题收集](#面试题收集)
    - [如何大量数据写入MySQL数据库](#如何大量数据写入mysql数据库)
    - [如何做一个高并发系统](#如何做一个高并发系统)
    - [App端高并发如何处理](#app端高并发如何处理)
    - [加密过程，用过哪些加密方法](#加密过程用过哪些加密方法)
- [数据结构](#数据结构)
    - [数组为什么访问快，链表访问慢？](#数组为什么访问快链表访问慢)
    - [时间复杂度和空间复杂度](#时间复杂度和空间复杂度)
- [什么是fail-fast？](#什么是fail-fast)
- [什么是fail-safe？](#什么是fail-safe)
- [CAS](#cas)
- [Guava缓存原理](#guava缓存原理)
- [JVM知识点](#jvm知识点)
- [Http2和Http3](#http2和http3)
- [什么是socket](#什么是socket)
- [限流算法](#限流算法)
- [动态代理和静态代理](#动态代理和静态代理)
- [分布式系统中事务如何处理](#分布式系统中事务如何处理)
- [MySQL如何支持事务的](#mysql如何支持事务的)
- [跨域请求](#跨域请求)
- [Servlet](#servlet)
- [分布式Session](#分布式session)
- [Zookeeper](#zookeeper)
- [分布式锁](#分布式锁)

<!-- /TOC -->
# 临时文档
## 锁、synchronized、ReetrantLock
[synchronized用法](https://blog.csdn.net/qq_32665329/article/details/84340332)

[一张图读懂非公平锁与公平锁](https://www.jianshu.com/p/f584799f1c77)

[阿里-JVM几把锁](https://mp.weixin.qq.com/s/h3VIUyH9L0v14MrQJiiDbw)

## JVM内存模型
[JVM内存模型](https://www.nowcoder.com/discuss/151138?type=1)

## 面试题
[一些面试题](https://www.jianshu.com/p/7c80de89cdb7)

# Mybatis面试题

# Mysql
## 索引原理

## 聚集索引

# LRU算法
最近最少使用

# 面试题收集
## 如何大量数据写入MySQL数据库
## 如何做一个高并发系统
## App端高并发如何处理
## 加密过程，用过哪些加密方法

# 数据结构
参考

[数据结构与算法博客](https://blog.csdn.net/qq_41907991/column/info/42035)

## 数组为什么访问快，链表访问慢？
- 数组

数组在内存中存储时是连续的内存地址，CPU缓存读取时会把一片连续的内存空间读入，对数组随机访问时只需对[基址+元素大小*元素偏移量k]就能找到第k个元素的地址。

- 链表

链表不需要一块块连续的内存，通过指针将一组零散的内存块串联起来使用。

如果我们需要在链表中访问一个元素，受链表底层数据结构的影响（不连续，通过指针串联）所以必须要一个个节点的遍历，不难得出时间复杂度为O(n)。

## 时间复杂度和空间复杂度

# 什么是fail-fast？
https://www.cnblogs.com/kubidemanong/articles/9113820.html


# 什么是fail-safe？
https://www.cnblogs.com/songanwei/p/9387745.html

# CAS
CompareAndSwap： 比较并替换
[面试必问的CAS](https://blog.csdn.net/v123411739/article/details/79561458)

# Guava缓存原理

# JVM知识点
[JVM知识点](https://blog.csdn.net/qq_35462834/article/details/81940222)

# Http2和Http3
https://blog.csdn.net/howgod/article/details/102597450

# 什么是socket
https://www.jianshu.com/p/066d99da7cbd

# 限流算法
令牌桶算法

# 动态代理和静态代理
https://blog.csdn.net/fox_bert/article/details/80891148

# 分布式系统中事务如何处理

# MySQL如何支持事务的

# 跨域请求
https://javadoop.com/post/cross-domain

# Servlet
1、客户端发起请求
2、服务器接收请求，发送到Servlet容器
3、Servlet容器检测是否实例化Servlet，交给Servlet实例
4、Servlet调用init方法
5、Servlet调用service方法，根据GET、POST不同请求来处理，调用doXXX()方法
6、Servlet返回相应内容给容器，容器返回给客户端
7、容器关闭时调用Servlet的destroy()方法
```
public interface Servlet {
    public void init(ServletConfig config) throws ServletException;

    public void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException;

    public void destroy();

    public ServletConfig getServletConfig();

    public String getServletInfo();
}
```
# 分布式Session
Redis


# Zookeeper
[Zookeeper](https://juejin.im/post/5b037d5c518825426e024473)

数据结构：树

集群：一主多从

ZAB协议

# 分布式锁
