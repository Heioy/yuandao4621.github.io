对于数据库产品而言，日志的记录是非常重要的。在我最开始接触 Oracle 数据库时，听到的最多的则是关于"Oracle数据库`日志先行`原则"。不难理解，Oracle数据库总是在进行操作之前会将操作的对象、时间、具体的操作记录到日志中。应用这种规则的日志在 Oracle 数据库中则称为 `redo log(重做日志)`。

Oracle 的 Online Redo Log是为了确保数据库中已经提交的事务不会丢失而建立的一个机制。在数据库正常工作的过程中，redo log 会记录关于 Oracle 数据库的变更历史，包含所有的**DML(数据操作语言)**操作(数据库的增删改操作)。正是基于日志记录的机制，当数据库发生故障时可以从redo log日志进行故障恢复，保证数据的完整。

Oracle 数据库记录的 redo log 包含六种状态，分别为  `CURRENT`、`ACTIVE`、`INACTIVE`、`UNUSED`、`CLEARING`、`CLEARING_CURRENT`。

#### CURRENT
`current` 状态的redo日志为当前活跃的日志，处于该状态下的日志为数据库当前正在写入的日志组。活跃中的日志组无法进行删除。删除前需要将日志组切换到 `INACTIVE` 状态。  
通过 `sqlplus / as sysdba` 进入数据库,查看当前数据库的redo日志组状态
```sql
> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 CURRENT
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 CURRENT
	 4	    2  104857600	  1 UNUSED
	 5	    1  104857600	  1 UNUSED
	 6	    1  104857600	  1 UNUSED
	 7	    1  104857600	  1 UNUSED
	 8	    2  104857600	  1 UNUSED
	 9	    2  104857600	  1 UNUSED
	10	    2  104857600	  1 UNUSED
```
当前数据库的日志组中存在两个 `CURRENT` 的活跃日志组，分别为 Group1 和 Group3，对应数据库的两个实例  
尝试在数据库中删除 `CURRENT` 状态的日志组时，会抛出 [ORA-01623](https://dbaclass.com/article/ora-01623-log-3-is-current-log-for-instance-cannot-drop/) 的错误  
```sql
> alter database drop logfile group 1;
alter database drop logfile group 1
*
ERROR at line 1:
ORA-01623: log 1 is current log for instance ss1 (thread 1) - cannot drop
ORA-00312: online log 1 thread 1: '***/SS/ONLINELOG/group_1.321.1096295273'
```

#### `ACTIVE`
`ACTIVE` 状态的redo log是可以用于数据库实例恢复的日志组，数据库并不会向处于该状态的日志组中写入日志。
该状态下的日志组中的事务尚未完全应用(Oracle需要将redo中记录的事务执行过后，并将事务执行中的数据从DB buffer cache中写入到数据文件中才算事务结束)，因此日志文件无法被覆盖。此种类型的日志文件无法被删除。  
尝试删除 `ACTIVE` 状态的日志文件时，会抛出 [ORA-01624](https://dbaclass.com/article/ora-01624-log-1-needed-for-crash-recovery-of-instance/) 的错误
```sql
> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 ACTIVE
	 2	    1  104857600	  1 ACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 CURRENT
	 6	    1  104857600	  1 INACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 ACTIVE
	 9	    2  104857600	  1 CURRENT
	10	    2  104857600	  1 UNUSED

10 rows selected.

> alter database drop logfile group 2;
alter database drop logfile group 2
*
ERROR at line 1:
ORA-01624: log 2 needed for crash recovery of instance ss1 (thread 1)
ORA-00312: online log 2 thread 1: '***/SS/ONLINELOG/group_2.322.1096301379'
```

#### `INACTIVE`
`INACTIVE` 状态的日志文件为数据库中的事务已经应用完毕的日志组，属于干净的日志文件。此种类型的日志文件可以被数据库删除。`ACTIVE` 状态的日志组在事务应用结束后，状态会变为 `INACTIVE`。
通过 `alter database drop logfile group {group_id};` 删除日志组
```sql
> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 INACTIVE
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 CURRENT
	 6	    1  104857600	  1 INACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 CURRENT
	10	    2  104857600	  1 UNUSED

10 rows selected.

> alter database drop logfile group 1;

Database altered.

> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 CURRENT
	 6	    1  104857600	  1 INACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 CURRENT
	10	    2  104857600	  1 UNUSED
```
#### UNUSED
`UNUSED` 状态的日志文件通常是数据库中新增加的还没有被数据库使用的日志文件。此种状态的日志组可以被删除，若需要使新增加的日志组工作，可以通过 `alter system switch logfile;` 将日志组切换为 `CURRENT` 状态。  
```sql
> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 UNUSED
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 INACTIVE
	 6	    1  104857600	  1 CURRENT
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 INACTIVE
	10	    2  104857600	  1 CURRENT

10 rows selected.

> alter system switch logfile;

System altered.

> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 CURRENT
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 INACTIVE
	 6	    1  104857600	  1 ACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 INACTIVE
	10	    2  104857600	  1 CURRENT

```

#### CLEARING
`CLEARING` 状态的日志组通常是由于数据库中对日志组执行了 `alter database clear logfile;` 出现的，日志文件内容被清除后，日志文件变为新的日志文件，日志组对应状态变为 `UNUSED`。实际生产中该状态并不常见。具体应用中对哪种类型的日志组可以应用 `clear`，请参考[alter database clear logfile group](https://blog.51cto.com/garlics/1112071)  

对状态为 `INACTIVE` 状态的日志组应用 clear 后，日志组状态变为 `UNUSED`  
```sql
> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 CURRENT
	 2	    1  104857600	  1 INACTIVE
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 INACTIVE
	 6	    1  104857600	  1 ACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 INACTIVE
	10	    2  104857600	  1 CURRENT

10 rows selected.

> alter database clear logfile group 2;

Database altered.

> select GROUP#,THREAD#,BYTES,MEMBERS,STATUS from v$log;

    GROUP#    THREAD#	   BYTES    MEMBERS STATUS
---------- ---------- ---------- ---------- --------------------------------
	 1	    1  104857600	  1 CURRENT
	 2	    1  104857600	  1 UNUSED
	 3	    2  104857600	  1 INACTIVE
	 4	    2  104857600	  1 INACTIVE
	 5	    1  104857600	  1 INACTIVE
	 6	    1  104857600	  1 INACTIVE
	 7	    1  104857600	  1 INACTIVE
	 8	    2  104857600	  1 INACTIVE
	 9	    2  104857600	  1 INACTIVE
	10	    2  104857600	  1 CURRENT

```

#### CLEARING_CURRENT
`CURRENT` 状态的日志文件在执行 `alter database clear logfile group {group_id};` 的过程中出现故障(thread closed)，或日志切换过程中出现错误(向新的日志文件头部写入元信息时出现I/O Error导致写入失败)时日志组状态会变为 `CLEARING_CURRENT`。  
目前尚未遇到过此状态的日志组，此处不做任何展示。  


Oracle的redo日志在数据库的启动、恢复、生产过程中有着至关重要的作用，一旦出现redo日志丢失，对于数据库的恢复将会是灾难性的。日常使用中，一定需要对redo进行正确的操作，避免误删日志。定期对日志进行备份也是很值得推崇的。  


参考：[详解Oracle数据库Redo log的六种状态](https://www.51cto.com/article/595972.html)
