---
title: 搭建WireGuard 
key: 20200416
tags: VPN
---

## 一、动机

* 尝试一下这个号称next generation的VPN工具
* 为现有方案做一个备选

<!--more-->

## 二、参考 

* [非官方文档](https://github.com/pirate/wireguard-docs) 
* [官方文档](https://www.wireguard.com/)
* [来自于linode的 - 如何在ubuntu上设置wireguard （这篇很棒，因为我的VPS就是搭建在linode）](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/)
* [WireGuard安装配置教程](https://www.lixh.cn/archives/2134.html)

## 三、环境及版本

WireGuard发展很迅速，我的这次尝试基于以下版本：

* 服务器端(ubuntu 18.04 LTS)
* 客户端环境(MacOS 10.15.4 (Catalina))

安装成功后可以通过下面的命令查看wireguard的版本
```shell
$ sudo wg version
```
或运行之后
```shell
$ sudo wg show
```

得到如下信息：
> wireguard-tools v1.0.20200319 - https://git.zx2c4.com/wireguard-tools/
或
> wireguard-go version 0.0.20200320

## 四、安装WireGuard服务器

### 添加WireGuard仓库到apt包管理器中

```shell
$ sudo add-apt-repository ppa:wireguard/wireguard 
```

### 安装wireguard

```shell
$ sudo apt-get update
$ sudo apt-get install wireguard
```

### 配置wireguard服务端 - 生成服务端私钥和密钥

```shell
$ cd /etc/wireguard
$ umask 077
$ wg genkey | tee privatekey | wg pubkey > publickey
```

### 配置wireguard服务端 - 服务端配置文件 (/etc/wireguard/wg0.conf)

```shell
# 定义服务器端的虚拟网卡
[Interface]
# 虚拟网卡的IP地址 （我曾经在这个地方犹豫了一会儿，不知道什么网段为好。这个其实要看你准备组的虚拟网要有多少设备）
Address = 10.0.0.1/24, fd86:ea04:1115::1/64
# 设为true表示配置文件在有新的peer接入时自动更新
SaveConfig = true
# 在虚拟网卡启动后的动作。这里将接受所有的客户端共享服务端的IP地址，并写入路由表中
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# 在虚拟网卡断开后的动作。这里将客户端共享服务端IP地址的规则从路由表中删除
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# 服务端的通讯端口，后面客户端配置的时候需要用到
ListenPort = 51820
# 上面产生的私钥
PrivateKey = <服务器私钥>
```

### 配置wireguard服务端 - 配置防火墙并确认

```shell
$ sudo ufw allow 22/tcp
$ sudo ufw allow 51820/udp
$ sudo ufw enable
$ sudo ufw status verbose
```

### 启动服务端wireguard

```shell
$ sudo wg-quick up wg0
```

### 服务端配置 - 允许IP转发

```shell
$ vi /etc/sysctl.conf

# 找到下面这一行，去掉注释来允许IP转发
net.ipv4.conf.default.rp_filter=1

# 退出编辑后，通过下面命令使其生效
$ sysctl -p
$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

> :warning: 这个如果不做的话，你会发现即便wireguard建立的虚拟网可以ping通，但是客户端无法通过服务端上网

## 五、安装客户端

### 在MacBookPro上安装wireguard

```shell
$ sudo brew install wireguard-tools
```

### 客户端配置 - 生成密钥

```shell
$ cd /usr/local/etc/
$ umask 077
$ mkdir wireguard
$ cd wireguard
$ wg genkey | tee privatekey | wg pubkey > publickey
```

### 客户端配置 - 配置文件 (/usr/local/etc/wireguard/wg0.conf)

```shell
[Interface]
PrivateKey = <客户端私钥>
# 这里的客户端IP需要跟服务器端的IP同一网段，服务器端的网卡地址是10.0.0.1
Address = 10.0.0.10/32
DNS = 10.0.0.1,8.8.8.8,1.1.1.1

[Peer]
PublicKey = <服务端公钥>
Endpoint = <服务端公网IP>:51820
# 0.0.0.0/0为允许所有IP地址通讯，如果是多个客户端的话，可以省去一个个配置之繁琐
AllowedIPs = 0.0.0.0/0
# 每21秒检查一下连接。如果IP有变化，也是通过这个进行自动更新
PersistentKeepalive = 21
```

### 启动客户端wireguard

```shell
$ sudo wg-quick up wg0
```

这时应该可以ping通服务端的虚拟IP，及其它网站了。如果不行尝试重启一下wireguard。还不行的话，检查一下防火墙和IP转发的配置。