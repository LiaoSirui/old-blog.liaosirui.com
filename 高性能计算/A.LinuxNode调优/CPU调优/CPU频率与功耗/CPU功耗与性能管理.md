## CPU 频率

关闭调节

```bash
# 为什么关闭动态调节？？
intel_pstate=disable intel_idle.max_cstate=0 processor.max_cstate=0

amd_pstate=passive
```



下面是一个 Intel 处理器官方参数

![img](./.assets/CPU功耗与性能管理/i9-9900k-turbo-freq.png)

- 基频是 3.6GHz
- 超频到 5GHz 时，最多只有 2 个核能工作在这个频率
- 超频到 4.8GHz 时，最多只有 4 个核能工作在这个频率
- 超频到 4.7GHz 时，最多只有 8 个核能工作在这个频率

Turbo 高低跟 workload 使用的指令集（SSE/AVX/...）也有关系；能超到多少，跟跑的业务类型（或者说使用的指令集）也有关系，使用的指令集不同，能达到的最高频率也不同，比如：

| Mode          | Example Workload                                             | Absolute Guaranteed Lowest Frequency | Absolute Highest Frequency |
| ------------- | ------------------------------------------------------------ | ------------------------------------ | -------------------------- |
| Non-AVX       | SSE, light AVX2 Integer Vector (non-MUL), All regular instruction | Base Frequency                       | Turbo Frequency            |
| AVX2 Heavy    | All AVX2 operations, light AVX-512 (non-FP, Int Vect non-MUL) | AVX2 Base                            | AVX2 Turbo                 |
| AVX-512 Heavy | All heavy AVX-512 operations                                 | AVX-512 Base                         | AVX-512 Turbo              |

另外，在一些 CPU data sheet 中，还有一个所谓的 all-core turbo： 这是所有 core 同时超到同一个频率时，所能达到的最高频率。这个频率可能比 base 高一些， 但肯定比 max turbo frequency 低。例如，Xeon Gold 6150

- base 2.7 GHz
- all-core turbo 3.4 GHz
- turbo max 3.7GHz

（4）p-state vs. freq 直观展示

![img](./.assets/CPU功耗与性能管理/Intel-P-States.jpg)

（5）Linux node `lscpu/procinfo` 实际查看各种频率

``` bash
[root@k8s-prod250-wk003 ~]# lscpu |grep MHz
CPU(s) scaling MHz:                 100%
CPU max MHz:                        2250.0000
CPU min MHz:                        1500.0000
```

还可以通过 cpuinfo 查看，每个 Core 都可能工作在不同频率

```
cat /proc/cpuinfo | egrep '(processor|cpu MHz)'

processor	: 0
cpu MHz		: 3022.175
processor	: 1
cpu MHz		: 2250.000
processor	: 2
cpu MHz		: 2823.914
...
cpu MHz		: 2819.743
processor	: 128
```

这里的 CPU 本质上都是硬件线程（超线程），并不是独立 CORE；同属一个 CORE 的两个硬件线程（CPU 0 & CPU 128）可以工作在不同频率

## CPU 功耗

TDP (Thermal Design Power)：Base Freq 下的额定功耗

TDP 表示的处理器运行在基频时的平均功耗（average power）

这就是说，超频或 turbo 之后，处理器的功耗会比 TDP 更大。 具体到实际，需要关注功耗、电压、电流、出风口温度等等指标

以 AMD 的 turbo 技术为例：

![img](./.assets/CPU功耗与性能管理/AMD_PowerTune_Bonaire-20240906092439221.png)

## 参考资料

- <https://arthurchiao.art/blog/linux-cpu-2-zh/>

- <https://blog.csdn.net/xiaofeng_yan/article/details/108969545>

- <https://blog.mygraphql.com/zh/notes/low-tec/kernel/cpu-frequency/>

- <https://wiki.archlinuxcn.org/wiki/CPU_%E8%B0%83%E9%A2%91>