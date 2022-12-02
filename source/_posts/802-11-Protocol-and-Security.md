---
title: 802.11 Protocol and Security
mathjax: false
date: 2021-12-05 14:47:50
summary: 无线网络协议安全分析
categories: Protocol Security
tags:
  - security
---

# 无线网络协议 802.11

无线网络的优势：让网络使用更加自由；网络建设更加经济；通信更加便利；工作更加高效...

## 常用名词

SSID: Service Set ID 服务集识别码，无线名称

STA: Station，任何的无线终端设备

AP: Access Point，接入点

BSS: Basic Service Set, 基本服务集
> BSS 指代一整个服务范围，范围内 AP 提供服务，AP 的外在名词由 SSID 定义，STA 通过 SSID 选择连接哪个 AP

<img src="https://raw.githubusercontent.com/Coming98/pictures/main/image-20210923153802256.png" alt="image-20210923153802256" style="zoom:50%;" />

ESS: Extended Service Set, 扩展服务集，采用**相同的 SSID **的多个 BSS 形成更大规模的虚拟 BSS

DS: Distribution System, 分布式系统

<img src="https://raw.githubusercontent.com/Coming98/pictures/main/image-20210923153934280.png" alt="image-20210923153934280" style="zoom:50%;" />

Ad Hoc: 无线自组网络，构成一种特殊的无线网络应用模式，STA 间可直接互相连接，资源共享，而无需通过 AP

<img src="https://raw.githubusercontent.com/Coming98/pictures/main/image-20210923154117728.png" alt="image-20210923154117728" style="zoom:50%;" />

# 无线网络协议安全威胁

这里主要介绍恶意访问接入点，其方式有二：

## 恶意AP假冒成合法AP

恶意 AP 使用相同或相似的 SSID 标识，冒充正规 AP ，拦截流量
> 无线客户端探测软件会误认为该伪造 AP 是合法 AP，于是就连接上此无线接入点

## 局域网内部人员搭建无线 AP

内部人员搭设无线 AP 是不安全的，容易让黑客绕过防火墙


# 无线网络协议安全机制

隐藏 SSID 为最简单方便的保证无线网络安全的手段之一

## MAC 层用户接入管理过程

### Scanning

STA 寻找一个无线网络

1、Passive Scanning：找到时间较长但是省电
> 通过侦听 AP 定期发送的 Beacon 帧来发现网络

2、Active Scannign：能迅速找到
> STA 在每个信道上发送 Probe request 报文，从 AP 回应的 Probe response 中获取 SSID 信息
> 应用于发现隐藏 AP

### Authentication

![shared-key authentication](https://raw.githubusercontent.com/Coming98/pictures/main/20211205143718.png)

采用 shared-key authentication 进行认证，加密算法为 WEP
> Attackers 可以通过监听 AP 发送的明文和 STA 回复的密文计算出 WEP key
> 因此 WEP 已经不再安全，共享秘钥认证方式已经被 WPA/WPA2 代替

详情将 [加密方式](##加密方式)

### Association

![Association](https://raw.githubusercontent.com/Coming98/pictures/main/20211205143915.png)

关联，STA 和 AP 建立关联，后续的数据报文的收发只能和建立Association关系的AP进行

### 数据收发

略


## CSMA/CA

![CSMA/CA](https://raw.githubusercontent.com/Coming98/pictures/main/20211205144057.png)

带冲突避免的载波侦听多路访问

1、冲突避免：送出数据前监听媒体状态，确定没人使用，维持一段时间，再（随机）等待一段随机时间后，依然无人使用送出数据 

2、送出数据前，发送请求传送报文（RTS）给目标端，等待目标端回应 CTS 报文后才，开始传送

> 询问 + 应答：先站住通信时间
>
> 缺点：网络的效率下降
>
> 优点：两种控制帧都很短，开销小。相反，若不使用，则一旦发生冲突而导致数据帧重发，则浪费的时间就更大。

## 加密方式

(仅作了解，Forget it！)
WEP，Wired Equivalent Privacy，有线对等私有协议 与 WPA/WPA2，Wi-Fi Protected Access，无线保护接入

### WEP

Wired Equivalent Privacy，有线对等私有协议

1、初始化向量 IV + 密码 PASSWORD + DATA 明文数据 + CRC-32 明文的完整性校验值

2、得到 KSA = IV + PASSWORD
> IV向量太短，大量监听用户数据报文后，WEP加密很容易被破解

3、经过 RC4 伪随机数密钥加密得到 PRGA = RC4（KSA）
> RC4加密算法过于简单

4、对 PRGA 与 DATA + CRC-32 进行异或加冕算法

5、得到密文 ENCRYPTED DATA

6、将密文与 IV 一起发送

> 加密整个网络共用一个共享密钥，一旦丢失，整个网络都很危险

**解密**

![image-20211005192237440](https://raw.githubusercontent.com/Coming98/pictures/main/image-20211005192237440.png)

### WPA/WPA2

Wi-Fi Protected Access，无线保护接入，了解即可，不必深究

WPA密码破解时，只要能抓到用户和AP之间的4次握手包中包含的和密码有联系的信息，采用字典进行暴力破解，能否破解就取决于字典和密码的复杂程度了