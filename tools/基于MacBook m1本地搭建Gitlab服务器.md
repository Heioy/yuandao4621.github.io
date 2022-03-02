### 基于MacBook m1 本地搭建Gitlab服务器

#### 前言

最近使用 github 的时候，经常发生代码无法推送到远端库，在终端检查 github 的域名服务器，发现完全ping不通。况且平时使用 github 也并不顺畅，经常出现打不开 github 页面的情况。

基于此种情况，便萌生了在本地电脑上搭建一个 gitlab 服务器，解决推送代码的问题。

#### 搭建环境

既然是在本地搭建，就需要使用自己电脑的资源。因为我自己使用的电脑是 2020 年发售的 MacBook m1，是基于 ARM64 架构的。

安装 gitlab 之前去网上查了很多关于安装 gitlab 的资料，最初想着直接在 docker 上装一个 gitlab，后经历艰难险阻，最终 gitlab 的镜像启动失败放弃了。

第二次选择在虚拟机上搭建 gitlab，但出身 ARM 的 MacBook 本身对虚拟化的支持并不怎么好，最终我选择了 VMware 公司的 `VMware Fusion Public preview 21H1 ` 这款软件(为数不多的可以在 ARM 上支持虚拟化的产品)。

下载地址：[下载 VMware Fusion Public Tech Preview 21H1](https://customerconnect.vmware.com/cn/downloads/get-download?downloadGroup=FUS-PUBTP-2021H1)

对于 Mac 用户，下载解压后安装就可。

安装后，需要安装一个虚拟机，虚拟机的镜像一定要选择 arm 架构的，否则，虚拟机无法启动，我选择安装的是 `ubuntu-server-20.04 LTS` 

镜像下载地址：[Ubuntu for ARM | Download | Ubuntu](https://ubuntu.com/download/server/arm)

虚拟机安装没什么特别要注意的，如果电脑资源足够，尽可能分配足够的内存给虚拟机使用。我安装时分配了 2个G 的内存，安装 gitlab 后虚拟机可用内存只剩不到 60M 了。因此，如果可以，你应该分配不小于 4G 的内存给虚拟机使用。

| 电脑系统     | MacBook m1 - ARM64    |
| ------------ | --------------------- |
| 虚拟机       | ubuntu server - ARM64 |
| 虚拟机处理器 | 2 core                |
| 虚拟机内存   | 2G                    |
| 虚拟机硬盘   | 40G                   |

#### 安装Gitlab

当你安装好虚拟机之后，就可以开始准备安装 gitlab 了。不过，在这之前，你还需要对虚拟机进行一些配置

- 更新系统源

  ```bash
  # 将 /etc/apt/sources.list 文件中的 http://archive.ubuntu.com/ 替换为 mirrors.aliyun.com
  # vim 底线模式下执行 %s/archive.ubuntu.com/mirrors.aliyun.com/g
  ```

- 更新软件源

  ```bash
  sudo apt update && apt-get upgrade
  ```

- 安装依赖

  ```bash
  sudo apt-get install -y curl openssh-server ca-certificates tzdata perl postfix
  ```

- 下载 gitlab 仓库

  ```bash
  # gitlab 企业版
  root@ubuntu:~# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
  
  # gitlab 社区版
  root@ubuntu:~# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
  
  # 执行成功后，使用apt search gitlab-ce 有gitlab的软件信息即可
  root@ubuntu:~# apt search gitlab-ce
  Sorting... Done
  Full Text Search... Done
  gitlab-ce/focal,now 14.4.2-ce.0 arm64 [installed]
    GitLab Community Edition (including NGINX, Postgres, Redis)

- 安装 gitlab

  ```bash
  root@ubuntu:~# sudo apt-get install gitlab-ce
  ```

  等待安装即可

- 配置 gitlab web页面访问

  安装结束后，最后程序会提示你进行 gitlab 配置，这里主要进行 external_url，将 external_url 后面的地址修改为你虚拟机的ip。

  ```bash
  # 修改 /etc/gitlab/gitlab.rb 文件中的 EXTERNAL_URL
  # 修改结束后保存退出
  
  root@ubuntu:~# sudo gitlab-ctl reconfigure
  ```

- 访问 gitlab web端

  当做完上面的步骤后，就可以访问 gitlab web页面了，使用你配置的 external_url，将其输入浏览器的网址栏

  第一次登录后需要修改密码，你可以在web页面进行修改。选择 "Edit profiles" —>   "Password" 进行修改

- 设置 gitlab 开机启动

  ```bash
  root@ubuntu:~# systemctl enable gitlab-runsvdir
  ```

  这样，即便系统重启后你的 gitlab 也是可以访问的。

