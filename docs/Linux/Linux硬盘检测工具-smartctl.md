### 介绍

通过 `smartctl` 的手册页可以看到如下一段话

> smartctl  controls  the  Self-Monitoring,  Analysis  and  Reporting  Technology  (SMART) system built into most ATA/SATA and
>        SCSI/SAS hard drives and solid-state drives.  The purpose of SMART is to monitor the reliability of the hard drive and  pre‐
>        dict drive failures, and to carry out different types of drive self-tests.  smartctl also supports some features not related
>        to SMART.  This version of smartctl is compatible with ACS-2, ATA8-ACS, ATA/ATAPI-7 and earlier  standards  (see  REFERENCES
>        below).
>
> `smartctl` 工具用来实现操作系统上的 `ATA/SATA` 、`SCSI/SAS`、`SSD` 等物理设备的监控、分析及使用情况报告。`SMART` 指的是对硬盘等设备的可靠性监控及预测磁盘可能存在的故障，并根据硬盘形态进行不同程度的自检。`smartctl` 的版本可以兼容众多磁盘规范，例如 ACS-2、ATA8-ACS、ATA/ATAPI-7 及更早期的一些磁盘标准。

`Smartctl` 工具对于 Linux操作系统的日常运维非常有用，可以对多种接口类型的磁盘进行故障检查，并将数据以报告的形式展现出来。

很多的 Linux 发行版已经内置了 `smartctl` 工具，你可以通过 `rpm -qa | grep smartmontools` 查看软件包是否安装。

```bash
[root@server ~]# rpm -qa | grep smartmontools
smartmontools-6.5-1.el7.x86_64
```

若未安装，可通过 `yum` 或 `apt` 进行安装

```bash
# redhat
[root@server ~]# yum install smartmontools
# debian,ubuntu
[root@server ~]# apt-get install smartmontools
```

`smartmontools` 提供了两种使用模式

- 以服务模式运行：通过 `smartd` 服务及配置文件来对服务器上的磁盘按照规则进行自检，需要开启 `smartd.service` 服务
- 以命令行模式运行：以终端命令行的方式对磁盘进行自检，无需开启 `smartd` 服务

### 使用

`smartctl` 提供了众多用于查看硬盘信息的方法

- 查看硬盘的基本参数 (设备型号、厂商、驱动版本等)

  ```bash
  [root@server ~]# smartctl -i /dev/sda
  smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-957.27.2.el7.x86_64] (local build)
  Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org
  
  === START OF INFORMATION SECTION ===
  Model Family:     Seagate Momentus 7200.4
  Device Model:     ST9160412AS
  Serial Number:    5VG4H9G3
  LU WWN Device Id: 5 000c50 02470255d
  Firmware Version: B006HPM1
  User Capacity:    160,041,885,696 bytes [160 GB]
  Sector Size:      512 bytes logical/physical
  Rotation Rate:    7200 rpm
  Form Factor:      2.5 inches
  Device is:        In smartctl database [for details use: -P show]
  ATA Version is:   ATA8-ACS T13/1699-D revision 4
  SATA Version is:  SATA 2.6, 3.0 Gb/s
  Local Time is:    Thu Dec  9 15:11:08 2021 CST
  SMART support is: Available - device has SMART capability.
  SMART support is: Enabled

- 查看硬盘的所有`smart`信息

  ```bash
  [root@server ~]# smartctl -a /dev/sda
  # 对于raid设备, 需要加上 `-d megaraid,{n}`  n 为raid控制器的编号
  [root@server ~]# #smartctl -a /dev/sda -d megaraid,{n}
  ```

- 查看硬盘的所有`smart`和`非smart`的信息

  ```bash
  [root@server ~]# smartctl -x /dev/sdb
  ```

- 查看系统上的所有设备

  ```bash
  [root@server ~]# smartctl --scan
  /dev/sda -d scsi # /dev/sda, SCSI device
  /dev/sdb -d scsi # /dev/sdb, SCSI device
  /dev/sdd -d scsi # /dev/sdd, SCSI device
  /dev/sde -d scsi # /dev/sde, SCSI device
  ```

- 查看`non-smart`设备的配置

  ```bash
  # -g 可选参数 {aam, apm, lookahead, security, wcache, rcache, wcreorder}
  [root@server ~]# smartctl -g apm /dev/sda
  smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-957.27.2.el7.x86_64] (local build)
  Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org
  
  APM feature is:   Disabled
  ```

- 指定模式查询磁盘信息

  ```bash
  # -q 可选参数 {errorsonly, silent, noserial}
  [root@server ~]# smartctl -a -q noserial /dev/sda    # 不打印硬盘的序列号
  [root@server ~]# smartctl -a -q silent /dev/sda     # 静默模式,不会输出任何信息
  [root@server ~]# smartctl -a -q errorsonly /dev/sda -l error  # 将硬盘相关的错误信息记录到smart日志中，并在开机的时候展示在启动界面
  ```

- 查看指定类型的硬盘信息 

  ```bash
  # -d 可选参数 {ata, scsi, nvme[,NSID], sat[,auto][,N][+TYPE], usbcypress[,X], usbjmicron[,p][,x][,N], usbprolific, usbsunplus, marvell, areca,N/E, 3ware,N, hpt,L/M/N, megaraid,N, aacraid,H,L,ID, cciss,N, auto, test}
  [root@server ~]# smartctl -a -d megaraid,0 /dev/sda 
  [root@server ~]# smartctl -a -d scsi /dev/sda
  ```

- 查看硬盘的健康状况

  ```bash
  [root@server ~]# smartctl -H /dev/sda
  smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-957.27.2.el7.x86_64] (local build)
  Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org
  
  === START OF READ SMART DATA SECTION ===
  SMART overall-health self-assessment test result: PASSED
  ```

- `long` 类型的硬盘测试

  ```bash
  [root@server ~]# smartctl --test=long /dev/sda
  smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-957.27.2.el7.x86_64] (local build)
  Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org
  
  === START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
  Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".
  Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.
  Testing has begun.
  Please wait 42 minutes for test to complete.
  Test will complete after Thu Dec  9 16:50:50 2021
  
  Use smartctl -X to abort test.
  ```

- 启用硬盘的 `smart` 功能

  ```bash
  [root@server ~]# smartctl --smart=on --offlineauto=on --saveauto=on /dev/sda
  ```

  注：一些硬盘是不支持`smart`的，因此虽然可以通过`smartctl`展示出来，但无法查看硬盘的详细参数。