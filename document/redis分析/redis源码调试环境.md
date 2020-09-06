
## redis源码调试

### 系统GCC要求

- redis6.x需要gcc 5.3以上的版本,因此需要升级gcc版本
  ```
  $ yum install centos-release-scl scl-utils-build
  $ yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils tcl
  $ scl enable devtoolset-9 bash
  ```
### 源码下载
```
$ wget http://download.redis.io/releases/redis-6.0.7.tar.gz
```

### 源码编译
```
$ tar zxvf redis-6.0.6.tar.gz && cd redis-6.0.7
$ make CFLAGS="-ggdb3 -O0" && make install
```

### 环境验证

```
[root@CentOS1 ~/Debug/redis-6.0.7]$ gdb redis-server
Reading symbols from /usr/local/bin/redis-server...done.
(gdb) list
5120                return zmalloc_test(argc, argv);
5121            }
5122
5123            return -1; /* test not found */
5124        }
5125    #endif
5126
5127        /* We need to initialize our libraries, and the server configuration. */
5128    #ifdef INIT_SETPROCTITLE_REPLACEMENT
5129        spt_init(argc, argv);
(gdb) 
```