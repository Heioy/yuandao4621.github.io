
#### dbca.rsp 应答文件模版
```bash
##############################################################################
##                                                                          ##
##                            DBCA response file                            ##
##                            ------------------                            ##
## Copyright(c) Oracle Corporation 1998,2017. All rights reserved.          ##
##                                                                          ##
## Specify values for the variables listed below to customize               ##
## your installation.                                         		        ##
##                                                            		        ##
## Each variable is associated with a comment. The comment    		        ##
## can help to populate the variables with the appropriate   		        ##
## values.                                                  		        ##
##                                                               	        ##
## IMPORTANT NOTE: This file contains plain text passwords and   	        ##
## should be secured to have read permission only by oracle user 	        ##
## or db administrator who owns this installation.               	        ##
##############################################################################
#-------------------------------------------------------------------------------
# Do not change the following system generated value.
#-------------------------------------------------------------------------------
responseFileVersion=/oracle/assistants/rspfmt_dbca_response_schema_v12.2.0

# 数据库全局名称
gdbName=test

# 数据库实例名称,可以与数据库名称一致
sid=test

# 数据库类型,可选择{SI、RAC、RACONENODE}
databaseConfigType=RAC

# RACONENODE安装时需要指定的实例名,其他安装选项可以保持默认
RACOneNodeServiceName=

# 数据库管理策略；Oracle数据库提供两种管理策略 "Admin-Managed" 和 "Policy-Managed", 如果使用 "Admin-Managed" 模式, 则如下的 "policyManaged" 的值应为 false
policyManaged=false

# 是否创建[服务器池](https://blog.csdn.net/tianlesoftware/article/details/7753607) 
createServerPool=false

# 服务器池名称, 根据 "createServerPool" 的值来设置, 这里不用配置
serverPoolName=

# Oracle基数, 其值应为整数
cardinality=

# 是否强制创建服务器池
force=false

# 并行查询池(Parallel Query (PQ) pool)的名称
pqPoolName=

# 并行查询基数
pqCardinality=

# 是否创建容器类型的数据库
createAsContainerDatabase=false

# 容器数据库的个数; 根据 "createAsContainerDatabase" 来设置,
numberOfPDBs=0

# 容器数据库名称
pdbName=

# 所有的容器数据库是否使用本地的undo表空间
useLocalUndoForPDBs=true

# 容器数据库Admin用户密码
pdbAdminPassword=

# 安装数据库的所有节点
nodelist=node1,node2

# 创建数据库时使用的模版 {Data_Warehouse.dbc、General_Purpose.dbc}
# Valid values  : 
templateName={ORACLE_HOME}/assistants/dbca/templates/Data_Warehouse.dbc

# Oracle数据库sys用户密码
sysPassword=oracle

# Oracle数据库system用户密码
systemPassword=oracle

# Password for Windows Service user
serviceUserPassword=

# 配置企业管理方式(Enterprise Manager Configuration Type), 可选 {CENTRAL、DBEXPRESS、BOTH、NONE}
emConfiguration=DBEXPRESS

# EM运行端口
emExpressPort=5501

# 集群一致性检查(Cluster Verification Utility) 
runCVUChecks=true

# 数据库默认监控用户dbsnmp密码
dbsnmpPassword=oracle

# EM management server host name
omsHost=

# EM management server port number
omsPort=0

# EM Admin user
emUser=

# EM Admin user password
emPassword=

# enable Oracle Database vault
dvConfiguration=false

# DataVault用户
dvUserName=

# DataVault Owner用户密码
dvUserPassword=

# DataVault Account Manager
dvAccountManagerName=

# DataVault Account Manager密码
dvAccountManagerPassword=

# Oracle标签安全性(Oracle Label Security)
olsConfiguration=false

# 模版文件路径
datafileJarLocation={ORACLE_HOME}/assistants/dbca/templates/

# 数据文件路径
datafileDestination=+NVMEDG/{DB_UNIQUE_NAME}/

# 快速恢复区使用的数据文件路径 默认 $ORACLE_BASE/flash_recovery_area
recoveryAreaDestination=

# 存储类型 FS (CFS for RAC), ASM
storageType=ASM

# 存储介质名称
diskGroupName=+NVMEDG/{DB_UNIQUE_NAME}/

# GI asmsnmp监控用户密码
asmsnmpPassword=

# 快速恢复区组名称
recoveryGroupName=

# 数据库字符集 默认 "US7ASCII"
characterSet=ZHS16GBK

# 本地字符集 默认 "AL16UTF16"
nationalCharacterSet=AL16UTF16

# 是否注册目录服务
registerWithDirService=false

# 目录服务名称
dirServiceUserName=

# 目录服务用户所属密码
dirServicePassword=

# 加密外部口令文件, "registerWithDirService" 值为true时设置该值
walletPassword=

# 数据库监听 默认加载 $ORACLE_HOME/network/admin/listener.ora  
listeners=

# 变量对儿 格式为 <variable>=<value> 
variablesFile=

# 变量对儿, 格式为 name=value
# 注意:  variables 会覆盖掉 variablesFile 的配置
# 这里的 DB_UNIQUE_NAME、ORACLE_BASE、PDB_NAME、DB_NAME、ORACLE_HOME、SID 需要根据实际配置
variables=DB_UNIQUE_NAME=test,ORACLE_BASE=/opt/oracle,PDB_NAME=,DB_NAME=test,ORACLE_HOME={ORACLE_HOME},SID=test

# 初始化参数 
# 注意: initParams 会覆盖掉 templates 的配置
# 这里的 *.thread、sga_target、等变量需要根据实际配置
initParams=test1.thread=1,test2.instance_number=2,sga_target=3GB,test2.thread=2,diagnostic_dest={ORACLE_BASE},cluster_database=true,audit_file_dest={ORACLE_BASE}/admin/{DB_UNIQUE_NAME}/adump,local_listener=-oraagent-dummy-,compatible=12.2.0,test1.instance_number=1,open_cursors=300,family:dw_helper.instance_mode=read-only,test2.undo_tablespace=UNDOTBS2,processes=2000,nls_language=AMERICAN,pga_aggregate_target=1GB,dispatchers=(PROTOCOL=TCP) (SERVICE=racXDB),db_block_size=8192BYTES,star_transformation_enabled=TRUE,nls_territory=AMERICA,control_files=("+NVMEDG/{DB_UNIQUE_NAME}/control01.ctl", "+NVMEDG/{DB_UNIQUE_NAME}/control02.ctl"),db_name=test,audit_trail=db,test1.undo_tablespace=UNDOTBS1,remote_login_passwordfile=exclusive

# 创建数据库的过程中创建示例表结构
sampleSchema=false

# Oracle可使用的物理内存比例
memoryPercentage=40

# 数据库类型 可选 {MULTIPURPOSE、DATA_WAREHOUSING、OLTP}  默认 MULTIPURPOSE
databaseType=MULTIPURPOSE

# 启用内存自动管理
automaticMemoryManagement=false


# 为Oracle分配的总内存
totalMemory=0
```

安装命令
```bash
oracle@node1:/home/oracle>dbca -createDatabase -silent -responseFile dbca-test.rsp
[WARNING] [DBT-10102] The listener configuration is not selected for the database. EM DB Express URL will not be accessible.
   CAUSE: The database should be registered with a listener in order to access the EM DB Express URL.
   ACTION: Select a listener to be registered or created with the database.
Enter SYS user password:

Enter SYSTEM user password:

[WARNING] [DBT-06208] The 'SYS' password entered does not conform to the Oracle recommended standards.
   CAUSE:
a. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9].
b.The password entered is a keyword that Oracle does not recommend to be used as password
   ACTION: Specify a strong password. If required refer Oracle documentation for guidelines.
[WARNING] [DBT-06208] The 'SYSTEM' password entered does not conform to the Oracle recommended standards.
   CAUSE:
a. Oracle recommends that the password entered should be at least 8 characters in length, contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9].
b.The password entered is a keyword that Oracle does not recommend to be used as password
   ACTION: Specify a strong password. If required refer Oracle documentation for guidelines.
[WARNING] [DBT-09102] Target environment does not meet some optional requirements.
   CAUSE: Some of the optional prerequisites are not met. See logs for details. /opt/oracle/cfgtoollogs/dbca/trace.log_2022-02-10_05-47-40-PM
   ACTION: Find the appropriate configuration from the log file or from the installation guide to meet the prerequisites and fix this manually.
Copying database files
1% complete
2% complete
15% complete
27% complete
Creating and starting Oracle instance
29% complete
32% complete
36% complete
40% complete
41% complete
43% complete
45% complete
Creating cluster database views
47% complete
63% complete
Completing Database Creation
64% complete
65% complete
68% complete
69% complete
71% complete
72% complete
Executing Post Configuration Actions
100% complete
Look at the log file "/****/cfgtoollogs/dbca/test/test.log" for further details.
```