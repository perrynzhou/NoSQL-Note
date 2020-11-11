## redis 字符串实现分析


| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/05/24 |



- redis字符串定义结构如下

```
typedef char *sds;

//按照一个字节对齐
struct __attribute__ ((__packed__)) sdshdr5 {
    //flags的低3位保存类型(2的三次方)，高5位保存长度(2的五次方)
    unsigned char flags; 
    //用来存储长度<31字符串长度 
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    //len保存字符串使用的长度
    uint8_t len; 
    //buf中申请的长度
    uint8_t alloc; 
    //flags低3位保存类型，其他5位保留未使用
    unsigned char flags;
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; 
    uint16_t alloc; 
    unsigned char flags; 
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; 
    uint32_t alloc; 
    unsigned char flags; 
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; 
    uint64_t alloc; 
    unsigned char flags; 
    char buf[];
};
```

- redis为什么要定义5种类型的字符串？
  - 核心是为了节省内存，不同类型的字符串存储不同长度的，sdshdr5用来存储长度小于31的字符串，sdshdr8存储长度小于2的8次方的字符串等，这样做的好处是节省内存，在kv系统中,字符串存储更加灵活,同时达到了空间节省的目的，具体判断字符串长度的方法如下
    ```
    static inline char sdsReqType(size_t string_size) {
        if (string_size < 1<<5)
            return SDS_TYPE_5;
        if (string_size < 1<<8)
            return SDS_TYPE_8;
        if (string_size < 1<<16)
            return SDS_TYPE_16;
    #if (LONG_MAX == LLONG_MAX)
        if (string_size < 1ll<<32)
            return SDS_TYPE_32;
        return SDS_TYPE_64;
    #else
        return SDS_TYPE_32;
    #endif
    }
    ```

- redis提供了哪几种常用的字符串操作
  - sdsnewlen:为init的字符串长度,从sdsReqType找到对应存储结构，然后进行初始化操作
  - sdsnew:判断字符串是否为空，然后调用sdsnewlen为字符串初始化
  - sdscat:append字符串到sds,首选判断当前sds剩余空间是否满足当前字符串，不满足需要扩容；满足直接进行字符串的append
  - sdscpy:拷贝字符串到sds
  - sdscatvprintf:按照格式化的要求来append字符串到sds
  - sdscatfmt:解析format字符串,根据fmt的格式化要求来append字符串到sds
  - sdstrim:剔除sds中自动的字符串集合
  - sdsclear:清空sds的内内容
  - sdsfree:释放sds结构
