查看当前cpu运行频率：

```bash
cat /proc/cpuinfo | grep -i "cpu mhz"
```

开始设置：

```bash
cpupower -c all frequency-set -g performance
# 或者
cpupower frequency-set -g performance
```

再次查看当前 cpu 运行频率，最大频率运行。

```bash
cat /proc/cpuinfo | grep -i "cpu mhz"
```

AMD 设置空载最大频率

```bash
# 查看状态
cpupower idle-info

# 设置
cpupower idle-set -D 0
```

频率调节器

```bash
# 查看当前的调节器
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# 查看频率信息
cpupower frequency-info
```

使用  stress 压测

```bash
stress -c 100
```

