---
title: SSL/TLS Protocol and Security
mathjax: false
date: 2021-12-04 19:58:10
summary: SSL/TLS协议与协议安全分析
categories: Protocol Security
tags:
  - security
---
# 传输层安全协议 SSL/TLS

HTTPS = HTTP + SSL/TLS

## SSL / TLS 简述

SSL，Secure Socket Layer，安全套接字（IP 地址 + 端口）层，是一种在 TCP 协议之上为两个应用层端实体（End Entity）之间提供安全通道的协议，利用数据加密技术确保数据在网络上传输过程中不会被窃取或篡改

TLS，Transport Layer Security，传输层安全协议，用于两个应用程序之间提供保密性和数据完整性

> 通常来讲 SSL 与 TLS 是一个东西，因为 SSL 是企业提出的，所以 TLS 1.0 才是 IETF 制定的 SSL 的互联网标准版本，当时的 SSL 版本是 3.0，因此 TLS v1 与 SSL v3 的差别非常微小

## IPSec 与 SSL

![IPSec 与 SSL](https://raw.githubusercontent.com/Coming98/pictures/main/20211204183341.png)


谁更安全？

安全可以从 CIA（机密性，完整性，可用性），不可抵赖性，可控性 这五个维度来评价下，IPSec 与 SSL 都能保证机密性，完整性，不可抵赖性（私钥签名与公钥验证）

但是不同的是 IPSec 在网络层，SSL 在应用层，因为 TCP 协议更为复杂，所以 TCP 安全威胁也较多，因此很多对 TCP 的攻击都将会影响到 SSL 的可用性与可控性，因此工作在较可靠的 IP 上的 IPSec 更为安全

## SSL 提供的安全服务

SSL 基于身份认证，机密性与完整性安全服务建立服务器与客户端之间的安全数据通道

### 身份认证

利用数字证书技术和可信任的第三方认证机构，为客户端和服务器之间的通信提供身份认证功能，以便于彼此之间进行身份识别

1、使用 X.509 v3  数字证书

2、客户对服务器的身份认证：使用标准的公钥加密技术和可靠认证中心（CA）的证书，来确认服务器的合法性

3、服务器对客户的身份认证：通过公钥技术和证书进行认证或通过用户名 + 口令来认证

> 双向认证才能保证一定的安全，单向认证一定会存在漏洞

### 机密性

在 SSL 客户端和服务器之间传输的所有数据都经过了加密处理，以防止非法用户进行窃取、篡改和冒充：Triple DES，IDEA，RC2，RC4 …

### 完整性

SSL 利用加密算法和 Hash 函数来保证客户机和服务器之间传输的数据的完整性，MAC with MD5 or SHA-*

## SSL 体系结构

### 连接与会话的定义

**连接**：连接是能提供合适服务类型的传输；对 SSL 来说，连接是点对点的而且是暂时的；每一条连接都与一个会话相关联

**会话**：SSL 会话是指在客户端和服务器之间的一种虚拟的关联关系，由一组连接组成

> 会话由握手协议创建；定义了一组可以被多个连接共用的密码安全参数（协商）；对于每个连接，可以利用会话来避免对新的安全参数进行代价昂贵的协商（**会话重用**）

![image-20211126105635371](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126105635371.png)

### 状态

每一个会话或连接都与若干个会话或连接状态相关联

> 一旦建立起一个会话或连接，对于发送和接收就存在一个当前操作状态
>
> 在握手协议中还会创建发送挂起和接受挂起状态，等握手协议结束后又回到当前状态
>
> 会话或连接状态由会话或连接状态参数定义

![image-20211126110715060](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126110715060.png)

### 工作原理

![image-20211118140429315](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211118140429315.png)

1、采用握手协议建立客户与服务器之间的安全通道，该协议包括：双方的相互认证，交换密钥参数（密码协商）

2、采用告警协议向对端指示其安全错误

3、采用变更密码规格协议告知改变密码参数

4、采用记录协议封装以上三种协议或应用层数据（比如HTTP）

## SSL 握手协议

### 基本功能

1、客户端和服务器之间相互认证

2、客户端和服务器之间协商加密算法和 MAC 算法

3、客户端和服务器之间协商用于加密 SSL 载荷的共享密钥：机密型秘钥

4、客户端和服务器之间协商用于产生 MAC 的共享密钥：完整型秘钥

> 握手协议在任何应用数据被传输之前使用

### 报文格式

典型的 TLV 格式，类型 + 长度 + 内容

![image-20211118141706256](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211118141706256.png)

![image-20211126112239897](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126112239897.png)

0、hello_request（可选），无参数，服务器告知客户端需要启动握手协议
> Tips ：SSL 的握手都是由客户端先发起的，如果服务器主动需要建立 SSL，也会用 hello_request 告知客户端，让其主动握手（傲娇）

**1、建立安全能力**

client_hello，参数有版本、随机数、会话 id、密码参数、压缩方法，客户端发出 c_h 启动 SSL 会话

server_hello，参数有版本、随机数、会话 id、密码参数、压缩方法，服务器响应客户端的 c_h

![image-20211126111750083](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126111750083.png)

**2、服务器认证与秘钥交换**

certificate，X.509 v3 证书链，服务器发出的向客户端验证自己身份的消息

server_key_exchange，参数为签名信息，由服务端发起的秘钥交换

certificate_request，参数为 CAs，服务器要求进行客户端认证（可选）

server_done，无参数，指示服务器的 Hello 消息发送完毕

![服务器认证与秘钥交换](https://raw.githubusercontent.com/Coming98/pictures/main/20211204184156.png)


**3、客户端认证与秘钥交换**：得到了域主秘钥

certificate_verify，参数为签名信息，对客户证书进行验证

client_key_exchange，参数为签名信息，表示秘钥交换

![](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126112603353.png)

**4、变更密码规格并结束握手**

finished，参数为 hash 值，验证秘钥交换和鉴别过程是成功的

![image-20211126112201222](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126112201222.png)

### 协议流程

**0x01 建立安全能力**

1、Client Hello：客户端明文发送，包含：版本（客户端 SSL 最高版本），随机数（32位时间戳+28字节随机序列，用于秘钥生成），会话 ID （0，在新会话上建立一条新连接；非0，为现有会话创建一个新连接或更新现有连接参数），客户支持的密码算法列表（CipherSuite，也被称作密码套件），客户支持的压缩方法列表
> 每个加密套件对应前面 TLS 原理中的四个功能的组合：认证算法 Au (身份验证)、密钥交换算法 KeyExchange(密钥协商)、对称加密算法 Enc (信息加密)和信息摘要 Mac(完整性校验);

![image-20211126113459835](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126113459835.png)

2、Server Hello：服务器发送，包含：版本（客户端建议的较低版本以及服务器支持的最高本），会话 ID（若客户端会话 ID 非0，采用相同值；否则应为一个新会话ID），随机数（独立于客户端随机数，同样参与密钥生成），服务器从客户端建议的密码套件中挑出一组，服务器从客户端建议的压缩方法中挑出一种

![image-20211126113510948](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126113510948.png)

**0x02 服务器认证与秘钥交换**

3、certificate：服务器发送，包含一个 X.509 证书或者一条 X.509 证书链；

4、（可选）server_key_exchange：若使用 RSA 密钥交换方法或服务器已发送了包含 DH 算法参数的证书，则无须发送该消息（固态 DH）
> Tips：如果是暂态或匿名 DH，则要发送 server_key_exchange 消息

5、（可选）Client Certificate Request：服务器发送，表示向客户请求一个证书，请求内容包含证书类型和证书机构（CAs）

> 使用 HTTPS 浏览网站通常不需要客户端提供证书，但是某些情况下，客户端在浏览过程中的某些操作可能会触发服务器要求验证客户端证书，比如网银转账

6、Server Done，服务器发送，表示服务器的 hello 及相关消息已经结束，等待客户端响应（协商的密码套件版本等客户端是否接收）

**0x03 客户端认证与秘钥交换**

7、Server Done，客户端接收到，根据需要验证服务器提供的证书是否有效；判断 server_hello 的参数是否可以接受；如果都没有问题的话，发送一个或多个消息给服务器
> 验证证书链的可信性；证书是否吊销；证书是否在有效期内；证书中的域名是否与当前访问的域名匹配；

8、（可选）Client Certificate，客户端发送，如果服务器请求证书，则客户端首先发送一个certificate 消息，作为该阶段的开始；若客户没有证书，则发送一个 no_certificate 警告

9、Client Key Exchange，客户端发送，客户端计算产生随机数字 Pre-master，并用证书公钥加密，发送给服务器；
> 消息具体内容取决于密钥交换的类型
> 
> RSA：客户端产生一个 48 字节的 pre_master_secret，并用服务器公钥加密
>
> 固态 DH：以 certificate 消息的形式发送客户端 DH 公钥参数，该消息为空
>
> 暂态或匿名 DH：发送客户端的 DH 公钥参数 C
> 这一步完成后客户端已经获取全部的计算协商密钥需要的信息，以 RSA 为例：两个明文随机数 random_C 和 random_S 与自己计算产生的 pre_master_secret，并用服务器公钥加密，计算得到协商密钥

10、（可选）Certificate Verify，客户端发送，发送一个 certificate_verify 消息验证客户端证书

> 该消息是对一个 HMAC 的签名，该 HMAC 基于之前的握手消息

11、Change Cipher Spec，客户端发送 change_cipher_spec 消息，通知服务器后续的通信都采用协商的通信密钥和加密算法进行加密通信，并且把协商得到的密码套件拷贝到当前连接的状态之中；

**0x04 变更密码规格并结束握手**

12、Client Finished，随后客户端用本次连接协商的算法和密钥参数加密发送一个 finished 消息

> 这条消息用于验证密钥交换和鉴别过程是否成功：其中包括一个校验值（verify_data），对所有以来的消息进行校验

13、Change Cipher Spec，Server Finished，服务器计算出协商秘钥后同样发送 change_cipher_spec 消息和 finished （加密，并且包含相同的 verify_data）消息

客户端计算所有接收信息的 hash 值，并采用协商密钥解密 encrypted_handshake_message，验证服务器发送的数据和密钥，验证通过则握手完成握手过程完成，客户和服务器可以传送应用层数据

**0x05 应用数据传输**

## 记录协议

主要功能：封装握手协议、告警协议、变更密码规格协议或应用层协议的数据

![记录协议](https://raw.githubusercontent.com/Coming98/pictures/main/20211204185938.png)


加密时要注意：加密内容为消息数据域 MAC；加密增加长度不超过 1024 字节；总长度不超过214 + 2048 字节；对于流加密算法，不需要填充数据，消息和 MAC 一起被直接加密；对于 CBC 模式的分组加密算法，需要填充数据，加密算法由 Cipher Spec 指定，并且需要指定初始化向量 IV

添加 SSL 记录头：主要有四部分内容：

内容类型（1字节）：上层协议类型，包含握手协议、告警协议、变更密码规格协议和应用数据四种

主版本号（1字节）：SSL 协议的主版本号，对于 SSL v3.0 该值为 3

副版本号（1字节）：SSL 协议的副版本号，对于 SSL v3.0 该值为 0

压缩后的长度（1字节）：加密后的数据长度，最大值为 214 + 2048 字节

## 密钥交换方法

密钥交换的目的是让双方获得相同的 pre_master_secret，预备主密钥，后续将由 pre_master_secret 生成 master_secret（主密钥）

### RSA

1、客户端生成一个 48 字节的随机数 k 作为 pre_master_secret

2、**数字信封**：客户端用服务器公钥加密 k 得到 k' 并发送给客户端，服务器收到 k' 后用自己的私钥解密得到 k

3、此时双方都得到了 k 即 pre_master_secret，密钥交换完成

### DH

共有 6 个参数，分别是 DH 算法参数（P，G）、服务器私钥参数 s 和公钥参数 S、客户端私钥参数 c 和公钥参数 C

1、服务器选定 DH 算法参数（P，G），可包含在证书中（固态 DH），或通过server_key_exchange 消息发送给客户端（暂态或匿名 DH）

2、客户端生成一个随机数 c，根据（P，G）计算出 C，C 可通过证书（固态 DH）或者client_key_exchange 消息发送给服务器（暂态或匿名DH）

3、同时服务器生成一个随机数 s，根据（P，G）计算出 S，S 可通过证书（固态 DH）或者通过server_key_exchange 消息发送给客户端（暂态或匿名DH）

4、客户端和服务器各自根据（c，S）和（s，C）计算出相同的 pre_master_secret（DH 算法保证该值相同）

## 密钥生成

通过秘钥交换算法得到的秘钥为预备主密钥，随后将通过预备主密钥生成主密钥

### 主密钥生成

Master Secret，服务器和客户端都有一份相同的 Pre_Master_Secret 和 随机数，这个随机数将作为产生 Master_Secret 的种子
> 生成方法可以多样

![image-20211126163441460](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126163441460.png)

### 密码参数生成

密码规格要求的客户端写 MAC 密钥、服务器写 MAC 密钥(用于数据完整性校验)、客户端写密钥、服务器写密钥（用于对称加密数据）、客户端写初始向量 IV、服务器写初始向量 IV（作为很多加密算法的初始化向量使用，具体可以研究对称加密算法），这些都是按顺序由主密钥生成

生成的方法是：主密钥利用散列函数产生安全字节序列（key_block），序列足够长以便生成所有需要的参数；具体的需要进行 12 次迭代，得到 12 个 hash 值，分组成 6 个元素：client / server mac key，encryption key，IV：共两组加密元素，分别被客户端和服务器使用，这两组元素都被两边同时获取

客户端使用 client 组元素加密数据，服务器使用 client 元素解密;服务器使用 server 元素加密，client 使用 server 元素解密;
> 双向通信的不同方向使用的密钥不同，破解通信至少需要破解两次

## 会话重用

SSL 认为会话通常是具有较长的生命期，在一个会话基础上可派生出多个连接；本质是基于 session_id 的复用

1、对于已经建立的 SSL 会话，使用 session_id 为 key，master_secret 为 value 组成一对键值，保存在本地，服务器和客户端都保存一份

2、当第二次握手（发起新的连接）时，客户端若想使用会话复用，则将 client hello 消息中 session id 设为相应的值

3、服务器收到这个 client hello，查找本地是否有该 session_id，如果有，判断当前的加密套件和上个会话的加密套件是否一致，一致则允许使用会话复用，将 server hello 中 session_id 也设为相同的值，然后计算对称秘钥，执行后续操作

4、如果服务器未查到客户端的 session_id（可能是会话已经老化），则会重新握手，session id 要么重新计算（和 client hello 中的不一样），要么置成 0，这两个方式都会告诉客户端这次会话不进行会话复用

## TLS 与 SSL 的差异

1、版本号：TLS 记录格式与 SSL 记录格式相同，但版本号的值不同，TLSv1.0 使用的版本号为SSLv3.1

2、SSLv3.0 和 TLS 的 MAC 算法及 MAC 计算的范围不同：TLS 使用了 RFC 2104 定义的 HMAC 算法；SSLv3.0 使用了相似的算法，两者差别在于 SSLv3.0 中，填充字节与密钥之间采用的是连接运算，而 HMAC 算法采用的是异或运算
> 但是两者的安全程度是相同的

3、伪随机函数：TLS 使用称为 PRF 的伪随机函数来将密钥扩展成数据块，是更安全的方式

4、告警代码：TLS 支持几乎所有的 SSLv3.0 报警代码，而且 TLS 还补充定义了很多报警代码

5、密码套件和客户证书：SSLv3.0 和 TLS 存在少量差别，即 TLS 不支持 Fortezza 密钥交换、加密算法和客户证书

6、Certificate Verify 和 Finished 消息：SSLv3.0 和 TLS 在用 MD5 和 SHA-1 计算 Certificate Verify 和 Finished 消息中的 HMAC 时，计算的输入有少许差别，但安全性相当

7、加密计算：TLS 与 SSLv3.0 在计算主密钥（master_secret）时采用的方式不同

8、用户数据加密之前需要增加的填充字节：在 SSL 中，填充后的数据长度要达到密文块长度的**最小**整数倍；在 TLS 中，填充后的数据长度可以是密文块长度的任意整数倍（但填充的最大长度为 255 字节），这种方式可以**防止基于报文长度分析的攻击**

# SSL/TLS 安全威胁

## Strip 攻击

基于用户习惯的安全威胁

Principle：多数用户不会主动请求 SSL 协议来保护通信内容，即不会主动在浏览器中输入 https 访问域名；一般情况下都是刘浏览器帮我们重定向到安全的 https 了

![image-20211126172249577](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126172249577.png)

但是如果通信中间存在中间人，那就会被欺骗：

1、攻击者利用 ARP 欺骗等方式成功监听客户端

2、客户端向服务器端发送 http 请求，攻击者如实转发请求

3、服务器端回复 https 链接给客户端，攻击者收到请求并将该链接篡改为 http 链接回复给客户端

4、客户端再次发送 http 请求给服务器端，攻击者将其改为 https 发送至服务器端

5、客户端与攻击者此时建立一个 http 明文链接，攻击者与服务器端建立 https 加密链接，攻击者因此可以获取客户端的所有信息

> 用户到攻击者的连接是 http，而攻击者却通过 https 连接到安全服务器，这就意味着浏览器上警告提示已经被阻止，浏览器看起来正常工作，在此期间所有的用户敏感信息都可以轻易被截获
>
> Tips：各大主流浏览器对未部署 SSL 证书的 HTTP 网站会发出 不安全 的警告，但是目标网站是合法的，所以浏览器警告会被阻止

![image-20211126172341706](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126172341706.png)

## 重协商攻击

基于协议漏洞的安全威胁

重协商指客户端和服务器在已经协商好的 SSL/TLS 连接上重新协商，用以更换算法、更换数字证书、重新验证对方身份、更新共享密钥等等；SSL/TLS 协议本身支持重协商，且 RFC 建议 SSL/TLS 实现（OpenSSL 等）也应该默认支持重协商

两种协商方式：重协商都是客户端主动发起，如果服务器想发起，那么需要向客户端通知

图A：ClientHello1 表示首次协商；ClientHello2 表示客户主动提出重协商；

图B：ClientHello1 表示首次协商；Hello Request，服务器请求客户端发起重协商；ClientHello2 表示客户主动提出重协商；

![image-20211126175358531](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126175358531.png)

1、客户端发出 ClientHello1 进行首次协商

2、中间人缓存 ClientHello1，自己构造 ClientHello2，对服务器发起首次协商

3、协商好后中间人将精心伪造的数据给服务器（服务器的 APP 程序通常需要处理粘包，所以中间人可以构造不完整的数据，让 APP 程序认为发生粘包而暂缓处理数据）

> 粘包：故意构造的不完整数据，让服务应用程序因为不完整而暂缓处理

4、中间人将 ClientHello1 发送给服务器（在同一个连接中），服务器收到后会认为这是一次重协商请求

5、重协商好后客户端发送真正的数据给服务器（服务器的 APP 程序收到后，将其与之前缓存的中间人精心构造的数据粘合起来，进行业务处理）

至此，中间人在不需要劫持或解密 SSL/TLS 连接的情况下，成功地将自己伪造的数据插入到用户真正数据之前（数据注入）

![image-20211126180212571](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126180212571.png)

关键点：客户端认为的首次协商却被服务器认为是重协商，首次协商和重协商直接缺少关联性

防御：

1、禁用重协商

2、禁用不安全的重协商而允许安全重协商（需要在首次协商时携带“支持安全重协商”标识）

![image-20211126180538546](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211126180538546.png)

3、允许安全重协商，但是只允许在首次协商时携带“支持安全重协商”标识，并且会验证两次协商时Finish 消息中的 verify_data 是否一致

![](https://raw.githubusercontent.com/Coming98/pictures/main/20211204193125.png)


## 三次握手攻击

基于协议漏洞的安全威胁

### 阶段一：未知密钥共享

1、客户端访问攻击者搭建的**带有证书**的恶意网站（比如钓鱼网站等）

2、客户端生成一个预备主密钥（使用攻击者公钥加密）和一个随机数，发送给攻击者
> 这里可以看做是客户端和攻击者建立 TLS

3、恶意网站向目标网站服务器发起连接，复用客户端的预备主密钥（使用服务器公钥加密）和随机数
> 这里复用客户端预备主密钥和随机数，也就是攻击者用相同的参数与目标服务器建立 TLS

4、恶意网站将目标服务器的随机数传给客户端
> 将目标服务器的相关参数也告诉客户端一份

5、客户端与攻击者、攻击者与服务器分别进行证书验证与密钥协商

6、密钥协商结束后，将会形成两个 TLS 连接，通信三方拥有相同主密钥（由相同的预备主密钥和随机数生成），共享相同的连接参数

> 但是由于两个连接使用的证书是不一样的，而且每个连接都拥有不同的 verify_data 值，这就需要利用会话恢复的机制来进行下一步的破解

### 阶段二：全同步

1、客户端再次向恶意网站发起请求时，客户端与恶意网站之间采用会话重用机制，主密钥不变；恶意网站直接转发请求到目标网站，同样采用会话重用机制

2、两个连接分别使用相同的主密钥实现会话重用（**会话重用不需要证书，仅使用主密钥就可实现通信双方的身份验证**），并且协商出相同的连接参数
> 仅检查 session_id 是否存在，加密参数是否一致

3、会话重用结束，两个连接的 Finish 消息一致，verify_data 相同

> 在会话重用的时候，不要求进行证书的验证，也没有身份的验证。而仅仅通过主密钥进行通讯双方的身份认证，从而导致在握手结束的时候，Finished消息将一致，verify_data相同

### 阶段三：身份仿冒

1、攻击者浏览到一个需要进行身份验证的资源（比如转账），提交 HTTP 请求；对此，目标服务器发出重协商请求并要求**客户端提供证书**(新的连接需要重协商；转账请求需要客户端证书验证身份)

2、因为安全连接参数在两个连接上均相同，攻击者就可以复制消息，让受害者和目标服务器进行重协商；由于两个连接上一次协商的 verify_data 相同，满足安全重协商要求，重协商能够完成

3、重协商结束，客户端证书通过验证，攻击者之前提交的 HTTP 请求（转账）被执行，攻击完成

或者

攻击者提前向客户端浏览器注入恶意代码，完全控制客户端浏览器：在重协商完成之后，攻击者能够以客户端的身份向服务器提交不受限制的 HTTP 请求并且可以随意获取结果（比如敏感的企业或个人数据等）

> 攻击者再发起重协商，利用客户端的证书伪造自己的身份，从而控制两边的连接，发送任意修改的数据内容。从而完成攻击

**关键点**：

攻击能够成功的原因在于重协商之前发往目标服务器的数据是由攻击者发出的，之后的数据是由经过身份验证的受害者发出的，而目标服务器却无法进行区分

防御：最简单的方法就是禁止重协商

## 心脏滴血

基于协议实现的安全威胁

# Cookies

Q：TLS 是否可以穿过 Net

A：TLS 是一种在 TCP 协议之上为两个应用层端实体（End Entity）之间提供安全通道的协议，并未涉及到 IP 和 端口，可以穿越 Net，

# Refs

[参考 - CSDN - 
hherima - HTTPS协议详解(四)：TLS/SSL握手过程](https://blog.csdn.net/hherima/article/details/52469674)

[参考 - wosign - HTTPS加密协议详解(四)：TLS/SSL握手过程](https://www.wosign.com/FAQ/faq2016-0309-04.htm)