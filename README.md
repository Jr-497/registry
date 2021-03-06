Registry注册中心
====
在学习了SpringCloud的各个组件的使用以及研究过相关源码后，为了更好地理解其内部实现原理，模仿了nacos的部分设计思想，自己实现了该注册中心，代码量4000+，代码中有详细的注释，使用netty实现了服务注册，服务心跳，服务健康检查，服务发现，服务重连等功能，使用Http请求完成集群间的同步以及心跳问题。迟点会画出该项目的剖析图，后面可能还会继续实现负载均衡器，网关路由，限流等功能......

registry核心功能点:
-------
**服务注册**：服务消费者(Client端)在启动后会通过netty发送注册请求的方式向Server端注册自己的服务，提供Client端自身的一些数据，比如ip地址、端口、服务名等信息。Server端接收到Client端发来的注册请求后，就会把这些数据信息存储在一个双层的内存Map中，服务端通过异步读取阻塞队列来防止同时向一个Server注册多个实例间的线程安全问题。<br>
**服务心跳**：Client端在启动连接上Server端后，会通过netty的写空闲事件来发送心跳信息来持续通知Server端，说明服务一直处于可用状态，防止该被实例被Server端剔除。默认5s发送一次心跳。<br>
**服务发现**：Client在连接上服务提供者的服务时，会先发送请求给Server，获取Server内存注册表上面注册的服务清单，并且缓存在Client本地的Map中，同时会在Client本地开启一个定时任务定时拉取服务端最新的注册表信息更新到本地缓存，默认15s拉取一次。<br>
**服务健康检查**：Server通过netty的读空闲事件来检查Client注册服务实例的健康情况，对于超过15s没有收到Client端心跳的实例，会先把该实例的读空闲次数记录下来，如果长达1分钟都没有收到该实例的心跳，会关闭Server服务端与该Client端的链路。<br>
**服务同步**：Server集群之间在添加实例或者剔除实例时会互相同步服务实例，用来保证服务信息的一致性，使用redis的分布式锁来解决数据覆盖的问题以及通过保证部分操作的顺序来防止数据丢失。<br>
**服务重连**: Client在Server端宕机或者重启或者Client端因为网络问题连接不上Server集群，会自动重连，直到连接上集群上其中的一个Server。<br>
**Server集群服务心跳**：Server集群间通过http请求来互相发送心跳，每个Server会记录着健康的Server实例，等客户端服务发现时也把健康的Server实例传给Client，以便Client所连接的Server宕机了也能连接上集群上的其他Server节点，默认5s发送一次心跳。<br>
**支持动态修改集群节点**:在Server集群启动后，可在不影响原本服务的使用的前提下，动态地添加或删除集群节点。<br>

有疑问的可以加我qq咨询:983470394

