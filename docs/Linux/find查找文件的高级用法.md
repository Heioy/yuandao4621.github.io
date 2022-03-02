Linux 操作系统为查找文件提供了众多的方法，针对不同类型的文件通常使用的方式也不尽相同。一般情况下，查找操作系统的系统命令或二进制文件会使用到 `which` 或 `where`。但很多时候，单纯依靠 `which` 或 `whereis` 两个命令远远无法满足我们查找文件的需求。
`find` 命令用来查找 Linux 操作系统上的所有文件，包括系统命令、文件文件、块设备、字符设备等等一切文件。`find` 命令提供了基于各式各样的属性搜索一个指定目录下的满足条件的文件。

#### 基本格式
find 命令由 路径、参数、操作表达式、执行命令组成
```bash
find START_DIRECTORY [TEST] [OPTIONS] [EXPRESSION]
```

#### 常用参数
- -type: 按照类型查找文件。常用的类型包含：`b(块设备)`、`c(字符设备)`、`d(目录)`、`f(文本文件)`、`p(管道文件)`、`l(符号链接文件)`、`s(套接字)`、`D(door)`.  
	常见用法
	```bash
	# 查找当前用户家目录下的符号链接文件
	[root@server ~]# find ~/ -type l
	```
- -size: 按照文件大小查找文件。常用的单位包含：`k(KB)`、`M(MB)`、`G(GB)`  
	常见用法
	```bash
	# 查找/var/log/目录下日志文件超过50M的日志文件
	[root@server ~]# find /var/log/ -type f -size +5M
	```
- -amin: 查找最后一次访问文件的时间。指定访问时间少于 `n分钟`，使用 `-n`；指定访问文件的时间大于 `n分钟`，使用 `+n`.  
	> 类似用法包括: -amin, -atime, -cmin, -ctime, -mmin, and -mtime 

	常见用法
	```bash
	# 查找当前用户家目录下30分钟被访问过的文件
	[root@server ~]# find ~/ -type f -amin -30
	# 查找当前用户家目录下30分钟前被访问过的文件
	[root@server ~]# find ~/ -type f -amin +30
	```
- -group: 根据文件的属组进行匹配。  
	> 类似用法包括: -user  

	常见用法
	```bash
	# 查找/home 目录下属组为docker的文件
	[root@server ~]# find /home/  -group docker
	```
- -perm: 根据文件权限查找文件。  
	常见用法
	```bash
	# 查找当前用户家目录下权限为 rw------- 的文件
	[root@server ~]# find ~/ -type f -perm 600
	# 查找当前家目录下其他用户拥有可执行权限的文件
	[root@server ~]# find ~/ -type f -perm -o=x
	```
- -name: 根据文件名模式进行匹配。支持正则匹配   
	> 类似用法包括: -iname (忽略文件名大小写)  

	常见用法
	```bash
	# 查找/var/log/ 目录下文件名中包含 messages 目录下日志文件
	[root@server ~]# find /var/log/ -type f -name messages*
	```
- maxdepth: 查找目录层级查找文件。  
	> 类似用法包括: -mindepth

	常见用法
	```bash
	# 查找/etc 目录下的hostname文件, 可进入的目录层级为2级
	[root@server ~]# find /etc -maxdepth 2 -name hostname
	```
- path: 根据指定路径匹配满足规则的文件。  
	常见用法
	```bash
	# 查找满足 /e*c/ho*me 的文件
	[root@server ~]# find /etc -path "/e*c/ho*me"
	```
- -or: 通过组合条件间的逻辑关系进行文件查找。格式为 ( expression 1 ) -or[and,not] (expression 2 )    
	> 类似用法包括: -and, -not

	常见用法
	```bash
	# 查找权限不是 0644 的文件 或 权限不是 0755 的目录
	[root@server ~]# find ~/ \( -type f -not -perm 0644 \) -or \( -type d -not -perm 0755 \)
	```	
- -delete: 查找符合规则的文件并删除。格式为 find xxx -delete  
	> 类似用法包括: -ls, -print, -quit(一旦找到一个满足规则的文件程序立即退出)

	常见用法
	```bash
	# 查找 /var/log/ 目录下的历史messages日志文件并删除
	[root@server ~]# find /var/log/ -name messages-2021* -delete
	```	
- print0: print0 会产生由 null 字符分离的输出。  
	常见用法
	```bash
	[root@server ~]# find /var/log/ -name sa* -type f -print0 # 同样格式的文件会输出在同一行
	/var/log/sa/sa17/var/log/sa/sar17/var/log/sa/sa18
	[root@server ~]# find /var/log/ -name sa* -type f -print0 | xargs --null ls -l # xargs --null 可以接受 null 字符分离的输入。通过配合 find ... -print0 可以保证所有格式的文件都被正确处理
	```

- -exec: 执行自定义命令。格式为 find xxx -exec command '{}' ';'  
	常见用法	
	```bash
	# 查找 /var/log/ 目录下的启动日志并查看日志信息
	[root@server ~]# find /var/log/ -name boot.log-* -type f -exec ls -lh '{}' ';'
	```
	

#### 高级用法

查找文件时使用 `-exec command '{}' ';'` 可以对查找到的结果执行自定义操作，但通常而言，通过 find 找到的文件往往不止一个。

执行 `-exec ls -lh '{}' ';'` 时的操作为，当 find 命令查找到一个匹配结果时，会调用一次 `-exec ls -lh '{}' ';'`。也就是说，倘若通过 find 命令查找到10个匹配的文件，会调用10次 `-exec ls -lh '{}' ';'`。

这种导致命令执行多次而不是一次的情况，显然不符合使用习惯。例如，你可能更习惯于使用 `ls file1 file2` 来代替 `ls file1; ls file2` 这样的用法。

要解决上面的问题，有两种方式。传统方式是通过 `xargs`，另一种方式，是通过 `find` 命令的另一个功能。

- 将 `-exec command '{}' ';'` 表达式中的 `;` 修改为 `+`，`+` 模式下，find 命令会将所有的搜索结果结合为一个「参数列表」，然后用于表达式中一次执行。  
  例如：# 查找 /var/log/ 目录下的启动日志并查看日志信息
  ```bash
  [root@server ~]# find /var/log/ -name boot.log-* -type f -exec ls -lh '{}' '+'
  ```
  在 `+` 模式下，系统对匹配到的文件只执行一次 `ls -lh `

- `xargs`：xagrs 会执行一个函数，将从标准输入接受输入，并将输入转换为一个命令的参数列表。
  例如：# 查找 /var/log/ 目录下的启动日志并查看日志信息
  ```bash
  [root@server ~]# find /var/log/ -name boot.log-* -type f | xargs ls -lh
  ```
  使用 `xargs` 后，find 命令的输出被管道到 「xargs」命令体中，之后，「xargs」会为 「ls -lh」命令构建参数列表，然后执行 「ls -lh」命令。
