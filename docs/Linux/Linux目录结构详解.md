### Linux目录结构结构的构成

#### 前言

对于绝大多数编程人员来说，对于 Linux 操作系统的目录应该是非常熟悉的。但往往只限于几个常用的目录，因此，本文以 Linux 操作系统的目录结构为基本，详细解释每个目录在操作系统中的意义所在。

#### 文件系统的目录结构

前面的文章中提到过 Linux 文件系统 「一切皆文件」，Linux 操作系统的目录结构以树形来看的话，相当于一棵倒立的树。如果你使用 `tree` 来查看操作系统的目录结构，你会看到下面这种结构

```bash
[root@server /]# tree -L 1
.
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib64 -> usr/lib64
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

我们根据顺序来看各目录的功能和用处。

##### `/bin` 目录

`bin` 是 `Binaries(二进制文件)` 的缩写，这个目录下存放着 操作系统的系统命令。这些命令都是经过编译后的二进制文件，使用 `cat` 或 `vim` 无法看到源程序的内容。可以通过 `ls` 查看该目录下的所有文件。

```bash
[root@server bin]# ls
[                                    fipscheck                    mv                        sgm_dd
a2p                                  fipshmac                     nail                      sg_modes
abrt-action-analyze-backtrace        firefox                      namei                     sg_opcodes
abrt-action-analyze-c                firewall-cmd                 ndptool                   sgp_dd
abrt-action-analyze-ccpp-local       firewall-offline-cmd         needs-restarting          sg_persist
...
find2perl                            msgmerge                     sg_logs                   znew
find-jar                             msgunfmt                     sg_luns                   zsoelim
findmnt                              msguniq                      sg_map
find-repos-of-install                ms_print                     sg_map26
```

`/bin` 目录下的文件都具有可执行权限。当你登录终端后，仅需要在终端输入对应的命令和参数即可执行该二进制文件。例如，你最常用的 `cp`、`mv`、`ls` 等命令都位于 `/bin` 目录下。

##### `/boot` 目录

`/boot` 目录，是 Linux操作系统中至关重要的一个目录。该目录下存放着操作系统启动所需要的镜像文件和内核映像文件。若镜像文件损坏或丢失，服务器是无法启动的。

通过在 `tree` 命令查看 `/boot` 目录包含哪些文件

```bash
[root@server boot]# tree -L 2
.
├── config-3.10.0-1160.45.1.el7.x86_64
├── config-3.10.0-1160.el7.x86_64
├── efi
│   └── EFI
├── grub
│   └── splash.xpm.gz
├── grub2
│   ├── device.map
│   ├── fonts
│   ├── grub.cfg
│   ├── grubenv
│   ├── i386-pc
│   └── locale
├── initramfs-0-rescue-c0dbfe367e164c868e38906295f5dfa1.img
├── initramfs-3.10.0-1160.45.1.el7.x86_64.img
├── initramfs-3.10.0-1160.el7.x86_64.img
├── symvers-3.10.0-1160.45.1.el7.x86_64.gz
├── symvers-3.10.0-1160.el7.x86_64.gz
├── System.map-3.10.0-1160.45.1.el7.x86_64
├── System.map-3.10.0-1160.el7.x86_64
├── vmlinuz-0-rescue-c0dbfe367e164c868e38906295f5dfa1
├── vmlinuz-3.10.0-1160.45.1.el7.x86_64
└── vmlinuz-3.10.0-1160.el7.x86_64

7 directories, 16 files
```

可以看到，所有以 `.x86_64` 为后缀的文件表示当前操作系统是基于 「x86」架构的 64 位系统，不同架构的服务器对应的标识是不一样的；所有以 `.img` 位后缀的文件就是镜像文件了，其中以 `initramfs-` 为前缀的则表明是开机启动所需的镜像文件。以 `vmlinuz-` 为前缀的文件为服务器开机时需要加载的内核启动文件。

以第 17 行 `initramfs-3.10.0-1160.45.1.el7.x86_64.img` 为例进行说明

![内核结构](../../../WorkSpace/notes/img/linux-2-6610266.png)

`grub` 和 `efi` 两个目录分别对应两种启动模式：`legacy` 和 `UEFI` ，目录下的文件为对应启动引导项的配置文件。

`system.map` 文件为索引文件，以内存地址的格式记录内核中富豪连接的位置。

##### `/dev` 目录

`dev` 是 `device(设备)` 的简写。顾名思义，`/dev/` 目录下存放操作系统上的所有外设设备，包含 `块设备` 和 `字符设备`。进入 `/dev/` 目录下查看目录下的文件

```bash
[root@server dev]# ls
autofs           dm-0       log                 ptmx      tty    tty20  tty33  tty46  tty59  uinput   vcs4   vga_arbiter
block            dm-1       loop-control        pts       tty0   tty21  tty34  tty47  tty6   urandom  vcs5   vhci
bsg              dri        mapper              random    tty1   tty22  tty35  tty48  tty60  usbmon0  vcs6   vhost-net
btrfs-control    fb0        mcelog              raw       tty10  tty23  tty36  tty49  tty61  usbmon1  vcsa   virtio-ports
bus              fd         mem                 rtc       tty11  tty24  tty37  tty5   tty62  usbmon2  vcsa1  vport2p1
cdrom            full       mqueue              rtc0      tty12  tty25  tty38  tty50  tty63  usbmon3  vcsa2  vport2p2
centos           fuse       net                 sg0       tty13  tty26  tty39  tty51  tty7   usbmon4  vcsa3  zero
char             hidraw0    network_latency     shm       tty14  tty27  tty4   tty52  tty8   usbmon5  vcsa4
console          hpet       network_throughput  snapshot  tty15  tty28  tty40  tty53  tty9   usbmon6  vcsa5
core             hugepages  null                snd       tty16  tty29  tty41  tty54  ttyS0  usbmon7  vcsa6
cpu              hwrng      nvram               sr0       tty17  tty3   tty42  tty55  ttyS1  vcs      vda
cpu_dma_latency  initctl    oldmem              stderr    tty18  tty30  tty43  tty56  ttyS2  vcs1     vda1
crash            input      port                stdin     tty19  tty31  tty44  tty57  ttyS3  vcs2     vda2
disk             kmsg       ppp                 stdout    tty2   tty32  tty45  tty58  uhid   vcs3     vfio
```

这里以 `/dev/` 目录下常用的一些作为说明

`block`: 该目录下为服务器上的总线设备的映射，例如磁盘等设备的槽位映射关系，以符号链接文件的形式存在；

`bus`: 该目录下的文件为服务器上的总线设备； 

`cdrom`: 光驱设备。一般在文件系统中挂载镜像文件，会将镜像挂载到 `cdrom` 商，和 `sr0` 是同一个设备(链接文件，通过 `ls -lh /dev/cdrom` 可以看到改文件是指向 `/dev/sr0` 的) 

`cpu`: 该目录下为 cpu 设备对应的硬件文件。该目录的子目录中以 cpu core 为命名，数量应与 cpu 核数一致；

`dm-*`: 对应服务器上的硬件设备。`dm-0` 对应系统盘上的第一个分区，`dm-1` 对应系统盘上的第二个分区；其他 dm-* 设备可能是存储介质(SSD/HDD等)，或其他一些设备；

`stdin|stdout|stderr`: 分别为 `标准输入`、`标准输出`、`标准错误`。操作系统上的每个进程启动时，都会产生对应的 `标准输出`、`标准输入`、及 `标准错误`；

`tty`: 终端设备。用户登陆到操作系统时，操作系统会为登陆的用户分配一个 `tty`。常见的有 `Last login: Mon Nov  8 19:24:56 on ttys000`

`null|random|zero`: 特殊设备。`/dev/null` 俗称 黑洞。所有试图写入这里的数据都会被丢弃；`/dev/random` 为生成随机字符的文件；`/dev/zero` 为 “0”设备，使用时会生成全部为 0 的数据，常用于清除磁盘的元数据或创建 “0” 文件。

##### `/etc` 目录

`/etc` 目录是Linux操作系统上的系统管理配置目录，基本上所有的服务启动、用户权限、主机(系统)设置、内核(驱动)配置 等相关的配置文件都存放在该目录下；

对于Linux系统来说，配置文件在操作系统上起着一场重要的作用。往往安装一个软件、自定义一项服务，都需要使用配置文件来达到需要；

##### `/home` 目录

`/home` 目录为用户目录，Linux 操作系统为 `多用户&多任务 `操作系统，允许用户间的来回切换。通过 `su - {user}` 切换用户角色后，默认进入的就在用户的家目录下。

Linux 上崇尚权限原则，并为每个用户配置了不同的权限，以保证文件不被随意篡改和整个操作系统的安全性。

##### `/lib` 目录 & /lib64 目录

`lib` 是 `Library（库）`的缩写，这个目录下放着系统最基本的动态链接库，一般是32 bit的。

正如操作系统分为 `32 bit` 和 `64 bit`位操作系统，系统上的 动态链接库为了能满足不同 bit 位的需求，也分为 `32 bit(/lib)` 和 `64 bit(/lib64)`。类似的还有 `x86` 和 `x86_64`。

Linux 在安装软件或服务时，往往会下载很多依赖，这些依赖文件大部分都是动态链接文件，因此众多依赖文件也被保存在 `lib` 目录下；

##### `/media` 目录

`media` 目录常用于挂载操作，通过将镜像文件挂载到本地文件系统上，以方便访问镜像中的文件。例如，通过 `iso` 镜像安装软件包；

##### `/mnt` 目录

`mnt` 为 `mount(挂载)` 的缩写。该目录主要用于作为操作系统的挂载点，常用于挂载光驱、USB灯设备。通过 `mount` 挂载的设备通常会展示在该目录下；

通过 `mount` 挂载一个光驱设备，在对应的挂载目录中可以看到光驱设备中的文件

```bash
[root@server mnt]# mkdir /mnt/cdrom
[root@server mnt]# mount /dev/sr0 /mnt/cdrom
mount: /dev/sr0 写保护，将以只读方式挂载
[root@server mnt]# cd cdrom/
[root@server cdrom]# ls
CentOS_BuildTag  EULA  images    LiveOS    repodata              RPM-GPG-KEY-CentOS-Testing-7
EFI              GPL   isolinux  Packages  RPM-GPG-KEY-CentOS-7  TRANS.TBL
```

通过 `umount` 命令可以取消挂载，光驱设备被卸载掉之后，对应挂载目录会变成空目录

```bash
[root@server mnt]# umount /dev/sr0
[root@server mnt]# ls cdrom/
[root@server mnt]# 
```

##### `/opt` 目录

`opt` 目录用来安装附加或第三方的软件包，是用户级的程序目录。例如，当你需要在Linux操作系统上安装 `Docker` 容器时，就可以将 容器 安装在 `/opt/` 目录下。

作为用户级的程序目录，你完全可以在磁盘空间不够或不想使用时通过 `rm -rf` 进行源文件的删除，这并不会对系统造成什么影响。

甚至，你可以将 `/opt` 目录作为一个设备挂载到操作系统的其他目录下，以此来保证空间的充分利用。

例如，你需要在操作系统上安装一个基于RAID卡的管理工具 `MegaCli`，那么首选位置应该就是 `/opt/` 目录。

```bash
[root@server opt]# ls MegaRAID/
MegaCli  perccli  storcli

```

##### `/proc` 目录

`proc` 的意思是 `process(程序/进程)`。Linux系统上的 `/proc` 目录是一种伪文件系统，存储的是当前操作系统上所有硬件设备和正在运行的进程的信息。

`/proc/` 目录下包含很多以数字命名的目录，这些目录名表示当前正在运行的进程id，你可以通过 `ps -ef | grep {proc_id}` 来查看该进程的名称及运行状态。

```bash
[root@server proc]# ls
1     1365  21   28   367  388  4446  657   710  7641  buddyinfo  driver       kcore       misc          self           tty
10    1372  22   283  368  389  469   658   717  7976  bus        execdomains  keys        modules       slabinfo       uptime
1099  14    23   29   38   39   488   659   718  7998  cgroups    fb           key-users   mounts        softirqs       version
11    15    24   293  381  390  5     660   719  8     cmdline    filesystems  kmsg        mtrr          stat           vmallocinfo
1101  16    266  294  382  391  505   661   721  8185  consoles   fs           kpagecount  net           swaps          vmstat
1104  17    27   295  383  4    56    6763  731  8432  cpuinfo    interrupts   kpageflags  pagetypeinfo  sys            zoneinfo
1109  18    275  296  384  40   6     6785  738  8438  crypto     iomem        loadavg     partitions    sysrq-trigger
1120  19    276  30   385  41   654   684   745  9     devices    ioports      locks       sched_debug   sysvipc
1269  2     277  357  386  42   655   7     755  91    diskstats  irq          mdstat      schedstat     timer_list
13    20    278  358  387  43   656   708   762  acpi  dma        kallsyms     meminfo     scsi          timer_stats
```

这里我们以 `7976` 这个目录名为例，通过 `ps` 查看该进程id对应的 进程信息

```bash
[root@server proc]# ps -ef | grep 7976
root      7976     1  0 01:38 ?        00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 5d342dd503ef4662b99b9b59fec092fdbdf3a084a77b05073cd5f24726712cbb -address /run/containerd/containerd.sock
root      7998  7976  0 01:38 pts/0    00:00:00 /bin/bash
root      8710  6785  0 01:51 pts/0    00:00:00 grep --color=auto 7976
```

可以看到，`7976` 号进程对应的程序为 `/usr/bin/containerd-shim-runc-v2` 。如果你想了解进程的详细信息，那么你需要进入到 进程号 对应的目录下，去查找你需要的信息了。

例如，你想要了解该进程运行时可以使用的资源限制时，你可以

```bash
[root@server proc]# cd 7976/
[root@server 7976]# ls
attr        cmdline          environ  io         mem         ns             pagemap      sched      stack    task
autogroup   comm             exe      limits     mountinfo   numa_maps      patch_state  schedstat  stat     timers
auxv        coredump_filter  fd       loginuid   mounts      oom_adj        personality  sessionid  statm    uid_map
cgroup      cpuset           fdinfo   map_files  mountstats  oom_score      projid_map   setgroups  status   wchan
clear_refs  cwd              gid_map  maps       net         oom_score_adj  root         smaps      syscall
[root@server 7976]# cat limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        unlimited            unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             unlimited            unlimited            processes
Max open files            1048576              1048576              files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       3871                 3871                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```

对于硬件设备来说，当操作系统启动的那一刻，所有的硬件设备都以文件的形式存在于 `/proc` 目录下，这些文件反应着硬件设备的运行情况。

例如，`/proc/cpuinfo` 代表当前操作系统中 cpu 的信息

```bash
[root@server proc]# cat /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 13
model name	: QEMU Virtual CPU version 2.5+
stepping	: 3
microcode	: 0x1
cpu MHz		: 2599.998
cache size	: 16384 KB
physical id	: 0
siblings	: 1
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pse36 clflush mmx fxsr sse sse2 syscall nx lm rep_good nopl xtopology eagerfpu pni cx16 x2apic hypervisor lahf_lm
bogomips	: 5199.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:
```

类似的还有很多，例如内存、磁盘、网络等等。简而言之，`/proc` 在操作系统中扮演着非常重要的角色，如果你对 `/proc` 目录非常熟悉，那么你完全通过 `/proc` 来监控系统的运行状态。

##### `/root` 目录

`root` 目录是 root 用户的家目录。作为一个用户，root 区别于其他的用户，它是Linux 操作系统的超级管理员，具有操作系统上的最高权限。

当你使用 root 用户登录操作系统时，意味着你可以对操作系统的一切资源进行分配。但事实非这样，尽管 root 用户作为超级管理员，但也并不意味着可以无法无天。

试想， root 用户想要去修改一些私有资源或正在运行中的进程的状态，最终导致资源异常或进程异常退出。那么，操作系统会允许这样的情况发生吗？答案是不会的。

Linux在声明 root 用户的的超级权限时，为了增强系统安全，又对 root 用户的权限做了限制，而这正是 `SELinux(安全增强型Linux)`。在保障超级权限的同时又以资源所属对用户做出限制。

##### `/run` 目录

操作系统上的 `/run` 目录是一个内存中的临时文件系统，主要用来存储系统启动以来的信息。当系统重启时，`/run` 目录下的目录和文件会被清理掉。通常，操作系统上的`/var/run` 会指向挂载在根目录上的 `/run`。

因此，不妨重启试试看 `/run` 目录会发生什么变化。

这个目录下主要包含两种格式的文件，分别是 `.pid` 和 `.sock。pid 文件表示当前进程使用的 进程号；sock 文件为服务对应的守护进程与socket套接字通信使用的监听通道。参考自：([关于/var/run/docker.sock](https://www.cnblogs.com/fundebug/p/6723464.html))

##### `/sbin` 目录

`/sbin` 目录与 `/bin` 目录非常类似，区别在于 `/sbin` 目录下存放的是系统级的管理命令，而 `/bin` 目录下存放的用户级的指令。

为什么会说 `/sbin` 目录下是系统级的管理命令呢？我们先来看看 `/sbin` 目录下有什么

```bash
[root@server sbin]# ls
abrt-auto-reporting      e2image                    iptables-restore     partprobe                   swaplabel
abrt-configuration       e2label                    iptables-save        partx                       swapoff
abrtd                    e2undo                     iptunnel             pcscd                       swapon
abrt-dbus                e4defrag                   irqbalance           pdata_tools                 switch_root
...
ifconfig	reboot		shutdown
```

其实不难看出，这个目录下的程序多是对于系统进行配置的指令。例如，`ifconfig`、`xfs_repair`、`lvcreate`等等。通过这些指令配置后往往操作系统上的所有用户都可以感知到，属于系统级的公共资源。

那为什么又说 `/bin` 目录是用户级的管理目录。我们同样来到 `/bin` 目录下查看其文件

```bash
[root@server bin]# ls
[                                    fipscheck                    mv                        sgm_dd
a2p                                  fipshmac                     nail                      sg_modes
abrt-action-analyze-backtrace        firefox                      namei                     sg_opcodes
abrt-action-analyze-c                firewall-cmd                 ndptool                   sgp_dd
...
findmnt                              msguniq                      sg_map
find-repos-of-install                ms_print                     sg_map26
```

例如最常用的 `ls`、`find`、`mount`等命令都是放在 `/bin` 目录下的，属于功能商的应用，操作这些指令并不会对操作系统造成严重的影响，大部分仅限于资源上的消耗。我们在通过 `ls -lh /bin` 查看，会发现 `/bin` 目录是指向 `/usr/bin` 的。

```bash
[root@server bin]# ls -lh /bin
lrwxrwxrwx. 1 root root 7 9月   2 08:54 /bin -> usr/bin
```

##### `/srv` 目录

主要用来存储本机或本服务器提供的服务或数据。一般很少关注，这里不做解释。

##### `/sys` 目录

在了解 `/sys` 目录前，先简单来了解下 `sysfs` 是什么东西？

Linux 操作系统上，`sysfs(System File System)` 是类似于 `/proc` 这样的特殊文件系统，可以帮助用户更方便的对系统硬件资源进行管理。主要用于将系统上的设备组织成层次结构，并由用户态的程序调用和使用这些资源。

操作系统上的 `sysfs` 文件系统正是挂载在 `/sys` 挂载点上的。倘若需要查看该挂载点的信息，你可以通过 `df -lTh /sys` 进行查看

```bash
[root@server ~]# df -lTh /sys
文件系统       类型   容量  已用  可用 已用% 挂载点
sysfs          sysfs     0     0     0     - /sys
```

实际上，`/sysfs` 文件系统将服务器上的所有硬件设备以树形目录的形式挂载到 `/sys` 目录下，正如你在 `/sys` 目录下看到的

```bash
[root@server sys]# ls -F
block/  bus/  class/  dev/  devices/  firmware/  fs/  hypervisor/  kernel/  module/  power/
```

`/sys/block` 目录下是所有设备的真实对象。例如，`/sys/block/dm-0`、`/sys/block/sr0` 等都指向真实的物理设备。

`/sys/bus` 目录是按照总线类型分层存放的，其子目录下的 devices 目录中的所有设备都是连接在总线上的设备，通过该目录下的具体设备可以找到总线分配给设备的bus号。

```bash
[root@server sys]# ls -lh bus/usb/devices/*
lrwxrwxrwx. 1 root root 0 11月  9 04:47 bus/usb/devices/1-0:1.0 -> ../../../devices/pci0000:00/0000:00:05.0/usb1/1-0:1.0
lrwxrwxrwx. 1 root root 0 11月  9 04:47 bus/usb/devices/2-0:1.0 -> ../../../devices/pci0000:00/0000:00:07.0/usb2/2-0:1.0
```

如上为操作系统中的 usb 设备对应在总线上的符号链接，可以看到 devices/1-0:1.0 这个设备通过总线分配的 bus 号为 0000:00:05.0。pci0000:00 则表示该设备是一个pcie设备，使用的是 pcie 总线。

`/sys/class` 目录是按照设备功能分类的结构模型，通过将操作系统上的所有设备以功能来区分，具有相同功能的设备则被归类到同一个目录下。

例如，`/sys/class/block/` 目录则包含操作系统上的所有块设备，无论这个设备是以哪种总线连接在系统上的。

```bash
[root@server sys]# ls -lh class/block/*
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/dm-0 -> ../../devices/virtual/block/dm-0
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/dm-1 -> ../../devices/virtual/block/dm-1
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/sr0 -> ../../devices/pci0000:00/0000:00:01.1/ata1/host0/target0:0:1/0:0:1:0/block/sr0
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/vda -> ../../devices/pci0000:00/0000:00:0a.0/virtio3/block/vda
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/vda1 -> ../../devices/pci0000:00/0000:00:0a.0/virtio3/block/vda/vda1
lrwxrwxrwx. 1 root root 0 11月  9 04:47 class/block/vda2 -> ../../devices/pci0000:00/0000:00:0a.0/virtio3/block/vda/vda2
```

`/sys/kernel/` 目录下为内核所有可调整参数的位置。不过，更多情况下更改内核参数会通过 `/etc/sysctl.conf` 或 `/proc/sysctl/kernel/` 下的文件来修改内核参数。

`/sys/module` 目录下包含系统上所有的模块信息。

`/sys/power` 目录下为系统中的电源相关的选项，可以向该目录下的文件中写入指令控制操作系统的开关机。参考自：[Linux 内核/sys 文件系统介绍](http://blog.chinaunix.net/uid-29616823-id-4456462.html)

##### `/tmp` 目录

`/tmp` 为英文单词 `temporary(临时)` 的意思，`/tmp` 目录在Linux操作系统中的作用比较奇特，当你将一个文件放在该目录下，且该文件没有任何程序或文件饮用的情况下，通过重启操作系统，系统会将 `/tmp` 目录下没有被引用的文件清理掉来减少已使用的存储空间。

而这，是区别于操作系统其他目录的。因此，重要文件通常是不能放在该目录下，而能被放在 `/tmp` 目录下的，可以是一些程序的安装包、不重要的日志信息等。

另一方面，`/tmp` 目录同样为内存文件系统，对应的挂载文件系统格式为 tmpfs。`/tmp` 目录消耗的是内存容量，而非物理磁盘。因此，若是向 `/tmp` 目录下写入大量文件，也会造成内存资源的严重消耗，而这往往是不被允许的。

##### `/usr` 目录

`/usr` 目录是Linux操作系统的核心目录，包含所有的共享文件。正如你所了解的 `/bin`、`/sbin`、`/lib`、`/lib64` 等目录都是作为符号链接文件共享到 `/usr` 目录下的。

你可以通过 `ls -lh /` 来查看这些链接文件

```bash
[root@server usr]# ls -lh /
总用量 20K
lrwxrwxrwx.   1 root root    7 9月   2 08:54 bin -> usr/bin
dr-xr-xr-x.   5 root root 4.0K 10月 27 06:56 boot
drwxr-xr-x.  21 root root 3.2K 11月  9 04:47 dev
drwxr-xr-x.  95 root root 8.0K 10月 27 07:14 etc
drwxr-xr-x.   3 root root   20 9月   8 04:10 home
lrwxrwxrwx.   1 root root    7 9月   2 08:54 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 9月   2 08:54 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 4月  11 2018 media
drwxr-xr-x.   3 root root   19 11月  9 01:22 mnt
drwxr-xr-x.   3 root root   24 9月   8 04:20 opt
dr-xr-xr-x. 108 root root    0 11月  9 04:47 proc
dr-xr-x---.   4 root root  265 11月  4 04:05 root
drwxr-xr-x.  29 root root  880 11月  9 04:47 run
lrwxrwxrwx.   1 root root    8 9月   2 08:54 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 4月  11 2018 srv
dr-xr-xr-x.  13 root root    0 11月  9 04:47 sys
drwxrwxrwt.  12 root root 4.0K 11月 10 03:23 tmp
drwxr-xr-x.  13 root root  155 9月   2 08:54 usr
drwxr-xr-x.  19 root root  267 9月   2 21:21 var
```

 `/usr` 目录专门存放各种程序和数据，即便是自定义安装的一些程序也不例外。通常，一般会将自定义安装的程序安装在 `/usr/local/` 目录下；`/usr/src` 目录下存放的是操作系统内核的源码相关的文件；`/usr/share` 目录包含各种程序间使用的字体、文档、图标等资源。

##### `/var` 目录

`/var` 目录下主要存放操作系统中不断发生变化(常态性变化)的文件，例如，操作系统自身产生的日志文件，进程运行时产生的锁文件、用户登录活动等等。

这里对最常用的几个目录加以说明

`/var/log` : 该目录下为操作系统中的日志文件，包含系统日志、启动日志、用户登录日志，程序运行日志等等，你都可以在这个目录下找到相应的日志；

`/var/crash` : 该目录主要用于存储因系统异常崩溃产生的转储文件，最常见的有 因系统异常断电发生panic、系统运行时内存不够产生的crash等现象所产生的日志都会被记录在该目录下；

`/var/spool` : 这个目录主要包含用户级的邮件、定时任务、等产生的相关信息



#### 最后

对于一个操作系统来说，其上的目录错综复杂，人的精力终究有限，难以对每个目录和文件进行深入研究。因此，我认为更应该将精力放在全局上，大致清楚目录是做什么的也是可以的。例如，当你需要修改网卡配置的时候，你应该知道自己该去哪个目录下去修改。

当然，倘若你有足够的精力，那么你完全可以深入去挖掘Linux的目录结构。本文篇幅过长，仅限于此。

