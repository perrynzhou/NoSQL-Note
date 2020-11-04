### memcached 线程模型分析

| author | update |
| ------ | ------ |
| perrynzhou@gmail.com | 2020/06/24 |

### memcached 有几种类型的线程?
memcached有2类线程，一类是main线程，一类是worker线程，main线程负责监听listenfd放到main_base中进行监听，一旦listenfd中有IO事件,也就是有新的连接进来，则转发通过dispatch_conn_new函数初始化一个CQ_ITEM，把对应的clientfd封装在CQ_ITEM中，同时把item放到线程的CQ_ITEM队列中，然后写入标记数据到线程的notify_send_fd中，worker线程则是监听工作线程中的notify_recv_fd中的IO事件，根据读取到的标记数据来初始化一个新连接，接着处理这个链接，释放CQ_ITEM

### memcached中工作线程如何和main线程交互的
工作线程会有一个管道来实现和main线程进行通信的，工作线程通过pipe系统调用初始化一对FD。

- memcached中的工作线程的结构
```
	typedef struct {
		pthread_t thread_id;        /* 工作线程ID*/
		struct event_base *base;    /* libevent的句柄 */
		struct event notify_event;  /* listen event */
		int notify_receive_fd;      /* 客户端线程监听这个套接字接受来自服务端通知，比如新连接 */
		int notify_send_fd;         /* 主线程有消息，会往这个fd中进行写入数据 */
		struct thread_stats stats;  /* 线程的状态 */
		struct conn_queue *new_conn_queue; /* 该队列存储CQ_ITEM把服务端接受到的客户端fd状态为CQ_ITEM，放到这个队列中 */
		cache_t *resp_cache;        /* response objects */
		cache_t *rbuf_cache;        /* static-sized read buffers */
		cache_t *io_cache;          /* IO objects */
		void *storage;              /* data object for storage system */
		logger *l;                  /* logger buffer */
		void *lru_bump_buf;         /* async LRU bump buffer */
		char   *ssl_wbuf;
	} LIBEVENT_THREAD;
```
-  memcached的工作线程的初始化函数
```
	//memcached的工作线程的初始化函数
	void memcached_thread_init(int nthreads, void *arg) {
	//每个工作线程的初始化
		threads = calloc(nthreads, sizeof(LIBEVENT_THREAD));
		if (! threads) {
	    perror("Can't allocate thread descriptors");
	    exit(1);
		}
	
		for (i = 0; i < nthreads; i++) {
	    	int fds[2];
	    	if (pipe(fds)) {
	        perror("Can't create notify pipe");
	        exit(1);
	    	}
	    //当有新连接时候，主线程会选择性的写数据到线程的notify_send_fd，然后工作线程在从notify_receive_fd读取数据，然后取出连接开始处理
			//工作线程从notify_receive_fd读取数据
	    threads[i].notify_receive_fd = fds[0];
		//主线程从notify_send_fd写入数据
	    threads[i].notify_send_fd = fds[1];
		//设置每个线程中event监听notify_receive_fd中的数据
	    setup_thread(&threads[i]);
	    /* Reserve three fds for the libevent base, and two for the pipe */
	    stats_state.reserved_fds += 5;
	}
	
	/* Create threads after we've done all the libevent setup. */
	//启动每个工作线程,每个线程执行event_base_loop
	for (i = 0; i < nthreads; i++) {
	    create_worker(worker_libevent, &threads[i]);
	}
}
```

### memcached主线程如何初始化服务端的socket?

在这里我们需要大概了解下main函数的执行逻辑前基本介绍全局的数据结构如下：

- listen_conn:全局的监听套接字的连接
- max_fds:默认最大支持的连接数
- main_base:主线程的libevent的核心结构体
- conns:全局的连接数组，类型是conn指针类型
	
- 基本的执行逻辑如下
	```
	//函数入口，在memcached的入口
	main 
	//初始化全局的conns,max_fds = settings.maxconns + headroom + next_fd;
	conn_init();
	//main线程的event初始化
	main_base = event_init();
	//和server_socket类型，不同的仅仅是制定网卡绑定
	server_sockets 
		//main线程通过socket/bind/listen来初始化listen_conn，设置该连接的类型，listen_conn中的sfd放到main_base监听
		server_socket 
			//根据sfd初始化一个conn放到main_base中，设置回函数event_handler
			conn_new 
	//初始化工作线程
	memcached_thread_init 
		//设置工作线程，把notify_receive_fd放到工作线程的event_base中监听
		setup_thread 
			//设置notify_receive_fd中IO事件处理函数
			thread_libevent_process 
		//启动单独线程开始监听notify_receive_fd的IO时间
		worker_libevent 
	event_base_loop
	event_base_free
	```
	
### memcached主线程如何处理一个新来的连接
- 服务端的socket(listenfd)监听套接字封装成conn,设置该连接状态为conn_listening，放到main_base中，一旦有listenfd有事件，event_handler就会被触发，则调用drive_machine函数接受客户单请求套接字
```
//针对main线程，把服务端的listenfd放到main_base中，针对工作线程则注册
event_handler notify_receive_fd注册到event_base中
	drive_machine //接受来自客户端IO请求，这是最重要的函数
		dispatch_conn_new //把请求的FD封装到CQ_ITEM中，然后放到工作线程的队列中，然后通知工作线程有新连接进去
```
- 服务端调用drive_machine接受客户端fd后，调用dispatch_conn_new函数客户端的FD封装成CQ_ITEM
- 客户端的FD封装成CQ_ITEM后，根据哈希取模策略选择一个线程，把CQ_ITEM放到该工作线程的new_conn_queue中，然后往该线程的notify_send_fd写入数据
- 由于工作线程在memcached启动后就已经初始化几个工作线程，然后每个线程中调用thread_libevent_process监听该线程中的notify_recv_fd中的IO事件，该工作线程从notify_recv_fd中读取到了数据，从自己线程队列中取出一个CQ_ITEM,初始化conn，放到全局conns数组中，这个数据的下表就是客户请求的FD，然后释放CQ_ITEM资源，到此整个memcached的从主线程接受请求，到工作线程接受和初始化conn结束，后面就是处理这个客户端请求

### memcached处理客户端请求的连接流程是怎么样的？

- 1.listenfd封装为conn，走conn_new函数，listenfd的conn结构的状态为conn_listening
- 2.conn_new函数初始化一个listenfd的conn,同时把conn放到去全局的conns的数组中，把conn中的fd放到miab_base中监听,IO触发函数event_handler
- 3.event_handler 调用 drive_machine函数
- 4.drive_machine函数根据conn的状态，如果conn的状态是conn_listening，则accept这个请求，然后传递客户端的请求的fd到dispatch_conn_new
- 5.dispatch_conn_new函数选择一个工作线程，把初始化一个CQ_ITEM放到这个工作线程的队列中，然后往这个工作线程的notify_send_fd写入一个标记数据
- 6.由于之前每个工作线程监听了notify_recieved_fd，然后调用thread_libevent_process函数从线程队列中取出一个item，然后根据item初始化一个conn放到该线程的event_base，同时设置回调函数为event_handler
- 7.当这个请求有IO进来时候，调用event_handler函数，触发drive_machine函数，该函数根据conn的状态来进行对应的IO操作
- 8.需要说明如下两点
```
1.main_base监听listenfd的IO事件，触发函数event_handler
2.工作线程中的base监听notify_receive_fd和客户端的fd,针对notify_receive_fd设置thread_libevent_process 函数；针对客户端的FD设置event_handler的回调函数
```
### memcached注释
- [1.memcached.c注释](../memcached-1.6.6/memcached.c)
