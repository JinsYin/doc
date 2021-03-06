# ARP 协议

地址解析协议（Address Resolution Protocol，缩写：**ARP**）是通过 `IP 地址`来定位 `MAC 地址` 的网络传输协议（即已知 `IP 地址` 求 `MAC 地址`）。

在 OSI 模型中，ARP 协议属于 **数据链路层**（主流）；而在 TCP/IP 模型中，ARP 协议属于 **网络层**。

## 概述

在 `以太网协议` 中规定，同一局域网中的一台主机要和另一台主机进行直接通信，必须要知道目标主机的 MAC 地址。而在 `TCP/IP协议` 中，网络层和传输层只关心目标主机的IP地址。这就导致在以太网中使用 IP 协议时，数据链路层的以太网协议接到上层 IP 协议提供的数据中，只包含目的主机的 IP 地址。于是需要一种方法，根据目的主机的 IP 地址，获得其 MAC 地址。这就是 ARP 协议要做的事情。所谓地址解析（address resolution）就是主机在发送 `帧` 前将目标 IP 地址转换成目标 MAC 地址的过程。

![ARP](.images/arp.png)

## 工作流程

局域网中，如果知道了 IP 地址，不知道 MAC 地址该怎么办？靠 “吼”（`广播`）。

![ARP Workflow](.images/arp-workflow.png)

1. 源主机查看本地 ARP 缓存表，如果没有对应的 MAC 地址就广播 ARP 请求
2. 目标主机收到请求后，进行 ARP 应答
3. 为了避免每次都用 ARP 请求，源主机会进行 ARP 缓存；由于机器可能上下线，IP 也可能会变，所以缓存的 MAC 地址也有过期时间

ARP 请求和应答的报文格式：

![ARP 报文格式](.images/arp-packet-format.png)

## 命令

* 查看本地 ARP 缓存表

```sh
$ arp -v # arp -a
Address                  HWtype  HWaddress           Flags Mask            Iface
172.17.0.3               ether   02:42:ac:11:00:03   C                     docker0
192.168.16.101                   (incomplete)                              eth1
172.17.0.4               ether   02:42:ac:11:00:04   C                     docker0
192.168.16.1             ether   f4:ec:38:1a:39:f8   C                     eth1
172.17.0.2               ether   02:42:ac:11:00:02   C                     docker0
Entries: 5  Skipped: 0  Found: 5
```

* 删除 IP-MAC 映射

```sh
# 对应的 HWaddress 会变成 incomplete
$ sudo arp -d 172.17.0.3
```

* 向目标主机发送 ARP 请求

```sh
# 可能收不到响应
$ arping -I eth1 192.168.16.101

# 如果收到了响应会将 IP-MAC 映射缓存下来
$ arping -I docker0 172.17.0.3
ARPING 172.17.0.3 from 172.17.0.1 docker0
Unicast reply from 172.17.0.3 [02:42:AC:11:00:03]  0.555ms
Unicast reply from 172.17.0.3 [02:42:AC:11:00:03]  0.544ms
Unicast reply from 172.17.0.3 [02:42:AC:11:00:03]  0.538ms
```

## 参考

* [Linux ARP 表相关操作](https://blog.csdn.net/letterwuyu/article/details/78277243)
