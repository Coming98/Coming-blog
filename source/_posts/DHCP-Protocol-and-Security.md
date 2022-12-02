---
title: DHCP Protocol and Security
mathjax: false
date: 2021-12-06 09:22:48
summary: DHCP 协议安全分析
categories: Protocol Security
tags:
  - security
---

# DHCP

Dynamic Host Configuration Protocol，动态主机配置协议，提供动态 IP 配置，它增强了 BOOTP，并与 BOOTP 向后兼容

## BOOTP

引导程序协议，Bootstrap protocol，BOOTP，使一个客户工作站能够用一个最小的 IP 堆栈进行初始化，并向 BOOTP 服务器请求它的 IP 地址、网关地址以及名字服务器的地址，是 DHCP 协议的前身

### BOOTP 工作过程

1、客户确定它自己的硬件地址，地址一般在硬件的 ROM 内

2、BOOTP 客户在一个 UDP 数据报中把它的硬件地址发送到服务器

3、服务器接收数据报，并在它的配置文件中查找客户的硬件地址，这个文件包含客户的 IP 地址

## DHCP 功能

1、保证任何 IP 地址在同一时刻只能由一台 DHCP 客户机所使用

2、DHCP 应当可以给用户分配永久固定的 IP 地址

3、DHCP 应当可以同用其他方法获得 IP 地址的主机共存（如手工配置 IP 地址的主机）

4、DHCP 服务器应当向现有的 BOOTP 客户端提供服务（向下兼容）

## DHCP 数据包

DHCP 属于应用层协议，建立在 UDP 协议之上，因此数据包形式如下：

![DHCP 数据包](https://raw.githubusercontent.com/Coming98/pictures/main/20211206090640.png)

1、最外层的以太网标头（ MAC 帧）源 MAC 地址为客户 MAC 地址，目的 MAC 地址未知，因此填入广播 MAC `FF-FF-FF-FF-FF-FF`

2、内部是 IP 标头，源 IP 地址未知，这里设置为 `0.0.0.0` 目的 IP 也未知，这里设置为 `255.255.255.255`
> `255.255.255.255` 在 [IPv4 Protocol and Security](https://coming98.github.io/Coming-blog/2021/12/05/ipv4-protocol-and-security/#toc-heading-7) 我们提到过这是受限广播地址，应用于 DHCP 中

3、UDP 标头，源端口固定为 68，目的端口固定为 67

### 数据包发送过程

1、广播发送，子网内每个主机都能收到该包

2、根据 MAC 地址无法判断目的地是谁，因此查看 IP 信息(`0.0.0.0` -> `255.255.255.255`) DHCP 服务器就能意识到这是 DHCP DISCOVER 包，其它主机将丢弃该包

3、DHCP 分配好 IP 地址后将返回一个 DHCP OFFER 包，源 MAC 与目的 MAC 都明确了，源 IP 为 DHCP 服务器的 IP，目的 IP 未知，此时仍设置为 `255.255.255.255` ，源端口为 67，目的端口为 68，分配给请求端的 IP 地址和本网络的具体参数则包含在 Data 部分
> 通过 MAC 寻址

## DHCP 工作过程

1、客户在它的本地物理子网上广播一个 DHCP DISCOVER 消息

2、每个服务器可以用一个 DHCP OFFER 消息作出响应，这个消息包含一个可用的网络地址和其他配置选项

3、客户从一个或多个服务器接收到一个或者多个 DHCP OFFER 消息，选择一个

4、服务器从客户接收 DHCP REQUEST

5、客户接收到带有配置参数的 DHCP ACK 消息

6、客户通过发送一个 DHCP RELEASE 消息给服务器，它可以选择不再继续租用地址

## DHCP 欺骗

1、攻击者首先发起大量的 DHCP 请求（伪造大量的 DHCP 请求包），伪装成 DHCP 客户端

2、很快正常的 DHCP 服务器的 IP 地址被消耗殆尽，DHCP服务器停止为其他 DHCP 客户端分配 IP 地址

3、攻击者在网络架设自己的 DHCP 服务器，为真正需要 IP 地址的客户端分配 IP 地址

4、攻击者的 DHCP 服务器为客户端分配 IP 参数，这些参数里可能包含人为错误设置的参数
> DNS 可以指向恶意 DNS 服务器
> 网关指向攻击者控制的路由器，实现中间人攻击

## DHCP 防御

1、可以通过建立一张包含有用户 MAC 地址、IP 地址、租用期、VLAN ID、交换机端口等信息的一张表，限制同一端口下的数目，限制 MAC等

2、将交换机的端口分为可信任端口和不可信任端口，当交换机从一个不可信任端口收到 DHCP 服务器的报文时，比如 DHCP OFFER 报文、DHCP ACK报文、DHCP NAK 报文，交换机会直接将该报文弃

# Refs

[知乎 - wuxinliulei - DHCP属于OSI的哪一层？](https://www.zhihu.com/question/23337642?sort=created)