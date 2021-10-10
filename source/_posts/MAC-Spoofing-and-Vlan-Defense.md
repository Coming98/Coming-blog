---
title: MAC Spoofing and Vlan Defense
mathjax: false
date: 2021-10-10 19:50:36
summary: 局域网下MAC欺骗与VLAN防御实践
categories: Security
tags:
  - Experiment
  - Security
---

# Prepare

**Packet Tracer**：[官网](https://www.netacad.com/courses/packet-tracer/introduction-packet-tracer)完成注册后下载即可

# MAC 欺骗

## 1、搭建网络结构

需要注意 PC 与交换机之间采用**直通线**互连，交换机与交换机之间采用**交叉线**互连

> 直通线：一般连接不同的设备
>
> 交叉线：一般用于相同设备的连接

![image-20211009160503527](https://gitee.com/Butterflier/pictures/raw/master/image-20211009160503527.png)

## 2、配置 IP

点击目标 PC ，在 Config - FastEthernet0 中配置静态 IP 地址

| Host | IP           | MAC            |
| ---- | ------------ | -------------- |
| PC0  | 192.168.1.10 | 0090.0CB0.55B6 |
| PC1  | 192.168.1.11 | 0002.1625.5207 |
| PC2  | 192.168.1.12 | 0006.2A16.0A3C |

![image-20211009160855791](https://gitee.com/Butterflier/pictures/raw/master/image-20211009160855791.png)

## 3、测试

使用 ICMP 报文（ping）完成 PC0,1,2 之间的通信测试

点击 PC0 在 Desktop - Command Prompt 中输入 ping 命令即可

![image-20211009161413510](https://gitee.com/Butterflier/pictures/raw/master/image-20211009161413510.png)

![image-20211009161841196](https://gitee.com/Butterflier/pictures/raw/master/image-20211009161841196.png)

## 4、查看正确的转发表

Tips：每台 PC 都要对其它 PC 进行一次成功的 ping 操作才能得到完整的转发表；

交换机转发表查看方式：

a、点击目标交换机进入 CLI 窗口

b、窗口中一直输入 `exit` 命令（小白做法）直到到达用户模式（`Swtich>`）

c、输入 `enable` 进入特权模式

d、输入 `show mac-address-table`

![image-20211009162900224](https://gitee.com/Butterflier/pictures/raw/master/image-20211009162900224.png)

## 5、模拟操作模式查看正常通信过程

a、点击右下角的 simulation 进入模拟操作模式

b、点击 Edit Filters 配置好过滤器，使得能够检测到 ICMP 报文的转发过程

![image-20211009164036277](https://gitee.com/Butterflier/pictures/raw/master/image-20211009164036277.png)

c、使用 PC1 ping PC0 查看过程动画

![image-20211009164047994](https://gitee.com/Butterflier/pictures/raw/master/image-20211009164047994.png)

## 6、MAC 欺骗

切换回实时操作模式，将 PC2 的 MAC 地址更改为 PC0 的 MAC 地址

![image-20211009165759568](https://gitee.com/Butterflier/pictures/raw/master/image-20211009165759568.png)

## 7、通信欺骗

使用 PC2 对 PC1通信后查看转发表信息：可以看到 PC2 成功伪造了 PC0 的身份，欺骗了各个交换机：Switch0 认为 0090…. 应该走端口 3 去找 Switch 1，Switch 1 认为 0090 应该走端口 2 去找 Switch 2，Switch 2 认为 0090…. 应该走端口 1 ，即找 PC2 

![image-20211009170222140](https://gitee.com/Butterflier/pictures/raw/master/image-20211009170222140.png)

## 8、模拟 PC1 与 PC0 的通信

切换到模拟操作模式：

a、PC1 ping PC0 的 IP 地址

![image-20211009170655929](https://gitee.com/Butterflier/pictures/raw/master/image-20211009170655929.png)

b、查看动画，发现 PC2 欺骗成功

![image-20211009170826102](https://gitee.com/Butterflier/pictures/raw/master/image-20211009170826102.png)

# VLAN 防 MAC 欺骗

Target：通过 VLAN 的划分，隔离攻击者 PC2 与 PC1，使得 PC2 无法冒充”友军“

![image-20211009171601483](https://gitee.com/Butterflier/pictures/raw/master/image-20211009171601483.png)

## 1、添加 VLAN

```shell
Switch>enable					# 进入特权模式
Switch\# configure terminal 	# 进入全局配置模式
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)\#vlan 2			# 创建vlan（id=2）并进入其配置模式
Switch(config-vlan)\#name vlan2 # 对该 vlan 定义一个名称
```

## 2、分配接入端口

```shell
Switch\#configure terminal 
Enter configuration commands, one per line.  End with CNTL/Z.

# 进入 1 号端口的配置模式
Switch(config)\#interface FastEthernet 0/1
# 将该端口指定为接入端口
Switch(config-if)\#switchport mode access 
# 将该接入端口分配给 vlan 2
Switch(config-if)\#switchport access vlan 2
Switch(config-if)\#exit			# 回到全局配置模式
# 同理配置 2 号端口，并分给给 vlan 2
Switch(config)\#interface FastEthernet 0/2
Switch(config-if)\#switchport mode access 
Switch(config-if)\#switchport access vlan 2
```

## 3、查看配置结果

在特权模式下输入命令 `show running-config`

可以查看到 1 号端口和 2 号端口均分配给了 VLAN 2

![image-20211009172655898](https://gitee.com/Butterflier/pictures/raw/master/image-20211009172655898.png)

## 4、检验结果

a、当前状态 Switch 转发表 遗忘了之前的信息，攻击者 PC2 有机可乘

![image-20211009172940185](https://gitee.com/Butterflier/pictures/raw/master/image-20211009172940185.png)

> 该 mac-address 是 Switch 0 与 Switch 1 之间的

b、PC2 使用 PC0 的 MAC 地址，ping PC1，可以看出到达 Switch 0 后，因为不属于一个 VLAN 包就被丢弃了。欺骗失败！

![image-20211009173426989](https://gitee.com/Butterflier/pictures/raw/master/image-20211009173426989.png)

c、在检测下 PC0 能否与 PC1 正常通信：通信成功

![image-20211009173638696](https://gitee.com/Butterflier/pictures/raw/master/image-20211009173638696.png)
