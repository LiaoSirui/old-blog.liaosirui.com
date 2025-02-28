## PTP 简介

PTP是英文 “Precision Timing Protocol” 的缩写，即“精确时间协议”。IEEE 1588标准中对它进行了描述，它是一种通过分组网络分配时间的协议，主时钟发送报文到从时钟，告诉从时钟主时钟所处的时间在这个过程中，报文延迟是个棘手的问题，而精确时间协议（PTP）大部分都致力于解决这个问题。

协议使用三种关键技术来减少延迟估计中的误差：

1. 硬件时间戳 —— 精确记录事件报文通过物理接口的时间。这消除了识别和处理报文所涉及的软件延迟。

2. 边界时钟 —— 这类时钟恢复网络中间点的时间，并将时间转发到新的一组报文中。这些时钟通常可在网络中的交换机和路由器中找到，它们有助于减小网络延迟变化所带来的影响，列队延迟等因素都有可能导致这类影响。

3. 透明时钟 —— 这类时钟也可在网络中的交换机和路由器中找到。不同的是，它并不恢复并转发时间，而是记录报文通过交换机或路由器的时间。当报文最终到达从时钟时，透明时钟包含报文通过网络所累积的延迟信息，使得从时钟更准确地将其本地时间与主时钟对齐。

## PTP 配置

要在 Linux 系统中实现与 Cisco 3548 交换机的 PTP 对时，需结合 PTP 协议配置及硬件支持。以下是具体步骤：

### 硬件与交换机准备

- 确认硬件支持

需使用支持PTP协议的网络接口卡（NIC），且交换机（如Cisco 3548）需启用PTP功能并配置为时钟源或从时钟。

示例配置（Cisco 3548）：

```
switch(config)# ptp clock
switch(config-ptp-clock)# priority1 128  # 设置时钟优先级（0-255）
switch(config-ptp-clock)# announce-interval 1  # 广播间隔（秒）
```

- 网络配置

确保Linux系统与交换机处于同一子网，且PTP流量通过专用端口传输（如配置VLAN或Trunk）。

### Linux系统配置

- 安装PTP工具

```bash
switch(config)# ptp clock
switch(config-ptp-clock)# priority1 128  # 设置时钟优先级（0-255）
switch(config-ptp-clock)# announce-interval 1  # 广播间隔（秒）
```

- 配置 PTP 参数

编辑 `/etc/ptp4l.conf` 文件，指定网络接口及时钟驱动：

```bash
# 使用默认时钟驱动（如 e1000 或 mcap）
interface eth0
clockdriver phc2sys
```

- 启动 PTP 服务

```bash
sudo ptp4l -i eth0 -m  # 以主时钟模式运行（若需从模式可省略-m）
```

- 同步硬件时钟

使用 phc2sys 将 PTP 时间同步到系统时钟：

```bash
sudo phc2sys -s -w -c eth0  # -s表示同步，-w表示等待PTP信号
```

### 验证与调试

- 检查PTP状态

```bash
ptp4l -i eth0 -m -v  # 启用详细日志
```

日志应显示 Clock state: locked，表示已成功同步。

- 测试时间偏差

```bash
ptping -i eth0 <交换机IP>  # 测试与交换机的PTP延迟
```

- 监控同步精度

使用ntpq -p或chronyc tracking查看时间同步状态。

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