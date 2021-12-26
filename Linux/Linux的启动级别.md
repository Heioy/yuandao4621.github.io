通常来说，通过 `VMWare` 或 `Virtual Box` 等虚拟化软件安装虚拟机后，安装后默认进入的都是 Linux 操作系统的图形界面，但在企业中你也能看到很多操作系统启动之后是没有图形界面的，只有一个黑乎乎的 `terminal` 终端。

然而，出现两种差异的根本原因在于操作系统启动时设置的`runlevel(运行级别)`不同。`运行级别` 用来控制操作系统启动后应该以什么样的模式运行，根据不同的运行模式关闭或开启某些操作系统的功能。

对于我们看到的图形化界面来讲，正是设置了运行级别为 5 `graphical(图形界面)`，对于 `rhel 7.+` 系列的操作系统，默认的启动级别一般也是 5。也就是说，你可以在操作系统启动后进入图形界面。对于启动操作系统后默认进入终端模式下的，则设置的启动级别为 3级 `multi-user(多用户)`。 

Linux 将操作系统上的启动模式分为 `0 ~ 6` 几种级别

- 0 级别(`halt`)：系统停机模式
- 1 级别(`single user`)：单用户模式。只允许系统管理员登录，一般系统维护时会设置该模式
- 2 级别(`multiuser without nfs`)：多用户模式，不开启网络
- 3 级别(`full multiuser`)：完整的多用户模式
- 4 级别(`unused`)：预留的自定义模式，一般不会使用
- 5 级别(`x11`)：图形界面
- 6 级别(`reboot`)：重启模式

查看系统当前的运行级别

```bash
[root@server ~]# runlevel
N 3
```

可以看到，当前操作系统的运行级别为 3级，对应的模式为 `multi-user(多用户模式)`。

> 需要注意的是，`multiuser` 和 `x11` 两种模式并无太大本质上的区别， `x11` 模式支持图形化的界面。而在 `multiuser` 模式中，是无法在操作系统中调用图形界面的任何程序的。

你也可以通过 `systemctl get-default` 查看

```bash
[root@server ~]# systemctl get-default
multi-user.target
```

将启动级别从 `multi-user` 切换到 `graphic` 

```bash
[root@server ~]# systemctl set-default graphical.target
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.
```

重启操作系统后再次查看运行级别

```bash
[root@server ~]# runlevel
N 5
[root@server ~]# systemctl get-default
graphical.target
```

尽管此时的运行级别为 `graphic` 图形模式，但是重启后进入的仍然不是图形化界面，这是因为部分操作系统安装时选择的镜像模式为 `server`，这种类型的镜像一般提供给工作站使用，安装后是没有图形界面的。



当修改操作系统的启动级别时，实际上是对 `/etc/systemd/system/default.target` 文件的修改。

- 首先删除掉旧的运行级别对应的文件
- 然后在 `/usr/lib/systemd/system/` 目录下寻找运行级别对应的 `*.target`  文件，
- 最后创建 `*.target` 的符号链接文件





参考

[Linux中shutdown，halt，poweroff，init 0区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/165436141)