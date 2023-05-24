# 阿里云+FRP内网穿透详细教程

首先需要租用一台云服务器，为的是利用云端服务器的端口转发我们的内容

为了方便区分，我们将阿里云服务器称为云端服务器，本地服务器称为客户端服务器

frp要在云端和客户端都要安装，frps安装在云端服务器上，frpc安装在客户端服务器上

# 安装ssh

客户端服务器电脑安装ssh服务

```Bash
# 安装ssh
sudo apt-get install ssh
# 启用SSH服务
service ssh start
# 查看SSH服务运行状态
service ssh status
```

# frp安装

## 下载最新版本的frp

下载地址：https://github.com/fatedier/frp/releases

下载frp文件到两台服务器上，然后执行下面命令解压：

```Bash
tar -zxvf frp_*_linux_amd64.tar.gz
```

## 云端服务器配置frp

#### 1. 进入解压后的目录中，打开配置文件
   
   ```Bash
   vim frps.ini
   ```
#### 2. 将内容修改如下：
   
   ```Bash
   [common]
   # frp监听的端口，默认是7000，可以改成其他的
   bind_port = 7000
   # 授权码，客户端要跟云端相同
   token = limei210417
   # frp管理后台端口，请按自己需求更改，一般默认7500
   dashboard_port = 7500
   # frp管理后台用户名和密码，请改成自己的
   dashboard_user = wangtao
   dashboard_pwd = wangtao10010
   enable_prometheus = true
   
   # frp日志配置
   log_file = /var/log/frps.log
   log_level = info
   log_max_days = 3
   ```
#### 3. 为了使用systemd控制frp及开机自启等，我们需要配置`frps.service`文件，
   
   如Linux服务端上没有安装 `systemd`，可以先安装 `systemd`。
   
   ```Bash
   apt install systemd
   ```
#### 4. 使用文本编辑器，如 `vim` 创建并编辑 `frps.service` 文件。
   
   ```Bash
   vim /etc/systemd/system/frps.service
   ```
   
   写入内容
   
   ```Bash
   [Unit]
   # 服务名称，可自定义
   Description = frp server
   After = network.target syslog.target
   Wants = network.target
   
   [Service]
   Type = simple
   # 启动frps的命令，需修改为您的frps的安装路径
   ExecStart = /home/wt/frps -c /home/wt/frps.ini
   
   [Install]
   WantedBy = multi-user.target
   ```
#### 5.  之后便可以使用systemd命令管理frps
   
   启动frps `systemctl start frps`
   
   停止frps `systemctl stop frps`
   
   重启frps `systemctl restart frps`
   
   查看frp状态`systemctl status frps`

#### 6. 配置 frps 开机自启。
   ```Bash
   systemctl enable frps
   ```

#### 7. 启动云端服务器frps
   ```Bash
   sudo systemctl start frps
   ```

#### 8. 设置云服务器防火墙：
   ```Bash
   添加监听端口
      sudo   firewall-cmd --permanent --add-port=7000/tcp
   添加管理后台端口
      sudo   firewall-cmd --permanent --add-port=7500/tcp
      sudo   firewall-cmd --reload
      ubuntu用户使用ufw工具设置防火墙
   ```
   
   ```Bash
   sudo ufw allow 7000/tcp
   sudo ufw allow 7500/tcp
   ```
   
   查看防火墙状态`ufw status`
   启动防火墙 `ufw enable`
   关闭防火墙 `ufw disable`
   重启防火墙 `ufw reload`
   放行一个端口 `ufw allow 7000`
   添加放行一个协议端口策略 `ufw allow 7000/tcp`

#### 9. 云服务器上一般还有虚拟防护墙，这里也还要放行端口

​虚拟防护墙上要放行 tcp 7000 7500 5299 的端口

## 客户端配置frp

#### 1. 打开配置文件

```Bash
vim frpc.ini#(注意哦，不是frps.ini)
```

修改内容如下：

```Bash
# 客户端配置
[common]
server_addr = 服务器ip
 # 与frps.ini的bind_port一致
server_port = 7000
 # 与frps.ini的token一致
token = limei210417
tls_enable = true

# 配置ssh服务
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
 # 这个自定义，之后再ssh连接的时候要用，转发的接口
remote_port = 5299
```

#### 2. 客户端服务器防火墙开放端口

```Bash
sudo firewall-cmd --permanent --add-port=5299/tcp
sudo firewall-cmd --reload
```

#### 3. 云服务器虚拟防火墙出放行转发端口和ICMP协议

​如果ICMP不放行，ping 公网ip 不通，轻量级应用服务器不需要放行ICMP，只要能ping通就可以

#### 4. 启动frp

在frp_*_darwin_amd64目录下执行

```Bash
./frpc -c frpc.ini
```

#### 5. 设置客户端开机自行后台启动

##### 5.1 将frpc.service 复制到系统文件夹 /usr/lib/systemd/system/内

​frpc.service内容如下：

```Bash
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/bin/frpc -c /etc/frp/frpc.ini
ExecReload=/usr/bin/frpc reload -c /etc/frp/frpc.ini

[  Install  ]
WantedBy=multi-user.target
```
##### 5.2 按照frpc.service内的命令配置，将frpc启动文件何frpc.ini配置文件复制到指定路径

```Bash
mkdir   /etc/frp
cp   /root/software/frp/frpc.ini /etc/frp/frpc.ini
cp   /root/software/frp/frpc /usr/bin/frpc
```

##### 5.3 配置生效

```Bash
# 刷新配置文件
systemctl daemon-reload
# 设置开机自启动
systemctl enable frpc
# 查看状态
systemctl status frpc
```

##### 5.4 如果启动失败。可能是文件的权限问题。可以将frpc.service，frpc.ini，frpc这几个文件的权限设为 777

# 连接SSH

输入命令

```
ssh -p 5299 wt@*.*.*.*
```

看是否能够连接成功