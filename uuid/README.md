# uuid
### 分布式UUID生成系统

------
### 简介：

分布式UUID生成系统，俗称“发号器”。
是为了解决分布式数据库部署过程中，数据唯一标识冲突的问题而提出的解决方案。
目前在微博FEED系统以及一些其他系统中得到了应用。
目前“发号器”使用数据库的AUTO_INCREMENT特性来生成，需要定期清理数据，存在较高的开销。
在UUID的生成算法方面，目前的“发号器”还存在着一定的缺陷，UUID并不是顺序递增的。

### 新版分布式UUID生成系统

为了解决旧版“发号器”的问题，我设计开发了新版分布式UUID生成系统：
新版系统采用了高性能的服务器框架，源于Redis系统的网络IO部分代码，具有业界公认的高性能和高稳定性。
新版系统实现了Memcache协议中的GET指令，用于客户端请求UUID，具有很好的通用性，应用改造成本很低。
新版系统采用了精密的算法，保证了UUID在“秒”级别能够保持顺序递增的特性。
新版系统具有良好的可扩展性，最多可以部署255个实例，理论上每个实例每秒最多可以生成1600万个的UUID。
新版系统每个实例保持独立，实例间无任何通信过程，可以方便的进行七层、四层、客户端等形式的负载均衡。
新版系统在单CPU虚拟机上进行简单测试，单实例每秒可以响应20000次以上的请求。
理论上每个机房部署两台服务器即可满足整个机房所有应用所有UUID的需求。

### 使用说明：

#### 可执行程序：redis-uuid
##### 参数选项：
```
Redis server version 2.0.4
-p <num>      TCP port number to listen on (default: 6376)
-l <ip_addr>  interface to listen on (default: INADDR_ANY, all addresses)
-d            run as a daemon
-c <num>      max simultaneous connections (default: 1024)
-v            verbose (print errors/warnings while in event loop)
-u <num>      the unique server id, must be a number between 1 and 255
```
##### 必须指定的参数：

-p 端口号 -d 以daemon在后台运行 -u 系统全局唯一的标识，必须在1-255之间，绝对不能重复。

### 运行示例：
```
redis-uuid -p 6000 -d -u 100
```
建议将多个实例的启动命令行写为脚本程序，防止发生错误。


### 测试方法：

使用任何支持文本协议的Memcache客户端或者telnet均可测试。
请求key为uuid。

#### 示例：
```
[root@vm10110041 ~]# telnet 10.210.74.152 6376
Trying 10.210.74.152...
Connected to vm10110041 (10.210.74.152).
Escape character is '^]'.
get uuid
VALUE uuid 0 19
5548557806297767423
END
```
