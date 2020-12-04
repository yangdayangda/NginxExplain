为什么要有负载均衡

•客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕后，再将结果返回给客户端。这种架构模式对于早期的系统相对单一，并发请求相对较少的情况下是比较适合的，成本也低。但是随着访问量和数据量的飞速增长，以及系统业务的复杂度增加，这种架构会造成服务器相应客户端的请求日益缓慢，并发量特别大的时候，还容易造成服务器直接崩溃。那么如何解决这种情况呢？我们首先想到的可能是升级服务器的配置，比如提高CPU执行频率，加大内存等提高机器的物理性能来解决此问题，但是硬件的性能提升已经不能满足日益提升的需求了。

![image-20201128174152195](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174152195.png)

•最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能 够满足需求的。那么怎么办呢？上面的分析我们去掉了增加服务器物理配置来解决问题的办法，也就是说纵向解决问题的办法行不通了，那么横向增加服务器的数量呢？这时候集群的概念产生了，单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡

![image-20201128174223927](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174223927.png)



原理实现

•效果：浏览器栏输入地址localhost/edu/test.html后(这就可以模拟向服务器发起请求)，负载均衡效果，平均到8080，8083，8084端口中

![image-20201128174302654](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174302654.png)

![image-20201128174308554](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174308554.png)

![image-20201128174314945](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174314945.png)

准备工作：

•（1）准备三台tomcat服务器，一台8080，一台8083，一台8084（和上述的反向代理原理类似） 

•（2）在三台tomcat 里面webapps目录中，创建名称是edu文件夹，在edu文件夹中创建 页面test.html，用于测试

![image-20201128174334507](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174334507.png)

•在nginx的配置文件（文件在conf目录下，文件为nginx.conf）中进行负载均衡的配置（# 表示注释），配置好后启动nginx

![image-20201128174353151](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174353151.png)



nginx 分配服务器策略 

•第一种 轮询（默认） 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。

•第二种weight 代表权重默认为 1,权重越高被分配的客户端越多

（指定轮询几率，weight和访问率成正比，用于后端服务器性能不均的情况）

![image-20201128174504229](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174504229.png)

•如图所示，如果把8080端口模拟的服务器权重设为2，那么这三台服务器的访问率为2：1：1，即刷新多次页面出现“8080”的次数为其他页面的两倍

•第三种ip_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器 （解决session共享问题）

![image-20201128174526443](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174526443.png)

•加入一行代码”ip_hash;”，然后即可达到此效果：一旦连接到某个服务器，多次刷新页面仍然显示同一个服务器

宕机

•当某个服务器宕机的时候，nginx会将请求在后台转发给其他服务器。如例，当我们关掉8084端口模拟的服务器时，再次刷新到请求8084端口模拟的服务器时，会出现卡顿一段时间的现象，可以在配置文件中修改如图所示的标蓝位置，设置请求的最长时长

![image-20201128174603860](C:\Users\Clearlove\AppData\Roaming\Typora\typora-user-images\image-20201128174603860.png)



注意事项

1.在做准备工作时要确定启动了你打算用的服务器

（服务器没有启动就根本不会出现效果）

2.在每次修改好配置文件后要记得重启nginx，即在出现了和自己意料之外的结果时要确认一下修改的配置文件是否有保存，以及是否有重新启动nginx

3.确认好文件的所在位置和输入的网址是否有出入