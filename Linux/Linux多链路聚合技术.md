一般而言，在单体结构的操作系统中，一块物理磁盘会接在总线设备上，并经由总线分配 `PCI-Bus` 号，这个时候一个 `bus` 往往对应一个真实可见的设备。

但在多主机的集群环境中，多个主机之间使用交换机进行通信，多台存储服务器接在同一台(或多台)交换机上，此时通过同一台主机可能会看到来自存储上的多个物理设备，这些物理设备的路径各不相同，但对主机的操作系统来说，操作系统会将不同路径的设备设备看作是一块物理磁盘，区别仅仅是通向这个物理盘的路径不同而已。

但对操作系统来说很好辨认的路径，对于普通用户使用来讲可就不是那么回事了。事实上，对于普通用户来说，他们很难得知数据 I/O 活跃在哪条链路上。而使用 `DM Multipath(Device-Mapper Multipath, 设备映射多路径)` 则可以解决这个问题。

#### 什么是 `multipath`

`DM Multipath` 是一种将服务器节点和存储阵列间的多个 I/O 路径配置为一个单一设备的技术。这些 I/O 设备是可包含「独立电缆」、 「交换器」、「控制器」的实体 SAN 链接。多路径集合了 I/O 路径，并生成由这些路径组成的新设备。

使用 `multipath`，不仅可以获得 链路上的冗余功能，而且可以充分发挥存储设备的性能。以下为 `multipath` 多路径的功能

- 冗余：`DM Multipath` 提供两种配置模式 `主动` 和 `被动`，并可以在两种模式下切换用来实现故障转移。在两种模式下，只有一半的路径在每次进行 I/O时会被使用。若一条 I/O 路径的任一元素(电缆、交换器、控制器)出现故障后，`multipath` 将切换到另一条正常的路径上。
- 性能提升：`multipath` 提供的两种配置模式，可以将 I/O 以`轮循(round-robin)` 的方式分布到所有的路径中。在某些配置中，`multipath` 能够检测 I/O 路径的负载，并重新动态平衡负载。

![带两个RAID设备的主/被动多路径配置](../../../WorkSpace/notes/img/multipath-1.png)

如图为，带两个RAID设备的主动/被动多路径配置，其中下面的是两个硬件RAID设备、中间为两台 SAN 交换机、最上面为一台 server 节点的主机。

每个RAID设备都有两个 I/O路径， 以RAID A 为例，两条链路分别为 `RAID-A ---> SAN 1 ---> hba 1` 和 `RAID A ---> SAN 2 ---> hba 2`。

倘若配置了 multipath多路径，当 `RAID-A ---> SAN 1 ---> hba 1` 路径发生故障时，multipath 会则将 I/O 切换到 `RAID A ---> SAN 2 ---> hba 2` 这条链路上。

#### `multipath` 相关组件

- `dm-multipath kernel`：为路径和路径组重新指定 I/O 并进行故障转移。
- `mpathconf`：配置并启用 `DM Multipath`。
- `multipath`：列出并配置多路径设备，通常使用 `/etc/rc.sysinit` 启动，还可以在添加块设备时通过 `udev` 启动。
- `multipathd`：`multipath` 守护进程，若出现故障路径、`multipathd` 可能会启动路径组切换。对 `/etc/multipathd.conf` 配置文件的任何修改，都需要重新启动 `multipathd` 服务。
- `kpartx`：为设备分区生成设备映射器。`kpartx` 命令包含在自己的软件包中，但是 `DM Multipath` 软件包需要依赖它

#### 安装&配置

在 Linux RedHat 系列中，`multipath` 的软件包默认已经安装，你可以通过 `rpm -qa | grep device-mapper`查看

```bash
[root@server ~]# rpm -qa | grep device-mapper
device-mapper-libs-1.02.170-6.el7_9.5.x86_64
device-mapper-event-1.02.170-6.el7_9.5.x86_64
device-mapper-multipath-libs-0.4.9-135.el7_9.x86_64
device-mapper-persistent-data-0.8.5-3.el7_9.2.x86_64
device-mapper-1.02.170-6.el7_9.5.x86_64
device-mapper-event-libs-1.02.170-6.el7_9.5.x86_64
device-mapper-multipath-0.4.9-135.el7_9.x86_64
```

倘若缺少 `device-mapper-multipath-lib` 和 `device-mapper-multipath`，使用`yum install -y device-mapper device-mapper-multipath` 安装即可。

安装之后，加载 `device-mapper` 相关的驱动

```ba
[root@server ~]# modprobe dm_multipath dm_round_robin
[root@server ~]# lsmod | grep dm
dm_round_robin         12819  0
dm_multipath           27792  1 dm_round_robin
dm_mirror              22326  0
dm_region_hash         20813  1 dm_mirror
dm_log                 18411  2 dm_region_hash,dm_mirror
dm_mod                124501  9 dm_multipath,dm_log,dm_mirror
```

启动 `multipath` 服务

(PS: 第一次启动服务会失败，是因为缺少multipath的配置文件)

```bash
[root@server ~]# systemctl start multipathd.service
[root@server ~]# systemctl status multipathd
● multipathd.service - Device-Mapper Multipath Device Controller
   Loaded: loaded (/usr/lib/systemd/system/multipathd.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
Condition: start condition failed at 日 2021-11-28 02:13:45 EST; 2s ago
           ConditionPathExists=/etc/multipath.conf was not met
```

配置 `multipath`

配置 multipath 有两种方式，你可以使用 `mpathconf` 程序设置多路径，它可以创建多路径配置文件 `/etc/multipathd.conf`。

使用 `mpathconf` 配置时，有以下几种情况

- 如果 `/etc/multipath.conf` 文件已存在， `mpathconf` 程序会编辑它
- 如果 `/etc/multipath.conf` 文件不存在， `mpathconf` 程序会使用 `/usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf` 作为初始文件
- 如果 `/usr/share/doc/device-mapper-multipath-0.4.9/multipath.conf` 文件不存在， `mpathconf` 程序则会创建一个新的 `/etc/multipath.conf` 文件

如果不需要编辑 `/etc/multipath.comf` ，可以使用 `mapthconf --enable --with_multipathd y` 设置基本故障切换。

```bash
[root@server ~]# mpathconf --enable --with_multipathd y
```

如果需要编辑 `/etc/multipath.comf` ，可以使用 `mpathconf --enable` 设置基本的故障切换配置。

```bash
[root@server ~]# mpathconf --enable
```

以下将以编辑 `/etc/multipath.conf` 的方式来说明

```bash
[root@server ~]# cat /etc/multipath.conf
# This is a basic configuration file with some examples, for device mapper
# multipath.
#
# For a complete list of the default configuration values, run either
# multipath -t
# or
# multipathd show config
#
# For a list of configuration options with descriptions, see the multipath.conf
# man page

## By default, devices with vendor = "IBM" and product = "S/390.*" are
## blacklisted. To enable mulitpathing on these devies, uncomment the
## following lines.
#blacklist_exceptions {
#	device {
#		vendor	"IBM"
#		product	"S/390.*"
#	}
#}

## Use user friendly names, instead of using WWIDs as names.
defaults {
	user_friendly_names yes
	find_multipaths yes
}
##
## Here is an example of how to configure some standard options.
##
#
#defaults {
#	polling_interval 	10
#	path_selector		"round-robin 0"
#	path_grouping_policy	multibus
#	uid_attribute		ID_SERIAL
#	prio			alua
#	path_checker		readsector0
#	rr_min_io		100
#	max_fds			8192
#	rr_weight		priorities
#	failback		immediate
#	no_path_retry		fail
#	user_friendly_names	yes
#}
##
## The wwid line in the following blacklist section is shown as an example
## of how to blacklist devices by wwid.  The 2 devnode lines are the
## compiled in default blacklist. If you want to blacklist entire types
## of devices, such as all scsi devices, you should use a devnode line.
## However, if you want to blacklist specific devices, you should use
## a wwid line.  Since there is no guarantee that a specific device will
## not change names on reboot (from /dev/sda to /dev/sdb for example)
## devnode lines are not recommended for blacklisting specific devices.
##
#blacklist {
#       wwid 26353900f02796769
#	devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
#	devnode "^hd[a-z]"
#}
#multipaths {
#	multipath {
#		wwid			3600508b4000156d700012000000b0000
#		alias			yellow
#		path_grouping_policy	multibus
#		path_selector		"round-robin 0"
#		failback		manual
#		rr_weight		priorities
#		no_path_retry		5
#	}
#	multipath {
#		wwid			1DEC_____321816758474
#		alias			red
#	}
#}
#devices {
#	device {
#		vendor			"COMPAQ  "
#		product			"HSV110 (C)COMPAQ"
#		path_grouping_policy	multibus
#		path_checker		readsector0
#		path_selector		"round-robin 0"
#		hardware_handler	"0"
#		failback		15
#		rr_weight		priorities
#		no_path_retry		queue
#	}
#	device {
#		vendor			"COMPAQ  "
#		product			"MSA1000         "
#		path_grouping_policy	multibus
#	}
#}

blacklist {
}
```

以上为使用 `mpathconf --enbale` 后默认生成的 `/etc/multipath.conf` 配置文件。其中

```bash
defaults {		# multipath 的常规配置
	#user_friendly_names yes
	find_multipaths yes
}
# user_friendly_names 表示使用 /dev/mapper/mpath{n} 类型的设备名称代替 WWID 类型的设备

blacklist {		# 不被视为多路径的具体设备列表, 例如，系统盘等
	wwid 26353900f02796769
	devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
	devnode "^hd[a-z]"
}

blacklist_exceptions {		# 根据 blacklist 部分中的参数列出不在黑名单中的设备
	device {
		vendor	"IBM"
		product	"S/390.*"
	}
}

multipaths {		# 各个独立多路径设备的特性设备。如果在 default 和 devices 中配置了，则会被覆盖掉
	multipath {
		wwid			3600508b4000156d700012000000b0000
		alias			yellow
		path_grouping_policy	multibus
		path_selector		"round-robin 0"
		failback		manual
		rr_weight		priorities
		no_path_retry		5
	}
	multipath {
		wwid			1DEC_____321816758474
		alias			red
	}
}

devices {		# 各个存储控制器的设置。
	device {
		vendor			"COMPAQ  "
		product			"HSV110 (C)COMPAQ"
		path_grouping_policy	multibus
		path_checker		readsector0
		path_selector		"round-robin 0"
		hardware_handler	"0"
		failback		15
		rr_weight		priorities
		no_path_retry		queue
	}
	device {
		vendor			"COMPAQ  "
		product			"MSA1000         "
		path_grouping_policy	multibus
	}
}
```

编辑完成后，保存退出。

使用 `systemctl start multipathd.service` 重新启动 multipath 服务。

```bash
[root@server ~]# systemctl restart multipathd
[root@server ~]# systemctl status multipathd
● multipathd.service - Device-Mapper Multipath Device Controller
   Loaded: loaded (/usr/lib/systemd/system/multipathd.service; enabled; vendor preset: enabled)
   Active: active (running) since 日 2021-11-28 03:15:03 EST; 9s ago
  Process: 17887 ExecStart=/sbin/multipathd (code=exited, status=0/SUCCESS)
  Process: 17885 ExecStartPre=/sbin/multipath -A (code=exited, status=0/SUCCESS)
  Process: 17884 ExecStartPre=/sbin/modprobe dm-multipath (code=exited, status=0/SUCCESS)
 Main PID: 17890 (multipathd)
    Tasks: 6
   Memory: 1.6M
   CGroup: /system.slice/multipathd.service
           └─17890 /sbin/multipathd
```

注：当修改了 `/etc/multipath.conf` 文件后，一定要执行 `systemctl reload multipathd.conf` 重新加载服务。

通过 `multipath -v2` 查看生成的多路径盘符

```bash
[root@server ~]# multipath -v2
create: 3600a0b80001327510000009a436215ec undef 
size=12G features='0' hwhandler='0' wp=undef 
`-+- policy='round-robin 0' prio=1 status=undef
  |- 2:0:0:1 sdc 8:32 undef ready running 
  `- 3:0:0:1 sdg 8:96 undef ready running
create: 3600a0b80001327d800000070436216b3 undef 
size=12G features='0' hwhandler='0' wp=undef 
`-+- policy='round-robin 0' prio=1 status=undef
  |- 2:0:0:2 sdd 8:48 undef ready running 
  `- 3:0:0:2 sdg 8:112 undef ready running
```

create: 后面的 `3600a0b80001327510000009a436215ec` 即为创建的多路径设备，你可以在 `/dev/mapper/` 路径下找到它们。

#### `multipath` 的管理及故障排除

使用 `multipath -l` 查看多路径设备

```bash
[root@server ~]# multipath -l
3600d0230000000000e13955cc3757800 dm-1 WINSYS,SF2372 size=269G features='0' hwhandler='0' wp=rw
|-+- policy='round-robin 0' prio=1 status=active
| `- 6:0:0:0 sdb 8:16 active ready running
`-+- policy='round-robin 0' prio=1 status=enabled
  `- 7:0:0:0 sdf 8:80 active ready running
```

删除多路径设备

```bash
[root@server ~]# multipath -f 3600d0230000000000e13955cc3757800
```

删除所有的多路径设备

```bash
[root@server ~]# multipath -F
```

以上列出了3条 `multipath` 日常使用的命令，而这也是我日常使用最频繁的命令，其他的命令我也还在学习🤪。

#### 最后

当你通过 `multipath` 创建了多路径设备后，你可以像使用普通设备那样使用它。创建分区、挂载文件系统、创建系统卷，具体的功能还需要你去探索。

