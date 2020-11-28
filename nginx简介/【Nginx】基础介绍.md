# 1.什么是Nginx
- Nginx ，是一个 Web 服务器和反向代理服务器，用于 HTTP、HTTPS、SMTP、POP3 和 IMAP 应用层协议。
目前使用的最多的 Web 服务器或者代理服务器，像淘宝、新浪、网易、迅雷等都在使用。

- Nginx 的主要功能如下：
作为 http server (代替 Apache ，对 PHP 需要 FastCGI 处理器支持)

- FastCGI：Nginx 本身不支持 PHP 等语言，但是它可以通过 FastCGI 来将请求扔给某些语言或框架处理。
- 反向代理服务器
- 实现负载均衡
- 虚拟主机
# 2.Fastcgi 与 cgi 的区别？
## 1）cgi

web 服务器会根据请求的内容，然后会 fork 一个新进程来运行外部 c 程序（或 perl 脚本…）， 这个进程会把处理完的数据返回给 web 服务器，最后 web 服务器把内容发送给用户，刚才 fork 的进程也随之退出。
如果下次用户还请求改动态脚本，那么 web 服务器又再次 fork 一个新进程，周而复始的进行。
## 2）fastcgi

web 服务器收到一个请求时，他不会重新 fork 一个进程（因为这个进程在 web 服务器启动时就开启了，而且不会退出），web 服务器直接把内容传递给这个进程（进程间通信，但 fastcgi 使用了别的方式，tcp 方式通信），这个进程收到请求后进行处理，把结果返回给 web 服务器，最后自己接着等待下一个请求的到来，而不是退出。
? 综上，差别在于是否重复 fork 进程，处理请求。

# 3.Nginx 有哪些优点？
·跨平台、配置简单。

·非阻塞、高并发连接

·处理 2-3 万并发连接数，官方监测能支持 5 万并发。
·内存消耗小

·开启 10 个 Nginx 才占 150M 内存。
·成本低廉，且开源。

·稳定性高，宕机的概率非常小。

# 4.使用“反向代理服务器”的优点是什么？

- 反向代理服务器可以隐藏源服务器的存在和特征。
它充当互联网云和 Web 服务器之间的中间层。
这对于安全方面来说是很好的，特别是当我们使用 Web 托管服务时。

## 什么是正向代理？
- 一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。
客户端才能使用正向代理。
正向代理总结就一句话：代理端代理的是客户端。例如说：? 我们使用的翻墙软件，OpenVPN 等等。

## 什么是反向代理？

- 反向代理（Reverse Proxy）方式，是指以代理服务器来接受 Internet上的连接请求，然后将请求，发给内部网络上的服务器并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。
反向代理总结就一句话：代理端代理的是服务端。

# 5.Nginx 和 Apache 之间的不同点？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128010421716.png)
·轻量级，同样起 web 服务，Nginx 比 Apache 占用更少的内存及资源。
·抗并发，Nginx 处理请求是异步非阻塞的，而 Apache 则是阻塞型的，在高并发下 Nginx 能保持低资源低消耗高性能。
·最核心的区别在于 Apache 是同步多进程模型，一个连接对应一个进程；Nginx 是异步的，多个连接（万级别）可以对应一个进程。
·Nginx 高度模块化的设计，编写模块相对简单。
·Nginx处理静态文件好，耗费内存少，只适合静态和反向。
Apache在处理动态有优势，nginx并发性比较好，CPU占用内存低，
如果rewrite频繁，选用apache最佳。

# 6.Nginx 如何处理 HTTP 请求？
- 首先，Nginx 在启动时，会解析配置文件，得到需要监听的端口与 IP 地址，然后在 Nginx 的 Master 进程里面先初始化好这个监控的Socket(创建 Socket，设置 addr、reuse 等选项，绑定到指定的 ip 地址端口，再 listen 监听)。

- 然后，再 fork(一个现有进程可以调用 fork 函数创建一个新进程。由 fork 创建的新进程被称为子进程 )出多个子进程出来。

- 之后，子进程会竞争 accept 新的连接。此时，客户端就可以向 nginx 发起连接了。当客户端与nginx进行三次握手，与 nginx 建立好一个连接后。此时，某一个子进程会 accept 成功，得到这个建立好的连接的 Socket ，然后创建 nginx 对连接的封装，即 ngx_connection_t 结构体。
- 接着，设置读写事件处理函数，并添加读写事件来与客户端进行数据的交换。

- 最后，Nginx 或客户端来主动关掉连接，到此，一个连接就结束了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128011037549.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128010636269.png)
# 7.Nginx 是如何实现高并发的？
- 如果一个 server 采用一个进程(或者线程)负责一个request的方式，那么进程数就是并发数。那么显而易见的，就是会有很多进程在等待中。等什么？最多的应该是等待网络传输。
- 而 Nginx 的异步非阻塞工作方式正是利用了这点等待的时间。在需要等待的时候，这些进程就空闲出来待命了。
因此表现为少数几个进程就解决了大量的并发问题。

Nginx是如何利用的呢，简单来说：
同样的 4 个进程，如果采用一个进程负责一个 request 的方式，那么，同时进来 4 个 request 之后，每个进程就负责其中一个，直至会话关闭。期间，如果有第 5 个request进来了。就无法及时反应了，因为 4 个进程都没干完活呢，因此，一般有个调度进程，每当新进来了一个 request ，就新开个进程来处理。

每进来一个 request ，会有一个 worker 进程去处理。但不是全程的处理，处理到什么程度呢？处理到可能发生阻塞的地方，比如向上游（后端）服务器转发 request ，并等待请求返回。那么，这个处理的 worker 不会这么傻等着，他会在发送完请求后，注册一个事件：“如果 upstream 返回了，告诉我一声，我再接着干”。于是他就休息去了。此时，如果再有 request 进来，他就可以很快再按这种方式处理。而一旦上游服务器返回了，就会触发这个事件，worker 才会来接手，这个 request 才会接着往下走。

## 为什么 Nginx 不使用多线程？
- Apache: 创建多个进程或线程，而每个进程或线程都会为其分配 cpu 和内存（线程要比进程小的多，所以 worker 支持比 perfork 高的并发），并发过大会榨干服务器资源。

- Nginx: 采用单线程来异步非阻塞处理请求（管理员可以配置 Nginx 主进程的工作进程的数量）(epoll)，不会为每个请求分配 cpu 和内存资源，节省了大量资源，同时也减少了大量的 CPU 的上下文切换。所以才使得 Nginx 支持更高的并发。
# 8.什么是动态资源、静态资源分离？
动态资源、静态资源分离，是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后我们就可以根据静态资源的特点将其做缓存操作（server push服务器推送），这就是网站静态化处理的核心思路。

动态资源、静态资源分离简单的概括是：动态文件与静态文件的分离。

## 为什么要做动、静分离？
- 在我们的软件开发中，有些请求是需要后台处理的（如：.jsp,.do 等等），有些请求是不需要经过后台处理的（如：css、html、jpg、js 等等文件），这些不需要经过后台处理的文件称为静态文件，否则动态文件。
- 因此我们后台处理忽略静态文件。这会有人又说那我后台忽略静态文件不就完了吗？当然这是可以的，但是这样后台的请求次数就明显增多了。在我们对资源的响应速度有要求的时候，我们应该使用这种动静分离的策略去解决动、静分离将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问

- 我们也将静态资源放到 Nginx 中，动态资源转发到 Tomcat 服务器中去。

## HTTP/2 多路复用——基于帧的流传输
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128011412814.png?)
**建立HTTP连接**：
- stream id = 0，client的2个frame（一个settings一个window-update）、server的3个frame（一个settings一个window-update一个ACK）

- client通过URL建立stream id 0设置请求头部大小、窗口大小、客户端的最大并发数大小，更新接收端窗口
- server返回stream id 0设置服务器的最大并发数、流缓冲容量、帧最大值、更新发送端窗口，并发送代表ACK的flag状态码表示认同client的settings

**数据传输**
- client建立stream id = 奇数，只发请求的headers，
- server先发headers再发data，stream id 都是= 奇数

- 中途server要给client进行server push提前加载的话，
- stream id就为偶数

## Server Push
当我们使用HTTP/1.1的时候，同域下Chrome最多同时加载6个TCP连接：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128011630247.png)
- 我们观察到图片的加载完成时间是从上往下的，这个就应该是上面提7到的优先级依赖影响的，后面的图片会依赖于前面的图片，所以前面的图片优先级会更高一点，优先传递。
时间线里面绿色的是表示Waiting TTFB（Time To First Byte）的时间，即发出请求之后到收到第一个字节所需要的时间，蓝色是内容下载时间。这里可以看到等待时间TTFB依次增长。

 - 虽然使用了HTTP/2没有了6个的限制，但是我们发现css/js需要在html解析了之后才能触发加载，而图片是通过JS的new Image触发加载，所以它们需要等到JS下载完并解析好了才能开始加载。
- 所以Server Push就是为了解决这个加载延迟问题，提前把网页需要的资源Push给浏览器。Nginx 1.13.9版本开始支持，给nginx.conf添加以下配置：指定需要Push的资源然后观察加载的时间线：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128011707969.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128011721114.png)
## CDN 服务
CDN ，即内容分发网络。

其目的是，通过在现有的 Internet中 增加一层新的网络架构，将网站的内容发布到最接近用户的网络边缘，使用户可就近取得所需的内容，提高用户访问网站的速度。

一般来说，因为现在 CDN 服务比较大众，所以基本所有公司都会使用 CDN 服务。
# 9.Nginx 有哪些负载均衡策略？
## 什么叫负载均衡？
负载均衡，即是代理服务器将接收的请求均衡的分发到各服务器中。

Nginx 默认提供了 3 种负载均衡策略：
1. 轮询（默认）round_robin
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

2. IP 哈希 ip_hash
每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 共享的问题。
当然，实际场景下，一般不考虑使用 ip_hash 解决 session 共享。

3. 最少连接 least_conn
下一个请求将被分派到活动连接数量最少的服务器

等等

## 什么叫惊群？
- 惊群是指多个进程/线程在等待同一资源时，每当资源可用，所有的进程/线程都来竞争资源的现象。

- Nginx采用的是多进程的模式。假设Linux系统是2.6版本以前，
当有一个客户端要连到Nginx服务器上，
Nginx的N个进程都会去监听socket的accept的，
如果全部的N个进程都对这个客户端的socket连接进行了监听，就会造成资源的竞争甚至数据的错乱。
我们要保证的是，一个连接在Nginx的一个进程上处理，包括accept和read/write事件

## 事件分配策略
- Nginx的N个进程会争抢文件锁，当只有拿到文件锁的进程，才能处理accept的事件。
- 没有拿到文件锁的进程，只能处理当前连接对象的read事件
当单个进程总的connection连接数达到总数的7/8的时候，就不会再接收新的accpet事件。
- 如果拿到锁的进程能很快处理完accpet，而没拿到锁的一直在等待（等待时延：ngx_accept_mutex_delay），容易造成进程忙的很忙，空的很空。
### ngx_process_events_and_timers


此方法为进程实现的核心函数。
主要作用：事件分发；惊群处理；简单的负载均衡。

负载均衡：
- 当事件配置初始化的时候，会设置一个全局变量：ngx_accept_disabled = ngx_cycle->connection_n / 8 - ngx_cycle->free_connection_n;
当ngx_accept_disabled为正数的时候，connection达到连接总数的7/8的时候，就不再处理新的连接accept事件，只处理当前连接的read事件

惊群处理：

- 通过ngx_trylock_accept_mutex争抢文件锁，拿到文件锁的，才可以处理accept事件。
- ngx_accept_mutex_held是拿到锁的一个标志，当拿到锁了，flags会被设置成NGX_POST_EVENTS，这个标志会在事件处理函数ngx_process_events中将所有事件（accept和read）放入对应的ngx_posted_accept_events和ngx_posted_events队列中进行延后处理。
- 当没有拿到锁，调用事件处理函数ngx_process_events的时候，可以明确都是read的事件，所以可以直接调用事件ev->handler方法回调处理。
- 拿到锁的进程，接下来会优先处理ngx_posted_accept_events队列上的accept事件，处理函数：ngx_event_process_posted
处理完accept事件后，就将文件锁释放
- 接下来处理ngx_posted_events队列上的read事件，处理函数：ngx_event_process_posted


```cpp
/**
 * 进程事件分发器
 */
void ngx_process_events_and_timers(ngx_cycle_t *cycle) {
	ngx_uint_t flags;
	ngx_msec_t timer, delta;
 
	if (ngx_timer_resolution) {
		timer = NGX_TIMER_INFINITE;
		flags = 0;
 
	} else {
		timer = ngx_event_find_timer();
		flags = NGX_UPDATE_TIME;
 
#if (NGX_WIN32)
 
		/* handle signals from master in case of network inactivity */
 
		if (timer == NGX_TIMER_INFINITE || timer > 500) {
			timer = 500;
		}
 
#endif
	}
 
	/**
	 * ngx_use_accept_mutex变量代表是否使用accept互斥体
	 * 默认是使用，可以通过accept_mutex off;指令关闭；
	 * accept mutex 的作用就是避免惊群，同时实现负载均衡
	 */
	if (ngx_use_accept_mutex) {
 
		/**
		 * 	ngx_accept_disabled = ngx_cycle->connection_n / 8 - ngx_cycle->free_connection_n;
		 * 	当connection达到连接总数的7/8的时候，就不再处理新的连接accept事件，只处理当前连接的read事件
		 * 	这个是比较简单的一种负载均衡方法
		 */
		if (ngx_accept_disabled > 0) {
			ngx_accept_disabled--;
 
		} else {
			/* 获取锁失败 */
			if (ngx_trylock_accept_mutex(cycle) == NGX_ERROR) {
				return;
			}
 
			/* 拿到锁 */
			if (ngx_accept_mutex_held) {
				/**
				 * 给flags增加标记NGX_POST_EVENTS，这个标记作为处理时间核心函数ngx_process_events的一个参数，这个函数中所有事件将延后处理。
				 * accept事件都放到ngx_posted_accept_events链表中，
				 * epollin|epollout普通事件都放到ngx_posted_events链表中
				 **/
				flags |= NGX_POST_EVENTS;
 
			} else {
 
				/**
				 * 1. 获取锁失败，意味着既不能让当前worker进程频繁的试图抢锁，也不能让它经过太长事件再去抢锁
				 * 2. 开启了timer_resolution时间精度，需要让ngx_process_change方法在没有新事件的时候至少等待ngx_accept_mutex_delay毫秒之后再去试图抢锁
				 * 3. 没有开启时间精度时，如果最近一个定时器事件的超时时间距离现在超过了ngx_accept_mutex_delay毫秒，也要把timer设置为ngx_accept_mutex_delay毫秒
				 * 4. 不能让ngx_process_change方法在没有新事件的时候等待的时间超过ngx_accept_mutex_delay，这会影响整个负载均衡机制
				 * 5. 如果拿到锁的进程能很快处理完accpet，而没拿到锁的一直在等待，容易造成进程忙的很忙，空的很空
				 */
				if (timer == NGX_TIMER_INFINITE
						|| timer > ngx_accept_mutex_delay) {
					timer = ngx_accept_mutex_delay;
				}
			}
		}
	}
 
	delta = ngx_current_msec;
 
	/**
	 * 事件调度函数
	 * 1. 当拿到锁，flags=NGX_POST_EVENTS的时候，不会直接处理事件，
	 * 将accept事件放到ngx_posted_accept_events，read事件放到ngx_posted_events队列
	 * 2. 当没有拿到锁，则处理的全部是read事件，直接进行回调函数处理
	 * 参数：timer-epoll_wait超时时间  (ngx_accept_mutex_delay-延迟拿锁事件   NGX_TIMER_INFINITE-正常的epollwait等待事件)
	 */
	(void) ngx_process_events(cycle, timer, flags);
 
	delta = ngx_current_msec - delta;
 
	ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
			"timer delta: %M", delta);
	/**
	 * 1. ngx_posted_accept_events是一个事件队列，暂存epoll从监听套接口wait到的accept事件
	 * 2. 这个方法是循环处理accpet事件列队上的accpet事件
	 */
	ngx_event_process_posted(cycle, &ngx_posted_accept_events);
 
	/**
	 * 如果拿到锁，处理完accept事件后，则释放锁
	 */
	if (ngx_accept_mutex_held) {
		ngx_shmtx_unlock(&ngx_accept_mutex);
	}
 
	if (delta) {
		ngx_event_expire_timers();
	}
 
	/**
	 *1. 普通事件都会存放在ngx_posted_events队列上
	 *2. 这个方法是循环处理read事件列队上的read事件
	 */
	ngx_event_process_posted(cycle, &ngx_posted_events);
}




```
### ngx_trylock_accept_mutex 获取accept锁

ngx_accept_mutex_held是拿到锁的唯一标识的全局变量。
当拿到锁，则调用ngx_enable_accept_events，将新的connection加入event事件上
如果没有拿到锁，则调用ngx_disable_accept_events。

```cpp
/**
 * 获取accept锁
 */
ngx_int_t ngx_trylock_accept_mutex(ngx_cycle_t *cycle) {
	/**
	 * 拿到锁
	 */
	if (ngx_shmtx_trylock(&ngx_accept_mutex)) {
 
		ngx_log_debug0(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
				"accept mutex locked");
 
		/* 多次进来，判断是否已经拿到锁 */
		if (ngx_accept_mutex_held && ngx_accept_events == 0) {
			return NGX_OK;
		}
 
		/* 调用ngx_enable_accept_events，开启监听accpet事件*/
		if (ngx_enable_accept_events(cycle) == NGX_ERROR) {
			ngx_shmtx_unlock(&ngx_accept_mutex);
			return NGX_ERROR;
		}
 
		ngx_accept_events = 0;
		ngx_accept_mutex_held = 1;
 
		return NGX_OK;
	}
 
	ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
			"accept mutex lock failed: %ui", ngx_accept_mutex_held);
 
	/**
	 * 没有拿到锁，但是ngx_accept_mutex_held=1
	 */
	if (ngx_accept_mutex_held) {
		/* 没有拿到锁，调用ngx_disable_accept_events，将accpet事件删除 */
		if (ngx_disable_accept_events(cycle, 0) == NGX_ERROR) {
			return NGX_ERROR;
		}
 
		ngx_accept_mutex_held = 0;
	}
 
	return NGX_OK;
}




```
### ngx_enable_accept_events 和 ngx_disable_accept_events

```cpp
/**
 * 开启accept事件的监听
 * 并将accept事件加入到event上
 */
static ngx_int_t ngx_enable_accept_events(ngx_cycle_t *cycle) {
	ngx_uint_t i;
	ngx_listening_t *ls;
	ngx_connection_t *c;
 
	ls = cycle->listening.elts;
	for (i = 0; i < cycle->listening.nelts; i++) {
 
		c = ls[i].connection;
 
		if (c == NULL || c->read->active) {
			continue;
		}
 
		if (ngx_add_event(c->read, NGX_READ_EVENT, 0) == NGX_ERROR) {
			return NGX_ERROR;
		}
	}
 
	return NGX_OK;
}
 
/**
 * 关闭accept事件的监听
 * 并将accept事件从event上删除
 */
static ngx_int_t ngx_disable_accept_events(ngx_cycle_t *cycle, ngx_uint_t all) {
	ngx_uint_t i;
	ngx_listening_t *ls;
	ngx_connection_t *c;
 
	ls = cycle->listening.elts;
	for (i = 0; i < cycle->listening.nelts; i++) {
 
		c = ls[i].connection;
 
		/* 如果c->read->active，则表示是活跃的连接，已经被使用中 */
		if (c == NULL || !c->read->active) {
			continue;
		}
 
#if (NGX_HAVE_REUSEPORT)
 
		/*
		 * do not disable accept on worker's own sockets
		 * when disabling accept events due to accept mutex
		 */
 
		if (ls[i].reuseport && !all) {
			continue;
		}
 
#endif
 
		/* 删除事件 */
		if (ngx_del_event(c->read, NGX_READ_EVENT,
				NGX_DISABLE_EVENT) == NGX_ERROR) {
			return NGX_ERROR;
		}
	}
 
	return NGX_OK;
}




```
### ngx_event_process_posted 事件队列处理
- 对ngx_posted_accept_events或ngx_posted_events队列上的accept/read事件进行回调处理

```cpp
/**
 * 处理事件队列
 *
 */
void ngx_event_process_posted(ngx_cycle_t *cycle, ngx_queue_t *posted) {
	ngx_queue_t *q;
	ngx_event_t *ev;
 
	while (!ngx_queue_empty(posted)) {
 
		q = ngx_queue_head(posted);
		ev = ngx_queue_data(q, ngx_event_t, queue);
 
		ngx_log_debug1(NGX_LOG_DEBUG_EVENT, cycle->log, 0,
				"posted event %p", ev);
 
		ngx_delete_posted_event(ev);
 
		/* 事件回调函数 */
		ev->handler(ev);
	}
}




```
### ngx_process_events 事件的核心处理函数

- 这个方法，我们主要看epoll模型下的ngx_epoll_process_events方法（ngx_epoll_module.c）

- 如果抢到了锁，则会将accpet/read事件放到队列上延后处理。
- 没有抢到锁的进程都是处理当前连接的read事件，所以直接进行处理。

```cpp
        /* 读取事件 EPOLLIN */
        if ((revents & EPOLLIN) && rev->active) {
 
#if (NGX_HAVE_EPOLLRDHUP)
            if (revents & EPOLLRDHUP) {
                rev->pending_eof = 1;
            }
 
            rev->available = 1;
#endif
 
            rev->ready = 1;
 
            /* 如果进程抢到锁，则放入事件队列 */
            if (flags & NGX_POST_EVENTS) {
                queue = rev->accept ? &ngx_posted_accept_events
                                    : &ngx_posted_events;
 
                ngx_post_event(rev, queue);
 
            } else {
            	/* 没有抢到锁，则直接处理read事件*/
                rev->handler(rev);
            }
        }
```
# 10.Nginx 虚拟主机
- 把一台运行在因特网上的服务器主机分成一台台“虚拟”的主机

- 利用虚拟主机，不用为每个要运行的网站提供一台单独的Nginx服务器或单独运行一组Nginx进程。

- 虚拟主机提供了在同一台服务器、同一组Nginx进程上运行多个网站的功能。

## 基本配置
Nginx的主配置文件是：nginx.conf，nginx.conf主要组成如下：

```cpp
		# 全局区   有一个工作子进程，一般设置为CPU数 * 核数
		worker_processes  1; 
 
		events {
				# 一般是配置nginx进程与连接的特性
				# 如1个word能同时允许多少连接，一个子进程最大允许连接1024个连接
				worker_connections  1024;
		}
 
		# 配置HTTP服务器配置段
		http {
 
				# 配置虚拟主机段
					server {
					
						# 定位，把特殊的路径或文件再次定位。
		        location  {
		           
		        } 
		    }
 
		    server {
		   			...
		    }
		}


```
## 基于域名的虚拟主机
1、在http大括号中添加如下代码段：

```cpp
    server {  
        #监听端口 80  
        listen 80;   
                                
        #监听域名abc.com;  
        server_name abc.com;
          
        location / {              
                # 相对路径，相对nginx根目录。  
            root    abc;  
            
            # 默认跳转到index.html页面  
            index index.html;                 
        }  
    } 


```
2、切换安装目录：cd/usr/local/software/nginx

3、创建目录：mkdir abc

4、新建index.html文件：vi /usr/local/software/nginx/abc/index.html，文件内容：

```cpp
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        </head>
        <body>
            <h2>基于域名的虚拟主机-index</h2>
        </body>
    </html>
```

5、重新读取配置文件：

/usr/local/software/nginx/sbin/nginx-s reload

kill -HUP进程号

6、配置windows本机host：

192.168.197.142 abc.com  #linux服务器IP地址

7、访问：http://abc.com:80/

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128013036586.png)
## 基于端口的虚拟主机配置

1、编辑配置文件：vim/usr/local/software/nginx/conf/nginx.conf

```cpp
    server {
        listen  2022;
        server_name     abc.com;
        location / {
           root    /home;
           index index.html;
        }
    }


```
2、新建index.html文件：vi /home/index.html，文件内容：

```cpp
      <html>
          <head>
              <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
          </head>
          <body>
              <h2>基于端口的虚拟主机配置-index</h2>
          </body>
      </html>


```
3、重新读取配置文件：
/usr/local/software/nginx/sbin/nginx-s reload

4、访问：http://abc.com:2022/
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128013157352.png)
## 基于IP地址虚拟主机配置

1、编辑配置文件：vim/usr/local/software/nginx/conf/nginx.conf

```cpp
    server {
      listen  80;
      server_name  192.168.197.142;
      location / {
              root    ip;
              index index.html;
      }
    }
```
2、新建index.html文件：

执行命令：

mkdir /usr/local/software/nginx/ip

vi /usr/local/software/nginx/ip/index.html
文件内容：

```cpp
      <html>
          <head>
              <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
          </head>
          <body>
              <h2>基于IP地址虚拟主机配置-index</h2>
          </body>
      </html>


```
3、重新读取配置文件：

/usr/local/software/nginx/sbin/nginx-s reload

4、访问：http://192.168.197.142/
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128013303990.png)

