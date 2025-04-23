使用方式

```bash
exasock nc -u -l 1234
```

如果找不到 <https://github.com/cisco/exanic-software/blob/master/util/exasock-stat.c>

需要先安装

```bash
dnf install -y libnl3-devel
```

然后单独编译此工具

```bash
make -C util exasock-stat
```

执行

```bash
./util/exasock-stat
```

可以看到如下输出

```bash
Active ExaNIC Sockets accelerated connections (servers and established):
 Proto | Recv-Q   | Send-Q   | Local Address            | Foreign Address          | State       
 UDP   | 0        | 0        | *:1234                   | *:*                      | -  
```

