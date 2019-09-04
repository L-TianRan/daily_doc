# gdb带参数调试
今天跑程序跑出一个coredump。不知道怎么回事，需要调试一下，但是程序a是带参数的，不知道怎么调。
```bash
$ ./a -N 2 < address
coredump
```

查了一下,应该这样执行
```
$ gdb a
$ r -N 2 < address
```
