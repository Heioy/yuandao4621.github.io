最近在数据库中查询日志应用情况，发现了好几种 `redo` 日志的状态，简单介绍下不同状态的含义。

```sql
SQL> select GROUP#,THREAD#,BYTES/1024/1024/1024 GB,STATUS from v$log;
    GROUP#    THREAD#	      GB STATUS
---------- ---------- ---------- ----------------
	 1	    1	      20 ACTIVE
	 2	    1	      20 CURRENT
	 3	    2	      20 INACTIVE
	 4	    2	      20 CURRENT
	11	    1	      20 ACTIVE
	12	    2	      20 INACTIVE
```



#### 什么是redo log

`redo log` 又被称为 `联机重做日志`，日志中记录了 Oracle 数据库的变更历史，包括 `insert`、update、delete、create、drop、alter等操作，主要用来做数据库的故障恢复。

#### redo log 的六种状态

Oracle redo log 总共包含六种状态，分别为 `CURRENT`、`ACTIVE`、`INACTIVE`、`UNUSED`、`CLEARING`、`CLEARING_CURRENT`。

##### `CURRENT`



