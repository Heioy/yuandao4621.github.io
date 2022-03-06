对于Oracle 11gR2之后的版本，集群中的所有组件统一被资源所替代(OHASD服务除外)，其中最重要的两个资源分别为 **CSS** 和 **CRS**。`CSS (Cluster Synchronization Service)` 主要负责构建和维护集群的一致性，其资源以 **ocssd.bin** 守护进程的形式存在于Oracle集群中，其功能主要包含 **NM (Node Management, 节点管理)** 和 **GM (Group Management, 组管理)**，通过集群内部节点间的心跳机制和组内部共享资源的共享和隔离来保证集群在运行中资源始终处于一致性的状态。


集群中的另一个重要的资源为 `CRS (Cluster Ready Service, 集群就绪服务)`，主要用来管理集群中的资源，以此来实现Oracle集群的高可用性。除此之外，`CRS` 还负责管理 **OCR (Oracle Cluster Register, 集群注册表)**，包括OCR的更新和备份。
> OCR 是一个包含集群内部所有资源的注册表，CRSD启动时会首先读取本地节点上OCR的信息，从中获取集群内部注册的所有资源及属性。集群内部的每个节点都会有一个注册表，但通常只有集群内最先启动节点的OCR注册表进程才能执行更新OCR注册表的写入操作，其他节点的OCR注册表仅可提供读服务，不提供写服务。


#### `CRS` 的启动步骤

**CRS** 的启动需要依赖 **CSS** 的启动，也就是说，集群中必须等待 **CSS** 服务完全开启后，集群中的 **CRS** 服务才能开启。从这一点上不难看出，Oracle集群在启动时会首先启动 **OHASD**，然后启动 **CSS**，最后启动 **CRS**。当集群中的**CRS**服务就绪后，整个集群可以对外提供服务了。


而整个**CRS**服务的启动过程相对而言是比较简单的，启动过程大致如下

* 启动**crsd.bin**守护进程，守护进程启动后会读取OCR的信息，判断注册到OCR中的资源的属性、状态并尝试启动OCR中的资源

* **crsd.bin**守护进程读取OCR的信息，并启动OCR中的资源

* **crsd.bin**守护进程建立和集群其他节点crsd服务的通信  

**CRS** 在集群资源启动成功后并不会中止服务，Oracle 需要定期对集群注册表进行更新(通常是每4个小时更新一次)，另外还需要处理用户的请求。


#### 资源管理

Oracle中提供了两种基于命令行的资源管理工具，分别为 `crsctl` 和 `srcvtl`。通过这些工具，可以很方便的对集群资源进行启动、关闭、配置等。  


通过 `crsctl` 查看集群资源状态  *通常集群启动、停止等操作会使用crsctl来完成，对单一资源的操作会使用srvctl来完成*
```bash
# crsctl status resource -t
--------------------------------------------------------------------------------
NAME           TARGET  STATE        SERVER                   STATE_DETAILS
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.DATADG.dg
               ONLINE  ONLINE       com1
               ONLINE  ONLINE       com2
ora.LISTENER.lsnr
               ONLINE  ONLINE       com1
               ONLINE  ONLINE       com2
ora.OCRVOTE.dg
               ONLINE  ONLINE       com1
               ONLINE  ONLINE       com2
ora.asm
               ONLINE  ONLINE       com1                     Started
               ONLINE  ONLINE       com2                     Started
ora.gsd
               OFFLINE OFFLINE      com1
               OFFLINE OFFLINE      com2
ora.net1.network
               ONLINE  ONLINE       com1
               ONLINE  ONLINE       com2
ora.ons
               ONLINE  ONLINE       com1
               ONLINE  ONLINE       com2
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.LISTENER_SCAN1.lsnr
      1        ONLINE  ONLINE       com1
ora.com1.vip
      1        ONLINE  ONLINE       com1
ora.com2.vip
      1        ONLINE  ONLINE       com2
ora.cvu
      1        ONLINE  ONLINE       com1
ora.oc4j
      1        ONLINE  ONLINE       com1
ora.rac.db
      1        ONLINE  ONLINE       com1                     Open
      2        ONLINE  ONLINE       com2                     Open
ora.scan1.vip
      1        ONLINE  ONLINE       com1
```

