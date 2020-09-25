# kv存储系统 阅读笔记

| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/05/24 |


## 目标

- 梳理redis/memcached的读写流程、复制详细过程

- 针对该版本在代码层面梳理一份有C语言基础的同学都可以读懂


## 分布式系统基本理论
- [1.分布式系统基本理论](./document/分布式系统原理/分布式系统基本理论.md)
- [2.缓存的基本介绍](./document/分布式系统原理/缓存的基本介绍.md)

## memcached分析
- [1.memcached介绍](./document/memcached分析/memcached基本介绍.md)

- [2.memcached线程模型分析](./document/memcached分析/memcached线程模型分析.md)

## redis分析
- [1.redis调试环境](./document/redis分析/redis源码调试环境.md)
- [2.redis支持的数据结构和编码方式](./document/redis分析/redis支持的几种数据结构和编码方式.md)
- [3.redis字符串的实现原理](./document/redis分析/redis字符串的实现原理.md)

## redis运维
- [1.redis客户端问题案例](./document/redis运维/Redis客户端问题案例.md)