PTP是英文 “Precision Timing Protocol” 的缩写，即“精确时间协议”。IEEE 1588标准中对它进行了描述，它是一种通过分组网络分配时间的协议，主时钟发送报文到从时钟，告诉从时钟主时钟所处的时间在这个过程中，报文延迟是个棘手的问题，而精确时间协议（PTP）大部分都致力于解决这个问题。

协议使用三种关键技术来减少延迟估计中的误差：

1. 硬件时间戳 —— 精确记录事件报文通过物理接口的时间。这消除了识别和处理报文所涉及的软件延迟。

2. 边界时钟 —— 这类时钟恢复网络中间点的时间，并将时间转发到新的一组报文中。这些时钟通常可在网络中的交换机和路由器中找到，它们有助于减小网络延迟变化所带来的影响，列队延迟等因素都有可能导致这类影响。

3. 透明时钟 —— 这类时钟也可在网络中的交换机和路由器中找到。不同的是，它并不恢复并转发时间，而是记录报文通过交换机或路由器的时间。当报文最终到达从时钟时，透明时钟包含报文通过网络所累积的延迟信息，使得从时钟更准确地将其本地时间与主时钟对齐。

## 参考资料

- <https://haocst.com/use-case/time-sync/linux-ptp-%E9%AB%98%E7%B2%BE%E5%BA%A6%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5%E5%AE%9E%E8%B7%B5/>

- <https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-using_the_ptp_management_client>

- <https://zhuanlan.zhihu.com/p/607427373>

- <https://ty-chen.github.io/ptp/>