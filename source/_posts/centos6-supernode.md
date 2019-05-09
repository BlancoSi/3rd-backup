---
title: N2N简易食用指南
date: 2019-05-09 14:08:26
tags:
---

# N2N简易食用指南

## 简介：

1. N2N是一款开源的软件，作者是著名的开源网管软件ntop的作者Luca Deri。
2. N2N是基于P2P协议之上的两个私有网络间的加密层。
3. 加密是在edge节点上执行的，使用开放的协议，用户自己定义密钥，过程完全自治。
4. 每个n2n用户可以同时隶属于多个网络
5. 请了解[内网穿透](https://baike.baidu.com/item/内网穿透/8597835?fr=aladdin)的原理

## N2N架构组件：

1. Supernode：超级节点
它在edge节点间建立握手，或为位于防火墙之后的节点中转数据。它的基础作用是注册节点的网络路径，并为不能直通的节点做路由，能够直通的节点间通信，是P2P的。
2. Edge：边缘（节点）
用户PC机上安装的用于建立n2n网络的软件。几乎每个edge节点都会建立一个tun/tap设备，作为接入n2n网络的入口。
### 原理：super node提供场所，让两个位于NAT/防火墙之后的edge node进行会面，一旦双方完成首次握手，剩下的数据流就之发生在两个edge node之间，如果有一方的NAT属于对称型(symmetrical)，supernode则还需继续为双方提供数据包的转发;edge node负责数据流的加解密。加密算法采用了twofish，开源、简便，处理速度快。

## 源码地址：

### `https://github.com/meyerd/n2n.git`

#### 注1：此源码地址为n2n的开发分支，meyerd/n2n，由于不明原因，官方即ntop/n2n的源码是不能编译的。
#### 注2：n2n源码没有包含gui应用程序，这里也没有提供，所以本文中介绍的全部关于使用n2n的操作都是命令行操作。
#### 注3：n2n目前有v1 v2两个版本，区别是v2安全性高一点。本文使用v2。使用时注意edge和supernode版本一致，最好同时git源码在本地编译。

## 本人使用环境：

1. Supernode：vultr上的64位centos6
2. Edge：win10

## 配置Supernode端：

1. 建立或者使用已有的vps，然后用Xshell或者JuiceSSH连接到vps
2. yum install gcc gcc-c++ openssl-devel git cmake
3. 下载源码
```	
git clone https://github.com/meyerd/n2n.git
```
4. 编译安装：	
```
		cd n2n/n2n_v2
		mkdir build
		cd build
		cmake ..
		make
	//不install也行，不过需要在n2n/n2n_v2/build文件夹里面运行，而且要加./前缀
	//如./supernode -l 5000
		make install
		cd
```
5. 测试下能不能开启Supernode服务：	
```
		supernode -l 5000 -f
	//-f为前台运行，关闭终端或ctrl+c来停止服务
```
6. 开放端口：	
```
		/sbin/iptables -I INPUT -p tcp --dport 5000 -j ACCEPT
		/sbin/iptables -I INPUT -p udp --dport 5000 -j ACCEPT
		/sbin/iptables -I INPUT -p udp --dport 5645 -j ACCEPT
	//5645换成53也行，原理暂时不明
```
7. 查看端口状态：
```
		/etc/init.d/iptables status
```
8. 保存端口设置：
```
		/etc/rc.d/init.d/iptables save
	//不保存也行，不过每次重启都要再次开放端口
```
9. 开启supernode：
```	
		cd n2n/n2n_v2/build
		./supernode 5000
```
10. （可选）在服务器上开启edge：
```
		./edge -a 10.0.0.0 -c 群组名 -k 此处密码 -l supernode的IP地址:5000
```

## 配置Edge端：
1. 我用的是GitHub Desktop，所以这里直接把n2n源码clone下来，你也可以去直接下载zip包
2. cmake（也可以用Cmake-GUI代替）：	
```
		cd E:\github\n2n_meyerd\n2n_v2
		md build
		cmake ..
```
注：由于种种原因微软vs中的MSBuild在Powershell下的操作默认是不开放的，默认情况下只能在Developer command prompt中使用（在开始菜单的vs项目里面），而配置Powershell下使用也会很麻烦，所以不推荐使用。
3. 编译：
>
		打开build文件夹中的n2n.sln
		生成项目ALL_BUILD
		生成完成之后就可以在Release文件夹中看到edge.exe和supernode.exe等文件了
>
	//~~没卵用的题外话：官方即ntop/n2n的源码cmake时会报错，请关闭CMakeList.txt文件中的ssl~~
4. win10（最好在管理员限权）powershell开启edge：
```
		cd E:\github\n2n_meyerd\n2n_v2\build\Release
		.\edge -a 10.0.0.100 -c 群组名 -k 此处密码 -l supernode的IP地址:5000
```

## 至于命令的用法：

### 1. 一般命令行用法：
```
sudo ./edge -d n2n0 -c mynetwork -k encryptme -u 99 -g 99 -m 3C:A0:12:34:56:78 -a 1.2.3.4 -l a.b.c.d:xyw
或者
N2N_KEY=encryptme sudo ./edge -d n2n0 -c mynetwork -u 99 -g 99 -m 3C:A0:12:34:56:78 -a 1.2.3.4 -l a.b.c.d:xyw
```
### 2. you can join a private network with your friends by:
1. same -l parameter, connect to same a supernode
2. same -c and -k parameters, you can understand as "open same a lock with the same key"
3. difference -a parameter, also difference -m parameter if you specified
4. difference -d parameter with your local existed network interface card
	
### 3. 注意在powershell下 ./ 要用 .\ 代替
## 请参考：
	1. https://github.com/meyerd/n2n/wiki/run
	2. https://github.com/ntop/n2n
	3. https://www.baidu.com/
	4. https://baike.baidu.com/item/内网穿透/8597835?fr=aladdin
