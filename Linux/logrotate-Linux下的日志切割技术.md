### 什么是日志切割？

日志切割指的是当应用程序或操作系统的日志文件满足设定的触发条件，对其进行切割/分割处理。切割后的日志会在原有日志的基础上多出一个新的日志文件，且后续产生的日志也会被写入到新的日志文件中，直到下一次满足设定的触发条件时。

一般情况下，我们习惯于将各种应用程序(Web端程序、应用服务、数据库等)软件部署在 Linux操作系统上，但众多软件部署后运行时会产生对应的日志记录，以便于出现故障后能够及时排查问题原因。但久而久之，随着时间的推长，应用程序的日志文件可能会变得很庞杂，这对于运维、管理、故障排查等来说非常不方便，因此，及时对日志进行定期切割和清理时非常有必要的。

常用的日志切割方式：`按时间` 和 `按日志大小`。

`按时间切割`：在进行切割日志时，以时间为标准，日志出现的时间满足设定的时间阈值时，则进行日志切割。类似的典型用法有：`/var/log/messages` 日志即按每7天切割一次的规则进行日志切分。

`按日志大小切割`：在进行切割日志时，以日志大小为参考标准，日志的大小满足设定的大小时进行日志切割。一般应用程序的日志多使用容量进行切割，例如，jenkins



### `logrotate-日志轮转`

Linux 操作系统上切割日志可以通过 `logrotate` 来实现。

相比其他日志切割软件来看，使用 `logrotate` 有以下优点：

* `logrotate` 是Linux操作系统上自带的一款开源的日志切割软件，因此你无需安装
* `logrotate` 自身已经集成进操作系统的定时任务中，因此你无需再配置定时任务
* `logrotate` 自身支持日志压缩



#### `logrotate` 配置文件解析

`logrotate` 的全局配置文件为 `/etc/logrotate.conf`。

```bash
[root@server ~]# cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```

- `weekly`：表示每周轮转一次，常见用法有  `daily`、 `weekly`、`monthly`、`yearly`
- `rotate`：保留的日志副本数
- `create`：日志轮转时创建一个新的日志文件用来接受新产生的日志
- `dateext`：切割日志时使用日期作为文件后缀
- `compress`：轮转后的日志是否进行压缩
- `include /etc/logrotate.d/`：日志轮转时会加载 `/etc/logrotate.d/` 目录下的所有配置文件（ps：该目录下的配置必须在操作系统中有rpm包存在）

   `/var/log/btmp` 和 `/var/log/wtmp` 两个日志因为在操作系统中没有对应的 rpm 包，因此被当作"孤儿"日志，配置在 `/etc/logrotate.conf` 全局配置中。

#### 自定义`logrotate`配置

Linux 操作上的一些系统服务默认已经配置了日志切割规则，可以通过查看 `/etc/logrotate.d/` 目录下的文件来查看

```bash
[root@server ~]# ls /etc/logrotate.d/
bootlog  chrony  firewalld  jenkins  syslog  wpa_supplicant  yum
```

倘若操作系统上新部署了一项应用程序，或需要对其他的日志文件配置切割规则，你可以自定义 `logrotate` 的配置文件使其生效。这里以 `/var/log/audit/audit.log` 日志为例

```bash
[root@server ~]# cat /etc/logrotate.d/audit

/var/log/audit/*.log {

	missingok			# 日志切割时缺少该日志不会报错
	weekly				# 每周切割一次
	rotate 10			# 切割后最多保留10个文件
	size +100M			# 当前日志容量超过100M时,立即进行日志切割
	compress			# 切割后的日志进行压缩
	dateext				# 切割后的日志以时间'年月日'为后缀
	notifempty			# 日志为空时不进行切割
	create 0600 root root		# 切割时创建一个新日志文件,模式为0600,日志属组为root root

}
```

`logrotate` 提供了一些命令行参数，用来测试并查看配置及运行结果

```bash
# 测试配置文件的语法是否合法
[root@server ~]# logrotate --debug --force /etc/logrotate.d/audit
reading config file /etc/logrotate.d/audit
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /var/log/audit/*.log  forced from command line (10 rotations)
empty log files are not rotated, old logs are removed
considering log /var/log/audit/audit.log
  log needs rotating
rotating log /var/log/audit/audit.log, log->rotateCount is 10
dateext suffix '-20211124'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding old rotated logs failed
fscreate context set to system_u:object_r:auditd_log_t:s0
renaming /var/log/audit/audit.log to /var/log/audit/audit.log-20211124
creating new /var/log/audit/audit.log mode = 0600 uid = 0 gid = 0
compressing log with: /bin/gzip
```



#### 参数释义

##### 压缩

- `compress`：使用压缩，默认的压缩方式为 `gzip`
- `compresscmd`：自定义压缩的命令，默认压缩方式为 `gzip`
- `uncompresscmd`：自定义解压的命令，默认解压工具为 `gunzip`
- `compressext`：压缩时使用后缀，默认 `gzip` 压缩格式下后缀为 `.gz`
- `compressoptions`：压缩选项，默认使用 `gzip`；如果使用其他压缩选项，需要设置 `compressoptions` 与之匹配
- `delaycompress`：延迟压缩，实际压缩生效的时间发生在下一次日志切割之时
- `nodelaycompress`：不使用延迟压缩，即轮转时压缩（默认配置）
- `nocompress`：不实用压缩

##### 归档方式

- `copy`：轮转时复制完整的日志文件，常用来做当前日志文件的镜像备份。当指定 `copy` 选项时， `create` 选项会失效
- `copytruncate`：轮转时复制完整的日志文件并清空原来的日志文件(相当于 `echo > logfile`)，新写入的日志会继续往清空后的日志文件中写入。但在日志 `copy`和  `truncate` 的过程中写入的日志可能会丢失。当指定 `copytruncate` 选项时，`create` 选项会失效
- `nocopy`：轮转时不会复制原日志文件
- `nocopytruncate`：轮转时复制原日志文件后不会清空原日志文件的内容
- `create`：轮转时创建一个新的日志文件，可以设置创建文件的权限、所有者、及属组
- `nocreate`：轮转时不会创建新的日志文件
- `createolddir`：轮转时如果指定的目录不存在，则会创建，支持设置目录的权限、所有者和属组
- `nocreateolddir`：轮转时指定的目录不存在时不会进行创建

##### 归档路径

- `olddir directory`：配置目录后，轮转后的日志会保存在指定的目录下
- `noolddir`：轮转后的日志保存在日志原有的目录下

##### 归档删除

- `shred`：删除日志时使用 `shred -u(粉碎式删除)` 的方式（默认关闭）
- `shredcycles count`：删除日志前会先重写覆盖日志文件，达到设定的次数后才进行删除
- `noshred`：删除日志时使用 `unlink` 的方式删除（还没搞懂这个删除逻辑）

##### 归档规则

- `hourly`：每小时进行一次日志切割

- `daily`：每天进行一次日志切割

- `weekly`：每周进行一次日志切割

- `monthly`：每月进行一次日志切割

- `yealy`：每年进行一次日志切割

- `size`：按日志大小进行切割

- `rotate count`：日志轮转时保存的归档文件数量 

- `start count`：日志轮转时从指定的count开始，例如，`start 9`，则日志轮转后会跳过 `0-8` 生成 `xxx.9` 这样的日志，

- `maxage count`：轮转后的日志超过设定的日期会被删除，只有被轮转的日志才会应用到此规则

- `maxsize size`：设置maxsize后，如果被轮转的日志大小超过设置的 size，则会在设置的轮转时间(例如：weekly)之前进行日志轮转

- `minsize  size`：设置minsize后，如果被轮转的日志大小不满足设置的size，即便到了设置的轮转时间也不会触发日志轮转

  

##### 日期格式

- `dateext`：切割后日志的后缀名以 "YYYYMMDD" 为格式
- `nodateext`：轮转日志时不使用后缀名为日期的格式
- `dateformat`：自定义日志后缀的时间格式，仅 `%Y %m %d %H %s` 可以被使用。 例如 `dateformat %m/%d/%Y`
- `dateyesterday`：切割日志时使用昨天的时间而不是今天的时间
- `extension`：日志文件在轮转后使用指定的 ext 扩展名。如果使用压缩，通常ext还会加上压缩文件的扩展名，通常是 `.gz`。例如，你有一个日志文件名为 `mylog.foo`，你可以通过 `extension ext` 将日志轮转为 `mylog.1.foo.gz` 而不是 `mylog.foo.1.gz`

##### 轮转规则

- `ifempty`：日志为空时也会按照规则进行轮转
- `notifempty`：日志为空时不进行轮转
- `missingok`：轮转的日志不存在时，继续下一个日志的轮转，不会报错
- `nomissingok`：轮式时日志不存在，会有报错提示 （默认配置）

##### 邮件配置

- `mail address`：配置邮件地址后，会将轮转的日志信息发送到邮箱
- `nomail`：不发送轮转的日志信息到任何邮箱 （默认配置）
- `mailfirst`：配置邮箱后，轮转后将本次生成的日志文件发送到邮箱
- `maillast`：配置邮箱后，将上一次轮转的日志文件发送到邮箱 （默认配置）

##### 归档时执行的脚本

- `include file_or_directory`：轮转前会尝试读取 include 配置的文件或目录，如果配置的是目录，则目录下的所有文件都会被加载到轮转的配置中；但对于文件扩展名以 `taboo` 结尾的文件或配置路径为多个目录、管道等时，加载配置时会被忽略
- `prerotate/endscript`：日志轮转前会执行自定义的命令(脚本)。通常，轮转的日志的完整路径会作为传入的第一个参数
- `postrotate/endscript`：日志轮转后会执行自定义的命令(脚本)。通常，轮转的日志的完整路径会作为传入的第一个参数
- `firstaction/endscript`：执行 `prerotate/endscript` 前且最少一个日志会被轮转时执行该语句，整个模式会作为第一个参数传递给该语句，当语句执行异常时，不会再向下执行
- `lastaction/endscript`：执行 `postrotate/endscript` 前且最少一个已经被轮转后执行该语句，整个模式会作为第一个参数传递给该语句，当语句执行异常时，仅仅展示错误信息
- `preremove/endscript`：仅仅当删除轮转过的日志前执行该语句。即将被删除的日志名会作为参数被传递进该语句
- `sharedscripts`：共享模式。启用共享模式后，当 `prerotate` 和 `postrotate` 语句执行时匹配到多个日志时，`prerotate` 和 `postrotate` 语句仅仅只会执行一次。正常模式下，轮转时匹配到的每个日志文件都会单独执行一次 `prerotate` 和 `postrotate`  语句。
- `nosharedscripts`：轮转时匹配到多个日志文件时，每个日志文件都会执行一次 `prerotate` 和 `postrotate`  语句。（默认配置）

