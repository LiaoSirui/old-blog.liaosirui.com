## 从文件过滤

示例

```bash
OUTPUT_PKG_DIR=/root/output
PKG_SUFFIX="$(date -d yesterday +%Y%m%d)-06.pcap"
PKG_SLIM_SUFFIX="$(date -d yesterday +%Y%m%d)-slim-06.pcap"

tcpdump \
    -r "${OUTPUT_PKG_DIR}/sgx-38-enp23s0f0-${PKG_SUFFIX}" \
    -w "${OUTPUT_PKG_DIR}/sgx-38-enp23s0f0-${PKG_SLIM_SUFFIX}" \
    --time-stamp-precision=nano -nn -G 86400 -Z root \
    'udp dst port 2042 or udp dst port 4042 or udp dst port 12042 or udp dst port 14042'

```



## 参考文档

- 过滤：<https://zhuanlan.zhihu.com/p/417275080>