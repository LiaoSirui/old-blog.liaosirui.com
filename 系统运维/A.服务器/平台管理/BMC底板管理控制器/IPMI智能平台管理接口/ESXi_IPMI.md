查看信息

```bash
esxcli hardware ipmi bmc get
```

设置 IP 信息

下载 IPMI Tool

```bash
# 下载
wget https://vswitchzero.com/wp-content/uploads/2019/08/ipmitool-esxi-vib-1.8.11-2.zip
unzip ipmitool-esxi-vib-1.8.11-2.zip

# 安装
esxcli software acceptance set --level=CommunitySupported
esxcli software vib install -v /tmp/ipmitool-1.8.11-2.x86_64.vib
esxcli software vib get -n ipmitool

# 运行
/opt/ipmitool/ipmitool
```

修改 IP

```bash
# 打印当前 ipmi 地址配置信息
/opt/ipmitool/ipmitool lan print 1

# 设置 id 1 为静态 IP 地址
/opt/ipmitool/ipmitool lan set 1 ipsrc static

# 设置 IPMI 地址
/opt/ipmitool/ipmitool lan set 1 ipaddr 192.168.17.37

# 设置 IPMI 子网掩码
/opt/ipmitool/ipmitool lan set 1 netmask 255.255.255.0

# 设置 IPMI 网关
/opt/ipmitool/ipmitool lan set 1 defgw ipaddr 192.168.17.254
```

