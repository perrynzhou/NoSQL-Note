## 2020-11-11-redisrebloom源码编译和加载

| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/11/11 |

- download redisrebloom source
```
$ wget https://github.com/RedisBloom/RedisBloom/archive/v2.2.4.tar.gz
```

- make 

```
$ tar zxvf v2.2.4.tar.gz  && cd RedisBloom-2.2.4
$ make CFLAGS="-ggdb3 -O0" 
$ ls
changelog  contrib  Dockerfile  docs  LICENSE  Makefile  mkdocs.yml  ramp.yml  README.md  redisbloom.so  rmutil  src  tests

```

- stop redis and config redisrebloom

```
$ echo "loadmodule /root/zhoulin/RedisBloom-2.2.4/redisbloom.so" >> /root/zhoulin/redis-6.0.7/redis.conf

$ redis-server /root/zhoulin/redis-6.0.7/redis.conf
```


- restart redis
```
nohup redis-server /root/zhoulin/redis-6.0.7/redis.conf &
```

- redisrebloom usage

```
$ redis-cli
127.0.0.1:6379> BF.ADD newFilter foo
(integer) 1
127.0.0.1:6379> bf.add test u1
(integer) 1
127.0.0.1:6379> bf.add test u2
(integer) 1
127.0.0.1:6379> bf.exists test u1
(integer) 1
127.0.0.1:6379> bf.exists test u2
(integer) 1
127.0.0.1:6379> bf.exists test u4
(integer) 0
127.0.0.1:6379> bf.exists test u100
(integer) 0
127.0.0.1:6379> 
```