# database存储系统 阅读笔记

| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/05/24 |


## 目标

- 梳理redis/memcached的读写流程、复制详细过程

- 针对该版本在代码层面梳理一份有C语言基础的同学都可以读懂

## 文件解压缩

```
//分割大文件为小文件
split -b 30m  postgres.tar.gz  postgres.tar.gz.  –verbose

//合并小文件为大文件
cat  postgres.tar.gz.a*  > postgres.tar.gz 
```
## redis/memcached存储系统
- [分布式系统基本理论](./document/分布式系统原理/分布式系统基本理论.md)
- [缓存的基本介绍](./document/分布式系统原理/缓存的基本介绍.md)
- [缓存系统中的高并发](./document/分布式系统原理/缓存系统中的高并发.md)
- [memcached介绍](./document/memcached分析/memcached基本介绍.md)
- [memcached线程模型分析](./document/memcached分析/memcached线程模型分析.md)
- [redis调试环境](./document/redis分析/redis源码调试环境.md)
- [redis支持的数据结构和编码方式](./document/redis分析/redis支持的几种数据结构和编码方式.md)
- [redis字符串的实现原理](./document/redis分析/redis字符串的实现原理.md)
- [redis客户端问题案例](./document/redis运维/Redis客户端问题案例.md)

## postgresql数据库
