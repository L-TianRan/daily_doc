# linux的shell中`ctrl+s`是什么意思
刚才在shell里想按`ctrl+a`不小心按到了`ctrl+s`，然后整个终端就不动了。还以为死机了差点reboot。

查了一下才知道有几个特殊的快捷键：
- `ctrl+s`和`ctrl+q`
  - `ctrl+s`: 在终端下是有特殊用途的，那就是暂停该终端，
  - `ctrl+q`: 退出这种状态，让终端继续运行
  - **即便终端在锁定状态下，你输入的命令虽然无法在屏幕上显示出来，但是敲下回车的时候还是会执行的，锁定的时候可别在键盘上乱按乱点**
  
- `ctrl+c`、`ctrl+d`和`ctrl+z`
  - `ctrl+c`: 发送terminal，终止当前正在执行的程序。
  - `ctrl+d`: 是发送一个exit信号，退出终端或者退出root用户
  - `ctrl+z`: 是把当前的程序挂起，暂停执行这个程序，但是进程仍然存在。需要配合`fg`/`bg`命令使用。
    - fg命令重新启动前台被中断的任务。
    - bg命令把被中断的任务放在后台执行。
    - ps：我怀疑我这个mtcp程序报错就是因为用了`ctrl+z`，之后又忘了切换回来。[Cannot init mbuf pool, errno: 17](https://github.com/L-TianRan/daily_doc/blob/master/dpdk错误码查找.md)
