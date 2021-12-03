---
title: 隧道与IPsec VPN
mathjax: false
date: 2021-10-26 16:26:22
summary: Packet Tracer 实现隧道与IPSecVPN
categories: Protocol Security
tags:
  - security
  - experiment
---

# Prepare

虚拟专用网络，Virtual Private Network，VPN

1、在内部网络各个子网之间建立点对点 IP 隧道，解决由互联网互联的内部网络各个子网之间的通信问题

2、通过在点对点 IP 隧道两端之间建立安全关联，解决内部网络各个子网之间的安全通信问题

3、通过 VPN 接入技术解决远程终端访问内部网络资源的问题

# 点对点 IP 隧道实验

## 1、路由器添加接口卡

**需要对公网路由器 R4，R5，R6 添加额外的接口卡**

a、点开路由器的配置界面，先关闭路由器

![image-20211017173557977](https://gitee.com/Butterflier/pictures/raw/master/image-20211017173557977.png)

b、左侧选择 NM - 2FE2W 将其拖动进指定的插槽

![image-20211017173647869](https://gitee.com/Butterflier/pictures/raw/master/image-20211017173647869.png)

c、再次启动路由器

## 2、配置外网路由器

主要配置 R4，R5，R6 路由器全球 IP 地址与子网掩码

Tips：配置时推荐打开接口：`no shut`

![image-20211017184925520](https://gitee.com/Butterflier/pictures/raw/master/image-20211017184925520.png)

最终配置结果如图所示：

![image-20211017194622660](https://gitee.com/Butterflier/pictures/raw/master/image-20211017194622660.png)

## 3、配置内网路由器

主要配置 R1，R2，R3 路由器连接公共网络接口与内网接口的 IP 与 子网掩码

![image-20211017190327310](https://gitee.com/Butterflier/pictures/raw/master/image-20211017190327310.png)

## 4、OSPF 配置

> OSPF，开放最短路径优先，其路由过程如下：
>
> 1、寻找邻居
>
> 2、建立邻接关系
>
> 3、链路状态信息传递
>
> 4、计算路由
>
> 因此对 R1-6 路由器配置 OSPF 后，能够自动寻找邻居（路由器），交换路由信息，保证路由器之间的连通性

以 R2 为例，配置过程如下：将 192.1.2.0 / 24 配置为一个 OSPF 区域（或者是自治系统 AS）

![image-20211017192234046](https://gitee.com/Butterflier/pictures/raw/master/image-20211017192234046.png)

最终整体的初始架构如下：

![image-20211017194711614](https://gitee.com/Butterflier/pictures/raw/master/image-20211017194711614.png)

以 R4 的路由表为例，我们查看一下是否配置正确：其中 O 表示 OSPF

![image-20211017194746959](https://gitee.com/Butterflier/pictures/raw/master/image-20211017194746959.png)

## 5、隧道配置

对 R1，R2 与 R3 配置隧道

```shell
interface tunnel 1 # 创建编号为 1 的 IP 隧道接口并进入配置模式
# 为隧道接口配置私有 IP 地址 192.168.4.1 和子网掩码 255.255.255.0
# 因为我们发送的
ip address 192.168.4.1 255.255.255.0 

# 指定隧道源端的公网 IP （与路由器接口绑定）
tunnel source FastEthernet 0/1

# 指定隧道目的端的公网 IP
tunnel destination 192.1.2.1
```

![image-20211017202015250](https://gitee.com/Butterflier/pictures/raw/master/image-20211017202015250.png)

## 6、RIP 配置

为什么要进行 RIP 配置，为了能使得 R1-3 发现（直连）隧道的私有 IP 地址

> RIP，Routing Information Protocol，是一种距离向量协议

以 R1 为例

```shell
router rip
# 下方表示该路由器直连的网段
network 192.168.1.0 # 自身内网私有地址
network 192.168.4.0 # Tunnel 1 的私有地址
network 192.168.5.0 # Tunnel 2 的私有地址
```

## 7、路由表分析

以 R1 为例

![image-20211017202946682](https://gitee.com/Butterflier/pictures/raw/master/image-20211017202946682.png)

C 类路由表项，表示直接连接的网络路邮项目，有 外部网段 `192.1.1.0` 走接口 `0/1`，有内部网段 `192.168.1.0` 走接口 `0/0` ，还有隧道 1 的 `192.168.4.0` 隧道源端，隧道 2 的 `192.168.5.0` 隧道源端；

O 类路由表项，表示通往公共子网的传输路径（非直连），对于 R1 除了 `192.1.1.0` 网段直连外 `2.0, 3.0, 4.0, 5.0, 6.0` 均是通过OSPF 配置得到的动态路由项

R 类路由表项，表示通往内部子网的传输路径（非直连），有 RIP 动态路由习得，对于 R1 除了 `192.1.1.0` 这一内部子网直连外，还能习得 `192.168.2.0, 192.168.2.0` 这两个，这是通过 RIP 配置的 `192.168.4.0, 192.168.5.0`  隧道直连得到；看到底部还有一个 `192.168.6.0` 这是 R3 那边配置隧道和 RIP 后，R1 习得的结果。

## 8、配置 PC 与 Server

| HOST    | IP          | Gateway       |
| ------- | ----------- | ------------- |
| PC0     | 192.168.1.1 | 192.168.1.254 |
| PC1     | 192.168.1.2 | 192.168.1.254 |
| PC2     | 192.168.2.1 | 192.168.2.254 |
| PC3     | 192.168.2.2 | 192.168.2.254 |
| PC4     | 192.168.3.1 | 192.168.3.254 |
| PC5     | 192.168.3.2 | 192.168.3.254 |
| Server0 | 192.168.1.3 | 192.168.1.254 |
| Server1 | 192.168.2.3 | 192.168.2.254 |
| Server2 | 192.168.3.3 | 192.168.3.254 |

## 9、隧道测试

使用 PC0 ping PC4 `192.168.3.1`

在内网中 先给交换机，交换机给网关，没有问题

![image-20211017205508453](https://gitee.com/Butterflier/pictures/raw/master/image-20211017205508453.png)

进入公网，需要隧道 2 配合换上公网 IP `192.1.1.1`

![image-20211017223314685](https://gitee.com/Butterflier/pictures/raw/master/image-20211017223314685.png)

![image-20211018085748762](https://gitee.com/Butterflier/pictures/raw/master/image-20211018085748762.png)

真实的行动轨迹

![image-20211017223418362](https://gitee.com/Butterflier/pictures/raw/master/image-20211017223418362.png)

到了 Router 3 后：

![image-20211017223512176](https://gitee.com/Butterflier/pictures/raw/master/image-20211017223512176.png)

Tips：如果没有这条 Tunnel2 私有地址（`192.168.1.1 - 192.168.3.1`）是无法在公网上路由的，但是他应该能找寻到 Tunnel1 -> Tunnel3 -> PC3 这样一条路径，我们来尝试下：

## 10、智能路由测试

关闭 Tunnel2 很简单：在 R1 与 R3 中输入如下命令，重启 Packet Tracer 后，R1 和 R3 就无法发现Tunnel 2 了。

```shell
router rip
no network 192.168.5.0
```

随后我们尝试 PC0 Ping PC3，第一个关键节点是 R1 输出的 IP 包：可以看出他的确没有识别到 Tunnel 2，但是通过路由器之间互学习（路由表交换），R1 能够学习到通往 192.168.3.0/24 的路由项。

![image-20211018091036921](https://gitee.com/Butterflier/pictures/raw/master/image-20211018091036921.png)

R1 的部分路由表，192.168.3.0 /24 走 Tunnel1 即可

![image-20211018091244762](https://gitee.com/Butterflier/pictures/raw/master/image-20211018091244762.png)

随后能在公网上顺利到达 192.1.2.1 （R2），R2 能够通过 Tunnel 3，传给 R3，进而到达 PC3

![image-20211018091415881](https://gitee.com/Butterflier/pictures/raw/master/image-20211018091415881.png)

# IPSec

Target：通过 IPSec 实现经过点对点 IP 隧道传输内容的保密性以及完整性，分两个阶段完成，第一个阶段建立点对点 IP 隧道两端之间的安全传输通道；第二阶段是建立点对点 IP 隧道两端之间的双向安全关联。

具体包括：

a、建立安全传输通道：完成身份鉴别协议、密钥交换算法和加密/解密算法等协商过程

b、建立安全关联：确认安全协议、加密算法、HMAC 算法等

> 选择 AH 为安全协议时不需要选择加密算法

c、配置分组过滤器：筛选出需要经过 IPSec 安全关联传输的一组 IP 分组，只有与实现内部子网之间通信相关的 IP 分组才需要经过 IPSec 安全关联进行传输

## 相关命令解释

### 安全策略

协商安全传输通道所需的安全策略，然后协商安全策略中的身份鉴别机制，加密算法，hash 算法

```shell
# 两端可定义多个安全策略编号越小表示的优先级越高
crypto isakmp policy 1  # 定义编号和优先级为 1 的安全策略并进入其配置模式

# 可选的鉴别机制有很多：基于 RSA 的数值签名等
authentication pre-share # 指定身份鉴别机制 pre-share 表示共享密钥鉴别机制

# 支持的加密算法有很多：3DES、AES、DES
encryption 3des # 指定加密算法

# 可选 MD5 SHA 等
hash md5 # 指定报文摘要算法 MD5

# 预定义了 1 2 5 号组标识符
group 2 # 指定 Diffie-Hellman 组表示符，在 DH 算法同步密钥时，使用组标识符 2 对应的参数

# 超时后需要重新建立安全传输通道（安全关联）
lifetime 3600 # 指定该安全策略的失效时间 3600s

# 为需要建立安全传输通道 且 采用共享秘钥鉴别机制的隧道两端配置共享秘钥 1234
# 0.0.0.0/0.0.0.0 表示另一端 IP 任意
crypto isakmap key 1234 address 0.0.0.0 0.0.0.0 
```

### IPSec 变换集

指定安全协议，以及使用的加密算法和 HMAC 算法

```shell
crypto ipsec transform-set tunnel esp-3des esp-md5-hmac
# transform-set 表示变换集
# tunnel 为变换集的名称
# 选择 ESP 作为安全协议，可选的加密算法有：esp-3des, esp-des, esp-aes 可选的 HMAC 算法有：esp-md5-hmac, esp-sha-hmac
# 选择 AH 作为安全协议，可选的 HMAC 算法有：esp-md5-hmac, esp-sha-hmac
```

### 分组过滤器

指定需要经过安全关联传输的 IP 分组集，只有与实现内部子网之间通信相关的 IP 分组才需要经过 IPSec 安全关联进行传输

```shell
access-list 101 permit gre host 192.1.1.1 host 192.1.2.1
access-list 101 deny ip any any
# 101 组过滤，指定源 IP 为 192.1.1.1 目的 IP 为 192.1.2.1
# gre 表示 以 GRE 格式封装内层 IP分组
```

### 加密映射

指定安全关联所 使用的安全协议和相关算法的 IPSec 变换集与过滤后 IP 分组的绑定

```shell
# 创建名为 tunnel 序号为 10 的 ipsec-isakmp 环境下的加密映射并进入其配置模式
crypto map tunnel 10 ipsec-isakmp

set peer 192.1.2.1 # 指定安全关联的另一端 为 192.1.2.1

set pfs group2 # 指定 DH 算法和组标识符 2 对应的参数, 重新建立安全关联时使用 DH 算法交换同步密钥，并使用组标识符 2 对应的参数重新建立安全关联

# 指定安全关联的失效时间
set security-association lifetime seconds 900

# 指定安全关联使用的安全协议及加密，HMAC算法的变换集为 tunnel
set transform-set tunnel

# 指定经过安全关联传输的 IP 分组集为 101 分组过滤器
match address 101
```

## 1、安全策略配置

隧道两端都需要完成相应的 ISAKMP 安全策略配置过程，包括安全通道传输所使用的加密算法，报文摘要算法，共享秘钥鉴别机制和 DH 组号。

隧道每一端可以配置多个安全 策略，但两端必须存在匹配的安全策略

**Router 1**：R1 上定义编号为 1 优先级最高的安全策略，身份鉴别机制采用共享秘钥鉴别机制，加密算法采用 3des，hash 采用 md5，DH 秘钥交换算法

![image-20211020083237656](https://gitee.com/Butterflier/pictures/raw/master/image-20211020083237656.png)

**Route 2**：匹配 Tunnel 1 的 R1 端，采用相同的安全策略即可

![image-20211020084427570](https://gitee.com/Butterflier/pictures/raw/master/image-20211020084427570.png)

**Route 3**：匹配 Tunnel 2 的 R1 端，与 Tunnel 3 的 R2 端，采用相同的安全策略即可

![image-20211020084337530](https://gitee.com/Butterflier/pictures/raw/master/image-20211020084337530.png)

## 2、变换集配置

IPSec 参数配置：这属于阶段二中安全关联相关配置过程

```shell
crypto ipsec transform-set tunnel esp-3des esp-md5-hmac
```

## 3、分组过滤器配置

指定两端需要进行安全传输的 IP 分组范围

R1：配置 Tunnel 1 与 Tunnel 2 的源端 IP 与 目的端 IP

```shell
# tunnel 1
access-list 101 permit gre host 192.1.1.1 host 192.1.2.1
access-list 101 deny ip any any
# tunnel 2
access-list 102 permit gre host 192.1.1.1 host 192.1.3.1
access-list 102 deny ip any any
```

R2：配置 Tunnel 1 与 Tunnel 3 的源端 IP 与 目的端 IP

```shell
# tunnel 1
access-list 101 permit gre host 192.1.2.1 host 192.1.1.1
access-list 101 deny ip any any
# tunnel 3
access-list 102 permit gre host 192.1.2.1 host 192.1.3.1
access-list 102 deny ip any any
```

R3：配置 Tunnel 2 与 Tunnel 3 的源端 IP 与 目的端 IP

```shell
# tunnel 2
access-list 101 permit gre host 192.1.3.1 host 192.1.1.1
access-list 101 deny ip any any
# tunnel 3
access-list 102 permit gre host 192.1.3.1 host 192.1.2.1
access-list 102 deny ip any any
```

## 4、加密映射配置

将分组过滤器与变换集配置选项绑定至加密映射中

R1：需要配置 Tunnel 1 与 Tunnel 2 的加密映射

![image-20211020091651843](https://gitee.com/Butterflier/pictures/raw/master/image-20211020091651843.png)

R2：需要配置 Tunnel 1 与 Tunnel 3 的加密映射

![image-20211020091911504](https://gitee.com/Butterflier/pictures/raw/master/image-20211020091911504.png)

R3：需要配置 Tunnel 2 与 Tunnel 3 的加密映射

![image-20211020092049086](https://gitee.com/Butterflier/pictures/raw/master/image-20211020092049086.png)

## 5、绑定接口

将配置好的加密映射作用到路由器目标接口上

```shell
interface FastEthernet 0/1
crypto map tunnel 
```

## 6、测试

使用 PC0 ping PC4 `192.168.3.1`

![image-20211020093157410](https://gitee.com/Butterflier/pictures/raw/master/image-20211020093157410.png)

首先能够看到初始的内层 IP 分组：

![image-20211020092955818](https://gitee.com/Butterflier/pictures/raw/master/image-20211020092955818.png)

随后能够看到 R1 将原始 IP 分组封装后的报文：内部的信息（私有地址）已然看不到了

![image-20211020093116200](https://gitee.com/Butterflier/pictures/raw/master/image-20211020093116200.png)

然后该报文在公网上安全加密传输：

![image-20211020093245915](https://gitee.com/Butterflier/pictures/raw/master/image-20211020093245915.png)

直到 R3 开始解开外部 IP 报文，解密获取了内部 IP 分组，故而继续在私网上传输

![image-20211020093419574](https://gitee.com/Butterflier/pictures/raw/master/image-20211020093419574.png)

