### 谈谈ls命令的执行效率

#### 起因

相信对于 Linux 操作系统上的 `ls` 命令，很多人都是非常熟悉的。很多人对 `ls` 命令的直观感觉就是快，非常快。无论你在操作系统的哪个目录下，执行 `ls` 都能在瞬间看到当前目录下的文件。

但前几天，公司生产环境上出现了一次故障，最终经过定位，罪魁祸首竟然与 `ls` 命令有关。那么，究竟是什么让 `ls` 出现了故障呢？

### 分析

在分析为什么 `ls` 命令会引起生产故障前，先简要描述下问题是怎么出现的。

公司内部的生产环境上搭建着数据库，而隔三差五的五也会在环境上去看看数据库是否运行正常，操作系统中的系统日志是不是有什么报错。而就在前段时间，当我去看系统日志的时候，发现系统日志中出现了很多 `Call Trace` 的打印，熟悉 `Call Trace` 的伙伴可能就知道，这个时候的系统是出现故障了。

```bash
Nov  2 19:37:54 *** kernel: Call Trace:
Nov  2 19:37:54 *** kernel: [<ffffffffbcb7ffa5>] dump_stack+0x19/0x1b
Nov  2 19:37:54 *** kernel: [<ffffffffbcb7a8c3>] dump_header+0x90/0x229
Nov  2 19:37:54 *** kernel: [<ffffffffbc506d32>] ? ktime_get_ts64+0x52/0xf0
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c251e>] oom_kill_process+0x25e/0x3f0
Nov  2 19:37:54 *** kernel: [<ffffffffbc533a51>] ? cpuset_mems_allowed_intersects+0x21/0x30
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c1f7d>] ? oom_unkillable_task+0xcd/0x120
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c2026>] ? find_lock_task_mm+0x56/0xc0
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c2d76>] out_of_memory+0x4b6/0x4f0
Nov  2 19:37:54 *** kernel: [<ffffffffbcb7b3e0>] __alloc_pages_slowpath+0x5db/0x729
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c91f6>] __alloc_pages_nodemask+0x436/0x450
Nov  2 19:37:54 *** kernel: [<ffffffffbc618ea8>] alloc_pages_current+0x98/0x110
Nov  2 19:37:54 *** kernel: [<ffffffffbc5be427>] __page_cache_alloc+0x97/0xb0
Nov  2 19:37:54 *** kernel: [<ffffffffbc5c0fe0>] filemap_fault+0x270/0x420
Nov  2 19:37:54 *** kernel: [<ffffffffc097798e>] __xfs_filemap_fault+0x7e/0x1d0 [xfs]
Nov  2 19:37:54 *** kernel: [<ffffffffc0977b8c>] xfs_filemap_fault+0x2c/0x30 [xfs]
Nov  2 19:37:54 *** kernel: [<ffffffffbc5edf6a>] __do_fault.isra.61+0x8a/0x100
Nov  2 19:37:54 *** kernel: [<ffffffffbc5ee51c>] do_read_fault.isra.63+0x4c/0x1b0
Nov  2 19:37:54 *** kernel: [<ffffffffbc5f5d80>] handle_mm_fault+0xa20/0xfb0
Nov  2 19:37:54 *** kernel: [<ffffffffbc64c713>] ? do_sync_write+0x93/0xe0
Nov  2 19:37:54 *** kernel: [<ffffffffbcb8d653>] __do_page_fault+0x213/0x500
Nov  2 19:37:54 *** kernel: [<ffffffffbcb8d975>] do_page_fault+0x35/0x90
Nov  2 19:37:54 *** kernel: [<ffffffffbcb89778>] page_fault+0x28/0x30
```

比较奇怪的事情是，这个 `Call Trace` 的打印每隔一段时间就会出现，但系统并没有发生 panic 或者 重启的现象。当我再深入去看系统日志的时候，发现每次 `Call Trace` 打印出现的时候都伴随有 `oom(Out-of memory)` ，而我也通过 `free -lh` 查看了操作系统的内存使用情况，剩余内存还剩下20余G，完全够用。

```bash
Nov  2 19:38:48 *** kernel: Out of memory: Kill process 222648 (qflame_exporter) score 50 or sacrifice child
Nov  2 19:38:48 *** kernel: Killed process 39704 (su), UID 0, total-vm:193980kB, anon-rss:576kB, file-rss:1244kB, shmem-rss:0kB
Nov  2 19:38:48 *** kernel: orarootagent.bi invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0
Nov  2 19:38:48 *** kernel: orarootagent.bi cpuset=/ mems_allowed=0
Nov  2 19:38:48 *** kernel: CPU: 30 PID: 94082 Comm: orarootagent.bi Kdump: loaded Tainted: P           OE  ------------   3.10.0-1127.19.1.el7.x86_64 #1
```

看到这里我已经不明白问题原因到底是什么了，于是，我去找了公司的大佬帮忙看了这个问题，大佬看了相关的日志，看到是因为 exporter 程序异常导致操作系统内存不够用的，通过查看 exporter 程序的日志也发现程序在操作系统上进行了大量的采集，而且采集的程序通通报 timeout 的错误。因为 exporter 程序采集的是 `/dev/` 目录下的磁盘设备，但是系统上总共也只有 200 来个设备(很多设备进行了分区)，不可能会出现采集超时的情况。

于是，我通过终端进入 `/dev/` 目录下，执行 `ls` 查看到底是怎么回事。没想到，在 `/dev/` 目录下执行 `ls` 当时就卡住了。足足过了近10分钟才返回结果，只看看终端屏幕疯狂的刷新，而终端的最下面返回的都是一些我不认识的文件。

细数之下，目录下的文件数量达 700万 多个文件，看来正是因为巨量的文件导致的程序采集超时，而采集磁盘信息中正好是通过 `ls` 来采集的。

#### 补墙

发现问题后，自然需要及时恢复生产环境，毕竟生产第一嘛。

但是删除目录下的文件时，使用 `rm -rf` 删除文件时，提示了 `Argument list too long`，看来想要一次删除大量文件，使用 `rm` 也是不太能实现的，无奈之下只好通过文件名进行匹配，一次删除一部分，最终耗时2个小时，终于删除了多余文件。

删除文件，重启业务相关的服务，总算恢复了环境。

#### 反思

可能大家为什么会奇怪为什么我一直没有提到为什么会出现这么多巨量文件，事实上，这不是今天的重点。生产上因为程序出现的bug，都是可以被修复的。而我们需要做的，则是反思通过故障我们能吸取哪些经验教训。

通过这次事件，让我对 `ls` 命令产生了兴趣，于是，我写了这边博客来记录我对 `ls` 的效率的一些探索。

探索过程依旧模拟生产上的故障，通过在本地挂载一块存储介质，将其格式化为 `ext4` 文件系统的格式，然后再挂载的目录中分别创建大量的文件，测试看看 `ls` 执行所需要花费的时间。

首先，找一块未使用的磁盘，格式化为 `ext4` 格式

```bash
#mkfs -t ext4 /dev/dm-162
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
48824320 inodes, 195280384 blocks
9764019 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2344615936
5960 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
    102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

将设备挂载到文件系统上

```bash
#mkdir /data

#mount /dev/dm-162 /data

#df -lh /data
Filesystem                          Size  Used Avail Use% Mounted on
/dev/mapper/*******  734G   73M  696G   1% /data
```

创建一个大文件，

```bash
#dd if=/dev/zero of=/root/data bs=1M count=10 oflag=direct
10+0 records in
10+0 records out
10485760 bytes (10 MB) copied, 0.00536984 s, 2.0 GB/s

#ls -lh /root/data
-rw-r--r-- 1 root root 10M Nov 13 14:37 /root/data  
```

将其切分为10万个小文件，统计使用 `ls` 查看文件的时间

> 若要统计文件数量，可将目录下的文件文件名写入到文件中，再进行行数统计。
> 
> #find . > /tmp/temp && wc -l /tmp/temp
> 100001 /tmp/temp

```bash
#cd /data/

#split -n 100000 /root/data  

#time ls
real    0m0.920s
user    0m0.609s
sys        0m0.289s
```

​        这里使用 `time {command}` 来统计命令执行的时间。其中

​         `real`: 表示程序实际执行所花费的时间 

​          `user`: 表示程序执行时在用户态所消耗的时间

​          `sys` : 表示程序执行时在内核态所花费的时间

将刚刚切分的文件删除(为了避免系统缓存)，将文件切分为50万个小文件

```bash
#split -n 500000 /root/data

#time ls
real    0m8.134s
user    0m3.943s
sys        0m2.983s
```

> 当文件数量达到 50万的时候，使用 `rm -rf *` 就出现了 `Argument toolong  ` 的报错
> 
> #rm -rf *
> -bash: /bin/rm: Argument list too long

删除文件后再次将文件切分为 100万个小文件

```bash
#split -n 1000000 /root/data

#time ls
real    0m15.427s
user    0m8.207s
sys        0m4.737s
```

删除文件后再次将文件切分为 200万个小文件

```bash
#split -n 2000000 /root/data

#time ls
real    0m34.412s
user    0m17.231s
sys        0m14.149s
```

再将其切分为 500万文件

```bash
#split -n 5000000 /root/data

#time ls
real    2m3.960s
user    0m47.844s
sys        1m5.230s
```

最后，将文件切分为 1000万个小文件

```bash
#split -n 10000000 /root/data

#time ls
real    6m31.626s
user    1m40.799s
sys        4m16.432s
```

通过不同数量的文件，可以看到 `ls` 命令会随着文件数量的增加而返回时间也随之增加。倘若不考虑操作系统的 inode 分配，那么随着文件数量的增长 ls 的执行速度可能会呈现指数增长的趋势。

不过，你大可放心，操作系统上的 inode 是有限制的，因此，文件的数量也是有限的。你只能尽可能的保证日常程序产生的日志、数据文件不能超过百万级别，否则，可能对系统运行有不小的影响。

为了避免这种情况，应该及时删除程序产生的一些过期文件，保证系统的健康运行。