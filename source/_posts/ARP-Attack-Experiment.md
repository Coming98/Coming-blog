---
title: ARP Attack Experiment
mathjax: false
date: 2021-09-25 19:11:00
summary: 虚拟机中实践 ARP 欺骗
categories: Security
tags:
  - Experiment
  - Security
---

# Tools

## Cain & WinPcap

Cain 工具依赖于 WinPcap，使用 Cain 时需要给与管理员权限

Cain Version：4.9

Whole Download Url：http://www.pc0359.cn/downinfo/79463.html

![image-20210924193818343](https://gitee.com/Butterflier/pictures/raw/master/image-20210924193818343.png)

![image-20210924193834801](https://gitee.com/Butterflier/pictures/raw/master/image-20210924193834801.png)

## Wireshark

官网下载即可

# Prepare

Perpetrator A: Windows_7_sp1_x64

User B: Windows_xp_professional_x86

Server C: Windows_2008_r2，存在网页服务（www.flower.com）

# Process

## 简单的 ARP 欺骗

**0、**将 A，B，C 三台主机放到 VMnet 3 交换机中（网关统一设置为 10.1.1.1）：

A 的 ip 为 10.1.1.10，MAC 为 00-0C-29-30-A8-BD

B 的 ip 为 10.1.1.11，MAC 为 00-0C-29-F0-B1-3A

C 的 ip 为 10.1.1.12，MAC 为 00-0C-29-6B-91-F5

最后使用 ping 命令保证 A 与 B 与 C 之间的互相连通

1、打开 Cain 工具，单击工具栏中的配置，对 嗅探器 进行配置，选择 IP 地址与网关 IP对应的适配器即可

![image-20210924202538785](https://gitee.com/Butterflier/pictures/raw/master/image-20210924202538785.png)

2、在主界面 - 嗅探器 - 主机中，点击左侧第二个按钮 开始/停止嗅探后，在空白处右击即可选择开始扫描 MAC

![image-20210924202746250](https://gitee.com/Butterflier/pictures/raw/master/image-20210924202746250.png)

3、可以看出，我们扫到了 User B 与 Service C ！

![image-20210924202830077](https://gitee.com/Butterflier/pictures/raw/master/image-20210924202830077.png)

4、在主界面 - 嗅探器 - ARP 处，单击工具栏中的 加号按钮，添加 User B 到 Service C 的 ARP 欺骗

![image-20210924203217137](https://gitee.com/Butterflier/pictures/raw/master/image-20210924203217137.png)

5、攻击前先查看 User B 与 Service C 的 arp 缓存信息（之前 ping 命令获取的）

![image-20210924203404086](https://gitee.com/Butterflier/pictures/raw/master/image-20210924203404086.png)

![image-20210924203459563](https://gitee.com/Butterflier/pictures/raw/master/image-20210924203459563.png)

6、开始攻击！点击开始/停止 ARP 攻击按钮

![image-20210924203559237](https://gitee.com/Butterflier/pictures/raw/master/image-20210924203559237.png)

7、再次查看 User B 与 Service C 的 arp 缓存信息

![image-20210924204046638](https://gitee.com/Butterflier/pictures/raw/master/image-20210924204046638.png)

![image-20210924204103319](https://gitee.com/Butterflier/pictures/raw/master/image-20210924204103319.png)

攻击成功！

## 欺骗后监听通信

0、为了方便监听，咋 Service C 上开启 IIS 与 DNS 服务，开启网站 www.flower.com

![image-20210924210219293](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210219293.png)

设定好 DNS，并将 User B 的 默认 DNS 指向尾 Service C

![image-20210924210358844](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210358844.png)

验证

![image-20210924210523600](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210523600.png)

1、根据任务 1 完成 ARP 欺骗后，开启 Wireshark，监听源IP 为 User B 或 Service C 的包

![image-20210924210724905](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210724905.png)

2、使用 User B 访问 www.flower.com

首先应该看到的是 10.1.1.11 与 10.1.1.12 的 DNS 相关请求，可以获取 User B 的访问域名：

![image-20210924211232666](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211232666.png)

可以看到中间人将收到的包看了一眼后转发到了正确的地址，使得 User B 正常访问 www.flower.com 

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924211330279.png" alt="image-20210924211330279" style="zoom:50%;" />

随后就是 TCP 三次握手：

![image-20210924211425093](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211425093.png)

然后是 HTTP

![image-20210924211559686](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211559686.png)

3、接下来我尝试使用 GET 传送一些参数进去，看看是否能够获取到呢？

这是 URL：

![image-20210924211753656](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211753656.png)

查看抓包详细结果：

因为 DNS 缓存的原因，这次没有 DNS 的相关抓包信息

![image-20210924211857375](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211857375.png)

可以看出，GET 请求中的参数将会被完全轻松的获取到：

![image-20210924212025745](https://gitee.com/Butterflier/pictures/raw/master/image-20210924212025745.png)

## 欺骗网关之不同局域网的欺骗

Perpetrator 为 ubuntu 系统

使用 Perpetrator A 欺骗 User B 与 路由器

Perpetrator MAC 地址： 00-0c-29-8a-05-cc

User B MAC 地址：00-0C-29-F0-B1-3A

路由器 网关 MAC 地址：88-25-93-27-8B-EE

0、将 Perpetrator A 与 User B 桥接至路由器中

得到 Perpetrator A IP 为 192.168.1.110，网关为 192.168.1.1

![image-20210925135745437](https://gitee.com/Butterflier/pictures/raw/master/image-20210925135745437.png)

得到 User B 的 IP 为 192.168.1.109，网关为 192.168.1.1

![image-20210925131922146](https://gitee.com/Butterflier/pictures/raw/master/image-20210925131922146.png)

1、使用 User B 访问 www.baidu.com 后查看 User B 和 路由器的 ARP 映射表

User B 的 ARP 映射表：

![image-20210925132622087](https://gitee.com/Butterflier/pictures/raw/master/image-20210925132622087.png)

路由器 有关 User B 的 ARP 映射表：

![image-20210925132710815](https://gitee.com/Butterflier/pictures/raw/master/image-20210925132710815.png)

2、Perpetrator A 中执行 arpspoof.py 文件
