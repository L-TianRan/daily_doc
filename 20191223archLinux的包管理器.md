# archLinux的包管理器
要在archLinux下安装openMPI，不想通过源码安装，但是又不知道对应的包管理器是啥，怎么用。记一下。

## 包管理器pacman
```
# 搜索MPI包
$ pacman -Ss mpi

# 指定版本搜索
$ pacman -Ss "openmpi>=3.1.0"

# 安装包
pacman -S openmpi=4.0.1-1
```
