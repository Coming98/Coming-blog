---
title: IPv4 Protocol and Security
mathjax: false
date: 2021-12-05 19:12:45
summary: IPv4 协议安全分析
categories: Protocol Security
tags:
  - security
---

# IP 协议

来到了网络层协议安全，简单介绍下 IP 协议的功能：在互联网络间进行寻址和分组转发；对设备进行逻辑编址；提供不可靠和无连接的数据包传送服务；

并罗列一些名词：

1、最大传输单元  MTU：由各个物理网络中的硬件决定，对数据帧长度的限制；如果 IP 层数据报比链路层 MTU 大，那么 IP 层就需要负责对该数据报进行分片，使得每一片都小于 MTU
Tips：各个物理网络的最大传输单元 MTU 可能不同，A，B 两个路由器之间与 B，C 两个路由器之间的 MTU 可能不一致

## IP 地址

给连接到因特网上的每一台主机分配一个全世界范围内惟一的 32 位的标识符，包含了两个独立的信息段：网络号（net-id）+ 主机号（host-id），常用点分十进制记法表示

![IP 地址分类](https://raw.githubusercontent.com/Coming98/pictures/main/20211205164142.png)

A 类：1 - 126

B 类：128 - 191

C 类：192 - 233

D 类：224 - 239

E 类：240 - 255

### 专用地址

也称私有地址，private address，由本机构自行分配其 IP 地址，只在本机构内部有效，不会被路由器转发到公网中；不需要向因特网的管理机构申请，节省全球 IP 地址资源

专用地址也有范围：ABC 三类地址中各预留了一些地址专门用于上述情况，这样就可以区分专有与公网地址，也方便 NAT，因此凡是 Internet 上的网络设备均不会接收，发送或者转发源 IP 地址或目的 IP 地址在上述范围内的报文，这些 IP 地址只能用于私有网络
> 不加以区分的话，我私有地址的设置可能和公网地址在同一网段，那样就是在局域网通信，NAT 也帮不了你

A 类地址范围：10.0.0.0 - 10.255.255.255

B 类地址范围：172.16.0.0 - 172.31.255.255

C 类地址范围：192.168.0.0 - 192.168.255.255

### 公网地址

又称合法地址，可以在因特网上使用 A、B、C 类地址

## 划分子网

子网都在同一个网段，其子网掩码一致

子网掩码：长度 32 位的二进制数字，用于表示网络号与主机号的位数

网络地址：IP 地址和子网掩码进行与运算，将运算结果中的网络地址不变，主机地址变为 0

方式：从网络的主机号借用若干比特作为子网号 subnet-id

过程：此路由器收到 IP 数据报后，再按目的网络号 net-id 和子网号 subnet-id 找到目的子网，将 IP 数据报交付给目的主机

## 广播地址

### 受限广播地址

255.255.255.255，IP 地址的网络字段和主机字段全为 1

> 路由器都不转发目的地址为受限的广播地址的数据报，这样的数据报仅出现在本地网络中
>
> 应用：本机不知道所处的网络时 或 网络掩码 或 服务器 IP，就会使用受限广播地址向 DHCP 服务器所要地址

### 直接广播地址

包含一个有效的网络号和一个全 1 的主机号

> 应用：用于本地网络，也可以跨网段广播（路由器要开启定向广播功能）
> 比如主机 192.168.1.1/30 可以发送广播包到 192.168.1.7，使主机 192.168.1.5/30也可以接收到该数据包

# IP 数据报

![IP 数据报格式](https://raw.githubusercontent.com/Coming98/pictures/main/20211205162718.png)

## IP 数据报选项

该选项为变长部分，用于网络控制和测试目的（如源路由、记录路由、时间戳等），最大长度不能超过 40 字节
> IP 选项在使用时是可选的，但在 TCP/IP 软件的实现中却是必须有的，也就是说所有的IP 协议都具有 IP 选项的处理功能

选项格式如下：

<img src="https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127123337344.png" alt="image-20211127123337344" style="zoom:50%;" />

## 源路由选项

通常 IP 数据报在传输时，由路由器自动为其选择路由。为了使数据报绕开出错网络，或者为了对某特定网络的吞吐率进行测试，需要在信源机控制 IP 数据报的传输路径

### 严格源路由

要求信源机上的发送者指定数据报必须经过每一个路由器

![image-20211127124111380](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127124111380.png)

1、信源机从上层收到源路由 IP 地址表后，将第一个 IP 地址从列表中去掉，并作为当前数据报的目的地址

2、将剩余的表项前移（逻辑前移，已经经过的地址还会在表中）

3、将最终要去的目的地址写入到选项地址表的最后（最后一步真实的目的 IP 将被取出，路由表项中全部为中间指定的路由）

![image-20211127124347061](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127124347061.png)

### 宽松源路由

要求信源机上的发送者指定数据报必须经过指定的路由，指定的路由只是一些关键路由，可能并不临接，两个路由之间的通道由路由器自动路由决定

Tips：其 IP 选项的格式与严格源路由相同

![image-20211127124111380](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127124111380.png)

## 无连接数据报传输

IP 数据报传输是 IP 层要解决的重要问题之一，是影响数据传输效率的一个重要因素，IP 数据报在经过路由器进行转发时一般要进行三个方面的处理：首部校验，路由选择，数据分片。

### 首部校验

IP 数据报的首部通过校验和（Checksum）来保证其正确性

![首部校验](https://raw.githubusercontent.com/Coming98/pictures/main/20211205162443.png)

**为什么要进行首部校验：**

1、IP 首部属于 IP 层协议的内容，不可能由上层协议处理

2、IP 首部中的部分字段在点到点的传递过程中是不断变化的，只能在每个中间点重新形成校验数据，在相邻点之间完成校验

**为什么 IP 层不对数据进行校验：**

1、效率方面：上层传输层是端到端的协议，进行端到端的校验比进行点到点的校验开销小得多，在通信线路较好的情况下尤其如此

2、灵活性：上层协议可以根据对于数据可靠性的要求，选择进行校验或不进行校验，甚至可以考虑采用不同的校验方法，这给系统带来很大的灵活性

### 数据分片与重组

因为各个物理网络的最大传输单元 MTU 可能不同，所以 IPv4 中规定将数据报先以信源网络的 MTU 进行封装，在传输过程中再根据需要对数据报进行动态分片

但是 IPv6 中决定将数据报以从信源到信宿路径上的最小 MTU 进行封装

# IPv4 安全机制

## ACL

ACL ：Access Control List，访问控制列表，应用在**路由器接口**上的指令列表；告知路由器哪些数据报分组可以接收，哪些拒绝

Application：路由器上进行配置；防火墙的相关配置也基于 ACL 指令；一个访问控制列表可以被应用到多个接口上，接口又分进入方向与出去方向；一个接口的 in 和 out 每个方向各只能应用一个访问控制列表。

![image-20211127141040398](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127141040398.png)

### 标准访问控制列表

标准访问控制列表：基于**源地址**作为判断依据，编号为：1-99, 1300-1999

> 路由器对每一个数据包都按照列表顺序逐条检查，如果匹配某一条语句，要么允许数据包通过，要么拒绝通过，不再检查后面的语句；否则向下继续匹配；如果所有的语句都不匹配，就拒绝数据包通过（缺省策略）。

![image-20211007140314775](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211007140314775.png)

### 扩展访问控制列表

扩展访问控制列表：基于 **源地址、目标地址、源端口、目标端口、协议类型**，更为灵活，编号为 100-199, 2000-2699

Application：包过滤防火墙；

## NAT

Backdrop：合法的 IP 地址日益短缺：一个局域网内部有很多台主机，但不是每台主机都有合法的 IP 地址，为了使所有内部主机都可以连接因特网，需要使用地址转换（将私有地址转为公有地址）

Principle：改变 IP 包头，使目的地址、源地址或两个地址在包头中被公有地址替换

Strength：节省公有合法 IP 地址；处理地址交叉；增强灵活性与安全性；

Weakness：延迟增大；配置和维护的复杂性；不支持某些应用

### 地址转换技术

可以有效地隐藏内部局域网中的主机，具有一定的网络安全保护作用（包在互联网中是该子网的公有地址并非子网内的私有地址）

可以在局域网内部提供给外部FTP、WWW、Telnet服务

### 静态 NAT 配置

实现内部 IP 地址与 外部 IP 地址的映射

![静态 NAT 配置](https://raw.githubusercontent.com/Coming98/pictures/main/20211205185404.png)

### 端口地址转换

Port Address Translation，PAT，该技术出现之前使用多个公网地址单射多个私网地址，因此为了解决公网地址短缺问题，引入端口地址转换实现私有地址向公有地址的 n:1 的映射

Q：外面回来的包都是一个公网 IP 我咋知道发给私网中的谁呢？

A：使用端口地址转换技术，利用不同的端口区分私网内各个机器

![image-20211127142039263](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127142039263.png)

## VPN

虚拟专用网络，是企业网在因特网上的延伸

隧道：封装一个隧道报头（三层），直接发送到目标网络所在的网关

![image-20211007142318338](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211007142318338.png)

### VPN 功能

1、数据机密性保护：传输时进行加密传输

2、数据完整性保护：对数据进行 hash 得到摘要，将摘要与加密后的数据一起传输；接收方解密数据后再次计算摘要，与传输来的摘要进行比较，验证数据的完整性

![数据完整性保护](https://raw.githubusercontent.com/Coming98/pictures/main/20211205185841.png)

3、数据源身份认证：使用私钥对摘要进行加密得到数字签名，将摘要与加密后的数据一起传输；接收方解密数据后再次计算摘要，将传输来的数字签名解密得到源摘要，将两个摘要进行比对实现身份验证

![数据源身份认证](https://raw.githubusercontent.com/Coming98/pictures/main/20211205185935.png)

4、重放攻击保护：加时间戳；加随机数；加流水号 / 序列号；
> Replay Attacks，重播攻击，回放攻击，指攻击者发送一个目的主机已接收过的包，来达到欺骗系统的目的；比如身份认证重放，转账重放。

## VPN连接模式

### 传输模式

实现端到端的保护，没有封装 IP 包头，也没有添加新的 IP 包头

优点：效率高

劣势：不能直接在公网上传输；安全性低

Application：结合其他 VPN 一起使用或在企业内网部署

### 隧道模式

实现站点到站点的保护，对整个包头进行了封装，同时添加新的 IP 包头

优点：可以直接在公网上传输，安全性较高

劣势：传输效率低

## VPN 类型

站点到站点 VPN：站点是固定的

远程访问 VPN：IP 地址是不一样的
