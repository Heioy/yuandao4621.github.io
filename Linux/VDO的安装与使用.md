### 介绍

随着服务器的普及和数据量的爆炸式增长，企业对存储介质的容量和性能需求也变得越来越高。传统模式下的存储介质的利用效率，仅能在磁盘中写入同等容量的数据。直到 RedHat 7.5 版本中，RedHat 官方在操作系统中增加了一项新的软件(测试版)，用来提高存储介质的使用效率，这款软件就是 `VDO(Virtual Data Optimizer)`。

`VDO` 是一个基于虚拟块设备压缩的技术，通过数据的去重和压缩而实现的块设备的高效使用。通俗来讲，以前一块 1TB 的硬盘，通过使用 VDO 技术后，可以当作 1.5TB、2TB 甚至 3TB 来使用。

### 原理

`VDO` 能够使得存储介质的高效利用来自于两项技术，分别是 `去重` 和 `压缩`。通过 `VDO` 技术在物理存储介质上虚拟化出来一块逻辑设备，当上层应用产生数据时，数据在写入物理设备(磁盘)前，会先进入 `VDO` 创建的逻辑设备中，通过数据去重和压缩之后再将数据写入到物理设备中。

那么，数据是如何去重并压缩的呢？

![](../../../WorkSpace/notes/img/vdo-1.png) 

- 去重(**Deduplication**)：通过消除数据块中的多个重复块来减少存储资源的消耗。
  
  原理：VDO 会在每个数据块写入存储前判断该数据块是否已经在存储区存在原始块，若是存在，VDO 不会再次将数据写入存储设备中，而是提供原始块在存储设备中的地址记录下来提供给多个重复块使用。
  
  数据块中出现重复块后，多个逻辑块的地址会被映射到同一个物理块地址，这些相同的数据块被称为 `共享块`。当共享块被覆盖后，将分配新的物理块来存储新的块数据，确保映射到共享存储块的其他逻辑块地址不被修改。
  
  具体实现为：数据写入 VDO 逻辑设备后，首先将数据块中为 `零` 的块设备去除，然后将除 `0` 以外的块进行去重，即相同的数据保留只保留一份，其他的数据会在去重的阶段被清理掉。

- 压缩(**Compression**)：一种数据缩减技术，保证相同的数据尽可能占据少的存储空间，一般适用于日志文件系统或数据库。
  
  具体实现为：将去重后的数据通过 `LZ4` 压缩算法进行压缩。压缩后的数据以固定4KB的数据块存储在磁盘中。

### 相关组件

`VDO` 软件中有3个模块用来支持数据的去重、压缩、和 vdo 设备的管理。分别是 `kvdo`、 `uds`、 `xxx`。

- kvdo: 该模块是加载在 Linux系统设备映射层的内核模块，可提供重复数据删除、压缩和精剪配置的块存储卷
- uds: 一个内核模块，主要与卷上的通用数据删除服务(UDS) 进行通信，并分析数据中的重复数据 
- common line tools：提供带管理化的终端命令

### 安装

芯片要求：目前仅提供 `ADM64` 和 `Intel 64位` 系列架构的支持

内存要求：`vdo` 模块需要370MB的内存空间，并且每1TB的存储介质需要额外的268MB的内存空间。`uds` 需要最小250MB的内存空间。(PS: 要创建的vdo设备的容量越大，对内存空间的要求越高。)

对于 `rhel` 系列的操作系统，可使用 `yum` 进行安装

#### 安装

```bash
# yum install vdo kmod-kvdo
```

#### 创建vdo卷

```bash
# vdo create \
       --name=vdo_name \
       --device=block_device \
       --vdoLogicalSize=logical_size \
       [--vdoSlabSize=slab_size]
```

如果要创建的设备容量大于 `16TB`，创建时需要添加 `--vdoSlabSize=32G` 参数

- `--name`：创建的设备名称
- `--device`：创建vdo时要使用的物理设备
- `--vdoLogicalSize`：要创建的vdo设备的逻辑容量，可以是物理设备容量的 1.5 倍、2倍、或 3倍
- `--vdoSlabSize`：指定 `slab size` 的大小。默认是 2GiB
- `--writePolicy`：写入模式。可选模式 `async`, `sync`, `auto`

#### 查看vdo设备

```bash
# vdo list
```

#### 查看vdo设备当前活动数据

```bash
# vdo status
# vdo status --name=vdo_name
```

#### 开启vdo设备

```bash
# vdo start --name=vdo_name
# vdo start --all
```

#### 停止vdo设备

```bash
# vdo stop --name=vdo_name
# vdo stop --all
```

#### 修改vdo设备写入策略

```bash
# vdo changeWritePolicy --writePolicy=sync|async|auto --name=vdo_name
```

#### 删除vdo设备

```bash
# vdo remove --name=my_vdo
# vdo remove --force --name=my_vdo
```

#### 配置vdo设备开机自启

```bash
# vdo activate --name=my_vdo
# vdo activate --all
```

#### 关闭vdo设备开机自启

```bash
# vdo deactivate --name=my_vdo
# vdo deactivate --all
```

#### 禁用去重功能

```bash
# vdo disableDeduplication --name=my_vdo
```

#### 启用去重功能

```bash
# vdo enableDeduplication --name=my_vdo
```

#### 禁用压缩功能

```bash
# vdo disableCompression --name=my_vdo
```

#### 启用压缩功能

```bash
# vdo enableCompression --name=my_vdo
```

#### 查看vdo设备空间已使用情况

```bash
# vdostats --human-readable
```

#### 扩容vdo设备

```bash
# vdo growLogical --name=my_vdo --vdoLogicalSize=new_logical_size
```

#### 扩容vdo底层物理设备容量

> 创建vdo设备时，可选择保留物理设备的部分容量，当vdo设备空间不够用时，可进行物理设备的扩容以缓解vdo设备的容量告急。不过，当出现vdo设备容量告急时，更应该考虑更换更大容量的物理设备。

```bash
# vdo growPhysical --name=my_vdo
```

参考自: [30.3. Getting Started with VDO Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/vdo-quick-start)
