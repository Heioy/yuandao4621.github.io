Oracle 11g中所有的组件以**资源**的形式进行统一管理，而其中最重要的三个资源分别为**OHASD**、**css**、**crs**，`OHASD` 负责集群初始化服务及启动各个资源对应的代理进程；`CSS` 负责检查和维护集群资源的一致性；`CRS` 则对集群中的所有资源进行统一管理。

`Cluster Synchornization Service,集群同步服务` 简称 **CSS**，是Oracle集群管理软件的核心组件，主要负责构建集群、并且维护集群的一致性。在11gR2版本之后，集群中的**OHASD**高可用服务组件会率先启动，然后启动对应资源的代理进程，最后由代理进程启动对应组件的服务。

`CSS` 组件对应的代理进程为**ocssd**，当集群中的**oracssagent_root**代理进程服务开启后，**oracssagent_root**代理进程会尝试启动**ocssd.bin**资源进程，通过**ocssd**访问集群启动的配置信息，来完成整个集群的构建工作。

如下，通过ocssd启动日志分析**ocssd**的启动步骤

* **CSSD**守护进程开始启动
```log
2022-02-26 12:55:51.303 :    CSSD:907018304: (TLM) Starting CSS daemon, version 12.2.0.1.0 with uniqueness value 1645851351
2022-02-26 12:55:51.303 :    CSSD:907018304: clsu_load_ENV_levels: Module = CSSD, LogLevel = 2
...
```
启动*CSS Daemon*守护进程，并发现集群版本信息，并加载CSS启动过程中的各种组件配置

* **CSSD**守护进程开始和**gpnpd**守护进程通信
```log
2022-02-26 12:55:51.310 :    CSSD:907018304: clssscGetParameterOLR: OLR fetch for parameter auth rep (9) failed with rc 21
2022-02-26 12:55:51.310 :    CSSD:907018304: clssscagStartAgLsnr: Failed to get auth location from OLR, constructing manually 
2022-02-26 12:55:51.310 :    CSSD:907018304: clssscagStartAgLsnr: auth location '/****/12.2.0/auth/css/' 
2022-02-26 12:55:51.310 :    CSSD:907018304: clssscGPNPInit: PERF_TIME Initializing GPNP
2022-02-26 12:55:51.311 :    GPNP:907018304: clsgpnp_Init: [at clsgpnp0.c:703] '/****/12.2.0' in effect as GPnP home base.
2022-02-26 12:55:51.311 :    GPNP:907018304: clsgpnp_Init: [at clsgpnp0.c:769] GPnP pid=10168, cli= GPNP comp tracelevel=1, depcomp tracelevel=0, tlsrc:init, apitl:0, tstenv:0, devenv:0, envopt:0, flags=3
```
CSS启动后，开始从本地注册表(OLR)中获取集群的配置信息，因为本地注册表的信息注册在**gpnpd**进程中，而这个时候**gpnpd**进程尚未启动(Initializing GPNP)，以至于获取(OLR)信息失败，最后**gpnpd**进程启动，pid为10168  
```log
2022-02-26 12:55:51.311 :    CSSD:737126144: clsssclsnrsetup: endp 0x4a for ipc://agent_ag_node1_
2022-02-26 12:55:51.311 :    CSSD:737126144: clssscagOpenAgentEndp: listening on ipc://agent_ag_node1_ (0x4a)
2022-02-26 12:55:51.312 :    CSSD:737126144: clsssclsnrsetup: endp 0x5b for ipc://monitor_ag_node1_
2022-02-26 12:55:51.312 :    CSSD:737126144: clssscagOpenAgentEndp: listening on ipc://monitor_ag_node1_ (0x5b)
```
ocssd使用的挂载点(endp,*endpoint的缩写*)被创建，
```log
2022-02-26 12:55:51.314 :    GPNP:907018304: clsgpnpkwf_initwfloc: [at clsgpnpkwf.c:403] Using FS Wallet Location : /****/12.2.0/gpnp/node1/wallets/peer/
...
2022-02-26 12:55:51.334 :    GPNP:907018304: clsgpnp_profileCallUrlInt: [at clsgpnp.c:2402] Result: (0) CLSGPNP_OK. Successful get-profile CALL to remote "ipc://GPNPD_node1" disco ""
2022-02-26 12:55:51.334 :    CSSD:907018304: clssscGPNPInit: Successfully initialized GPNP
```
接着启动的**ocssd**进程与**gpnpd**守护进程进行通信，并从**gpnp profile**中获取集群的基本配置信息

* **ocssd**开始读取**gpnp profile**中的VF(Voting File)
```log
2022-02-26 12:55:51.386 :    CSSD:700241664: clssnmvDDiscThread: using discovery string /dev/* for initial discovery 
2022-02-26 12:55:51.386 :   SKGFD:700241664: Discovery with str:/dev/*:
2022-02-26 12:55:51.386 :   SKGFD:700241664: UFS discovery with :/dev/*:
...
2022-02-26 12:55:51.389 :   SKGFD:700241664: running stat on disk:/dev/voting
2022-02-26 12:55:51.389 :   SKGFD:700241664: Fetching UFS disk :/dev/voting:
2022-02-26 12:55:51.397 :   SKGFD:700241664: Handle 0x7f8bd0336080 from lib :UFS:: for disk :/dev/voting:
2022-02-26 12:55:51.399 :    CLSF:700241664: Opened hdl:0x7f8bd032f380 for dev:/dev/voting:
2022-02-26 12:55:51.400 :    CSSD:700241664: clssnmvDiskVerify: Successful discovery for disk /dev/voting, UID 6371c643-7e584f21-bf565295-4efcff72, SID 00112233-44556677-8899aabb-ccddeeff, Pending CIN 0:1641868903:0, Committed CIN 0:1641868903:0
2022-02-26 12:55:51.400 :    CSSD:700241664: clssnmvDiskVerify: Successful discovery of 1 disks
```
**ocssd**守护进程通过和**gpnpd**守护进程进行通信，成功获取到**gpnp profile**中的Voting File，并根据路径搜索对应的VF是否可用，最终找到可用的VF(/dev/voting)

* **ocssd**进程获取VF中关于集群的配置参数
```log
2022-02-26 12:55:51.400 :    CSSD:700241664:   misscount          30    reboot latency      3
2022-02-26 12:55:51.400 :    CSSD:700241664:   long I/O timeout  200    short I/O timeout  27
2022-02-26 12:55:51.400 :    CSSD:700241664:   rim hub timeout    30    grace period        0
2022-02-26 12:55:51.400 :    CSSD:700241664:   hub size           99  active version 12.2.0.1.0
2022-02-26 12:55:51.400 :    CSSD:700241664:   Listing unique IDs for 1 voting files:
2022-02-26 12:55:51.400 :    CSSD:700241664: voting file 1: 6371c643-7e584f21-bf565295-4efcff72
```
**ocssd**进程获取到VF中关于集群的配置参数，包含 *miscount*、*I/O timeout*、*reboot latency*等信息

* **ocssd**开始和**gipcd**进程通信
```log
2022-02-26 12:55:51.346 :GIPCXCPT:726787840: gipcInternalConnectSync: failed sync request, ret gipcretConnectionRefused (29)
2022-02-26 12:55:51.346 :GIPCXCPT:726787840: gipcConnectSyncF [clscrsconGipcConnect : clscrscon.c : 689]: EXCEPTION[ ret gipcretConnectionRefused (29) ]  failed sync connect endp 0x7f8c04105990 [0000000000000190] { gipcEndpoint : localAddr 'clsc://(ADDRESS=(PROTOCOL=ipc)(KEY=)(GIPCID=00000000-00000000-0))', remoteAddr 'clsc://(ADDRESS=(PROTOCOL=ipc)(KEY=CRSD_UI_SOCKET)(GIPCID=00000000-00000000-0))', numPend 0, numReady 0, numDone 0, numDead 1, numTransfer 0, objFlags 0x0, pidPeer 0, readyRef (nil), ready 1, wobj 0x7f8c04108370, sendp 0x7f8c04108120 status 13flags 0xa108071a, flags-2 0x0, usrFlags 0x0 }, addr 0x7f8c04106e20 [0000000000000197] { gipcAddress : name 'clsc://(ADDRESS=(PROTOCOL=ipc)(KEY=CRSD_UI_SOCKET)(GIPCID=00000000-00000000-0))', objFlags 0x0, addrFlags 0x4 }, flags 0x0
```
**ocssd**守护进程试图和**gipcd**进程进行通信，但失败了，这个时候**gipcd**守护进程尚未初始化成功
```log
2022-02-26 12:55:51.400 :    CSSD:907018304: clssnmOpenGIPCEndp: opening cluster listener on gipcha://node1:nm2_****
```
**gipcd**守护进程正常后，**ocssd**开启集群监听，并和**gipcd**进行通信
```log
2022-02-26 12:55:51.400 :GIPCXCPT:907018304: gipchaInternalReadGpnp: No HAIP network info configured in GPNP profile, using defaults, ret gipcretFail (1)
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: configuring default IPV4 multicast addresses
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: configuring default IPV6 multicast addresses
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: configuring default bootstrap communications modes
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: configuring bootstrap communications using:  broadcast and multicast
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: mcast address[  0 ]  224.0.0.251
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: mcast address[  1 ]  230.0.1.0
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaInternalReadGpnp: mcast address[  0 ]  [FF02::FB]
2022-02-26 12:55:51.400 :GIPCHGEN:907018304: gipchaGetClusterMode: gipchaGetClusterMode():REGULAR Cluster mode
```
**ocssd**进程开始访问**gipc**，获取集群私网的信息。由于**gpnp profile**中没有发现关于*HAIP*的信息，因此使用默认配置；
```log
2022-02-26 12:55:52.427 :GIPCHDEM:697079552: gipchaDaemonProcessInfUpdate: ip 172.xx.xx.xx subnet 172.xx.xx.0 mask 255.255.255.0 state 1 inc 0 flags 0x1
2022-02-26 12:55:52.427 :GIPCHGEN:697079552: gipchaNodeAddInterfaceF [gipchaDaemonProcessInfUpdate : gipchaDaemonThread.c : 4640]: adding interface info 0x7f8bc809e750 { host '', haName 'CSS_****', local (nil), ip '172.xx.xx.xx', subnet '172.xx.xx.0', mask '255.255.255.0', mac 'a0-00-02-20-fe-80-00-00-00-00-00-00-00-02-00-00-00-00-00-00', ifname 'ib0', numRef 0, numFail 0, idxBoot 0, flags 0x1841 }
2022-02-26 12:55:52.427 :GIPCHDEM:697079552: gipchaDaemonProcessInfUpdate: completed interface update host 'node1', haName '', hctx 0x26bc520 [0000000000000011] { gipchaContext : host 'node1', name 'CSS_****', luid '9c33f299-00000000', name2 00aa-381f-1b1a-2e5d, numNode 0, numInf 1, maxPriority 0, clientMode 3, nodeIncarnation 00000000-00000000 usrFlags 0x0, flags 0x80865 }
```
...
2022-02-26 12:55:52.427 :GIPCHTHR:698656512: gipchaWorkerCreateInterface: created local bootstrap broadcast interface for node 'node1', haName 'CSS_xxx', inf 'udp://172.xx.xx.255:42424' inf 0x7f8bc809e750
...
2022-02-26 12:55:52.428 :GIPCHTHR:698656512: gipchaWorkerCreateInterface: created local bootstrap multicast interface for node 'node1', haName 'CSS_xxx', inf 'mcast://224.0.0.251:42424/172.16.129.27' inf 0x7f8bc80abf90
**gipcd**将集群中默认的网络配置注册到服务中，并准备和集群中的其他节点进行通信(udp://172.xx.xx.255)

* **gipcd**进程开始与远程节点进行通信
```log
2022-02-26 12:55:53.493 :GIPCHTHR:698656512: gipchaWorkerCreateInterface: created remote interface for node 'node2', haName '3411-028b-77a0-cde1', inf 'udp://172.xx.xx.xx:57238' inf 0x7f8bc80aa020
2022-02-26 12:55:53.493 :GIPCHGEN:698656512: gipchaWorkerAttachInterface: Interface attached inf 0x7f8bc80aa020 { host 'node2', haName '3411-028b-77a0-cde1', local 0x7f8bc809e750, ip '172.xx.xx.xx:57238', subnet '172.xx.xx.0', mask '255.255.255.0', mac '', ifname 'ib0', numRef 0, numFail 0, idxBoot 0, flags 0x6 }
...
2022-02-26 12:55:53.496 :GIPCHALO:698656512: gipchaLowerProcessAcks: ESTABLISH finished for node 0x7f8bc80a6f90 { host 'node2', haName '3411-028b-77a0-cde1', srcLuid 9c33f299-2684e083, dstLuid 10c4e4e4-31bd223e numInf 2, sentRegister 0, localMonitor 0, baseStream 0x7f8bc80a9df0 type gipchaNodeType12001 (20), nodeIncarnation d5ca59d0-bd21895a, incarnation 2, cssIncarnation 0, roundTripTime 4294967295 lastSeenPingAck 0 nextPingId 1 latencySrc 0 latencyDst 0 flags 0x10280c}
...
2022-02-26 12:55:53.497 :GIPCGMOD:673437440: gipcmodGipcCompleteConnect: [gipc] completed connect on endp 0x7f8bac0337d0 [0000000000000525] { gipcEndpoint : localAddr 'gipcha://node1:4f02-73bf-f8e9-26d8', remoteAddr 'gipcha://node2:nm2_****/8e2a-e5a7-346a-9797', numPend 1, numReady 1, numDone 0, numDead 0, numTransfer 0, objFlags 0x0, pidPeer 0, readyRef 0x2e4c1f0, ready 0, wobj 0x7f8bac035d60, sendp (nil) status 13flags 0x200b8602, flags-2 0x0, usrFlags 0x0 }
2022-02-26 12:55:53.497 :    CSSD:673437440: clssscSelect: conn complete ctx 0x2e4e2b0 endp 0x525
...
2022-02-26 12:55:57.998 :    CSSD:673437440: clssnmConnComplete: node node2 softver 12.2.0.1.0
2022-02-26 12:55:57.998 :    CSSD:673437440: clssnmCompleteConnProtocol: Connect ack from node 2 (node2) ninf endp (nil), probendp 0x525, endp 0x7f8c00000525
2022-02-26 12:55:57.998 :    CSSD:673437440: clssnmCompleteConnProtocol: Connection from known node 2, node2, node unique 1645760034, msg unique 1645760034
2022-02-26 12:55:57.998 :    CSSD:673437440: clssnmCompleteConnProtocol: node node2, 2, uniqueness 1645760034, msg uniqueness 1645760034, endp 0x525 probendp 0x7f8c00000000 endp 0x525
```
**gipcd**服务正常后，发现了远端节点的网络信息，并与远端节点进行通信，最后建立(ESTABLISH)两个节点间的联系，clssnmConnComplete结束

* 集群重新配置
```log
2022-02-26 12:55:59.251 :    CSSD:675014400: clssnmRcfgMgrThread: Reconfig in progress...
2022-02-26 12:55:59.251 :    CSSD:907018304: NMEVENT_SUSPEND [00][00][00][00]
2022-02-26 12:55:59.251 :    CSSD:675014400: clssnmRcfgMgrThread: initial lastleader(2) unique(1645760034)
2022-02-26 12:55:59.251 :    CSSD:673437440: clssnmSendAck: node 2, node2, syncSeqNo(538081862) type(11)
2022-02-26 12:55:59.251 :    CSSD:907018304: clssscCompareSwapEventValue: changed CmInfo State  val 5, from 1, changes 1
2022-02-26 12:55:59.251 :    CSSD:907018304: clssgmSuspendAllGrocks: Issue SUSPEND
2022-02-26 12:55:59.251 :    CSSD:907018304: clssgmSuspendAllGrocks: done
2022-02-26 12:55:59.251 :    CSSD:907018304: clssscCompareSwapEventValue: changed CmInfo State  val 2, from 5, changes 2
2022-02-26 12:55:59.251 :    CSSD:907018304: clssscUpdateEventValue: ConnectedNodes  val 0, changes 1
2022-02-26 12:55:59.251 :    CSSD:907018304: clssgmStartNMMon:  completed node cleanup 
2022-02-26 12:55:59.251 :    CSSD:673437440: List of members seen by sync source: 2
2022-02-26 12:55:59.251 :    CSSD:673437440: List of connections seen by sync source: 1,2
2022-02-26 12:55:59.251 :    CSSD:673437440: clssnmHandleSync: Node node2, number 2, is EXADATA fence capable
2022-02-26 12:55:59.251 :    CSSD:673437440: clssnmValidateSyncMsg: Received additional sync from master node2, number 2 for the same reconfig
```
集群开始重新配置(ReConfigure, 进程名为clssnmRcfgMgrThread)
```log
2022-02-26 12:55:59.251 :    CSSD:907018304: clssgmSuspendAllGrocks: Issue SUSPEND
2022-02-26 12:55:59.251 :    CSSD:907018304: clssgmSuspendAllGrocks: done
```
> 集群进行重新配置的过程中, 会先进行**NM(Node Management,节点管理)**节点层面的配置，必须要等待**NM**配置结束后，集群才能进行**GM(Group Management,组管理)**层面的重新配置  

**NM**层面的集群开始配置后，**GM**层面的集群配置被*SUSPEND(暂停)*，进程名为clssgmSuspendAllGrocks  

```log
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmUpdateNodeState: node node1, number 1, current state 1, proposed state 3, current unique 1645851351, proposed unique 1645851351, prevConuni 0, birth 538081862
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmChangeState: oldstate 1 newstate 3 clssnmc.c 6924
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmUpdateNodeState: node node2, number 2, current state 4, proposed state 3, current unique 1645760034, proposed unique 1645760034, prevConuni 0, birth 538081860
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmChangeState: oldstate 4 newstate 3 clssnmc.c 6924
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmSendAck: node 2, node2, syncSeqNo(538081862) type(15)
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmQueueClientEvent:  Sending Event(1), type 1, incarn 538081862
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmQueueClientEvent: Node[1] state = 3, birth = 538081862, unique = 1645851351
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmQueueClientEvent: Node[2] state = 3, birth = 538081860, unique = 1645760034
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmHandleUpdate: SYNC(538081862) from node(2) completed
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmHandleUpdate: NODE 1 (node1) IS ACTIVE MEMBER OF CLUSTER
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmHandleUpdate: NODE 2 (node2) IS ACTIVE MEMBER OF CLUSTER
2022-02-26 12:55:59.253 :    CSSD:673437440: clssscUpdateEventValue: NMReconfigInProgress  val 4294967295, changes 3
2022-02-26 12:55:59.253 :    CSSD:673437440: clssnmHandleUpdate: local disk timeout set to 200000 ms, remote disk timeout set to 200000
```
集群结束了**NM**层面的配置(SYNC(538081862) from node(2) completed)，进行**NM**层面的配置修改了3项配置(NMReconfigInProgress  val 4294967295, changes 3)  

```log
2022-02-26 12:55:59.253 :    CSSD:907018304: clssgmStartNMMon: node 1 active, birth 538081862
2022-02-26 12:55:59.254 :    CSSD:907018304: clssgmStartNMMon: node 2 active, birth 538081860
...
2022-02-26 12:55:59.254 :    CSSD:2815416064: clssgmReconfigThread:  Setting initial active version for GMP to 12.2.0.1.0
2022-02-26 12:55:59.254 :    CSSD:2815416064: clssgmReconfigThread:  started for reconfig (538081862) with GM active version 12.2.0.1.0 and global active version 12.2.0.1.0
```
集群开始**GM**组层面的重新配置

```log
2022-02-26 12:55:59.985 :    CSSD:2815416064: clssgmCMReconfig: reconfiguration successful, incarnation 538081862 with 2 nodes, local node number 1, master node node2, number 2
```
集群**GM**组的重新配置结束，集群成员列表被更新，集群重新配置成功。至此，**OCSSD**启动结束