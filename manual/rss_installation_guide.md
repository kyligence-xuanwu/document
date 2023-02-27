## 部署 RSS 服务
RSS 是 Remote Shuffle Service 的简称，为计算引擎 Shuffle 数据提供远端存储服务，提高存算分离架构下计算引擎执行稳定性。
RSS 开源项目很多，我们以 Apache Celeborn 为例：

### 硬件要求
- CPU：>= 4 核
- 内存：>= 16 GB
- 节点数：>= 3 个
### 软件、系统要求
- JDK：8 版本
- 系统：Linux
- 环境变量：export RSS_HOME = <RSS 根目录>

### 部署
1. 解压安装包至 $RSS_HOME

2. 根据实际情况，修改环境变量 ```$RSS_HOME/conf/rss-env.sh```，例如
```Bash
#!/usr/bin/env bash
RSS_MASTER_MEMORY=4g
RSS_WORKER_MEMORY=2g
RSS_WORKER_OFFHEAP_MEMORY=4g
```
3. 根据实际情况，修改配置文件 ```$RSS_HOME/conf/rss-defaults.conf```，采用 HA 部署模式，如下
```YAML
# An Example:
rss.device.monitor.enabled                       false
rss.rpc.io.mode                                  NIO
rss.rpc.io.clientThreads                         4
rss.rpc.io.serverThreads                         4
rss.rpc.io.numConnectionsPerPeer                 4

rss.push.io.numConnectionsPerPeer                4
rss.push.io.threads                              4

rss.worker.flush.queue.capacity                  4096
rss.push.data.replicate                          true
rss.metrics.system.enabled                       true
rss.worker.prometheus.metric.port                9096
rss.master.prometheus.metric.port                9098
rss.worker.base.dirs                             /opt/rss-shuffle-data

# HA cluster
rss.master.port                                 9097
rss.ha.enabled                                  true
rss.ha.master.hosts                             10.1.2.146,10.1.2.147,10.1.2.148
rss.ha.service.id                               rss
rss.ha.nodes.rss                                node1,node2,node3
rss.ha.address.rss.node1                        10.1.2.146
rss.ha.address.rss.node2                        10.1.2.147
rss.ha.address.rss.node3                        10.1.2.148
rss.ha.port                                     9872
rss.ha.storage.dir                              /opt/rss_ratis/
rss.ha.ratis.snapshot.auto.trigger.enabled      true
rss.ha.ratis.snapshot.auto.trigger.threshold    200000
```
注意：请提前在所有节点上创建好数据目录、工作目录。对应的配置项为
  - rss.worker.base.dirs
  - rss.ha.storage.dir

4. 将安装包分发到所有 RSS 节点上
5. 在所有主节点上启动 Matser：```$RSS_HOME/sbin/start-master.sh```
6. 在所有工作节点上启动 Worker：```$RSS_HOME/sbin/start-worker.sh```
7. 启动成功后，可以看到 Master 的日志显示如下信息

```
21/12/21 20:06:18,964 INFO [main] Dispatcher: Dispatcher numThreads: 64
21/12/21 20:06:18,994 INFO [main] TransportClientFactory: mode NIO threads 8
21/12/21 20:06:19,113 WARN [main] ServerBootstrap: Unknown channel option 'TCP_NODELAY' for channel '[id: 0x8a9442f6]'
21/12/21 20:06:19,129 INFO [main] Utils: Successfully started service 'MasterSys' on port 9097.
21/12/21 20:06:19,150 INFO [main] HttpServer: HttpServer started on port 7001.
21/12/21 20:06:21,615 INFO [netty-rpc-connection-0] TransportClientFactory: Successfully created connection to /172.16.159.100:40115 after 4 ms (0 ms spent in bootstraps)
21/12/21 20:06:21,661 INFO [dispatcher-event-loop-9] Master: Registered worker
Host: 172.16.159.100
RpcPort: 40115
PushPort: 35489
FetchPort: 35689
TotalSlots: 4096
SlotsUsed: 0
SlotsAvailable: 4096
LastHeartBeat: 0
WorkerRef: NettyRpcEndpointRef(ess://WorkerEndpoint@172.16.159.100:40115)
.
21/12/21 20:06:23,785 INFO [netty-rpc-connection-1] TransportClientFactory: Successfully created connection to /172.16.159.98:39151 after 1 ms (0 ms spent in bootstraps)
21/12/21 20:06:23,817 INFO [dispatcher-event-loop-17] Master: Registered worker
Host: 172.16.159.98
RpcPort: 39151
PushPort: 40193
FetchPort: 37455
TotalSlots: 4096
SlotsUsed: 0
SlotsAvailable: 4096
LastHeartBeat: 0
WorkerRef: NettyRpcEndpointRef(ess://WorkerEndpoint@172.16.159.98:39151)
.
21/12/21 20:06:25,948 INFO [netty-rpc-connection-2] TransportClientFactory: Successfully created connection to /172.16.159.99:41955 after 1 ms (0 ms spent in bootstraps)
21/12/21 20:06:26,009 INFO [dispatcher-event-loop-25] Master: Registered worker
Host: 172.16.159.99
RpcPort: 41955
PushPort: 37587
FetchPort: 46865
TotalSlots: 4096
SlotsUsed: 0
SlotsAvailable: 4096
LastHeartBeat: 0
WorkerRef: NettyRpcEndpointRef(ess://WorkerEndpoint@172.16.159.99:41955)
```

### 重启步骤
请确保执行步骤 3 中的清理工作，这个目录保存着上个集群的过期数据
1. 停止所有 worker
2. 停止所有 masters
3. 清理 master ratis meta storage 目录 (rss.ha.storage.dir).
4. 启动所有 masters
5. 启动所有 workers
