---
title: IPSec
mathjax: false
date: 2021-12-02 17:31:49
summary: IPSec 摘抄汇总学习
categories: Protocol Security
tags:
  - security
---

# IPSEC
 
IPSec，IP Security，是 IETF 制定的一系列协议，端到端的确保 IP 层通信安全的机制/框架，用于保证在 Internet 上传送数据的安全保密性能

> 因为 IPv4 缺乏对通信双方身份真实性的鉴别能力；缺乏对传输数据的完整性和机密性保护的机制；IP 层存在业务流被监听和捕获、重放、IP 地址欺骗、信息泄露和数据项篡改等攻击

Process：特定的通信方之间，在 IP 层通过加密与数据源验证来保证数据包在 Internet 上传输的**机密性、完整性和真实性**

私有性 / 机密性：IPSec 在传输数据包之前，将其加密以保证数据的私有性

完整性：IPSec 在目的地使用单向散列函数验证数据包的完整性

身份验证：单向散列函数、数字签名和公开密钥加密来判断一份数据是否源于正确的创建者

防重放：通过查阅序列号，目的地会拒绝老的或重复的数据包

![image-20211007150244277](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211007150244277.png)

![image-20211007164204042](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211007164204042.png)

## 加密算法

对称加密算法：加密密钥与解密密钥相同；由于对称加密的运算速度快，所以 IPSec 使用对称加密算法来加密数据；Example ：DES，3DES，AES

非对称加密算法：加密密钥与解密密钥不同；由于非对称加密的运算速度慢，所以主要用来交换秘钥和身份认证；Example ：RSA，DH

### DH

Diffie-Hellman 密钥交换算法

1、主机1，产生一个很大的数 $a$ 一个素数 $p$ 和一个整数 $g$，计算得到 $A=g^a \mod p$

> 公钥：A，私钥：a

2、发送 $A, p, g$ 给主机2

3、主机2，产生一个很大的数 b，计算得到 $B=g^b \mod p$

> 公钥：B，私钥：b

4、发送 B 给主机1

5、$K_B = A^b \mod p = g^{ab} \mod p$

6、$K_A = B^a \mod p = g^{ab} \mod p$

End：得到共享密钥，$Key = g^{ab} \mod p$

## 验证算法

### HMAC

HMAC，Hash-based Message Authentication Code，一种基于Hash函数和密钥进行消息认证的方法，实现数据完整性验证与身份验证，Example ：MD5，SHA

> 对加密后的数据进行哈希（通常加 salt），得到数字签名；然后将加密后的数据与数字签名一起发送过去，供接收方进行验证。

## 密钥交换

通过非对称加密算法加密对称加密算法的密钥完成密钥共享后再用对称加密算法加密实际要传输的数据，但是如何对抗中间人攻击呢？

> 中间人完全可以更改非对称加密算法的过程，非对称加密算法共享公钥的过程

1、预共享密钥，PSK，Pre-shared key，指通信双方在配置时手工输入相同的密钥

2、数字证书，CA，Certificate authority，其将身份标识与公钥绑定在一起，并由可信任的第三方权威机构用其私钥签名，这样就可验证证书自身的有效性

> 由于一个 CA 不能满足所有的需求，因此形成了一个类似于 DNS 的层次 CA 结构

# IPSec 工作模式

![image-20211013181505022](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013181505022.png)

## 传输模式

实现端到端的保护：封装简单，传输效率不高，只对有效载荷进行了保护，IP 头未被保护

Application：较适用于两台主机之间的数据保护

![image-20211013181231672](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013181231672.png)

## 隧道模式

实现站点到站点保护：对整个报文进行了封装，IP 头被保护（实现了匿名）

Application：较适用于站点间建立安全 VPN 隧道，以保护多台主机

> 站点和站点之间（网关之上），一个站点的所有主机都指代同一个公网 IP 地址

![image-20211013181308850](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013181308850.png)

## 对比

传输模式适用于两台主机之间的数据保护；隧道模式适宜于在站点间建立安全 VPN 隧道：保护多台主机

# AH

AH 协议：Authentication Header，认证头协议，协议号：51

提的服务如下：数据完整性服务（哈希校验），数据源身份认证（共享秘钥），防止数据重放攻击（ESP 头中序列号字段），数据机密性服务（加密）

> 提供报头验证工能，但是不能提供数据加密工能，使得数据可以读取但是无法修改

## AH 头

![image-20211013181648238](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013181648238.png)

**下一头部**（8位）：表示紧跟在 AH 头部后面的协议类型：传输模式下，该字段是处于保护中的传输层协议的值，如6（TCP），17（UDP）或 50（ESP）；在隧道模式下，AH 保护整个 IP 包，该值是4，表示是 IP-in-IP 协议

**有效载荷长度**（8位）：其值是以 32 位为单位的整个 AH 数据（包括头部和变长验证数据）的长度再减 2。

**安全参数索引**，SPI，表示对安全策略的索引

> 用于标识有相同 IP 地址和相同安全协议的不同 SA（由 SA 的创建者定义，只有逻辑意义）

**序列号**，由单向递增的计数器进行维护，用于防重放攻击

**验证数据**：可变长，取决于采用何种消息验证算法。包含完整性验证码，也就是 HMAC 算法的结果，称为 ICV，它的生成算法由 SA 指定。

## 传输模式

保护有效载荷，验证时还需要验证 IP 报文的数据部分，以及 IP 头中的不变部分（ 可变部分预先置0）

![image-20211013181938033](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013181938033.png)

## 隧道模式

验证全部的内部 IP 报文，以及外部新 IP 头中的不变部分，对于可变部分如 TTL 等应提前置位为 0

![image-20211013182402554](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013182402554.png)

# ESP

ESP 协议：Encapsulating Security Payload，封装安全载荷安全协议，协议号：50

提的服务如下：数据完整性服务（哈希校验），数据源身份认证（共享秘钥），防止数据重放攻击（ESP 头中序列号字段），数据机密性服务（加密）

> 数据包（IP包）加密，（路由器端）数据流加密 —— 需要从 SA 处获取秘钥
>
> 但是传输模式中的 ESP 不对整个数据包进行签名，不提供报头验证

## ESP 头

安全参数索引 SPI 与 序列号与 AH 头作用一致

![image-20211013183211644](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013183211644.png)

## 传输模式

传输模式下对原始 IP 头不进行保护，仅对 IP 报文的有效数据部分提供加密

Tips ：ESP 头不加密只认证；ESP 尾部加密也认证；原始 IP 头不加密也不认证；

> ESP 头中有安全参数索引与序列号，传输过程中需要知道索引与序列号得知安全加密策略与序列号信息

![image-20211013183359384](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013183359384.png)

## 隧道模式

对整个内部 IP 报文进行加密，新 IP 头不加密也不认证

![image-20211013183443513](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013183443513.png)

# AH + ESP

![image-20211127155954664](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211127155954664.png)

AH 相较于 ESP 提供了 IP 头的认证，ESP 相较于 AH 提供了数据加密

> 所以需要结合使用 AH 和 ESP 才能保证 IP 报头的机密性和完整性
>
> AH 为 IP 报头提供尽可能多的验证保护，验证失败的包将被丢弃，不交给上层协议解密，这种操作模式可以减少拒绝服务攻击成功的机会。

## 传输模式下

![image-20211013183714583](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013183714583.png)

这里把 ESP 头也加密了，是因为 AH 头在外面呢，只需要用一个头即可

## 隧道模式下

![image-20211013183817158](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211013183817158.png)

## NAT 穿越问题

**Q：使用 IPSec 在跨域 NAT 出现什么问题？NAT 要变换 IP，如何保证 IPSec 认证依然成功？**

NAT 实现了私有地址向公有地址的映射，会对 IP 头进行更改，因此使用对 IP 报头进行验证的 AH 协议的 IPsec 是不能穿越 NAT 的；

而 ESP 不对外部的 IP 头进行完整性检查，IP 地址转换不会破坏 ESP 的 Hash 值。但 ESP 报文中 TCP/UDP 的端口已经加密无法修改，所以对于同时转换端口及 IP 地址的 NAT 来说，ESP 无法穿越 NAT（端口地址转换）。如果 NAT 只转换 IP 地址，那么不管是传输模式还是隧道模式，基于 ESP 的 IPSec 均能穿越 NAT

**Q：对于同时转换IP地址及端口的NAT而言，IPsec如何穿越：NAT-T（NAT Traversal）**

当需要穿越 NAT 设备时，ESP 报文会被封装在一个 UDP 头中，源和目的端口号均是 4500，有了这个 UDP 头就可以正常进行转换

**补充：穿越 NAT 后出现的问题：**

1、身份确认问题：在 IP 网络中 IP 地址是最好的身份标识，IPsec VPN 中标准身份标识也是 IP 地址。NAT 处理过程中会改变 IP 地址，因此 IPsec 的身份确认机制必须能够适应 IP 地址变化。目前解决此问题的方法主要有两种，第一种是使用数字证书替代 IP 地址作为身份标识，第二种是使用字符串取代 IP 地址作为身份标识

2、IP 地址复用问题：ESP 的 IP 协议号是 50，并不是基于 UDP 和 TCP 的协议（虽然其封装的是整个 IP 层数据，包含 TCP/UDP 等传输层协议），因此当 NAT 网关背后存在多个 ESP 应用端时，无法只根据协议号进行反向映射，为了使 ESP 能够在 NAT 环境中进行地址复用，ESP 必须做出改变

ref：https://blog.csdn.net/u014023993/article/details/86634339

# IKE

IKE 协议：Internet Key Exchange，因特网密钥交换协议，**用于动态建立 SA** 实现秘钥交换

## 基本概念

### 混合协议

IKE 属于混合协议：

1、**ISAKMP**（在两个实体间进行分组格式及状态转换的消息交换的体系结构）

2、**Oakley**（基于到达两个对等体间的加密密钥的机制）

3、**SKEME**（实现公钥加密认证的机制）

Strength: IPsec 需要的很多参数（密钥，算法）都可以自动协商建立，降低了手工配置的复杂度，还提供了失效时间这一机制

### IKE 的来源

IPSec SA 负责具体的数据流加密，能够使得 IPSec 区分对不同数据流提供的安全服务，并且 IPSec 能够对不同的数据流提供不同级别的安全保护；因此建立 IPSec 之前需要先建立相对应的安全联盟（SA），手工配置的方式非常繁琐困难，因此引入了 IKE，实现自动进行安全联盟建立与密钥交换的过程

### IKE 的用途

1、为 IPSec 协商生成密钥，供 AH/ESP 加解密和验证使用
> AH 和 ESP 两个协议都使用 SA 来保护通信，而 IKE 的主要功能就是在通信双方协商 SA

2、在 IPSec 通信双方之间，动态地建立安全关联（SA：Security Association），对 SA 进行管理和维护
> IPSec 需要 SA 来对不同数据流提供不同级别的安全保护

## 一句话解释

IPSec：端到端的确保 IP 层通信安全的机制/框架

IKE：用于动态建立 SA 实现秘钥交换，辅助 IPSec 的实施执行

ISAKMP：定义了一个通用的可以被任何密钥交换协议使用的框架（协商、建立、修改和删除SA的过程和包格式）辅助了 IKE 的实施执行

SA：是两个 IPSec 实体之间，经过协商建立起来的一种协定（内容包括各种 IPSec 用到的参数算发等）

SAD：将所有的 SA 以某种数据结构集中存储的一个列表

SP：决定对 IP 数据包提供何种保护，并以何种方式实施保护

SPD：SP 以某种数据结构集中存储的列表

## SA

SA，Security Association，安全联盟，负责 IPSec SA 的建立和维护，起控制作用，是两个 IPSec 实体（主机、安全网关）之间，经过协商建立起来的一种协定
> 内容包括：采用何种 IPSec 协议（AH，ESP）；运行模式（传输模式、隧道模式）；验证算法；加密算法；加密密钥；密钥生存期；抗重放窗口；计数器等；

### 唯一标识

SA 是单向的，每个通信双方都要有两种 SA，**三元组唯一标识**（SPI，目的 IP 地址，IPSec 协议）

1、SPI，Security Parameter Index，安全参数索引，用于标识具有相同 IP 地址和相同安全协议的不同的 SA
> 绑定统一 IP 下同一安全协议下的不同参数

2、目的 IP 地址，SA 的终端地址

3、IPSec 协议，采用 AH, ESP, AH-ESP

### SAD

**SAD**：存储 SA 的数据库（database）

**对于外出的流量**，如果需要使用 IPSec 处理，然而相应的 SA 不存在，则 IPSec 将启动IKE来协商出一个 SA，并存储到 SAD 中。

**对于进入的流量**，如果需要进行 IPSec 处理，IPSec 将从 IP 包中得到三元组（SPI,DST,Protocol），并利用这个三元组在 SAD 中查找一个 SA

## SP

Security Policy，安全策略，决定对 IP 数据包是否提供保护，提供何种保护，以何种方式保护（指向 SA）

> 主要根据源IP地址、目的IP地址、入数据还是出数据等来标识


### SPD 
SPD，Security Policy Database，安全策略数据库，将所有的 SP 以某种数据结构集中存储的列表

当接收或将要发出IP包时，首先要查找SPD来决定如何进行处理

1、丢弃：流量不能离开主机或者发送到应用程序，也不能进行转发

2、不用 IPSec：对流量作为普通流量处理，不需要额外的 IPSec 保护

3、使用 IPSec：对流量应用 IPSec 保护，此时这条安全策略要指向一个 SA。**对于外出流量**，如果该 SA 尚不存在，则启动 IKE 进行协商，把协商的结果连接到该安全策略上；

## ISAKMP

Internet Security Association Key Management Protocol，Internet 安全联盟密钥管理协议，定义了协商、建立、修改和删除 SA 的过程和包格式

> IKE 真正定义了一个密钥交换的过程，而 ISAKMP 只是定义了一个通用的可以被任何密钥交换协议使用的框架

## IKE’s Process

![image-20211014090730684](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211014090730684.png)

1、对等体之间建立一个 IKE SA 完成身份验证和密钥信息交换后，

2、在 IKE SA 的保护下，根据配置的 AH/ESP 安全协议等参数协商出一对 IPSec SA

此后，对等体间的数据将在IPSec隧道中加密传输

### 宏观流程

1、流量触发 IPSec VPN

> 并非主动建立，有流量过来走 VPN 才会触发被动建立

2、建立管理连接

> 即进行阶段 1，用于保证隧道的安全性

3、建立数据连接

> 阶段2，保证数据的安全

### 阶段 1 - 主动协商模式

通信双方协商和建立 IKE 协议本身使用的安全通道，即建立一个IKE SA，总共是三次交换过程，总共有 6 个消息交互

![阶段 1 主动协商模式](https://raw.githubusercontent.com/Coming98/pictures/main/20211202165713.png)

![阶段 1 主动协商模式2](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211015090109438.png)

1、协商对等体间的管理连接使用何种安全策略：A 发送一个或多个 IKE 安全提议；B 收到后查找最先匹配的 IKE 安全提议，并进行确认返回；A 将接收对端确认的策略
> 交换 ISAKMP / IKE 传输集
> 加密算法、HMAC 功能、设备验证的类型、DH 密钥组、管理连接的生存周期
> 匹配的原则为协商双方具有相同的加密算法、认证算法、认证方法和 Diffie-Hellman 组标识

2、通过 DH 算法产生并交换加密算法和 HMAC 功能所需的密钥：A 先发送本端秘钥生成的信息；B 收到后进行秘钥生成，然后返回 B 秘钥生成信息；A 收到后完成秘钥生成；
> 通常使用预共享秘钥防止中间人攻击：因为是网关之间所以方便实现，证书实现麻烦成本较高

```html
根据DH的公开信息都算出了双方相等的秘钥后，连通预共享密钥生成第一个skey_ID
SKEYID - 表示基准密钥，
HDR拆解为Ci，Cr，分别代表Initiator cookie和Responder Cookie。第一个包Cr为0
通过SKEYID推导三个密钥
SKEYID_d = prf(SKEYID, K | Ci | Cr | 0) -----------推导密钥，衍生密钥
SKEYID_a = prf(SKEYID, SKEYID_d | K | Ci | Cr | 1)---------验证密钥
SKEYID_e = prf(SKEYID, SKEYID_a | K | Ci | Cr | 2)---------加密密钥
```

3、使用预共享秘钥等方式执行对等体间的身份验证：A 发送本端身份和验证信息；B 端进行身份验证和交换过程验证，随后发送 B 的身份和验证信息；A 收到后完成身份验证和交换过程验证；

> DH 算法后，此消息就使用 SKEYID_e 加密传输了：使用密钥加密用户身份信息；使用密钥和用户信息通过 hash 算法计算数字签名;对方比对数字签名确认身份

**前四个报文为明文信息，后续报文为密文传输**

Q：为什么有了预共享秘钥还要用 DH 生成 skey_ID 然后在生成三个子秘钥呢？

A：预共享秘钥是预设并不能经常改变，而其它的秘钥是需要经常改变来保证安全性（随机生成），因此使用预共享秘钥负责身份认证，后续的秘钥随机生成使用

![image-20211015090753216](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211015090753216.png)

### 阶段 1 - 野蛮模式

野蛮模式交互过程少，所以在传输过程中，其传输的数据比较多，并且前两个数据为明文传输，仅消息3为加密传输。

![阶段 1 - 野蛮模式](https://raw.githubusercontent.com/Coming98/pictures/main/20211202170905.png)

1、发起方建议SA，发起DH交换：发起者发送 5 元组（IKE SA的各项参数），DH 公共值，辅助随机数 nonce 以及身份资料。响应者可以选择接受或者拒绝该建议

2、接收方接受SA, 认证接收方：回应一个选定的 5 元组，DH 公共值，辅助随机数nonce，身份材料以及一个“认证散列值”。
> Tips: 包含身份信息的消息未被加密, 所以和主模式不同，野蛮模式不提供身份保护

3、发起方认证接受方：发起者发送一个“认证散列值”，该消息被验证，让应答方能够确定其中的散列值是否与计算得到的散列值相同，进而确定消息是否有问题。实际上，这个消息认证发起者并且证明它是交换的参与者。这个消息使用前两个消息交换的密钥信息生成的密钥进行加密。

### 阶段 1 - 对比

![](https://raw.githubusercontent.com/Coming98/pictures/main/20211202171632.png)

### 阶段 2

双方协商 IPSec SA 安全参数，称为变换集 transform set，包括：加密算法、Hash算法、安全协议、封装模式、存活时间；其次还需要周期性的对数据连接更新密钥信息；

1、A 向 B 认证自己、建议安全关联、交换公开值、选择 nonce 等

2、B 向 A 认证自己、建议安全关联、交换公开值、选择 nonce 等

3、A 向 B 发送一个消息来证明自己的活性，该消息只包含一个 Hash 值
> 快速模式的协商是受 IKE SA 保护的，所以协商飞快
> 当第二阶段协商完毕之后，第一阶段的策略将暂时不会被使用，直到有新的VPN连接建立时或IPSEC SA加密密钥超时时，才会用第一阶段的策略重新生成并传递新的加密数据和认证的密钥。

Q：为什么又要协商 IPSec 的参数？

A：一个 IKE 下可以承载多个 IPSec（quick 模式），使得 IKE 可以复用；第一个 SA 作用与 IKE 保证了通道安全；随后的 IPSec SA 使得 IKE 可以复用；

## Demo 分析

![Demo 分析](https://raw.githubusercontent.com/Coming98/pictures/main/20211202172856.png)

![image-20211014134933959](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211014134933959.png)

1、数据传输

2、到 SPD 中查找安全策略：发现源地址为 `1.25` 目的地址为 `2.34` 的需要使用安全策略

3、需要建立相应的 IKE SA（安全关联）：查找策略表中是否存在 SA

> 如果没有，需要进行阶段 1 六步的关联
>
> 如果有，只需要进行阶段 2

4、建立关联后建立了相应的 SA

5、在 SAD 中查找对应 SA 的参数

6、基于 SPD 与 SAD 协商建立 IPSec SA

7、对原有数据报进行相应的安全处理

# Refs

[1] [CSDN - NEUChords - IPSec介绍
](https://blog.csdn.net/NEUChords/article/details/92968314)

[2] [CSDN - 叨陪鲤 - IPSec 专栏目录锦集(openswan)](https://blog.csdn.net/s2603898260/article/details/105831370)