测试环境中每次重新安装数据库后需要重新造数据，而造数据的时间一般会很久。通过swingbench造 100G数据一般需要5-6个小时的时间。为了提高在环境准备上的测试效率，可以对swingbench的数据进行复用。

Oracle 数据库提供了 `expdp` 数据导出的方法，可以将表空间中的数据通过 `expdp` 导出，后续需要使用的时候通过 `impdp` 重新导入到新的数据表空间中。

#### 整体思路

1. 首次需要在数据库中造足够量的数据

2. 通过 `expdp` 将数据表空间中的数据文件导出

3. 在新的环境中创建表空间用户和表空间

4. 将 `expdp` 导出的数据导入到新的表空间中

#### 操作步骤

1. 查询数据表空间对应的用户
   
   /* 查找数据库中SOE 和 TPCC用户 */
   
   ```sql
   16:13:02 >select userna from dba_users where username in ('TPCC', 'SOE');
   USERNAME
   --------------------
   TPCC
   SOE
   ```

2. 查找数据库中对应用户的数据表空间
   
   ```sql
   16:16:47 >select name from v$tablespace where name in ('SOE', 'TPCCTAB');
   NAME
   ------------------------------
   TPCCTAB
   SOE
   ```

3. 查询数据表空间数据文件大小
   
   ```sql
   16:18:41 >select sum(bytes/1024/1024/1024) from dba_segments where owner='TPCC';
   SUM(YTES/1024/1024/1024)
   -------------------------
             124.694641
   
   16:19:26 >select sum(bytes/1024/1024/1024) from dba_segments where owner='SOE';
   SUM(BYTES/1024/1024/1024)
   -------------------------
             45.8637695
   ```

4. 数据库中创建逻辑目录
   
   /* 将表空间中的数据导出到本地文件系统时, 注意预留足够容量的空间；若预留空间不足, 会导致导出数据文件失败 */
   
   ```bash
   # 检查本地存储目录是否存在 !!!注意, 目录属组为 oracle:oinstall  若目录属组不正确,执行chown -R oracle:oinstall {dir} 进行设置
   #ls -lh /opt/oracle/
   total 0
   drwxr-xr-x  2 oracle oinstall  38 Jan 13 12:53 dump
   ```
   
   ```sql
   16:40:20 >create directory dump as '/opt/oracle/dump/';
   Directory created.
   
   16:41:05 >select OWNER,DIRECTORY_NAME,DIRECTORY_PATH from dba_directories;
   
   OWNER         DIRECTORY_NAME             DIRECTORY_PATH
   ----------- ---------------------- ------------------------
   SYS             DUMP                     /opt/oracle/dump/
   ```

5. 导出表空间中的数据文件 
   
   > `expdp` 导出数据文件常见用法
   > 
   > COMPRESSION：导出时进行压缩。可选 [ALL, DATA_ONLY, [METADATA_ONLY] and NONE] 。数据导出可以节省本地存储的空间, 但数据导出的时间消耗会变大
   > 
   > DIRECTORY：导出时指定的目录。数据库中的数据文件导出时, 需要预先将本地的存储目录在数据库中创建一个逻辑目录, 逻辑目录与本地存储目录为对应关系
   > 
   > DUMPFILE：导出的数据文件名称
   > 
   > LOGFILE：导出数据时产生的日志信息
   > 
   > PARALLEL：导出数据时开启多少任务并行度
   > 
   > REUSE_DUMPFILES：导出数据文件时, 若本地存储目录中已经包含相同名称的数据文件,会进行覆盖
   > 
   > TABLESPACES：数据库中数据表空间名称
   > 
   > SCHEMAS：需要导出的数据表空间
   
   ```bash
   $expdp "'"/ as sysdba"'" directory=dump schemas=tpcc dumpfile=tpcc.dmp compression=all REUSE_DUMPFILES=yes logfile=tpcc.log
   ```

6. 检查要导入数据文件的环境中是否存在 `SOE` 或 `TPCC` 用户
   
   ```sql
   16:54:11 sys@ demo1>select username from dba_users where username in ('SOE', 'TPCC');
   no rows selected
   -- 没有对应的用户,需要创建对应的用户
   17:32:56 >create user soe identified by oracle;
   User created.
   
   17:33:28 >grant dba to soe;
   Grant succeeded.
   
   
   -- 创建数据表空间 /* 数据表空间创建时容量要大于导出数据的容量 */
   17:38:18 >create tablespace soe datafile '+NVMEDG' size 30G autoextend on;
   Tablespace created.
   ```

7.  导入数据文件
   
   将导出的数据文件拷贝到本节点上, 例如: /tmp/dump/
   
   ```sql
   17:46:21 >create directory dump as '/tmp/dump/';
   Directory created.
   ```

```bash
$impdp "'"/ as sysdba"'" directory=dump schemas=tpcc dumpfile=tpcc.dmp  logfile=tpcc.log
 
```




