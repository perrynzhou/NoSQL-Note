## Redis中的redisCommandTable如何被使用的

| 作者 | 时间 |QQ技术交流群 |
| ------ | ------ |------ |
| perrynzhou@gmail.com | 2020/11/01 |中国开源存储技术交流群(672152841) |

### 介绍

- redis中的大部分操作命令都是通过redisCommandTable进行定义的，比如set 命令会在redisCommandTable中定义对应的操作函数。当用户在redis-cli中输入set命令，经过网络传输到redis-server,然后校验命令、解析命令、最终调用setCommand函数把key和value按照用户输入插入到redis db中，完成此次set命令的操作。如下简单举例列举了redisCommandTable的2个命令

  ```
  // 定义在redis-6.0.9/src/server.c中,目前redis 6中有205个命令
  struct redisCommand redisCommandTable[] = {
  		/************其他命令*********/
      {"get",getCommand,2,
       "read-only fast @string",
       0,NULL,1,1,1,0,0,0},
  
      {"set",setCommand,-3,
       "write use-memory @string",
       0,NULL,1,1,1,0,0,0},
       
       /************其他命令*********/
  }
  ```

### redisCommandTable定义
```
typedef void redisCommandProc(client *c);
struct redisCommand {
		// redis中命令的名称
    char *name;
    // 回调函数，参数是客户端结构体
    redisCommandProc *proc;
    int arity;
    // redis命令中的flag的字符串表现形式,标识命令是读还是写
    char *sflags;   
    // 命令中的实际的flag，标识命令是读还是写
    uint64_t flags;
    // 在redis cluster中用于重定向
    redisGetKeysProc *getkeys_proc;
  
    int firstkey; 
    int lastkey;  
    int keystep; 
    // 从服务启动到现在命令执行总时间和总次数
    long long microseconds, calls;
    // 用于acl中，表示为命令的id
    int id;    
};
```

###  Redis命令重命名

- 禁用危险命令的用法
```
rename-command FLUSHALL ""

```
- 禁用危险命令的实现

```
int main(int argc, char **argv) {
	loadServerConfig(configfile,options);
}
void loadServerConfig(char *filename, char *options){
	loadServerConfigFromString(config);
}
void loadServerConfigFromString(char *config) {
	// 查找配置文件redis.conf中的是否配置有rename-command xxx ""
	else if (!strcasecmp(argv[0],"rename-command") && argc == 3) {
			 // 查找 xxx命令，如果不存在直接报错
       struct redisCommand *cmd = lookupCommand(argv[1]);
       int retval;

       if (!cmd) {
           err = "No such command in rename-command";
           goto loaderr;
       }
       // 成功查找到xxx命令,然后从server.commands中删除
       retval = dictDelete(server.commands, argv[1]);
       serverAssert(retval == DICT_OK);

       // 判断配置文件中 rename-command xxx "" 配置中第三个参数
       if (sdslen(argv[2]) != 0) {
           sds copy = sdsdup(argv[2]);
					// 把xxx命令替换为 "",然后被原来xxx对应的 redisCommand进行绑定。从而达到
           retval = dictAdd(server.commands, copy, cmd);
           if (retval != DICT_OK) {
               sdsfree(copy);
               err = "Target command name already exists"; goto loaderr;
           }
       }
  }
}
```
### redis命令加载过程

- redis命令加载流程函数调用链
```
// redis启动的入口函数
1.main

// 通过读取redis.conf来初始化redis配置
2.initServerConfig

// redis命令初始化
3.populateCommandTable
4.loadServerConfig
```
- main 函数
```
int main(int argc, char **argv) {
	// 基于内存无任何配置下的默认初始化
	initServerConfig(){
		// 初始化server.commands和server.orig_commands，把redis命令和对应的struct redisCommand建立kv结构分别存储在这2个结构中
    populateCommandTable();
	}
	// 加载redis.conf 读取配置，然后进行更新server config信息
	loadServerConfig(configfile,options);
}
```
- initServerConfig 函数

```
// initServerConfig主要是初始化struct redisServer server结构
void initServerConfig(void) {
    server.runid[CONFIG_RUN_ID_SIZE] = '\0';
    changeReplicationId();
    clearReplicationId2();
    /******************省略******************************/
    
    // 为struct redisServer server中的命令设置默认值
    server.commands = dictCreate(&commandTableDictType,NULL);
    server.orig_commands = dictCreate(&commandTableDictType,NULL);
    populateCommandTable();
    server.delCommand = lookupCommandByCString("del");
    server.multiCommand = lookupCommandByCString("multi");
    server.lpushCommand = lookupCommandByCString("lpush");
    server.lpopCommand = lookupCommandByCString("lpop");
    server.rpopCommand = lookupCommandByCString("rpop");
    server.zpopminCommand = lookupCommandByCString("zpopmin");
    server.zpopmaxCommand = lookupCommandByCString("zpopmax");
    server.sremCommand = lookupCommandByCString("srem");
    server.execCommand = lookupCommandByCString("exec");
    server.expireCommand = lookupCommandByCString("expire");
    server.pexpireCommand = lookupCommandByCString("pexpire");
    server.xclaimCommand = lookupCommandByCString("xclaim");
    server.xgroupCommand = lookupCommandByCString("xgroup");
    server.rpoplpushCommand = lookupCommandByCString("rpoplpush");

    
    /******************省略******************************/
}
```
- populateCommandTable 函数
```
// 把redisCommandTable中的redisCommand，初始化到server.commands和server.orig_commands两个结构中，server.commands用于redis每次执行命令时候的命令查找，server.orig_commands用于redis acl功能的命令查找
void populateCommandTable(void) {
    int j;
    int numcommands = sizeof(redisCommandTable)/sizeof(struct redisCommand);

    for (j = 0; j < numcommands; j++) {
        struct redisCommand *c = redisCommandTable+j;
        int retval1, retval2;
			 // 命令的设定特殊的flag 
        if (populateCommandTableParseFlags(c,c->sflags) == C_ERR)
            serverPanic("Unsupported command flag");
				// 把当前命令插入到acl的rax tree中，返回返回ID
        c->id = ACLGetCommandID(c->name); 
        // 把当前的命令字符串作为key,struct redisCommand *c作为value插入到server.commands字典中
        retval1 = dictAdd(server.commands, sdsnew(c->name), c);
        
        //  把当前的命令字符串作为key,struct redisCommand *c作为value插入到server.orig_commands字典中，这个用于redis中的acl功能
        retval2 = dictAdd(server.orig_commands, sdsnew(c->name), c);
        serverAssert(retval1 == DICT_OK && retval2 == DICT_OK);
    }
}

```

- loadServerConfig 函数

```
// 根据redis.conf更新或者重写 struct redisServer server 中的配置选项，这一切准备完毕后，redis就可以启动了
void loadServerConfig(char *filename, char *options) {
	loadServerConfigFromString(config);
}
```