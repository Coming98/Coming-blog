---
title: ICMP Protocol and Security
mathjax: false
date: 2021-12-05 19:11:02
summary: ICMP 协议安全分析
categories: Protocol Security
tags:
  - security
---
# ICMP

Internet Control Message Protocol，Internet 控制报文协议，用于在IP主机、路由器之间传递控制消息
> 控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息

![image-20211127135407400](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127135407400.png)

## 报文种类

攻击也是从正常的所提供的功能出发：

![image-20211007133301183](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211007133301183.png)

# ICMP 攻击

## Ping of death

又名 死亡之 ping

Backdrop：当一个 IP 包的长度超过以太网帧的最大尺寸时，包就会被分片，作为多个帧来发送。接收端的机器提取各个分片，并重组为一个完整的 IP 包；

Attack Point ：IP 协议规范规定了一个 IP 包的最大尺寸（65535）而大多数的包处理程序又**假设**包的长度不会超过这个最大尺寸。因此，包重组代码所分配的内存区域也不会超过这个尺寸

### 攻击方式

Attack：缓存溢出（Buffer Overflow）攻击；构造超过这个最大尺寸的包，包当中的额外数据就会被写入其他正常区域，使得系统进入非稳定状态
> 在防火墙一级对这种攻击进行检测是相当难的，因为每个分片包看起来都很正常，在内部重组后却出事了
> 
### Demo

ping 指令将发送一个 ICMP 回声请求消息给目的地并报告是否收到所希望的 ICMP echo，用于检查网络是否通畅或者网络连接速度的命令
> Tips ：ping 只是利用这个漏洞的工具之一

```shell
ping -l 65500 192.168.1.1 -t
# 最终的大小还有加上 ICMP(8) 与 IP(20-60)
```

### 防御

对操作系统打补丁，使内核将不再对超过规定长度的包进行重组

## ICMP Smurf 攻击

又名反射放大攻击，利用 IP 欺骗和 ICMP 回应包引起目标主机网络阻塞，实现 DoS 攻击

### 攻击原理

在构造数据包时将源地址设置为被攻击主机的地址，而将目的地址设置为广播地址，于是大量 ICMP echo 的回应包 ICMP reply 被发送给被攻击主机，使其因网络阻塞而无法提供服务

### 防御

防火墙源地址检查过滤

## ICMP Redirect 攻击

Backdrop：当路由器检测到主机在启动时具有一定的路由信息（但不一定是最优的）并检测到 IP 数据报经非最优路由传输，就通过 **ICMP 重定向报文**通知主机去往该目的地的最优路径；其目的旨在保证主机拥有动态的、既小且优的路由表；

### 攻击原理

![image-20211006104322321](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211006104322321.png)

Attack：攻击者通过发送 ICMP 重定向报文可以在受害者主机路由表中添加一条到达特定主机的路由信息，使得受害者发往特定主机的数据包被发往攻击者主机，攻击者进而实施监听、欺骗等攻击行为；

核心：欺骗路由表。通过添加特定路由表实施

### 限制

1、ICMP 重定向机制只能在同一网络的路由器与主机之间使用；

2、ICMP 重定向攻击一次只能指定一个目的地址；

3、指定的地址路由上必须是直达的；

4、重定向包必须来自去往目标的当前路由；

5、重定向包不能通知主机用自己做路由（需要合谋者）；

6、受害者和攻击者要处于同一网络环境中；

### 防御

修改系统和防火墙配置，拒绝接收 ICMP 重定向报文