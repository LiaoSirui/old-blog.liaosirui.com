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

设置权限

```bash
If the SmartNIC is to be used in kernel bypass mode, it may be useful to set permissions on the SmartNIC device files (/dev/exanic0, etc, and the /dev/exasock device file used by ExaNIC Sockets) so that they can be accessed by certain non-root applications. Permissions can be set with chown/chgrp/chmod as normal, however this will not persist across reboots. A persistent configuration can be achieved by appropriate configuration of the udev daemon. For example, to force the device files to be accessible by a group called exanic, create a /etc/udev/rules.d/exanic.rules file as follows:

KERNEL=="exanic*", GROUP="exanic", MODE="0660"
KERNEL=="exasock", GROUP="exanic", MODE="0660"
In the current software release, the SmartNIC device files also expose the device control registers so only trusted users should be given access to the SmartNIC in this way.

If non-root users require utilities such as exanic-config, this can be accomplished by setting the appropriate Linux capability on the relevant binary, eg:

setcap cap_net_admin+ep $(which exanic-config)
Would allow non-root users to enable/disable interfaces via exanic-config.

```

