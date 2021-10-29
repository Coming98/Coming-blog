---
title: ARP Attack Experiment
mathjax: false
date: 2021-09-25 19:11:00
summary: 虚拟机中实践 ARP 欺骗
categories: Security
tags:
  - experiment
  - network security
---
# Tools

## Cain & WinPcap

**Version**: 4.9

**Download**: http://www.pc0359.cn/downinfo/79463.html

**Tips**: Cain 工具依赖于 WinPcap，使用 Cain 时需要给与管理员权限

![image-20210924193818343](https://gitee.com/Butterflier/pictures/raw/master/image-20210924193818343.png)

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924193834801.png" alt="image-20210924193834801" style="zoom:50%;" />

## Wireshark

官网下载即可 [http://www.wireshark.org](http://www.wireshark.org/)

# Ⅰ同一局域网内的 ARP 欺骗

## 0 Prepare

| 主机          | OS                          | MAC               |
| ------------- | --------------------------- | ----------------- |
| Perpetrator A | Windows_7_sp1_x64           | 00-0C-29-30-A8-BD |
| User B        | Windows_xp_professional_x86 | 00-0C-29-F0-B1-3A |
| Server C      | Windows_2008_r2             | 00-0C-29-6B-91-F5 |

## 1 初始化各个主机

将 A，B，C 三台主机放到 VMnet 3 交换机中（网关统一设置为 10.1.1.1）,最后使用 ping 命令保证 A 与 B 与 C 之间的互相连通

| 主机 | IP        |
| ---- | --------- |
| A    | 10.1.1.10 |
| B    | 10.1.1.11 |
| C    | 10.1.1.12 |

## 2 初始化 Cain 嗅探器

打开 Cain 工具，单击工具栏中的配置，对嗅探器进行配置，选择 IP 地址与网关 IP 对应的适配器即可

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924202538785.png" alt="image-20210924202538785" style="zoom:50%;" />

## 3 开始嗅探 - 扫描 MAC

在主界面 - 嗅探器 - 主机中，点击左侧第二个按钮 开始/停止嗅探后，在空白处右击即可选择开始扫描 MAC  

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924202746250.png" alt="image-20210924202746250" style="zoom:50%;" />

可以看出，我们扫到了 User B 与 Service C ！

![image-20210924202830077](https://gitee.com/Butterflier/pictures/raw/master/image-20210924202830077.png)

## 4 准备攻击

在主界面 - 嗅探器 - ARP 处，单击工具栏中的 加号按钮，添加 User B 到 Service C 的 ARP 欺骗

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924203217137.png" alt="image-20210924203217137" style="zoom:50%;" />

攻击前先查看 User B 与 Service C 的 ARP 缓存信息（之前 ping 命令获取的）

User B 的 ARP 缓存信息：可以看出其正确的记录了 Service C 的 MAC 地址

![image-20210924203404086](https://gitee.com/Butterflier/pictures/raw/master/image-20210924203404086.png)

Service C 的 ARP 缓存信息：可以看出其正确的记录了 User B 的 MAC 地址

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924203459563.png" alt="image-20210924203459563" style="zoom:50%;" />

## 5 开启攻击

开始攻击！点击开始/停止 ARP 攻击按钮

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924203559237.png" alt="image-20210924203559237" style="zoom:50%;" />

## 6 攻击成功

再次查看 User B 与 Service C 的 ARP 缓存信息

User B 的 ARP 缓存信息：可以看出 Perpetrator A 成功欺骗了 User B 将 Service C 的 MAC 地址改为了自身的 MAC 地址

![image-20210924204046638](https://gitee.com/Butterflier/pictures/raw/master/image-20210924204046638.png)

Service C 的 ARP 缓存信息：可以看出 Perpetrator A 成功欺骗了 Service C 将 User B 的 MAC 地址改为了自身的 MAC 地址

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924204103319.png" alt="image-20210924204103319" style="zoom:50%;" />

# Ⅱ欺骗后监听通信

## 0 Prepare

| 主机          | OS                          | MAC               | IP        |
| ------------- | --------------------------- | ----------------- | --------- |
| Perpetrator A | Windows_7_sp1_x64           | 00-0C-29-30-A8-BD | 10.1.1.10 |
| User B        | Windows_xp_professional_x86 | 00-0C-29-F0-B1-3A | 10.1.1.11 |
| Server C      | Windows_2008_r2             | 00-0C-29-6B-91-F5 | 10.1.1.12 |

为了方便监听，在 Service C 上开启 IIS 与 DNS 服务，开启网站 www.flower.com

![image-20210924210219293](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210219293.png)

设定好 DNS，并将 User B 的 默认 DNS 指向为 Service C

![image-20210924210358844](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210358844.png)

验证

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924210523600.png" alt="image-20210924210523600" style="zoom:50%;" />

## 1 初始化 Wireshark

按照同一局域网内的 ARP 欺骗步骤完成 ARP 欺骗后，开启 Wireshark，监听源IP 为 User B 或 Service C 的包

![image-20210924210724905](https://gitee.com/Butterflier/pictures/raw/master/image-20210924210724905.png)

## 2 查看监听结果

使用 User B 访问 www.flower.com，首先应该看到的是 10.1.1.11 与 10.1.1.12 的 DNS 相关请求，可以获取 User B 的访问域名：

![image-20210924211232666](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211232666.png)

Tips：可以看到中间人将收到的包看了一眼后转发到了正确的地址，使得 User B 正常访问 www.flower.com 

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210924211330279.png" alt="image-20210924211330279" style="zoom:50%;" />

随后就是 TCP 三次握手：

![image-20210924211425093](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211425093.png)

然后是 HTTP：

![image-20210924211559686](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211559686.png)

## 3 尝试获取重要信息

接下来我尝试使用 GET 传送一些参数进去，看看是否能够获取到呢？

![image-20210924211753656](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211753656.png)

查看抓包详细结果：因为 DNS 缓存的原因，这次没有 DNS 的相关抓包信息

![image-20210924211857375](https://gitee.com/Butterflier/pictures/raw/master/image-20210924211857375.png)

可以看出，GET 请求中的参数将会被完全轻松的获取到：

![image-20210924212025745](https://gitee.com/Butterflier/pictures/raw/master/image-20210924212025745.png)

# Ⅲ 欺骗网关之不同局域网的欺骗

| 主机            | OS                          | MAC 地址          | IP            |
| --------------- | --------------------------- | ----------------- | ------------- |
| Perpetrator A   | Kail Linux 2021.2           | 00-0c-29-0c-b9-d0 | 192.168.1.111 |
| User B          | Windows_xp_professional_x86 | 00-0C-29-F0-B1-3A | 192.168.1.109 |
| 路由器 (网关) C | -                           | 88-25-93-27-8B-EE | 192.168.1.1   |

**Target**：使用 Perpetrator A 欺骗 User B 与 路由器

**Tips**：

& 为了方便实验，此时将 Perpetrator A 系统更为 Kail Linux 2021.2

& 所有设备都处于桥接状态，桥接模式中 VMWare 虚拟出来的操作系统就像是局域网中的一台独立的主机（主机和虚拟机处于对等地位）

## 1 初始化各个主机

将 Perpetrator A 与 User B 桥接至路由器中

得到 Perpetrator A IP 为 192.168.1.111，网关为 192.168.1.1

![image-20210925194951365](https://gitee.com/Butterflier/pictures/raw/master/image-20210925194951365.png)

得到 User B 的 IP 为 192.168.1.109，网关为 192.168.1.1

![image-20210925131922146](https://gitee.com/Butterflier/pictures/raw/master/image-20210925131922146.png)

## 2 查看攻击前状态

使用 User B 访问 www.baidu.com 后查看 User B 和 路由器的 ARP 映射表

User B 的 ARP 映射表：

![image-20210925132622087](https://gitee.com/Butterflier/pictures/raw/master/image-20210925132622087.png)

路由器 有关 User B 的 ARP 映射表：

![image-20210925132710815](https://gitee.com/Butterflier/pictures/raw/master/image-20210925132710815.png)

## 3 主机扫描

A 先使用 nmap 扫描一下本局域网，可以扫出网关 192.168.1.1，用户机 192.168.1.109

```shell
nmap -sP 192.168.1.0/24
```

![image-20210925195305365](https://gitee.com/Butterflier/pictures/raw/master/image-20210925195305365.png)



## 4 开始攻击

使用 arpspoof 命令开始 ARP 欺骗

```shell
# arpspoof -i eth0 -t 目标ip 主机ip（路由器）
arpspoof -i eth0 -t 192.168.1.109 192.168.1.1 # 将 109 发往 网关的包监听拦截
```

此时查看用户 B 的网络状态，发现已经无法上网：查看 B 的 ARP 缓存信息，可见 A 成功欺骗了 B，改写了网关 MAC 为 A 自己：

![image-20210925200125750](https://gitee.com/Butterflier/pictures/raw/master/image-20210925200125750.png)

查看路由器 C 的 ARP 缓存信息，并没有发现对 B MAC 的更改：是因为我们值攻击并监听了 192.168.1.109 -> 192.168.1.1 这一条信道，其实这已然可以实现我们想要的功能，只是很容易被 B 发现

![image-20210925202212149](https://gitee.com/Butterflier/pictures/raw/master/image-20210925202212149.png)

如果要双向监听，则在新开一个命令行窗口执行下方命令即可：

```shell
arpspoof -i eth0 -t 192.168.1.1 192.168.1.109
```

再次查看，发现路由器也被欺骗啦

![image-20210925202228390](https://gitee.com/Butterflier/pictures/raw/master/image-20210925202228390.png)

## 5 流量转发

因为 Linux 中默认禁止流量转发功能，因此 arpspoof 拦截了流量，客户机会立刻发现无法上网，为了更加隐蔽，需要开启流量转发功能

> 流量转发功能：当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的 ip 地址将包发往本机另一网卡，也就是我们监听拦截到数据报后，看一眼，然后 copy 一份，以另一个网卡发给目的地址

执行下方命令，开启流量转发，再次开启 ARP 双向欺骗后，这时 User B 是正常上网状态：

```shell
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210925203541401.png" alt="image-20210925203541401" style="zoom:50%;" />

<img src="https://gitee.com/Butterflier/pictures/raw/master/image-20210925203527816.png" alt="image-20210925203527816" style="zoom:50%;" />

## 6 流量嗅探

我们拦截了请求与响应包，应当得查看一眼里面有没有什么重要信息吧~

使用 `ettercap` 进行嗅探：尝试获取登录 sep.ucas.ac.cn 的用户名与密码信息

```shell
ettercap -Tq -i eth0
```

![image-20210925203914868](https://gitee.com/Butterflier/pictures/raw/master/image-20210925203914868.png)

开启成功后，当用户进行正常的请求时，`ettercap` 就能帮助我们嗅探到信息

![image-20210925204100160](https://gitee.com/Butterflier/pictures/raw/master/image-20210925204100160.png)

# END

实践下来真的十分的有意思，特别是 Kail Linux 中拥有丰富的工具，可以简单的让小白快速上手，实践所学，点赞！

