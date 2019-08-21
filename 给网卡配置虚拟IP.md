# 给网卡配置虚拟IP
## 我的网络拓扑
- 我现在本机A 的IP是`10.30.5.204/19`,我需要用远程桌面连接到虚拟机B，B 的IP是`10.30.14.242/19`
- B还有一块网卡B2的IP是`172.16.14.242/16`，B2再用ssh连接到服务器C上工作，C 的IP是`172.16.32.104/16`。
- A和B通信时的网关都是10.30.0.254
- B2没有默认网关，C的默认网关是172.16.0.254/16

## 我的需求
本来我是想让A能和C的第二块网卡C2进行通信的。但是A要是能和C进行通信，那和C2肯定也不成问题。

## 网管给出的解决方案
- 给C配置一个虚拟IP
  - 虚拟IP为10.30.0.104/19
  - 把C的默认网关设为10.30.0.254
我也不知道可不可行，我还是第一次听说虚拟IP这玩意，就是一块网卡多个IP。边查资料边整吧。

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
$ sudo ifconfig enp2s0f0:0 10.30.0.104 netmask 255.255.224.0 up
$ ifconfig
# 多了一个这个
enp2s0f0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.30.0.104  netmask 255.255.224.0  broadcast 10.30.31.255
        ether 9c:e3:74:16:ff:6c  txqueuelen 1000  (Ethernet)
        device memory 0x92500000-925fffff  
```

### 永久配置
如果需要永久配置，需要修改配置文件。这里不详细介绍了，有需要可以查看这里。
