#### 安装Cobbler依赖组件

```bash
yum install -y createrepo_c \
							 httpd \
							 dhcp \
							 httpd-devel \
							 xorriso \
							 mod_wsgi \
							 mod_ssl \
							 python-cheetah \
							 python-netaddr \
							 python-librepo \
							 python-schema \
							 PyYAML \
							 rsync \
							 syslinux \
							 tftp-server \
							 dnf-plugins-core 
python -m pip install sphinx coverage
```

#### 安装依赖源

```bash
yum install -y git \
							 make \
							 python3-devel \
							 openssl \
```

#### 下载cobbler

```bash
git clone https://github.com/cobbler/cobbler.git
cd cobbler
make install 
# make devintsall
```

#### 执行

```bash
cp /etc/cobbler/cobblerd.service /etc/systemd/system/cobblerd.service

systemctl enable cobblerd
```

#### 设置开机启动服务

```bash
systemctl enable httpd && systemctl restart httpd
systemctl enable dhcpd && systemctl restart dhcpd
systemctl enable tftp && systemctl restart tftp
systemctl enable rsyncd && systemctl restart rsyncd
```

