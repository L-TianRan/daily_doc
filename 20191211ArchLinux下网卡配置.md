# ArchLinux下网卡配置
ArchLinux很少用，过会不用就蛋疼。
网卡配置文件在`/etc/rc.conf`
```zsh
$ cat /etc/rc.conf 
interface=enp1s0f0
eth0="static"
address=172.16.32.210
netmask=255.255.0.0
gateway=172.16.0.254
```


重启网络服务用`systemctl restart netctl.service`

Read the wiki on [archwiki](https://wiki.archlinux.org/index.php/Systemd#Using_units)
