#### FIO工具介绍

FIO 工具是一款用于测试硬件存储性能的辅助工具，兼具灵活性、可靠性从而从众多性能测试工具中脱颖而出。磁盘的 `I/O` 是衡量硬件性能的最重要的指标之一，而 FIO 工具通过模拟 `I/O`负载对存储介质进行压力测试，并将存储介质的 `I/O` 数据直观的呈现出来。

根据实际业务的场景，一般将 `I/O` 的表现分为四种场景，`随机读`、`随机写`、`顺序读`、`顺序写`。FIO 工具允许指定具体的应用模式，配合多线程对磁盘进行不同深度的测试。

自 Red Hat Enterprise Linux 4 版本，FIO 工具已经集成在 Fedora/EPEL 仓库中。也可以通过 github 的地址 https://github.com/axboe/fio.git 进行访问。

#### 编译&安装

编译方式：下载解压安装包后执行

https://github.com/axboe/fio/archive/refs/tags/fio-3.29.tar.gz

```bash
$ ./configure
$ make
$ make install
```

#### 工作原理

使用 FIO 工具测试硬盘性能，首先需要编写一个作业(命令或者脚本)来预定义 FIO 将要以什么样的模式来执行任务。作业 中可以包含任意数量的线程或文件，一个作业的典型模式是定义一部分的全局共享参数和一个(或多个)作业的实体对象(即硬盘)。运行 FIO 作业时，FIO 会自上而下解析作业中的配置，并做相应 的配置。

FIO 中的基本参数包含：

- I/O Type：定义 FIO 工具的模式。包含 `随机读`、`随机写`、`顺序读`、`顺序写`四种模式。

- Block Size：使用 FIO 工具时测试的块大小。可选 `4k`、`8k`、`16k`、…、`1M`、...等

- I/O Size：按照 I/O type 向硬盘中写入/读取多少数据

- I/O engine：FIO 工作时使用的引擎。引擎决定了使用什么样的 I/O，同步模式、异步模式 

- I/O depth：I/O 引擎如果使用异步模式，要保持多少的队列深度

- Target file/device：将 I/O 负载分配到多少个硬盘上

- Threads, processes, job synchronization：使用多少的进程或线程数量

  

#### 常用参数

| 常用参数                   | 参数说明                                                     | 参数取值(eg:)                 |
| -------------------------- | ------------------------------------------------------------ | ----------------------------- |
| name                       | FIO 运行任务的名称 （可选）                                  | name=fio-test                 |
| description                | FIO 运行时的描述 （可选）                                    | description="this is a test"  |
| loops                      | FIO 是否要循环执行该任务 （可选）                            | loops=5                       |
| numjobs                    | 每个FIO 任务要开启多少数量的线程                             | numjobs=8                     |
| runtime                    | FIO 任务要执行的时间，单位为s                                | runtime=300                   |
| time_based                 | 设置后，即便FIO写完了整个磁盘，也不会退出任务。当任务的时间满足runtime设置的时间后，才会退出任务 | time_based                    |
| startdelay                 | 延迟作业的启动时间。如果指定区间类型的数值，将从区间中选择一个随机值用来启动任务。单位为s | startdelay=10                 |
| ramp_time                  | FIO 在执行任务时预热的时间。可以使性能测试的结果更加精确可靠。单位为s | ramp_time=60                  |
| filename                   | 测试文件的名称，通常为硬盘的盘符名称                         | filename=/dev/sdd             |
| pre_read                   | 在FIO下发 I/O 前，文件会被加载到内存中。当 pre_read=true 时会清除掉磁盘中的元数据，默认为false | pre_read=false                |
| max_open_zones             | FIO 执行随机写入任务时，允许写入/读取的磁盘的区域的最大数量。 | max_open_zones=1000000        |
| direct                     | 是否使用非缓冲I/O。默认为false                               | direct=true                   |
| readwrite                  | I/O 模式。read、write、randwrite、randread、randrw。混合读写时默认读写比例为50%:50% | readwrite=read                |
| percentage_random          | 随机写入时包含多少的随机数据                                 | percentage_random=30%         |
| blocksize                  | 读取/写入时块的大小                                          | bs=1M                         |
| zero_buffers               | 初始化所有零的缓冲区。默认用随机数填充缓冲区                 | zero_buffers                  |
| rwmixread                  | 混合模式中读取所占的百分比                                   | rwmixread=30                  |
| rwmixwrite                 | 混合模式中写入所占的百分比                                   | rwmixwrite=70                 |
| refill_buffers             | FIO 在每次提交时重新填充 I/O 缓冲区。使用 zero_buffers 参数后，该参数会失效 | refill_buffers                |
| buffer_compress_percentage | 压缩 I/O 缓冲区的百分比，配合 refill_buffers 可以降低硬盘中相邻块中数据的一致性 | buffer_compress_percentage=70 |
| size                       | FIO 执行任务时要读取或写入的数据总和。size为百分比时将按照硬盘容量 * percentage | size=300G or size=50%         |
| ioengine                   | FIO 任务工具时的引擎，sync, libaio, rdma...                  | ioengine=libaio               |
| iodepth                    | 任务执行时的 I/O 队列深度的单元数。任务的线程数 = numjobs * iodepth | iodepth=16                    |

#### 定义 FIO 任务

要执行一个 FIO 任务前，需要先定义一个 FIO 任务作业。有两种方式可以选择，命令行形式和脚本形式。实际使用推荐 脚本形式，更加直观和清晰，也容易保存。

FIO 任务脚本需要以 `.ini` 作为文件的后缀名

```ba
$cat fio.ini
[global]
ioengine=libaio
iodepth=16
direct=1
#runtime=300
size=98%
time_based
refill_buffers
group_reporting=1
numjobs=8
rw=write
bs=1M

[file1]
filename=/dev/sdc
[file2]
filename=/dev/sdd
[file3]
filename=/dev/sde
```

如上为 `(rw=write)顺序写` 模式的任务，I/O 队列深度为 `(iodepth=16)`，每个任务执行时的并发数量为 `(numjobs=8)`，写入时每次写入的块为 `(bs=1M)` ，写入数据量达到磁盘容量的 `(size=98%)` 后退出任务。

执行 FIO 任务脚本

在终端命令行任务脚本的同级目录下，执行 `fio fio.ini`

```bash
$fio fio.ini
file1: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file2: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file3: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file4: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file5: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file6: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file7: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file8: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
file9: (g=0): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=32
...
fio-3.7
Starting 72 processes

file1: (groupid=0, jobs=72): err= 0: pid=165889: Tue Jun  8 17:08:28 2021
  write: IOPS=315, BW=316MiB/s (331MB/s)(93.3GiB/302533msec)
    slat (usec): min=25, max=5930.2k, avg=223639.76, stdev=862552.86
    clat (msec): min=11, max=31055, avg=7017.73, stdev=2199.38
     lat (msec): min=11, max=31055, avg=7241.37, stdev=2264.17
    clat percentiles (msec):
     |  1.00th=[ 2635],  5.00th=[ 3574], 10.00th=[ 4329], 20.00th=[ 5604],
     | 30.00th=[ 6275], 40.00th=[ 6678], 50.00th=[ 7013], 60.00th=[ 7349],
     | 70.00th=[ 7684], 80.00th=[ 8087], 90.00th=[ 8926], 95.00th=[10671],
     | 99.00th=[14295], 99.50th=[16442], 99.90th=[17113], 99.95th=[17113],
     | 99.99th=[17113]
   bw (  KiB/s): min= 1896, max=36864, per=8.94%, avg=28915.47, stdev=8391.88, samples=6610
   iops        : min=    1, max=   36, avg=28.18, stdev= 8.20, samples=6610
  lat (msec)   : 20=0.01%, 50=0.03%, 100=0.02%, 250=0.06%, 500=0.06%
  lat (msec)   : 750=0.04%, 1000=0.02%
  cpu          : usr=0.10%, sys=0.02%, ctx=16780, majf=0, minf=2057
  IO depths    : 1=0.1%, 2=0.2%, 4=0.3%, 8=0.6%, 16=1.2%, 32=97.7%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,95570,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
  WRITE: bw=316MiB/s (331MB/s), 316MiB/s-316MiB/s (331MB/s-331MB/s), io=93.3GiB (100GB), run=302533-302533msec
```

