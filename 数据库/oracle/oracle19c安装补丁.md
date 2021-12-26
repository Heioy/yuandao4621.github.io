[toc]

以下为 Oracle 数据库 19.3.0 版本安装补丁记录

安装前 `GI` 和 `DB` 版本为: `Version 19.3.0.0.0`

要升级的版本: `Version 19.13.0.0.0`



### 备份目录

ps：因为是在测试环境上进行安装的，测试周期比较快，上面不涉及到重要的数据库文件，因此没有进行备份。若要进行进行备份，可将整个数据库所在的目录进行备份，例如 `/opt` 目录

### 安装 `GI` 和 `DB` 补丁

#### 检查 `opatch` 版本

升级oracle数据库的版本需要通过 `optach` 打补丁的方式升级，因此，升级的数据库版本要使用对应的 `opatch` 版本。

例如，测试环境中要升级的 `DB` 版本为 `19.13`，需要 `opatch` 版本不低于 `12.2.0.1.25` ，如果 `opatch` 版本低于要求版本，需要先升级 `opatch` 的版本

```bash
#grid@com92:/home/grid>opatch version
OPatch Version: 12.2.0.1.27

OPatch succeeded.
```

#### 更新 `opatch`

> 更新 `opatch` 的操作需要所有 `RAC` 集群的节点在 `grid` 和 `oracle` 用户下执行

- 下载解压 `opatch` 包

  ```bash
  #scp root@10.10.100.80:/Backup/QData/QData_Appliance/Regular/onekey_packages/.psu/opatch/12_2_0_1_27/p6880880_190000_Linux-x86-64.zip /tmp
  #unzip /tmp/p6880880_190000_Linux-x86-64.zip
  ```

- 备份原始版本的 `optach`

  Grid 的 `opatch`，使用 root 用户执行

  ```bash
  #mv /opt/grid/products/19.3.0/OPatch /opt/grid/products/19.3.0/OPatch.old
  #cp -r /tmp/OPatch /opt/grid/products/19.3.0/OPatch
  #chown -R grid:oinstall /opt/grid/products/19.3.0/OPatch.old
  #chown -R grid:oinstall /opt/grid/products/19.3.0/OPatch
  ```

  Oracle 的 `opatch`，使用 root 用户执行

  ```bash
  #mv /opt/oracle/products/19.3.0/OPatch  /opt/oracle/products/19.3.0/OPatch.old
  #cp -r /home/oracle/OPatch /opt/oracle/products/19.3.0/OPatch
  #chown -R oracle:oinstall /opt/oracle/products/19.3.0/OPatch
  #chown -R oracle:oinstall /opt/oracle/products/19.3.0/OPatch.old
  ```

- 检查新的 `opatch` 版本

  ```bash
  #su - grid
  Last login: Wed Dec 22 15:40:20 CST 2021
  
  
  ****************************************************************
  ***You login as grid,Please ask somebody to  double check!******
  ****************************************************************
  
  
  grid@com92:/home/grid>opatch version
  OPatch Version: 12.2.0.1.27
  
  OPatch succeeded.
  ```

  

### 补丁冲突检测

#### 下载解压补丁包

```bash
#scp root@10.10.100.80:/Backup/QData/QData_Appliance/Regular/onekey_packages/.psu/p33182768_190000_Linux-x86-64.zip /tmp/
#unzip p33182768_190000_Linux-x86-64.zip
```

#### 补丁冲突检测

> 打补丁前确认所有要安装的补丁没有依赖冲突才可进行安装，冲突检测要在所有 `RAC` 集群的节点的 `grid` 和 `oracle` 用户执行

补丁包解压后后在 `/tmp` 目录下创建一个目录，以补丁包编号命名, 目录结构如下

```bash
#ls -F /tmp/                 
p33182768_190000_Linux-x86-64.zip
33182768/      
#tree -L 1 /tmp/33182768/
/tmp/33182768/
├── 32585572
├── 33192793
├── 33208107
├── 33208123
├── 33239955
├── automation
├── bundle.xml
├── README.html
└── README.txt
```

这里看到的以数字编号命名的目录就是要检查的，具体 grid 和 oracle 的检查项查看补丁包中的 readme.html 

##### grid 用户

```bash
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/32585572
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-53-29PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33192793
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-53-55PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33208107
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-54-12PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33208123
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-54-23PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33239955
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-54-35PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
```

##### oracle用户

```bash
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33192793
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/oracle/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/oracle/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/oracle/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-56-06PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
#opatch prereq CheckConflictAgainstOHWithDetail -phBaseDir /tmp/33182768/33208123
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/oracle/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/oracle/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/oracle/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_14-56-34PM_1.log

Invoking prereq "checkconflictagainstohwithdetail"

Prereq "checkConflictAgainstOHWithDetail" passed.
```

#### 系统空间检测

##### grid 用户

Grid下编辑临时文件，将 grid 对应的补丁包目录写入到临时文件中，方便执行

```bash
#cat /tmp/patch_list_gihome.txt
/tmp/33182768/32585572
/tmp/33182768/33192793
/tmp/33182768/33208107
/tmp/33182768/33208123
/tmp/33182768/33239955
```

执行检测命令

```bash
#opatch prereq CheckSystemSpace -phBaseFile /tmp/patch_list_gihome.txt
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/grid/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/grid/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/grid/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_15-03-16PM_1.log

Invoking prereq "checksystemspace"

Prereq "checkSystemSpace" passed.

OPatch succeeded.
```



##### Oracle 用户

Oracle 用户下编辑临时文件，将 grid 对应的补丁包目录写入到临时文件中，方便执行

```bash
#cat /tmp/patch_list_dbhome.txt
/tmp/33182768/33192793
/tmp/33182768/33208123
```

执行检测命令

```bash
#opatch prereq CheckSystemSpace -phBaseFile /tmp/patch_list_dbhome.txt
Oracle Interim Patch Installer version 12.2.0.1.27
Copyright (c) 2021, Oracle Corporation.  All rights reserved.

PREREQ session

Oracle Home       : /opt/oracle/products/19.3.0
Central Inventory : /opt/oraInventory
   from           : /opt/oracle/products/19.3.0/oraInst.loc
OPatch version    : 12.2.0.1.27
OUI version       : 12.2.0.7.0
Log file location : /opt/oracle/products/19.3.0/cfgtoollogs/opatch/opatch2021-12-22_15-04-45PM_1.log

Invoking prereq "checksystemspace"

Prereq "checkSystemSpace" passed.

OPatch succeeded.
```

#### 一次性补丁冲突检测

> 必须：以上步骤在所有 `RAC` 节点都完成 
>
> 使用 root 用户在所有 `RAC` 节点执行

```bash
#su - grid
Last login: Wed Dec 22 14:57:29 CST 2021


****************************************************************
***You login as grid,Please ask somebody to  double check!******
****************************************************************


grid@com91:/home/grid>su root
Password:
#grid@com91:/home/grid>export PERL5LIB=/opt/grid/products/19.3.0/perl/lib
#grid@com91:/home/grid>opatchauto apply /tmp/33182768 -analyze
OPatchauto session is initiated at Wed Dec 22 15:16:28 2021

System initialization log file is /opt/grid/products/19.3.0/cfgtoollogs/opatchautodb/systemconfig2021-12-22_03-16-31PM.log.

Session log file is /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/opatchauto2021-12-22_03-17-16PM.log
The id for this session is UX7Y

Executing OPatch prereq operations to verify patch applicability on home /opt/grid/products/19.3.0

Executing OPatch prereq operations to verify patch applicability on home /opt/oracle/products/19.3.0
Patch applicability verified successfully on home /opt/grid/products/19.3.0

Patch applicability verified successfully on home /opt/oracle/products/19.3.0


Executing patch validation checks on home /opt/grid/products/19.3.0
Patch validation checks successfully completed on home /opt/grid/products/19.3.0


Executing patch validation checks on home /opt/oracle/products/19.3.0
Patch validation checks successfully completed on home /opt/oracle/products/19.3.0


Verifying SQL patch applicability on home /opt/oracle/products/19.3.0
SQL patch applicability verified successfully on home /opt/oracle/products/19.3.0

OPatchAuto successful.

--------------------------------Summary--------------------------------

Analysis for applying patches has completed successfully:

Host:com91
CRS Home:/opt/grid/products/19.3.0
Version:19.0.0.0.0


==Following patches were SUCCESSFULLY analyzed to be applied:

Patch: /tmp/33182768/33208123
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log

Patch: /tmp/33182768/33208107
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log

Patch: /tmp/33182768/32585572
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log

Patch: /tmp/33182768/33239955
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log

Patch: /tmp/33182768/33192793
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log


Host:com91
RAC Home:/opt/oracle/products/19.3.0
Version:19.0.0.0.0


==Following patches were SKIPPED:

Patch: /tmp/33182768/33208107
Reason: This patch is not applicable to this specified target type - "rac_database"

Patch: /tmp/33182768/32585572
Reason: This patch is not applicable to this specified target type - "rac_database"

Patch: /tmp/33182768/33239955
Reason: This patch is not applicable to this specified target type - "rac_database"


==Following patches were SUCCESSFULLY analyzed to be applied:

Patch: /tmp/33182768/33208123
Log: /opt/oracle/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log

Patch: /tmp/33182768/33192793
Log: /opt/oracle/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-17-35PM_1.log


OPatchauto session completed at Wed Dec 22 15:18:01 2021
Time taken to complete the session 1 minute, 33 seconds
```

#### 补丁安装检测

安装补丁前，grid 用户执行，两个节点都执行

```bash
#grid@com91:/home/grid>cluvfy stage -pre patch

Verifying cluster upgrade state ...PASSED
Verifying Software home: /opt/grid/products/19.3.0 ...PASSED

Pre-check for Patch Application was successful.

CVU operation performed:      stage -pre patch
Date:                         Dec 22, 2021 3:21:03 PM
CVU home:                     /opt/grid/products/19.3.0/
User:                         grid
```

### 安装补丁

> 安装补丁以 root 用户执行，一个节点安装完成后，另一个节点再执行

```bash
grid@com91:/tmp>export PERL5LIB=/opt/grid/products/19.3.0/perl/lib
bash: woquCommand: command not found
grid@com91:/tmp>opatchauto apply /tmp/33182768

OPatchauto session is initiated at Wed Dec 22 15:26:50 2021

System initialization log file is /opt/grid/products/19.3.0/cfgtoollogs/opatchautodb/systemconfig2021-12-22_03-26-52PM.log.

Session log file is /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/opatchauto2021-12-22_03-27-40PM.log
The id for this session is GWI1

Executing OPatch prereq operations to verify patch applicability on home /opt/grid/products/19.3.0

Executing OPatch prereq operations to verify patch applicability on home /opt/oracle/products/19.3.0
Patch applicability verified successfully on home /opt/oracle/products/19.3.0

Patch applicability verified successfully on home /opt/grid/products/19.3.0


Executing patch validation checks on home /opt/grid/products/19.3.0
Patch validation checks successfully completed on home /opt/grid/products/19.3.0


Executing patch validation checks on home /opt/oracle/products/19.3.0
Patch validation checks successfully completed on home /opt/oracle/products/19.3.0


Verifying SQL patch applicability on home /opt/oracle/products/19.3.0
SQL patch applicability verified successfully on home /opt/oracle/products/19.3.0


Preparing to bring down database service on home /opt/oracle/products/19.3.0
Successfully prepared home /opt/oracle/products/19.3.0 to bring down database service


Performing prepatch operations on CRS - bringing down CRS service on home /opt/grid/products/19.3.0
Prepatch operation log file location: /opt/ogrid/crsdata/com91/crsconfig/crs_prepatch_apply_inplace_com91_2021-12-22_03-28-30PM.log
CRS service brought down successfully on home /opt/grid/products/19.3.0


Performing prepatch operation on home /opt/oracle/products/19.3.0
Perpatch operation completed successfully on home /opt/oracle/products/19.3.0


Start applying binary patch on home /opt/oracle/products/19.3.0
Binary patch applied successfully on home /opt/oracle/products/19.3.0


Performing postpatch operation on home /opt/oracle/products/19.3.0
Postpatch operation completed successfully on home /opt/oracle/products/19.3.0


Start applying binary patch on home /opt/grid/products/19.3.0
Binary patch applied successfully on home /opt/grid/products/19.3.0


Performing postpatch operations on CRS - starting CRS service on home /opt/grid/products/19.3.0
Postpatch operation log file location: /opt/ogrid/crsdata/com91/crsconfig/crs_postpatch_apply_inplace_com91_2021-12-22_03-39-13PM.log
CRS service started successfully on home /opt/grid/products/19.3.0


Preparing home /opt/oracle/products/19.3.0 after database service restarted
No step execution required.........


Trying to apply SQL patch on home /opt/oracle/products/19.3.0
SQL patch applied successfully on home /opt/oracle/products/19.3.0

OPatchAuto successful.

--------------------------------Summary--------------------------------

Patching is completed successfully. Please find the summary as follows:

Host:com91
RAC Home:/opt/oracle/products/19.3.0
Version:19.0.0.0.0
Summary:

==Following patches were SKIPPED:

Patch: /tmp/33182768/33208107
Reason: This patch is not applicable to this specified target type - "rac_database"

Patch: /tmp/33182768/32585572
Reason: This patch is not applicable to this specified target type - "rac_database"

Patch: /tmp/33182768/33239955
Reason: This patch is not applicable to this specified target type - "rac_database"


==Following patches were SUCCESSFULLY applied:

Patch: /tmp/33182768/33192793
Log: /opt/oracle/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-29-50PM_1.log

Patch: /tmp/33182768/33208123
Log: /opt/oracle/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-29-50PM_1.log


Host:com91
CRS Home:/opt/grid/products/19.3.0
Version:19.0.0.0.0
Summary:

==Following patches were SUCCESSFULLY applied:

Patch: /tmp/33182768/32585572
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-34-16PM_1.log

Patch: /tmp/33182768/33192793
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-34-16PM_1.log

Patch: /tmp/33182768/33208107
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-34-16PM_1.log

Patch: /tmp/33182768/33208123
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-34-16PM_1.log

Patch: /tmp/33182768/33239955
Log: /opt/grid/products/19.3.0/cfgtoollogs/opatchauto/core/opatch/opatch2021-12-22_15-34-16PM_1.log


Patching session reported following warning(s):
_________________________________________________

[WARNING] The database instance 'rac1' from '/opt/oracle/products/19.3.0', in host'com91' is not running. SQL changes, if any,  will not be applied.
To apply. the SQL changes, bring up the database instance and run the command manually from any one node (run as oracle).
Refer to the readme to get the correct steps for applying the sql changes.


OPatchauto session completed at Wed Dec 22 15:43:04 2021
Time taken to complete the session 16 minutes, 14 seconds
```

### 确认补丁安装情况

> RAC 集群的所有 grid 用户下执行 `cluvfy stage -post patch`

```bash
#grid@com91:/home/grid>cluvfy stage -post patch

Verifying cluster upgrade state ...PASSED
Verifying Software home: /opt/grid/products/19.3.0 ...PASSED

Post-check for Patch Application was successful.

CVU operation performed:      stage -post patch
Date:                         Dec 22, 2021 4:26:43 PM
CVU home:                     /opt/grid/products/19.3.0/
User:                         grid
```

> root 用户下执行 `crsctl query crs releasepatch` && `crsctl query crs softwarepatch`。正常情况两个节点的输出一致

```bash
#crsctl query crs releasepatch
Oracle Clusterware release patch level is [2966572961] and the complete list of patches [32585572 33192793 33208107 33208123 33239955 ] have been applied on the local node. The release patch string is [19.13.0.0.0].
#crsctl query crs softwarepatch
Oracle Clusterware patch level on node com91 is [2966572961].
#kfod op=patches
---------------
List of Patches
===============
32585572
33192793
33208107
33208123
33239955
```

grid 用户下查看补丁组件版本

```bash
#grid@com91:/home/grid>opatch lspatches
33239955;TOMCAT RELEASE UPDATE 19.0.0.0.0 (33239955)
33208123;OCW RELEASE UPDATE 19.13.0.0.0 (33208123)
33208107;ACFS RELEASE UPDATE 19.13.0.0.0 (33208107)
33192793;Database Release Update : 19.13.0.0.211019 (33192793)
32585572;DBWLM RELEASE UPDATE 19.0.0.0.0 (32585572)
```

以上正常后，补丁安装就结束了。通过 `sqlplus -v` 查看数据库版本已经升级到 `19.13` 了。

```bash
#grid@com91:/home/grid>sqlplus -v

SQL*Plus: Release 19.0.0.0.0 - Production
Version 19.13.0.0.0
```

