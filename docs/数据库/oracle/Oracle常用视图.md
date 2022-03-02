日常使用Oracle的过程中，总是免不了要对各种表进行查询。在Oracle数据库中，通常通过查询视图来代替物理表的查询。视图是基于数据表创建的一种逻辑上的虚拟表结构，通过将一些常用的表结构和查询操作通过联合而形成的，在使用过程中使用视图代替数据表的查询可以使一些常见的查询操作事倍功半。

这里简单描述下**视图**和**数据表**之间的关系
* 数据表: 是物理上的结构，存储在底层的硬盘上，占用实际硬盘的物理空间；
* 视图: 是逻辑上的结构，是表与表之间的一种或多种逻辑映射，存储在数据字典里，不占用实际的物理硬盘空间；

进入数据库,通过 **all_views** 视图查看数据库的所有视图
```sql
> select OWNER,VIEW_NAME from all_views;   /* 视图中的字段会很多,使用`desc {view_name}`查看视图的字段名选择可用的字段名 */
```

#### 数据库常用视图
* `USER_TABLES`: 当前用户下的所有数据表
```sql
/* 查询表名称及所属表空间 */
> select TABLE_NAME,TABLESPACE_NAME from USER_TABLES;
```

* `ALL_TABLES`: 所有用户的数据表
```sql
/* 查询表所属用户、表名称及所属表空间 */
> select OWNER,TABLE_NAME,TABLESPACE_NAME from ALL_TABLES;
```

* `DBA_TABLES`: DBA用户可以访问的数据表
```sql
/* 查询表所属用户、表名称及所属表空间 */
> select OWNER,TABLE_NAME,TABLESPACE_NAME from DBA_TABLES;
```

* `ALL_INDEXES`: 存放数据表索引的视图
```sql
/* 查询索引对应名称、类型、及对应的表空间*/
> select OWNER,INDEX_NAME,INDEX_TYPE,TABLE_NAME,TABLESPACE_NAME from ALL_INDEXES;
```

* `USER_EXTENTS`: 存放用户所拥有对象的段的视图
```sql
/* 查询用户拥有的段中的段名称、分区名、段类型、表空间及所占用的字节数 */
> select SEGMENT_NAME,PARTITION_NAME,SEGMENT_TYPE,TABLESPACE_NAME,BYTES from USER_EXTENTS;
```

* `DBA_EXTENTS`: 数据库中所有段包含的区的视图
```sql
/* 查询所有用户拥有的段中的段名称、分区名、段类型、表空间及所占用的字节数 */
> select OWNER,SEGMENT_NAME,PARTITION_NAME,SEGMENT_TYPE,TABLESPACE_NAME,BYTES from DBA_EXTENTS;
```

* `DBA_DATA_FILES`: 数据库中所有数据文件相关的视图
```sql
/* 查询数据文件大小、名称、对应表空间及是否自动增长 */
> select FILE_NAME,TABLESPACE_NAME,BYTES/1024/1024/1024,STATUS,AUTOEXTENSIBLE from DBA_DATA_FILES;

FILE_NAME						     TABLESPACE_NAME	  BYTES/1024/1024/1024 STATUS	 AUT
------------------------------------------------------------ -------------------- -------------------- --------- ---
+DATADG/RAC/system01.dbf				     SYSTEM			    1.03515625 AVAILABLE YES
+DATADG/RAC/sysaux01.dbf				     SYSAUX			    21.1035156 AVAILABLE YES
```

* `V_$LOG`: Redo日志相关视图
```sql
/* 查询REDO日志相关组及日志大小 */
> select GROUP#,THREAD#,BYTES/1024/1024/1024,MEMBERS,ARCHIVED,STATUS from V_$LOG;

    GROUP#    THREAD# BYTES/1024/1024/1024    MEMBERS ARC STATUS
---------- ---------- -------------------- ---------- --- ----------------
	 9	    1			10	    1 NO  CURRENT
	23	    2			10	    1 NO  CURRENT
```

* `V_$LOGFILE`: REDO日志文件视图
```sql
/* 查看redo日志组成员 */
> select GROUP#,MEMBER,TYPE from V_$LOGFILE;

    GROUP# MEMBER					      									TYPE
---------- -------------------------------------------------- -------
	 6 	  +LOGDG/RAC/ONLINELOG/group_6.261.1095247683	      ONLINE
	 7 	  +LOGDG/RAC/ONLINELOG/group_7.262.1095247717	      ONLINE
```

* `V_$CONTROLFILE`: 控制文件视图
```sql
/* 查询数据库所使用的控制文件 */
> select STATUS,NAME,BLOCK_SIZE from V_$CONTROLFILE;

STATUS	NAME			       BLOCK_SIZE
------- ------------------------------ ----------
	+DATADG/RAC/control01.ctl	    16384
	+DATADG/RAC/control02.ctl	    16384
```

* `GV_$CONTROLFILE`: 集群级别的控制文件视图
```sql
> select INST_ID,STATUS,NAME,BLOCK_SIZE from GV_$CONTROLFILE;

   INST_ID STATUS  NAME 					      BLOCK_SIZE
---------- ------- -------------------------------------------------- ----------
	 1	   +NVMEDG/RAC/control01.ctl				   16384
	 2	   +NVMEDG/RAC/control02.ctl				   16384
```

* `DBA_DIRECTORIES`: 存放数据库所有用户可以访问的数据字典
```sql
> select OWNER,DIRECTORY_NAME,DIRECTORY_PATH from DBA_DIRECTORIES;

OWNER	 DIRECTORY_NAME			   DIRECTORY_PATH
------- -------------------------- ------------------------ 
SYS		 DBMS_OPTIM_LOGDIR		   /****/12.2.0/cfgtoollogs
```

* `DBA_SEGMENTS`: 数据库所有的段视图
```sql
/* 查询表空间索引占用的空间情况 */
> select OWNER,SEGMENT_NAME,SEGMENT_TYPE,TABLESPACE_NAME,sum(BYTES/1024/1024/1024) from dba_segments where OWNER='SOE' group by OWNER,SEGMENT_NAME,SEGMENT_TYPE,TABLESPACE_NAME;

OWNER	 SEGMENT_NAME	     SEGMENT_TYPE	 TABLESPACE_NAME	  SUM(BYTES/1024/1024/1024)
------- -------------------- -------------- -------------------- -------------------------
SOE		PRODUCT_DESCRIPTIONS TABLE 	          SOE				 .000305176

```

* `DBA_FREE_SPACE`: 数据库剩余表空间统计视图 
```sql
/* 查询数据库已使用表空间并根据表空间名称分组 */
> select TABLESPACE_NAME,sum(BYTES/1024/1024/1024) from DBA_FREE_SPACE group by TABLESPACE_NAME;

TABLESPACE_NAME      SUM(BYTES/1024/1024/1024)
-------------------- -------------------------
SYSTEM				    4.99133301
UNDOTBS2			    136.481812
```

* `V_$SQL_PLAN`: 实例级别的sql执行计划
```sql
/* 查询sql执行语句的ID */
> select SQL_ID,PLAN_HASH_VALUE,TIMESTAMP,OBJECT# from V_$SQL_PLAN;

SQL_ID	      PLAN_HASH_VALUE TIMESTAMP 	     OBJECT#
------------- --------------- ------------------- ----------
94qn6y14kw01g	   1388734953 2022-03-01 16:25:37
94qn6y14kw01g	   1388734953 2022-03-01 16:25:37
```


#### ASM常用视图
* `V$ASM_DISK`: ASM磁盘组中关于磁盘的详细信息
```SQL
/* 查询磁盘组中磁盘名称及状态大小 */
> select GROUP_NUMBER,DISK_NUMBER,MOUNT_STATUS,HEADER_STATUS,STATE,NAME,TOTAL_MB from V$ASM_DISK;

GROUP_NUMBER DISK_NUMBER MOUNT_STATUS	HEADER_STATUS		 STATE		  NAME				   TOTAL_MB
------------ ----------- -------------- ------------------------ ---------------- ------------------------------ ----------
	   1	       2 CACHED 	MEMBER			 NORMAL 	  LOGDG_0002			     204800
	   3	       1 CACHED 	MEMBER			 NORMAL 	  OCRVOTE_0001			       6144
```

* `V$ASM_DISKGROUP`: ASM磁盘组视图
```sql
/* 查询asm中所有磁盘组的信息 */
> select GROUP_NUMBER,NAME,ALLOCATION_UNIT_SIZE,STATE,TYPE,TOTAL_MB/1024 from V$ASM_DISKGROUP;

GROUP_NUMBER NAME			    ALLOCATION_UNIT_SIZE STATE			TYPE		     TOTAL_MB/1024
------------ ------------------------------ -------------------- ---------------------- -------------------- -------------
	   1 LOGDG					 1048576 MOUNTED		NORMAL			       600
	   3 OCRVOTE					 4194304 MOUNTED		NORMAL				18
```

* `V$ASM_OPERATION`: ASM磁盘组磁盘重平衡视图
```sql
/* 查询磁盘组重平衡进度 */
> select GROUP_NUMBER,OPERATION,STATE,POWER,ACTUAL,EST_WORK,EST_RATE,EST_MINUTES from V$ASM_OPERATION;

GROUP_NUMBER OPERATION	STATE	      POWER	ACTUAL	 EST_WORK   EST_RATE EST_MINUTES
------------ ---------- -------- ---------- ---------- ---------- ---------- -----------
	   2 REBAL	WAIT		 11	    11		0	   0	       0
	   2 REBAL	WAIT		 11	    11		0	   0	       0
	   2 REBAL	RUN		 11	    11	  2025371      72715	      12
	   2 REBAL	DONE		 11	    11		0	   0	       0
```

* `V$ASM_FILE`: 存放ASM磁盘组中的文件信息
```sql
/* 查询ASM数据文件大小及类型*/
> select GROUP_NUMBER,FILE_NUMBER,SPACE/1024/1024/1024,TYPE,REDUNDANCY from V$ASM_FILE;

GROUP_NUMBER FILE_NUMBER SPACE/1024/1024/1024 TYPE		   REDUNDANCY
------------ ----------- -------------------- -------------------- ------------
	   1	     256	   .395507813 ONLINELOG 	   MIRROR
	   1	     257	   .395507813 ONLINELOG 	   MIRROR
```

* `V$ASM_CLIENT`: ASM实例及数据库实例信息相关的视图
```sql
/* 查询ASM实例及数据库实例名称及版本信息 */
> select GROUP_NUMBER,INSTANCE_NAME,DB_NAME,CLUSTER_NAME,STATUS,SOFTWARE_VERSION from V$ASM_CLIENT;

GROUP_NUMBER INSTANCE_NAME	  DB_NAME	   CLUSTER_NAME 	STATUS			 SOFTWARE_VERSION
------------ -------------------- ---------------- -------------------- ------------------------ --------------------
	   1 rac1		  rac		   oracle		CONNECTED		 19.0.0.0.0
	   2 rac1		  rac		   oracle		CONNECTED		 19.0.0.0.0
	   3 +ASM1		  +ASM		   oracle		CONNECTED		 19.0.0.0.0
	   3 node1		  _OCR		   oracle		CONNECTED		 -
```

* `V$ASM_ALIAS`: ASM磁盘组别名
```sql
> select * from V$ASM_ALIAS;

NAME			       GROUP_NUMBER FILE_NUMBER FILE_INCARNATION ALIAS_INDEX ALIAS_INCARNATION PARENT_INDEX REFERENCE_INDEX AL SY     CON_ID
------------------------------ ------------ ----------- ---------------- ----------- ----------------- ------------ --------------- -- -- ----------
RAC					  1  4294967295       4294967295	   0		     1	   16777216	   16777269 Y  N	   0
redo01.log				  1	    256       1095246583	  53		     1	   16777269	   33554431 N  N	   0
```
