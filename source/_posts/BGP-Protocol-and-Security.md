---
title: BGP Protocol and Security
mathjax: false
date: 2021-12-06 13:19:52
summary: BGP 协议安全分析
categories: Protocol Security
tags:
  - security
---

# BGP

BGP 是外部网关协议，属于增强的距离矢量路由协议，用来在AS之间传递路由信息
> AS 之间的协议，并没有统一的管理机构，因此存在很大的安全威胁

## 名词速览

1、AS，自治域或自治系统

2、Speaker，发送 BGP 消息的路由器

3、Peers，Speaker 之间的邻居关系

4、EBGP，外部 BGP

5、IBGP，内部 BGP

6、IGP，内部网关协议

## 特点

(与同样是距离矢量协议的 RIP 比比看)

1、可靠的路由更新机制：传送协议为 TCP - 179；无需周期性更新；需要周期性发送 keepalive 报文校验 TCP 的连通性；路由更新时只发送增量路由；

2、丰富的 Metric 度量方法

3、从设计层面避免了环路的发生

4、为路由附带了属性信息

5、支持 CIDR - 无类别域间选路

6、非负的路由过滤和路由策略

> 优势点也可能成为攻击点

## BGP 基本概念

### AS

AS，自治域或自治系统，指拥有相同选路策略的由单一机构管理的网络集合，每个 AS 由 AS 号标识
> 目前的互联网大约由 5 万个 AS 和 55 万个地址前缀组成
> 
> 每个地址前缀代表一个网络空间，由连续的 IP 地址组成，如前缀 59.66.132.0/24 代表一个有 256 个 IP 地址的网络空间

### 发言者

Speaker，指发送 BGP 消息的路由器，它接收或产生新的路由信息并发布/通告给其它 Speaker
> 当 BGP Speaker 收到**来自其它 AS 的新路由时**，如果该路由比当前已知路由更优或者当前还没有该路由，它就把这条路由**发布给 AS 内所有其它 Speaker**

### 邻居

Peers，任何两个形成 TCP 连接来交换 BGP 路由信息的 Speaker 称为邻居(peers)或对等体

![Peers](https://raw.githubusercontent.com/Coming98/pictures/main/20211129105731.png)

### 外部与内部 BGP

外部 BGP 即 EBGP，当 BGP 邻居属于不同的自治系统，他们被称为 EBGP（相对而言的）
> EBGP 邻居, 默认情况下, 需要直连

![EBGP](https://raw.githubusercontent.com/Coming98/pictures/main/20211206113218.png)

内部 BGP 即 IBGP，指 BGP 邻居存在于同一个 AS 内，通常这两个 BGP 不需要直连，是该 AS 的两个边界 BGP

![IBGP](https://raw.githubusercontent.com/Coming98/pictures/main/20211206113154.png)

### 五类消息/报文

1、Open, 建立 BGP 连接

2、Keepalive, 检测和维护 BGP 连接

3、**Update**, 发送 BGP 路由更新

4、Notification, 中断 BGP 连接

5、Route-Refresh，通知对等体自己支持路由刷新能力

## BGP 工作机制

1、建立 TCP 连接 （179）

2、BGP Speaker 通过 Open 消息协商参数，建立 BGP 邻居关系

3、建立连接后，BGP 邻居会通过 Update 消息及时增量更新路由表

4、BGP 会发送 Keepalive 消息来维持邻居间的 BGP 连接

5、当 BGP 检测到网络中的错误状态时，BGP 发 Notification 消息报错，BGP 连接中断

## BGP 路由注入

内部网关协议信息注入外部网关协议中

### 纯动态注入

路由器将通过 IGP 路由协议动态获得的路由信息并直接注入到 BGP 系统中去

![动态注入](https://raw.githubusercontent.com/Coming98/pictures/main/20211129111408.png)


### 半动态注入

路由器有选择性的（通过配置命令）将 IGP 发现的动态路由信息注入到 BGP 系统中去
> 会检查该路由符合之前配置的注入条件

### 静态注入

路由器将静态配置的某条路由注入到 BGP 系统中
> 人为配置某条静态路由，然后将其注入到 BGP 路由表中

## BGP 路由通告原则

一旦建立新连接，Speaker 按照下述原则将自己所有路由通告信息通告给新邻居
> 存在多条路径时，Speaker 只选取最优的使用（不考虑负载均衡的情况）
> Speaker 只把自己使用的路由通告给邻居

1、对于 BGP 路由，如果当前路由器为该路由的始发路由器，则会将该路由传递给其他所有 IBGP 对等体以及所有 EBGP 对等体（始发信息全部传）

2、非始发路由，如果所接收路由条目为 IBGP 路由，则只会将该路由条目转发给所有 EBGP 对等体；所接收条目为 EBGP 路由时，则会发给所有 IBGP 与 EBGP 对等体（内部来源信息只传给外部，外部来源信息传给所有）
> Speaker 从 IBGP 获得的路由是否通告给它的 EBGP 邻居要依 IGP 和 BGP 同步的情况来决定
> 这个原则避免了 AS 内部环路

### EBGP 之间

1、Speaker 从 EBGP 获得的路由会向它所有 BGP 邻居通告（包括 EBGP 和 IBGP）
> 建立连接后会发送所有有效BGP路由，之后只**增量更新路由**

![增量路由](https://raw.githubusercontent.com/Coming98/pictures/main/20211206125443.png)

### IBGP 之间

Speaker 从 IBGP 获得的路由不向它的 IBGP 邻居通告（避免环路）

## BGP 路由属性

BGP 路由属性是包含在 BGP 路由器通告里的一套参数，它对特定的路由进行描述，使得路由器能够对路由进行过滤和选择
> IGP 使用 networks 通告路由，用度量值标识路径好坏
> EGP 中的 BGP 通告整条路径并使用属性标识路径好坏

四类属性：

1、公认必遵：所有 BGP 路由器都可以识别的属性，且必须存在于 Update 消息中。如果缺少这种属性，路由信息就会出错（我支持你也必须有）

2、公认任意：所有BGP路由器都可以识别，但不要求必须存在于 Update 消息中，可以根据具体情况来选择（我支持你可以没有）

3、可选过度：在 AS 之间具有可传递性的属性。BGP 路由器可以不支持此属性，但它仍然会接收带有此属性的路由，并通告给其他邻居（我如果不支持但是会帮你传递）

4、可选非过渡：如果 BGP 路由器不支持此属性，则相应的 Update 消息会被忽略，且不会通告给其他邻居（我如果不支持就不管那个属性）

常用属性如下：

![BGP路由属性](https://raw.githubusercontent.com/Coming98/pictures/main/20211129113701.png)

## BGP 路由更新策略

当 BGP 路由器从多个邻居接收到达到同一目的前缀的路由信息时，会根据路由属性选择并通告一条最优路由（顺序决策）

1、本地优先属性(Local_Pref)值最高的路由
> 优先属性由 AS 商业关系决定
> C2P 关系：Customer 向 Provider 购买接入互联网服务，支付流量费：顾客就是上帝
> P2C 关系：Provider 为 Customer 提供接入互联网服务，收取流量费
> P2P 关系：AS 之间可以互相为对方(及其Customer)免费转发流量：关系平等
> S2S 关系：AS 之间可以免费转发对方任何流量(少见)

![路由策略](https://raw.githubusercontent.com/Coming98/pictures/main/20211129115251.png)

Q：为什么优先级次序为 Customer > Peer > Provider

A：这是由流量所产生的的经济利益决定的，customer 为购买方其产生的经济利益最大，因此优先级最高；Peer 与我是同等身份，经济利益应介于 Provider 与 Customer 之间；

2、AS路径(AS_Path)长度最小的路由

3、路由源类型(Origin)最小的路由(IGP<EGP<INCOMPLETE)

4、MED值最小的路由(Next_Hop相同的前提下)

5、EBGP类型路由优于IBGP类型路由

6、IGP Metric最小的路由(对于出口边界路由器)

7、路由器ID最小的路由

## BGP 路由通告策略

BGP 是一种基于策略的路由协议，只有当 AS 愿意让邻居 AS 利用其资源去访问某些网络时，才会把自己掌握的特定路由信息通告给邻居 AS
> BGP 路由通告策略由 AS 商业关系决定

### 导出规则

1、来自 Customer 的路由通告给 Customer、Peer 以及 Provider

2、来自 Peer 或 Provider 的路由仅通告给 Customer，不向 Peer 和 Provider 传播

Q：为什么要这样指定规则？

A：与路由器通告原则中本地优先属性规定类似，考虑到的还是经济效益；我从顾客那里获得的路由当然要给我其他顾客，以及与我合作的 peer 与 Provider 共享；但是我从与我合作的 peer 与 Provider 获得的路由只能给与我的顾客，不能给与其它的与我合作的 peer 与 Provider（因为他们不一定是合作关系）

### 导入规则

1、优选 Local preference 高的，通常从高到低排列为 customer > peer > provider；

2、其次看 AS-Path 长度，长度越小越优先

# BGP 安全威胁

1、没有保障邻居之间通信报文的完整性、时效性和邻居身份的真实性

2、没有验证 AS 可发起 NLRI 的权限 -> 前缀劫持
> NIRI，网络层可达信息，它描述了一个路由和怎样到达它，就是一个路由前缀

3、没有保障 AS 通告路径属性的真实性 -> 路径伪造

4、无法验证路由传播行为是否合法 -> 路由泄露
> 异常路由通告：前缀劫持，路径伪造，路由泄露，是 BGP 面临的最主要安全威胁，轻可造成路由黑洞和中间人攻击，重则容易造成互联网大规模瘫痪

## 前缀劫持

AS 对外发起的路由通告中的前缀未获授权（我可以随便说到某个 IP 要走我自己）

![前缀劫持](https://raw.githubusercontent.com/Coming98/pictures/main/20211129123047.png)

## 路径伪造

AS 向邻居传播的路由通告中的 AS_PATH 路径属性非真（类似于前缀劫持）

![路径伪造](https://raw.githubusercontent.com/Coming98/pictures/main/20211129123141.png)

## 路由泄露

AS 向邻居传播的路由通告违反路由出站策略：路由通告合法，路由传播非法，会造成流量重定向
> 来自 Peer 或 Provider 的路由仅通告给 Customer，不向 Peer 和 Provider 传播

![路由泄露](https://raw.githubusercontent.com/Coming98/pictures/main/20211129123340.png)

## 典型事件分析

![异常路由事件分析](https://raw.githubusercontent.com/Coming98/pictures/main/20211206130550.png)

### 巴基斯坦电信恶意主动劫持 YouTube 前缀

1、巴基斯坦电信致使 YouTube 断网事件：巴基斯坦电信想要封掉 YouTube 的访问，因此在路由器在路由器上添加一条静态路由表项（人文静态注入）将目的地址设为黑洞路由地址；但是电信工程师不小心让这个路由注入到了 BGP 中，随后被通告给了巴基斯坦电信的 Provider 香港电讯盈科，Provider（香港电讯盈科） 一看 Customer （巴基斯坦电信）传来了路由，那我得给所有人通告啊，因此被逐渐同步到了全世界

# BGP 安全技术

BGP 安全技术分为**路由认证技术**和**异常检测技术**两大类

路由认证技术：以制定和完善 BGP 路由协议的安全机制为目标，利用证书、数字签名和其他加密技术来保护路由信息的真实性和完整性。可以从根本上解决 BGP 异常路由通告问题，但同时也需要付出不小的代价，主要包括：需要建立 PKI、路由器的性能开销、需要修改现有协议规范等

异常检测技术：提取 BGP 控制平面和数据平面的异常信息，对异常路由通告行为进行检测并报警。不能彻底解决 BGP 的安全问题，但在目前尚未部署完整 PKI体系的情况下，不失为一种轻量级的解决方案

![BGP 安全技术对比](https://raw.githubusercontent.com/Coming98/pictures/main/20211129124523.png)

## S-BGP

Secure BGP，基于PKI（公钥基础设施）的 BGP 路由认证技术：

### 组成

1、定义了两类 PKI 来表明 AS 号和 IP 地址的所有权，通过**绑定**来证实它们与某个 AS 的关系；

2、增加了一个新的路由属性 Attestation（证明），可携带数字签名，用于 BGP 系统合法性的确认；

3、应用 IPsecBGP 对话系统间的确认。IPsec 同 BGP 发言者发布的证书相关联，来证实 BGP 邻居的真实性，提供 TCP 连接的数据完整性保护，同时可以防止重放攻击；

### Attestation

Attestation 分为 Address Attestation（地址证明）与 Route Attestation（路由证明）：

![Attestation](https://raw.githubusercontent.com/Coming98/pictures/main/20211129133832.png)

### Attestation 地址证明

地址证明：Signer 是 ISP，ExplicitPA 字段包含有该 ISP 拥有的一段 I P前缀，Target 是 ISP 授权用于通告该 IP 前缀的 AS 号。ISP 使用其私钥对 IP 前缀和 AS 号码数据进行签名并置于 Signature 字段，以保护该段数据的完整性和真实性。“地址证明”表明了哪个 AS 被 ISP 授权来通告其所拥有的前缀
> 证明了 AS 拥有的 IP 地址；这样未被证明的 AS 就不能主动通告了

### Attestation 路由证明

路由证明：Signer 是 BGP 路由器，ExplicitPA 字段含有要通告的 IP 前缀，Target 是路由器邻居的 AS 号。“路由证明”用 BGP 路由器的私钥进行签名,“路由证明” 代表了 BGP 路由通告方对其邻居继续通告该 IP 前缀的授权
> 代表我对我邻居能够继续通告的授权，我认证了我的邻居；这样我没认证的邻居收到了通告后是不能更改或随意转发的；

### S-BGP 路由认证方法：

1、提取路由信息中的 IP 前缀，从其他地方获取与该 IP 前缀绑定的 ISP 证书和地址证明，用 ISP 的公钥验证地址证明，就可证实路由的 Origin AS 是否具有通告该前缀的合法授权

2、对于 AS_PATH 的认证，则使用 AS_PATH 中每一跳 AS 的路由器证书来对路由消息中携带的路由证明逐个验证，以证实该 AS_PATH 确实可信

Tips：S-BGP 虽然成功地解决了路由认证问题，不过也相应地带来计算开销大、路径收敛时间延长的问题，更加上建立 PKI 需要 IANA、RIR、ISP 以及路由器厂商的共同参与，致使 S-BGP 始终未能实际部署

## MOAS List

MOAS，Multi-Origin AS，冲突：一个前缀，Prefix，匹配多个 Origin AS 的行为
> 也就是有多个源 AS 对前缀 P

### 基本思想

创建一个包含所有授权通告某一前缀的 Origin AS 的列表，MOAS list，将该列表附于每一授权 AS 的路由通告中，当其他路由器接收到关于这一前缀的所有路由通告时，比较通告中的 MOAS List 是否一致，以此判断是否发生了前缀劫持

### 有效性证明

MOAS List 技术之所以有效，主要在于 Internet 是一个高度互联的 mesh 网络，无论是因恶意攻击还是因管理员误配所产生的前缀劫持，由于 BGP 路由传播的多路径特性，错误的 MOAS List 和正确的 MOAS List 最终都会被接收路由器收到，两者的不一致就会使接收路由器意识到发生了前缀劫持事件
