# NoSQL存储系统 阅读笔记

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/12/05 |中国开源存储技术交流群(672152841) |


## 目标

- 梳理md/redis/memcached的读写流程、复制详细过程
- redis模块原理和源码分析


## redis篇
- [分布式系统基本理论](./document/md/redis/分布式系统基本理论.md)
- [缓存的基本介绍](./document/md/redis/缓存的基本介绍.md)
- [缓存系统中的高并发](./document/md/redis/缓存系统中的高并发.md)
- [事务一致性解决方案](./document/md/redis/2020-11-04-事务一致性解决方案.md)
- [redis调试环境](./document/md/redis/redis源码调试环境.md)
- [redis支持的数据结构和编码方式](./document/md/redis/redis支持的几种数据结构和编码方式.md)
- [redis字符串的实现原理](./document/md/redis/redis字符串的实现原理.md)
- [redis客户端问题案例](./document/md/redis/Redis客户端问题案例.md)
- [RedisBloom源码编译和加载](./document/md/redis/2020-11-11-redisbloom源码编译和加载.md)
- [分布式系统数据分布策略.md](./document/md/redis/2020-11-24-分布式系统数据分布策略.md)
- [redis6.x启动过程-持续更新](./document/md/redis/redis6.x启动过程.md)
- [redis字符串操作实现-持续更新](./document/md/redis/2020-12-02-redis字符串操作实现.md)
- [redis6强制多线程调试](./document/md/redis/2020-12-04-Redis6强制多线程调试.md)
- [Redis集群技术介绍](./document/md/redis/2020-12-05-Redis集群技术介绍.md)
- [Redis-Cluster原理](./document/md/redis/2020-12-07-Redis-Cluster原理.md)
- [Redis中字典的实现](./document/md/redis/2020-12-15-Redis中字典的实现.md)
- [intset集合实现原理](./document/md/redis/2020-12-16-intset集合实现原理.md)
- [iRedis中的redisCommandTable如何被使用的](./document/md/redis/Redis中的redisCommandTable如何被使用的.md)



## memcached篇
- [memcached介绍](./document/md/memcached/memcached基本介绍.md)
- [memcached线程模型分析](./document/md/memcached/memcached线程模型分析.md)
  


## 文件解压缩

```
//分割大文件为小文件
split -b 30m  redis.tar.gz  redis.tar.gz.  

//合并小文件为大文件
cat  redis.tar.gz.a*  > redis.tar.gz
```