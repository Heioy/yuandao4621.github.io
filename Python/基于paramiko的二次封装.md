`paramiko` 是 Python 中的一个用来连接远程主机的第三方工具，通过使用 `paramiko` 可以用来代替以 ssh 连接到远程主机执行命令。

`paramiko` 模块提供了两个核心组件，分别是 `SSHClient` 和 `SFTPClient`。前者用于在远程主机上执行命令，是对于 ssh 会话的封装；后者用于对资源上传下载等操作，是对 sftp 的封装。

`paramiko` 模块提供了一些SSH核心组件

- `Channel`: 建立 ssh 连接后，paramiko会调用底层的 `channel` 类来打开一个socket连接，后续发送的命令会通过 `channel` 进行发送，当接收到命令的返回结果(标准输出或标准错误)后，结果又将通过 `channel` 发送到连接的客户端。
- `Client`: 建立的 ssh 会话的客户端。通过客户端可以去连接多个远程主机，并将执行的命令通过 `client.exec_command()` 方法发送到远程主机。
- `Message`: 一个用来将传输过程中的字符、整形、布尔及长类型的字符组合进行编码组成的字节流信息。
- `Packetizer`: `Implementation of the base SSH packet protocol.`
- `Transport`: 封装了一个加密的会话，调用该会话时会创建一个流式隧道(通常称为 `channel`)

封装的具体逻辑，为了保持功能的纯粹性，仅通过 `paramiko` 实现客户端的的建立及命令的发送。

以下为实现逻辑

```python
#!/usr/bin/env python3
# coding=utf-8

import sys
import socket
import paramiko
from paramiko.ssh_exception import NoValidConnectionsError, AuthenticationException

class Connect(object):
    
    def __init__(self, username: str, 
                 password: str, 
                 port: int = 22,
                 hostname: str,
                 pkey: str = None,
                 look_for_keys: bool = True,
                 allow_agent: bool = True,
                 timeout: float = None
                ):
        self.hostname = hostname
        self.username = suername
        self.port = port
        self.password = password
        self.pkey = pkey
        self.allow_agent = allow_agent
        self.look_for_keys = look_for_keys
        self.timeout = timeout
        
    def _connect(self):
        self.client = SSHClient()
        self.client.load_system_host_keys()
        self.client.set_missing_host_key_policy(paramiko.AutoAddpolicy())
        try:
            self.client.connect(hostname=self.hostname,
                               port=self.port,
                               username=self.username,
                               password=self.password,
                               pkey=self.pkey,
                               allow_agent=self.allow_agent,
                               look_for_keys=self.look_for_keys
                               )
        except BadHostKeyException as badKey:
            msg = "Receive a bad key"
            sys.exit()
        except AuthenticationException as auth:
            msg = "Invalid username or password"
            sys.exit()
        except SSHExpection as ssh:
            msg = "Establish ssh session error"
            sys.exit()
        exception scoket.error as sock:
            msg = "Connecting has a socket error"
            sys.exit()

        
    def run(self, command: str):
        stdin, stdout, stderr = self.client.exec_command(command, timeout=self.timeout)
        stdin.close()
        stdout.flush()
        
        try:
            output = stdout.read()
            err_msg = stderr.read()
            
            output = output.decode("utf-8") if isinstance(output, bytes) else output
            err_msg = err_msg.decode("utf-8") if isinstance(err_msg, bytes) else err_msg
            return output, err_msg
            
        except socket.timeout:
            raise(f"Exec Command {command} timeout")
            
    def close(self):
        self.client.close()
               
```

将ssh会话保存到会话池中

```python
import socket
import paramiko
from collections import deque
from paramiko import SSHExpection
from paramiko.ssh_exception import NoValidConnectionsError, AuthenticationException
from eventlet import pools


class SSHPool(pools.Pool):
    
    """创建 SSH 对象池 """
    _pool = deque()
    
    def __init__(self,
                 ip: str,
                 username: str,
                 password: str = None,
                 port: int = 22,
                 privatekey: str = None,
                 timeout: float = None,
                 **kwargs
                ):
        self.ip = ip
        self.port = port
        self.password = password
        self.username = username
        self.privatekey = privatekey
        self.timeout = timeout
        super(SSHPool, self).__init__(**kwargs)
        
    def _create(self):
        try:
            client = paramiko.SSHClient()
            client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            
            if self.password:
                ssh.connect(self.ip, 
                            port=self.port,
                            username=self.username,
                            password=self.password,
                            timeout=self.timeout
                           )
            elif self.privatekey:
                if isinstance(self.privatekey, paramiko.rsakey.RSAKey):
                    key = self.privatekey
                else:
                	keyfile = os.path.expanduser(self.privatekey)
                    key = paramiko.RSAKey.from_private_key_file(keyfile)
                
                ssh.connect(self.ip,
                            port=sel.port,
                            username=self.username,
                            look_for_keys=True,
                            pkey=key,
                            timeout=self.timeout
                           )
            else:
                raise SSHExpection("Invalid username or password")
                
            
            # Paramiko by default sets the socket timeout to 0.1 seconds,
            # ignoring what we set through the sshclient. This doesn't help for
            # keeping long lived connections. Hence we have to bypass it, by
            # overriding it after the transport is initialized. We are setting
            # the sockettimeout to None and setting a keepalive packet so that,
            # the server will keep the connection open. All that does is send
            # a keepalive packet every ssh_conn_timeout seconds.
            
            if self.timeout:
                transport = ssh.get_transport()
                transport.sock.settimeout(None)
                transport.set_keepalive(self.timeout)
            return ssh
        
        except socket.timeout:
            raise SSHExpection("Connect timeout")
        except NoValidConnectionsError as novalid:
            raise SSHExpection("Connect valid failed")
        except AuthenticationException as auth:
            raise SSHExpection("Invalid username or password")
        except Exception as e:
            raise SSHExpection("An exception happened")

    def get(self):
        """从会话池中获取 SSH 会话
        1. 返回一个存活的会话
        2. 不存在的会话或已经失效的会话, 会新建一个会话并返回该会话
		"""
        conn = super(SSHPool, self).get()
        if conn:
            if conn.get_transport().is_active():
                return conn
            else:
                conn.close()
        return self._create()
    
    #def put(self, ssh):
    #    """将 SSH 会话添加到会话池中"""
    #    conn = super(SSHPool, self).get()
    #    if ssh not in self._pool:
    #        self._pool.append(ssh)
            
    def remove(self, ssh):
        """关闭 SSH 会话,并将其从会话池中移除"""
        ssh.close()
        ssh = None
        if ssh in self.free_items:
            self.free_items.pop(ssh)
        if self.current_size > 0:
            self.current -= 1
        
```



