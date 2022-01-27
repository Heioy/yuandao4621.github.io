#### 安装相关依赖
```bash
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get install -y curl openssh-server ca-certificates tzdata perl
```
> 根据自己需要选择是否安装邮件服务
> ```bash
> root@ubuntu:~# apt-get install -y postfix
> ```
#### 添加Gitlab仓库源
安装Gitlab时可选的安装方式有两种, 分别为 `企业版` 和 `社区版`。`企业版` 使用需要收费, 这里推荐大家安装 `社区版`。
```bash
# 添加Gitlab仓库源(社区版)
root@ubuntu:~# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh -o script.deb.sh
# 安装仓库源
root@ubuntu:~# bash script.deb.sh
```
执行结束后看到 `The repository is setup! You can now install packages.` 说明gitlab仓库的源已经成功添加了。
执行 `apt search gitlab-ce` 即可看到gitlab已经在镜像仓库中。
```bash
root@ubuntu:~# apt search gitlab-ce
Sorting... Done
Full Text Search... Done
gitlab-ce/focal 14.6.3-ce.0 amd64
  GitLab Community Edition (including NGINX, Postgres, Redis)
```
#### 安装Gitlab
安装时执行 `apt-get install gitlab-ce` 即可。如果需要安装其他版本，执行 `apt-get install gitlab-ce-{version}`
安装时会有一些配置需要注意, 参考如下
```bash
root@ubuntu:~# apt-get install gitlab-ce
```
看到如下标示说明安装成功
```bash

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/

```
#### 申请SSL证书(可选)
> gitlab默认使用的是 `http` 协议, 如果要更换为 `https` 协议可参考本小节进行域名和ssl证书的申请。

[阿里云申请域名](https://wanwang.aliyun.com/?spm=a2c1d.8251892.help.16.22b95b76NouX1e)

[阿里云证书申请](https://www.aliyun.com/product/cas?spm=a2c6h.12873639.0.0.22f745f8zTGrOe&source=5176.11533457&userCode=r3yteowb)

购买域名
搜索可用的域名, 根据自己的需要选择合适的域名。
购买证书
`登录阿里云SSL证书申请平台--->点击"SSL证书"--->选择"免费证书"--->选择"DV单域名证书[免费试用]"--->点击"立即购买/支付"`  
申请证书
`点击"SSL证书"--->点击"免费证书"--->点击"创建证书"--->点击"证书申请"`
证书申请结束后, 在创建证书页面点击`下载`, 选择`nginx`, 下载申请的SSL证书, 下载后将证书文件上传到Gitlab服务的`/etc/gitlab/ssl/` 目录下(如果不存在,可以创建一个目录* ssl目录会用来存放ssl证书文件 * )

#### 配置Gitlab
Gitlab的配置文件为 `/etc/gitlab/gitlab.rb`, 配置文件中提供了两种访问方式, 分别为`http` 和 `https`。

- http 方式访问
    - 修改配置文件中的 `external_url "http://gitlab.example.com"`, 可以将 "http://gitlab.example.com" 修改为本机的ip 或 域名. 例如: "http://10.10.10.10"
    - xxx

- https 方式访问
    - 修改配置文件中的 `external_url "http://gitlab.example.com"`, 可以将 "http://gitlab.example.com" 修改为本机的ip 或 域名. 例如: "https://10.10.10.10"
    ```bash
    external_url "http://gitlab.example.com"
    external_url "https://xxxx:443" # 修改后
    ```
    - 禁用 `Let's Encrypt`。找到 `letsencrypt['enable']`, 将其修改为 `letsencrypt['enable'] = false`, 去掉注释
    ```bash
    # letsencrypt['enable'] = nil
    letsencrypt['enable'] = false # 修改后
    ```
    - 拷贝ssl证书到`/etc/gitlab/ssl/` 目录下
    ```bash
    chmod 755 /etc/gitlab/ssl
    cp gitlab.example.com.key gitlab.example.com.crt /etc/gitlab/ssl/
    ```
    - 修改 nginx 相关配置
    ```bash
    # nginx['redirect_http_to_https'] = false 
    nginx['redirect_http_to_https'] = true # 修改后

    # nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.crt"
    # nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.key"
    nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.crt"     # 修改后
    nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.key"
    ```

