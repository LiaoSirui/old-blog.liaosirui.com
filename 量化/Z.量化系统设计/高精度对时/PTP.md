## PTP 简介

PTP是英文 “Precision Timing Protocol” 的缩写，即“精确时间协议”。IEEE 1588标准中对它进行了描述，它是一种通过分组网络分配时间的协议，主时钟发送报文到从时钟，告诉从时钟主时钟所处的时间在这个过程中，报文延迟是个棘手的问题，而精确时间协议（PTP）大部分都致力于解决这个问题。

协议使用三种关键技术来减少延迟估计中的误差：

1. 硬件时间戳 —— 精确记录事件报文通过物理接口的时间。这消除了识别和处理报文所涉及的软件延迟。
2. 边界时钟 —— 这类时钟恢复网络中间点的时间，并将时间转发到新的一组报文中。这些时钟通常可在网络中的交换机和路由器中找到，它们有助于减小网络延迟变化所带来的影响，列队延迟等因素都有可能导致这类影响。
3. 透明时钟 —— 这类时钟也可在网络中的交换机和路由器中找到。不同的是，它并不恢复并转发时间，而是记录报文通过交换机或路由器的时间。当报文最终到达从时钟时，透明时钟包含报文通过网络所累积的延迟信息，使得从时钟更准确地将其本地时间与主时钟对齐。

精确时间协议 (PTP) 支持亚微秒级准确性，较 NTP 更优。PTP 支持分为内核和用户空间。

## PTP 配置

PTP 管理的时钟遵循主从层次结构。从属时钟将同步到其主时钟。层次结构由每个时钟上运行的最佳主时钟 (BMC) 算法更新。只有一个端口的时钟可以是主时钟也可以是从属时钟。此类时钟称为普通时钟 (OC)。具有多个端口的时钟可以在一个端口上为主时钟，在另一个端口上为从属时钟。此类时钟称为边界时钟 (BC)。顶层主时钟称为超级主时钟。超级主时钟可与全球定位系统 (GPS) 同步。这样，不同的网络便能够以高准确度实现同步。

硬件支持是 PTP 的主要优势。多种网络交换机和网络接口控制器 (NIC) 都支持 PTP。虽然可以在网络内部使用启用非 PTP 的硬件，但为所有 PTP 时钟之间的网络组件启用 PTP 硬件可实现最大的准确性。

由 `linuxptp` 软件包提供 PTP 实施，其中包含用于时钟同步的 `ptp4l` 和 `phc2sys` 程序。`ptp4l` 实施了 PTP 边界时钟和普通时钟。如果启用硬件时戳，`ptp4l` 会将 PTP 硬件时钟同步到主时钟。如果使用软件时戳，它会将系统时钟同步到主时钟。仅当使用硬件时戳将系统时钟同步到网络接口卡 (NIC) 上的 PTP 硬件时钟时，才需要 `phc2sys`。

要在 Linux 系统中实现与 Cisco 3548 交换机的 PTP 对时，需结合 PTP 协议配置及硬件支持。

### 硬件与交换机准备

- 确认硬件支持

需使用支持 PTP 协议的网络接口卡（NIC），且交换机（如 Cisco 3548）需启用 PTP 功能并配置为时钟源或从时钟。

示例配置（Cisco 3548）：

```bash
switch(config)# ptp clock
switch(config-ptp-clock)# priority1 128  # 设置时钟优先级（0-255）
switch(config-ptp-clock)# announce-interval 1  # 广播间隔（秒）
```

- 网络配置

确保Linux系统与交换机处于同一子网，且 PTP 流量通过专用端口传输（如配置 VLAN 或 Trunk）。

### Linux 系统配置

参考文档：<https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ptp_using_ptp4l#sec-Understanding_PTP>

- 安装 PTP 工具

```bash
sudo apt install linuxptp ptp4l phc2sys  # Debian/Ubuntu
sudo dnf install linuxptp       # CentOS/RHEL
```

- 检查网卡是否支持

PTP 要求使用的内核网络驱动程序支持软件时戳或硬件时戳。此外，NIC 还必须支持物理硬件中的时戳。可以使用 `ethtool` 校验驱动程序和 NIC 时戳功能：

```bash
ethtool -T eth0

Time stamping parameters for eth0:
Capabilities:
    hardware-transmit   (SOF_TIMESTAMPING_TX_HARDWARE)
    software-transmit   (SOF_TIMESTAMPING_TX_SOFTWARE)
    hardware-receive   (SOF_TIMESTAMPING_RX_HARDWARE)
    software-receive   (SOF_TIMESTAMPING_RX_SOFTWARE)
    software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
    hardware-raw-clock  (SOF_TIMESTAMPING_RAW_HARDWARE)
PTP Hardware Clock: 3
Hardware Transmit Timestamp Modes:
        off
        on
Hardware Receive Filter Modes:
        none
        ptpv1-l4-sync
        ptpv1-l4-delay-req
        ptpv2-event
```

软件时戳需要以下参数：

```
SOF_TIMESTAMPING_SOFTWARE
SOF_TIMESTAMPING_TX_SOFTWARE
SOF_TIMESTAMPING_RX_SOFTWARE
```

硬件时戳需要以下参数：

```
SOF_TIMESTAMPING_RAW_HARDWARE
SOF_TIMESTAMPING_TX_HARDWARE
SOF_TIMESTAMPING_RX_HARDWARE
```

`ethtool` 打印的 `PTP 硬件时钟`值是 PTP 硬件时钟的索引。它与 `/dev/ptp*` 设备的命名相对应。第一个 PHC 的索引为 0

- 配置 PTP 参数

`ptp4l` 默认使用硬件时戳。需要以 `root` 身份使用 `-i` 选项指定支持硬件时戳的网络接口。

```bash
# cat /usr/lib/systemd/system/ptp4l.service 
[Unit]
Description=Precision Time Protocol (PTP) service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/ptp4l
ExecStart=/usr/sbin/ptp4l $OPTIONS

[Install]
WantedBy=multi-user.target

# cat /etc/sysconfig/ptp4l 
OPTIONS="-f /etc/ptp4l.conf -i eth0"
# `-m` 指示 `ptp4l` 将其输出列显到标准输出，而不是列显到系统的日志记录工具
```

修改 `/etc/sysconfig/ptp4l` 中关于网卡的配置

- 启动 PTP 服务

```bash
systemctl enable --now ptp4l

# 查看状态
systemctl status ptp4l
```

示例

```bash
selected eth0 as PTP clock
port 1: INITIALIZING to LISTENING on INITIALIZE
port 0: INITIALIZING to LISTENING on INITIALIZE
port 1: new foreign master 00a069.fffe.0b552d-1
selected best master clock 00a069.fffe.0b552d
port 1: LISTENING to UNCALIBRATED on RS_SLAVE
master offset -23947 s0 freq +0 path delay    11350
master offset -28867 s0 freq +0 path delay    11236
master offset -32801 s0 freq +0 path delay    10841
master offset -37203 s1 freq +0 path delay    10583
master offset -7275 s2 freq -30575 path delay  10583
port 1: UNCALIBRATED to SLAVE on MASTER_CLOCK_SELECTED
master offset -4552 s2 freq -30035 path delay  10385
```

主偏移值是与主偏移量测得的偏移量（以纳秒为单位）。

`s0`、`s1`、`s2` 字符串表示不同的时钟伺服状态：`s0` 已解锁，`s1` 表示时钟步进，`s2` 已锁定。一旦处于锁定状态 （`s2`），时钟将不会步进（仅缓慢调整），除非在配置文件中将 `pi_offset_const` 选项设置为正值（现在是 `step_threshold`）。伺服器将通过更改时钟频率而不是步进时钟来校正最大偏移量。设置为 `0.0` 时，伺服器除启动外将永不步进时钟。

`freq` 值是以十亿分之一 （ppb） 为单位的时钟频率调整。

路径延迟值是从主服务器发送的同步消息的估计延迟（以纳秒为单位）。

端口 0 是用于本地 `PTP` 管理的 Unix 域套接字。端口 1 是 `eth0` 接口（基于上面的示例）。

INITIALIZING、LISTENING、UNCALIBRATED 和 SLAVE 是一些可能的端口状态，这些状态会在 INITIALIZE、RS_SLAVE MASTER_CLOCK_SELECTED 事件中发生变化。在最后一条状态更改消息中，端口状态从 UNCALIBRATED 更改为 SLAVE，表示与 `PTP` 主时钟同步成功。

- 软件时间戳

要启用软件时间戳，需要按如下方式使用 `-S` 选项：

```bash
ptp4l -i eth0 -m -S
```

- `ptp4l` 配置文件

配置文件分为若干部分。全局部分（通过 `[global]` 识别）用于设置程序选项、时钟选项和默认端口选项。其他部分与特定的端口相关，会覆盖默认端口选项。部分的名称是配置的端口的名称 — 例如 `[eth0]`。空白端口部分可用于替换命令行选项。

```bash
[global]
verbose               1
time_stamping         software
[eth0]
```

### 延迟测量

`ptp4l` 通过两种方法测量时间延迟：对等式 (P2P) 或端到端 (E2E)。

- P2P

  此方法使用 `-P` 指定。它可以更快地对网络环境中的更改做出反应，并可更准确地测量延迟。仅在每个端口都会与另一个端口交换 PTP 消息的网络中才会使用此方法。P2P 需受到通讯路径中所有硬件的支持。

- E2E

  此方法使用 `-E` 指定。此为默认设置。

- 自动选择方法

  此方法使用 `-A` 指定。自动选项以 E2E 模式启动 `ptp4l`，如果收到了对等延迟请求，则会切换为 P2P 模式。

单个 PTP 通讯路径上的所有时钟必须使用相同的方法来测量时间延迟。如果在使用 E2E 机制的端口上收到了对等延迟请求，或者在使用 P2P 机制的端口上收到了 E2E 延迟请求，则会列显警告。

### PTP 管理客户端

可以使用 `pmc` 客户端获取有关 `ptp41` 的更详细信息。pmc 从标准输入或命令行读取按名称和管理 ID 指定的操作。然后通过选定的传输方式发送操作，并列显收到的任何答复。pmc 支持以下三个操作：`GET` 可检索指定的信息，`SET` 可更新指定的信息，`CMD`（或 `COMMAND`）可发起指定的事件。

默认情况下，管理命令会在所有端口上寻址。可以使用 `TARGET` 命令为后续消息选择特定的时钟和端口。

```bash
pmc -u -b 0 'GET TIME_STATUS_NP'
```

`-b` 选项指定所发送消息中的边界跃点值。将此选项设置为 0 会将边界限制为本地 `ptp4l` 实例。如果将该值设置得更高，则还会检索距离本地实例更远的 PTP 节点发出的消息。返回的信息可能包括：

- stepsRemoved

超级主时钟的通讯节点数。

- offsetFromMaster、master_offset

该时钟与主时钟之间上次测得的偏差（纳秒）。

- meanPathDelay

从主时钟发送的同步消息的预计延迟（纳秒）。

- gmPresent

如果为 `true`，则表示 PTP 时钟已同步到主时钟；本地时钟不是超级主时钟。

- gmIdentity

此为超级主时钟身份。

### 使用 `phc2sys` 同步时钟

使用 `phc2sys` 可将系统时钟同步到网卡上的 PTP 硬件时钟 (PHC)。系统时钟被视为*从属时钟*，而网卡上的时钟则为*主时钟*。PHC 本身将与 `ptp4l` 同步

使用 `-s` 可按设备或网络接口指定主时钟。使用 `-w` 等待 `ptp4l` 进入同步状态。

```bash
phc2sys -s eth0 -w
```

`-w` 选项等待正在运行的 ptp4l 应用程序同步 `PTP` 时钟，然后从 ptp4l 检索 TAI 到 UTC 的偏移量。

PTP 按国际原子时 (TAI) 运行，而系统时钟使用的是协调世界时 (UTC)。如果您不指定 `-w` 来等待 `ptp4l` 同步，可以使用 `-O` 来指定 TAI 与 UTC 之间的偏差（以秒为单位）：

TAI 和 UTC 时间刻度之间的当前偏移量为 36 秒。当插入或删除闰秒时，偏移量会发生变化，这通常每隔几年发生一次。当不使用 `-w` 时，需要使用 `-O` 选项手动设置此偏移量，如下所示：

```bash
phc2sys -s eth0 -O -36
```

也可以将 `phc2sys` 作为服务运行：

```bash
systemctl start phc2sys
```

配置文件

```bash
# cat /usr/lib/systemd/system/phc2sys.service
[Unit]
Description=Synchronize system clock or PTP hardware clock (PHC)
After=ntpdate.service ptp4l.service

[Service]
Type=simple
EnvironmentFile=-/etc/sysconfig/phc2sys
ExecStart=/usr/sbin/phc2sys $OPTIONS

[Install]
WantedBy=multi-user.target

# cat /etc/sysconfig/phc2sys 
OPTIONS="-a -r"
# 指定网卡 -s eth0
```

`-a` 选项使 phc2sys 从 ptp4l 应用程序读取要同步的时钟。它将跟随 `PTP` 端口状态的变化，相应地调整 NIC 硬件时钟之间的同步。除非还指定了 `-r` 选项，否则系统时钟不会同步。

### 校验时间同步

当 PTP 时间同步正常工作并且使用了硬件时戳时，`ptp4l` 和 `phc2sys` 会定期向系统日志输出包含时间偏差和频率调节的消息。

`ptp4l` 输出示例：

```bash
tp4l[352.361]: port 1: INITIALIZING to LISTENING on INITIALIZE
ptp4l[352.361]: port 0: INITIALIZING to LISTENING on INITIALIZE
ptp4l[353.210]: port 1: new foreign master 00a069.eefe.0b442d-1
ptp4l[357.214]: selected best master clock 00a069.eefe.0b662d
ptp4l[357.214]: port 1: LISTENING to UNCALIBRATED on RS_SLAVE
ptp4l[359.224]: master offset       3304 s0 freq      +0 path delay      9202
ptp4l[360.224]: master offset       3708 s1 freq  -28492 path delay      9202
ptp4l[361.224]: master offset      -3145 s2 freq  -32637 path delay      9202
ptp4l[361.224]: port 1: UNCALIBRATED to SLAVE on MASTER_CLOCK_SELECTED
ptp4l[362.223]: master offset       -145 s2 freq  -30580 path delay      9202
ptp4l[363.223]: master offset       1043 s2 freq  -28436 path delay      8972
[...]
ptp4l[371.235]: master offset        285 s2 freq  -28511 path delay      9199
ptp4l[372.235]: master offset        -78 s2 freq  -28788 path delay      9204
```

`phc2sys` 输出示例：

```bash
phc2sys[616.617]: Waiting for ptp4l...
phc2sys[628.628]: phc offset     66341 s0 freq      +0 delay   2729
phc2sys[629.628]: phc offset     64668 s1 freq  -37690 delay   2726
[...]
phc2sys[646.630]: phc offset      -333 s2 freq  -37426 delay   2747
phc2sys[646.630]: phc offset       194 s2 freq  -36999 delay   2749
```

`ptp4l` 通常会频繁写入消息。可以使用 `summary_interval` 指令降低该频率。其值的表达式为 2 的 N 次幂。例如，要将输出频率降低为每隔 1024（等于 2^10）秒，请将下面一行添加到 `/etc/ptp4l.conf` 文件。

```
summary_interval 10
```

也可以使用 `-u *SUMMARY-UPDATES*` 选项降低 `phc2sys` 命令的更新频率。

### 配置示例

#### 使用软件时戳的从属时钟

`/etc/sysconfig/ptp4l`：

```
OPTIONS=”-f /etc/ptp4l.conf -i ethX”
```

未对分发包 `/etc/ptp4l.conf` 进行任何更改。

#### 使用硬件时戳的从属时钟

`/etc/sysconfig/ptp4l`：

```
OPTIONS=”-f /etc/ptp4l.conf -i ethX”
```

`/etc/sysconfig/phc2sys`：

```
OPTIONS=”-s ethX -w”
```

未对分发包 `/etc/ptp4l.conf` 进行任何更改。

#### 使用硬件时戳的主时钟

`/etc/sysconfig/ptp4l`：

```
OPTIONS=”-f /etc/ptp4l.conf -i ethX”
```

`/etc/sysconfig/phc2sys`：

```
OPTIONS=”-s CLOCK_REALTIME -c ethX -w”
```

`/etc/ptp4l.conf`：

```
priority1 127
```

#### 使用软件时戳的主时钟（一般不建议使用）

`/etc/sysconfig/ptp4l`：

```
OPTIONS=”-f /etc/ptp4l.conf -i ethX”
```

`/etc/ptp4l.conf`：

```
priority1 127
```

### PTP 和 NTP

NTP 和 PTP 时间同步工具可以共存，可实现双向时间同步。

#### NTP 到 PTP 的同步

使用 `chronyd` 同步本地系统时钟时，可将 `ptp4l` 配置为超级主时钟，以便通过 PTP 从本地系统时钟分发时间。在 `/etc/ptp4l.conf` 中包含 `priority1` 选项：

```
[global]
priority1 127
[eth0]
```

然后运行 `ptp4l`：

```
ptp4l -f /etc/ptp4l.conf
```

使用硬件时戳时，需要通过 `phc2sys` 将 PTP 硬件时钟同步到系统时钟：

```
phc2sys -c eth0 -s CLOCK_REALTIME -w
```

为了防止 `PTP` 时钟频率的快速变化，可以通过对 PI 伺服使用较小的 `P` （比例） 和 `I` （积分）常数来放松与系统时钟的同步：

```bash
phc2sys -a -r -r -P 0.01 -I 0.0001
```

#### 配置 PTP-NTP 网桥

在未配备支持 PTP 的交换机或路由器的网络中，如果可以使用高度精确的 PTP 超级主时钟，则计算机可以作为 PTP 从属和第 1 层 NTP 服务器运行。此类计算机需有两个或更多网络接口，并且靠近或者能够直接连接到超级主时钟。这可以确保在网络中实现高度精确的同步。

将 `ptp4l` 和 `phc2sys` 程序配置为使用一个网络接口来通过 PTP 同步系统时钟。然后将 `chronyd` 配置为使用另一接口提供系统时间：

```bash
bindaddress 192.0.131.47
hwtimestamp eth1
local stratum 1
```

默认情况下，当 DHCP 客户端命令 `dhclient` 收到 NTP 服务器列表时，会将这些服务器添加到 NTP 配置。为防止出现此行为，请设置

```
NETCONFIG_NTP_POLICY=""
```

### 验证与调试

- 检查 PTP 状态

```bash
ptp4l -i eth0 -m -v  # 启用详细日志
```

日志应显示 Clock state: locked，表示已成功同步。

- 测试时间偏差

```bash
ptping -i eth0 <交换机IP>  # 测试与交换机的PTP延迟
```

- 监控同步精度

使用 `ntpq -p` 或 `chronyc tracking` 查看时间同步状态。

### 注意事项

- 交换机配置：需确保 Cisco 3548 已启用 PTP 并配置为时钟源（如上述示例）。

- 硬件要求：PTP 对时依赖精确的硬件时间戳支持（如支持 PTP 的网卡）。

- 防火墙设置：需开放 PTP 端口（默认 UDP 319 和 320）。

通过以上步骤，Linux 系统可与 Cisco 3548 交换机实现亚微秒级时间同步。

## 参考资料

- <https://haocst.com/use-case/time-sync/linux-ptp-%E9%AB%98%E7%B2%BE%E5%BA%A6%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5%E5%AE%9E%E8%B7%B5/>

- <https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-using_the_ptp_management_client>

- <https://zhuanlan.zhihu.com/p/607427373>

- <https://ty-chen.github.io/ptp/>

- <https://blog.csdn.net/JiMoKuangXiangQu/article/details/135405068>