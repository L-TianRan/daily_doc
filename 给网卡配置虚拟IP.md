# 给网卡配置虚拟IP
## 我的网络拓扑
- 我现在本机A 的IP是`10.30.5.204/19`,我需要用远程桌面连接到虚拟机B，B 的IP是`10.30.14.242/19`
- B还有一块网卡B2的IP是`172.16.14.242/16`，B2再用ssh连接到服务器C上工作，C 的IP是`172.16.32.104/16`。
- A和B通信时的网关都是`10.30.0.254`
- B2没有默认网关，C的默认网关是`172.16.0.254/16`

## 我的需求
本来我是想让A能和C的第二块网卡C2进行通信的。但是A要是能和C进行通信，那和C2肯定也不成问题。

## 网管给出的解决方案
- 给C配置一个虚拟IP
  - 虚拟IP为`10.16.0.104/19`。
  - 把C的默认网关设为`10.16.0.254`。

看起来有点奇怪，但是是可行的，可能因为还有其他网络连接在一起。我还是第一次听说虚拟IP这玩意，就是一块网卡多个IP。这次就当学习怎么配虚拟IP吧。

## 配置命令
### 临时配置
有时候可能只是临时使用，那么用命令临时配置就行了。
```
# 查看C的现有配置
$ ifconfig           
enp2s0f0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.32.104  netmask 255.255.0.0  broadcast 127.16.255.255
        inet6 fdea:2e7d:6b69:0:9ee3:74ff:fe16:ff6c  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::9ee3:74ff:fe16:ff6c  prefixlen 64  scopeid 0x20<link>
        ether 9c:e3:74:16:ff:6c  txqueuelen 1000  (Ethernet)
        RX packets 87556813  bytes 5753915494 (5.3 GiB)
        RX errors 0  dropped 67810  overruns 3  frame 0
        TX packets 4361219  bytes 5654841610 (5.2 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device memory 0x92500000-925fffff  
```
```
# 增加一块虚拟网卡
$ sudo ifconfig enp2s0f0:0 10.16.0.104 netmask 255.255.255.0 up
$ ifconfig
# 多了一个这个
enp2s0f0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.16.0.104  netmask 255.255.255.0  broadcast 10.16.0.255
        ether 9c:e3:74:16:ff:6c  txqueuelen 1000  (Ethernet)
        device memory 0x92500000-925fffff  
```
```
# 添加默认路由
$ sudo route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.16.0.254 dev enp2s0f0:0
$ route
Kernel IP routing table       //两个默认路由
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.16.0.254     0.0.0.0         UG    0      0        0 enp2s0f0
default         172.16.0.254    0.0.0.0         UG    0      0        0 enp2s0f0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 enp2s0f0
........
$ route -n                    // -n查看详细信息
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.16.0.254     0.0.0.0         UG    0      0        0 enp2s0f0
0.0.0.0         172.16.0.254    0.0.0.0         UG    0      0        0 enp2s0f0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 enp2s0f0
........
```

### 取消配置
只需要删除虚拟IP，删除默认路由就行。
```
$ sudo ifconfig enp2s0f0:0 down
$ sudo route del -net 0.0.0.0 netmask 0.0.0.0
               //删默认路由的时候小心别删错了，删错了把原来的默认路由给加上
```

### 永久配置
如果需要永久配置，需要修改配置文件。这里不详细介绍了，有需要可以[查看这里](https://blog.csdn.net/ygm_linux/article/details/81532478)。

## 初步结果
按上述过程配置之后，A和B`ping 10.16.0.104`都能通了。这个IP就是C 网卡的虚拟IP。但是现在C不能上网了，`ping baidu.com` 不通。所以我得写个脚本一键配置、一键还原。这个后面写

## 另一块网卡
上面已经说了，我实际目的是让A、B能和C的另一块dpdk网卡通信，所以我把C的另一块dpdk网卡按上述过程配置一下。当然，之前`enp2s0f0`网卡的虚拟IP配置和默认路由不能删。
```
$ sudo ifconfig dpdk0 10.16.0.105 netmask 255.255.255.0 dev dpdk0
//这样让dpdk0和enp2s0f0:0 在同一个网段
```
现在在A和B上`ping 10.16.0.105`都是通的。这个IP就是我想通信的C2网卡。

## 配置脚本
写个Makefile，用来设置和删除服务器C上enp2s0f0的默认路由。
```
$ vim Makefile
# This file is to config a default gateway on enp2s0f0 for communication between my PC and server.
# Run this file as sudo or root.
# After communication, please clean the configure, or the server cannot connect to the Internet.

defgw:
	sudo route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.16.0.254 dev enp2s0f0:0

clean:
	sudo route del -net 0.0.0.0 netmask 0.0.0.0 dev enp2s0f0:0

```
需要和服务器通信的时候就使用`make defgw`，通信结束后使用`make clean`，使服务器恢复联网。
