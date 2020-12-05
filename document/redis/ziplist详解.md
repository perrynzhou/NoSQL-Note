## ziplist详解

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com |2020/06/29 |中国开源存储技术交流群(672152841) |

### 介绍
- ziplist本质是一个字节数组，为了节约内存的使用而设计线性结构。当存储的元素个数比较少，元素都是短字符时候，redis使用压缩列表作为底层数据存储的结构，ziplist实现是在ziplist.h/ziplist.c中

### ziplist具体应用在哪些命命令上？

  ![redis-encode-object](../images/redis-encode-object.jpg)
#### 哈希中如何使用ziplist

- 想要知道hset/hmset命令如果使用ziplist，需要搞明白hset/hmset命令处理逻辑，如下是hset命令的处理入口
```
void hsetCommand(client *c) {
    //hset命令处理逻辑
}
```
- 哈希编码方式，目前在redis 6中哈希编码有ziplist和hashtable两种方式，具体在源码中描述如下
```

int hashTypeSet(robj *o, sds field, sds value, int flags) {
    if (o->encoding == OBJ_ENCODING_ZIPLIST) {
        //do something
    } else if (o->encoding == OBJ_ENCODING_HT) {
        //do something
    } else{
        serverPanic("Unknown hash encoding");
}
```



### ziplist实现插入的调试信息

```
(gdb) bt
#0  __ziplistInsert (zl=0x2b0f02077c10 "\v", p=0x2b0f02077c1a "\377", s=0x2b0f02134c43 "name", slen=4) at ziplist.c:755
#1  0x00000000004454b1 in ziplistPush (zl=0x2b0f02077c10 "\v", s=0x2b0f02134c43 "name", slen=4, where=1) at ziplist.c:959
#2  0x0000000000476d30 in hashTypeSet (o=0x2b0f02077c30, field=0x2b0f02134c43 "name", value=0x2b0f0200e373 "perryn", flags=0) at t_hash.c:229
#3  0x0000000000477a78 in hsetCommand (c=0x2b0f0211ce80) at t_hash.c:543
#4  0x00000000004383bd in call (c=0x2b0f0211ce80, flags=15) at server.c:3334
#5  0x00000000004394e9 in processCommand (c=0x2b0f0211ce80) at server.c:3750
#6  0x000000000044aa29 in processCommandAndResetClient (c=0x2b0f0211ce80) at networking.c:1828
#7  0x000000000044ac7c in processInputBuffer (c=0x2b0f0211ce80) at networking.c:1910
#8  0x000000000044b057 in readQueryFromClient (conn=0x2b0f020150c0) at networking.c:1996
#9  0x00000000004e6f0a in callHandler (conn=0x2b0f020150c0, handler=0x44acfc <readQueryFromClient>) at connhelpers.h:79
#10 0x00000000004e7597 in connSocketEventHandler (el=0x2b0f0200b480, fd=8, clientData=0x2b0f020150c0, mask=1) at connection.c:285
#11 0x000000000042f47b in aeProcessEvents (eventLoop=0x2b0f0200b480, flags=27) at ae.c:479
#12 0x000000000042f672 in aeMain (eventLoop=0x2b0f0200b480) at ae.c:539
#13 0x000000000043d3f3 in main (argc=1, argv=0x7ffe6193eb08) at server.c:5310
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000444939 in ziplistNew at ziplist.c:579
        breakpoint already hit 2 times
2       breakpoint     keep y   0x000000000044580d in ziplistInsert at ziplist.c:1055
3       breakpoint     keep y   0x0000000000444e26 in __ziplistInsert at ziplist.c:744
        breakpoint already hit 1 time
4       breakpoint     keep y   0x0000000000445473 in ziplistPush at ziplist.c:958
5       breakpoint     keep y   0x00000000004e7457 in connSocketEventHandler at connection.c:249
6       breakpoint     keep y   0x00000000004e6eea in callHandler at connhelpers.h:78
7       breakpoint     keep y   0x000000000044ad0f in readQueryFromClient at networking.c:1927
8       breakpoint     keep y   0x000000000044aa78 in processInputBuffer at networking.c:1845
9       breakpoint     keep y   0x000000000044aa0b in processCommandAndResetClient at networking.c:1826
10      breakpoint     keep y   0x0000000000438a7a in processCommand at server.c:3508
11      breakpoint     keep y   0x00000000004382bd in call at server.c:3310
12      breakpoint     keep y   0x000000000047798c in hsetCommand at t_hash.c:531
13      breakpoint     keep y   0x0000000000476c0e in hashTypeSet at t_hash.c:203
14      breakpoint     keep y   0x0000000000445473 in ziplistPush at ziplist.c:958
```