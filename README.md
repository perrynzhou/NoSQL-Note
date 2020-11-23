# NoSQL存储系统 阅读笔记

| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/05/24 |


## 目标

- 梳理redis/memcached的读写流程、复制详细过程
- redis模块原理和源码分析


## redis篇
- [分布式系统基本理论](./document/redis/分布式系统基本理论.md)
- [缓存的基本介绍](./document/redis/缓存的基本介绍.md)
- [缓存系统中的高并发](./document/redis/缓存系统中的高并发.md)
- [事务一致性解决方案](./document/redis/2020-11-04-事务一致性解决方案.md)
- [redis调试环境](./document/redis/redis源码调试环境.md)
- [redis支持的数据结构和编码方式](./document/redis/redis支持的几种数据结构和编码方式.md)
- [redis字符串的实现原理](./document/redis/redis字符串的实现原理.md)
- [redis客户端问题案例](./document/redis/Redis客户端问题案例.md)
- [RedisBloom源码编译和加载](./document/redis/2020-11-11-redisbloom源码编译和加载.md)
- [分布式系统数据分布策略.md](./document/redis/2020-11-24-分布式系统数据分布策略.md)

## memcached篇
- [memcached介绍](./document/memcached/memcached基本介绍.md)
- [memcached线程模型分析](./document/memcached/memcached线程模型分析.md)
  


## 文件解压缩

```
//分割大文件为小文件
split -b 30m  redis.tar.gz  redis.tar.gz.  

//合并小文件为大文件
cat  redis.tar.gz.a*  > redis.tar.gz
```