## memcached 基本介绍 

### memcached应用场景
- memcache主要应用在于减少数据库的压力场景，第一次访问数据未命中，从数据库加到并存储到memcache中，第二次直接访问memcached获取数据

### memcached特性
- 协议简单，memcached客户端和服务通信协议采用文本或者二进制协议
- 基于libevent事件处理，libevent将常用的IO复用函数封装在libevent中，memcached采用这个库能在IO事件处理上获取高性能
- 纯内存的方式,memcached所有的数据都存储在memcached的内存中，memcached重启会导致数据全部丢失


### memcached存在的一些问题

- 无法备份，重后无法恢复，由于纯内存的粗出方式，一旦memcached进程死掉,内存中的数据立马丢失,同时也无法备份
- 无法查询,memcached无法提供范围查询，仅仅能支持set/get存储命令
- 没有内置的高可用方案，存在单点故障的风险

### memcached内存申请方式

- memcached采用slab allcation方式，按照预先规定的大小，将分配的内存分割为各种尺寸块，把相同大小的内存块分到一组。page是memcached分配给slab的空间，默认是1M；chunk是用于缓存记录的空间，slab class是特定大小的chunk组。当memcached收到数据大小时候，选择最适合的大小的slab存储
- memcached slab allcation解决了内存碎片，但是也会到来内存浪费的问题。比如99个字节使用了112个chunk，剩余12个字节就浪费了