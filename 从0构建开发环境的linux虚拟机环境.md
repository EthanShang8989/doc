# 从0构建开发环境的linux虚拟机环境

## 先决条件

- 使用Ubuntu镜像安装好了附GUI的虚拟机。（这里需要gui主要是为了配置v2rayA代理，使用命令行的话不是很方便）
- 拥有v2ray的订阅连接。

## 可能的网络问题

我在搭建的时候选择的是NAT，但是虚拟机连接不了网络。



sudo dhclient ens33
sudo ifconfig ens33

sudo dhclinet ens33 手动获取ip
sudo ifconfig ens33 来查看

```
ethanshang@ubuntu2:~/Desktop$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 00:0c:29:85:ad:03 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
```

`ens33` 网络接口处于 `DOWN` 状态，这表明网络接口未启用。我们需要将其启用并配置为获取IP地址。

### 1. 启用网络接口

首先，启用 `ens33` 网络接口：

```
sudo ip link set ens33 up
```

### 2. 配置网络接口以使用DHCP

使用 DHCP 为网络接口分配 IP 地址：

```
sudo dhclient ens33
```

### 3. 检查网络接口状态

再次检查网络接口状态，确保它已启用并获取 IP 地址：

```
ethanshang@ubuntu2:~/Desktop$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:85:ad:03 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.88.128/24 brd 192.168.88.255 scope global dynamic ens33
       valid_lft 1797sec preferred_lft 1797sec
    inet6 fe80::20c:29ff:fe85:ad03/64 scope link 
       valid_lft forever preferred_lft forever
```

## 配置apt国内源



## 安装常见工具

```
sudo su
apt install build-essential -y
apt install openssh-server -y
sudo apt install git curl cmake make gcc lld pkg-config libssl-dev libclang-dev libsqlite3-dev g++ protobuf-compiler vim tmux net-tools htop
```

### rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

vim /etc/profile
#在最后添加
# Load Rust environment variables
if [ -f "$HOME/.cargo/env" ]; then
    . "$HOME/.cargo/env"
fi

source /etc/profile
```

### go

官网是https://go.dev/dl/
选择自己对应的

```
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
```

将下载的二进制包解压至 /usr/local目录。

```
tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

添加环境变量

```
vim /etc/profile
#添加
export PATH=$PATH:/usr/local/go/bin

source /etc/profile
```

验证装好了

```
 go version
```

### nvm

官方git是https://github.com/nvm-sh/nvm

```

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

vim /etc/profile
#添加下面的内容
# Load NVM (Node Version Manager)
export NVM_DIR="$HOME/.nvm"
if [ -s "$NVM_DIR/nvm.sh" ]; then
    . "$NVM_DIR/nvm.sh"
fi
if [ -s "$NVM_DIR/bash_completion" ]; then
    . "$NVM_DIR/bash_completion"
fi

source /etc/profile

nvm --version #验证下有没有安装好
nvm install 20.11.0

node -v
npm -v
#安装yarn（1.x.x)
npm install -g yarn
#安装高版本的yarn 4.2.2
corepack enable
corepack prepare yarn@stable --activate
```

### 安装 OpenSSH Server

```
sudo apt install openssh-server -y
```

#### 启动并启用 SSH 服务

启动 SSH 服务并设置为开机自启动：

```
sudo systemctl start ssh
sudo systemctl enable ssh
```

#### 确认 SSH 服务运行

检查 SSH 服务是否正常运行：

```
sudo systemctl status ssh
```







## 安装v2rayA

https://v2raya.org/docs/prologue/installation/debian/

进行基本设置

**透明代理/系统代理** 选择 **启用：不进行分流**。

**透明代理/系统代理实现方式** 选择 redirect。

启用 **开启 IP 转发**和**开启端口转发**。

设置终端代理

```
vim /etc/profile
export http_proxy="http://127.0.0.1:20171"
export https_proxy="http://127.0.0.1:20171"
export all_proxy="socks5://127.0.0.1:20170"
```



## 共享文件夹

有一些奇妙的原因需要每次启动虚拟机都需要配置下

```
sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

这样就可以在/mnt/hgfs看到自己的共享文件夹了