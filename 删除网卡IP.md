# 删除网卡IP
今天我给网卡配了一个IP，接着想把它删除。不知道咋删。
```bash
$ ifconfig enp3 192.168.1.4 netmask 255.255.255.0 up
$ ifconfig enp3 down        // down了之后网卡IP还在
```
百度了之后说需要用IP命令
```bash
$ ip addr delete 192.168.1.4/24 dev enp3 
$ ifconfig enp3    // 没有IP了
```




