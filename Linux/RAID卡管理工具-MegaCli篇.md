对于机械磁盘及固态磁盘，往往随着使用的时间增长磁盘本身会出现各种各样的问题，尤其对于机械盘来讲，长时间或高数据量的读写均有可能导致磁盘出现坏道。然而一旦磁盘出现问题后，对于操作系统的影响也是致命的，严重的会导致数据丢失，系统无法开机等现象。  

通常情况下，企业会给存放重要数据的服务器配备RAID卡，并将系统盘制作成RAID-1模式，用来做重要数据的镜像，防止因为单盘系统故障导致系统无法无法开机，系统中数据无法访问的情况。  

对于配置了RAID卡的服务器，为了方便日常查看、监控、管理RAID卡及磁盘运行状态，自然需要一种能够管理硬件RAID卡的工具。目前市面上使用的大多数RAID卡支持`MegaCli`工具来进行RAID卡的日常维护、监控等。  

[toc]

#### 安装`MegaCli`
LSI的`MegaCli`属于操作系统外部软件，使用前需要安装。可以通过`rpm -qa|grep MegaCli`查看当前操作系统是否安装了该工具。
```bash
$ rpm -qa |grep MegaCli
MegaCli-8.07.14-1.noarch
```
下载[MegaCli](https://link.jianshu.com/?t=https://docs.broadcom.com/docs-and-downloads/raid-controllers/raid-controllers-common-files/8-07-14_MegaCLI.zip)
解压安装包后，选择对应的操作系统平台进行安装 *以Linux为例*
```bash
$ unzip 8-07-14_MegaCLI.zip -d /tmp/
$ cd Linux && rpm -ivh MegaCli-8.07.14-1.noarch.rpm
```

#### 常用命令
* 查询版本
```bash
$ MegaCli v
```
* 查看RAID控制器数量
```bash
$ MegaCli -adpCount
```
* 查询系统、硬件和存储概览
```bash
$ MegaCli -ShowSummary {-f filename} -a0  # a0 为RAID卡适配器编号
```
* 查询适配器的*Enclosures*信息
```bash
$ MegaCli -EncInfo -a0
```

##### RAID控制器相关
* 查询控制器信息
```bash
$ MegaCli -AdpAllInfo -a0
```
* 查询/设置控制器时间
```bash
$ MegaCli -AdpGetTime -a0
$ MegaCli -AdpSetTime yyyymmdd hh:mm:ss -a0
```
* 启用/禁用*auto select boot*
```bash
$ MegaCli -AdpBIOS EnblAutoSelectBootLd /*DsblAutoSelectBootLd*/ -a0
$ MegaCli -AdpBIOS -Dsply -a0
```
* 启用/禁用*Auto Rebuild(自动重建)*
```bash
$ MegaCli -AdpAutoRbld -Enbl /*-Dsbl*/ -a0
$ MegaCli -AdpAutoRbld -Dsply -a0
```

##### 磁盘相关
* 查看所有磁盘信息
```bash
$ MegaCli -PDList -a0
```
* 查看物理磁盘的数量
```bash
$ MegaCli -PDGetnum -a0
```
* 查看指定磁盘信息
```bash
$ MegaCli -pdInfo -PhysDrv[E0:S0] -a0  # E0 为Enclosure Device ID, S0 为磁盘的 Device Id
```
* online离线磁盘
```bash
$ MegaCli -PDOnline  -PhysDrv[E0:S0] -a0
```
* offline在线磁盘
```bash
$ MegaCli -PDOffline -PhysDrv[E0:S0] -a0
```
* 将磁盘配置为*Unconfigured Good*状态
```bash
$ MegaCli -PDMakeGood -PhysDrv[E0:S0] {-Force} -a0
```
* 配置阵列卡为*JBOD*模式
```bash
$ MegaCli -PDMakeJBOD -PhysDrv[E0:S0] -a0
```
* 查询RAID重建进度
```bash
$ MegaCli -PDRbld -ShowProg -PhysDrv[E0:S0] -a0 
$ MegaCli -PDRbld -ProgDsply -PhysDrv[E0:S0] -a0
```
* 开始/停止/暂停/恢复RAID重建进程
```bash
$ MegaCli -PDRbld -Start /*-Stop,-Suspend,-Resume*/ -ProgDsply -PhysDrv [E0:S0] -a0
```
* 查询清除磁盘配置进度
```bash
$ MegaCli -PDClear -ProgDsply -PhysDrv[E0:S0] -a0
```
* 开启/停止磁盘清除策略
```bash
$ MegaCli -PDClear -Stop /*-Start*/ -PhysDrv[32:13] -a0
```
* 开启/关闭磁盘定位指示灯
```bash
$ MegaCli64 -PDLocate -Start /*-Stop*/ -PhysDrv[E0:slot_id] -a0
```








##### RAID相关
* 创建RAID[0,1,5]
```bash
$ MegaCli -CfgLdAdd -r0[E0:S0] WT NORA Direct NoCachedBadBBU -Force -a0  # r0 表示raid级别为0
$ MegaCli -CfgLdAdd -r1[E0:S0,E0:S1] WB RA Direct NoCachedBadBBU -Force -a0
$ MegaCli -CfgLdAdd -r5[E0:S0,E0:S1,E0:S2] WB RA Direct NoCachedBadBBU -Force -a0
```
* 创建RAID[10,50,60]
```bash
$ MegaCli -CfgSpanAdd -r10 -Array0[E0:S0,E0:S1] -Array1[E0:S2,E0:S4] WB RA Direct NoCachedBadBBU -a0
```
* 删除RAID组
```bash
$ MegaCli -CfgLdDel -L0 {-Force} -a0  # L0 表示raid组的ID, 可通过 MegaCli -CfgDsply -a0 | grep -i "^Virtual Drive:"查询
```
* 配置/删除全局热备盘
```bash
$ MegaCli -PDHSP -Set -PhysDrv[E0:S0] -a0
$ MegaCli -PDHSP -Rmv -PhysDrv[E0:S0] -a0
```




##### CacheCade(缓存池)相关
* 创建缓存池
```bash
$ MegaCli -CfgCachecadeAdd -Physdrv[E0:S0] {-Name LdNamestring} WT - assign -L0 -a0
$ MegaCli -CfgCachecadeAdd -Physdrv[E0:S0] {-Name LdNamestring} WB - assign -L1 -a0
$ MegaCli -CfgCachecadeAdd -Physdrv[E0:S0] {-Name LdNamestring} ForcedWB - assign -L1 -a0
```
* 删除缓存池
```bash
$ MegaCli -CfgCachecadeDel -L0 -a0
```
* 查询控制器对应的缓存设备
```bash
$ MegaCli -CfgCacheCadeDsply -a0
```
* 关联/(移除)虚拟设备
```bash
$ MegaCli -Cachecade -assign -L0 -a0
$ MegaCli -Cachecade -remove -L0 -a0
```


##### 日志相关
* 收集硬件事件日志
```bash
$ MegaCli -AdpEventLog -GetEventLogInfo -a0
```
* 收集指定级别的硬件日志
```bash
$ MegaCli -AdpEventLog -GetEvents -warning {-f filename} -a0 # 事件级别包含 info、warning、critical、fatal
```
* 收集上次系统事件的日志
```bash
$ MegaCli -AdpEventLog -GetSinceShutdown {-info -warning -critical -fatal} {-f <fileName>} -a0
$ MegaCli -AdpEventLog -GetSinceReboot {-info -warning -critical -fatal} {-f <fileName>} -a0 
```

##### RAID卡电池相关
* 查看RAID卡电池总信息
```bash
$ MegaCli -AdpBbuCmd -a0
```
* 查看RAID卡电池状态
```bash
$ MegaCli -AdpBbuCmd -GetBbuStatus -a0
```
* 查看RAID卡电池设备信息
```bash
$ MegaCli -AdpBbuCmd -GetBbuCapacityInfo -a0
```
* 查看电池参数
```bash
$ MegaCli -AdpBbuCmd -GetBbuDesignInfo -a0
```
* 查看电池属性
```bash
$ MegaCli -AdpBbuCmd -GetBbuProperties -a0
```
* 配置*Auto-Learn(自动校准)*
```bash
$ MegaCli -AdpBbuCmd -BbuLearn -a0
```



