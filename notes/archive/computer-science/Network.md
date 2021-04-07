---
title: "计算机网络"
date: 2015-07-05T08:57:52+10:00
draft: true
categories: ["Computer Science"]
---

# 计算机网络

![cwnd](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/计算机网络体系结构.png)

- 作用及协议

|分层|作用|协议|
|---|---|---|
|物理层|通过媒介传输比特，确定机械及电气规范（比特 Bit）|RJ45、CLOCK、IEEE802.3（中继器，集线器）|
|数据链路层|将比特组装成帧和点到点的传递（帧 Frame）|PPP、FR、HDLC、VLAN、MAC（网桥，交换机）|
|网络层|负责数据包从源到宿的传递和网际互连（包 Packet）|IP、ICMP、ARP、RARP、OSPF、IPX、RIP、IGRP（路由器）|
|运输层|提供端到端的可靠报文传递和错误恢复（ 段Segment）|TCP、UDP、SPX|
|会话层|建立、管理和终止会话（会话协议数据单元 SPDU）|NFS、SQL、NETBIOS、RPC|
|表示层|对数据进行翻译、加密和压缩（表示协议数据单元 PPDU）|JPEG、MPEG、ASII|
|应用层|允许访问OSI环境的手段（应用协议数据单元 APDU）|FTP、DNS、Telnet、SMTP、HTTP、WWW、NFS|

## 链路层

### MAC 地址

- MAC 地址是链路层地址，长度为 6 字节（48 位），用于唯一标识网络适配器（网卡）；一台主机拥有多少个网络适配器就有多少个 MAC 地址

### 局域网
局域网是一种典型的广播信道，主要特点是网络为一个单位所拥有，且地理范围和站点数目均有限。

主要有以太网、令牌环网、FDDI 和 ATM 等局域网技术，目前以太网占领着有线局域网市场。

可以按照网络拓扑结构对局域网进行分类：

## 状态码