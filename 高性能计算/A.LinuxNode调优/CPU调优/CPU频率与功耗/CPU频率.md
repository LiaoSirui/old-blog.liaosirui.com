## 查看 CPU 频率

查看频率

```bash
watch -n1 lscpu -a -e

# 还是推荐使用 watch cat /sys/devices/system/cpu/cpu[0-9]*/cpufreq/scaling_cur_freq
```

输出示例

```bash
CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ   MINMHZ       MHZ
  0    0      0    0 0:0:0:0          yes 4700.0000 800.0000  800.0000
  1    0      0    1 1:1:1:0          yes 4700.0000 800.0000  800.0000
  2    0      0    2 2:2:2:0          yes 4700.0000 800.0000  800.0000
  3    0      0    3 3:3:3:0          yes 4700.0000 800.0000  800.0000
  4    0      0    4 4:4:4:0          yes 4700.0000 800.0000  800.0000
  5    0      0    5 5:5:5:0          yes 4700.0000 800.0000  800.0000
  6    0      0    0 0:0:0:0          yes 4700.0000 800.0000  800.0000
  7    0      0    1 1:1:1:0          yes 4700.0000 800.0000  800.0000
  8    0      0    2 2:2:2:0          yes 4700.0000 800.0000  800.0000
  9    0      0    3 3:3:3:0          yes 4700.0000 800.0000 2902.3259
 10    0      0    4 4:4:4:0          yes 4700.0000 800.0000 2899.9929
 11    0      0    5 5:5:5:0          yes 4700.0000 800.0000  800.0000
```

`lscpu` 呈现了一个简短的表格，有助于理解 CPU 拓扑：

有 12 个逻辑 CPU （ `CPU 0..11` ），它们都位于同一个节点 （ `NODE 0` ） 和相同的插槽 （ `SOCKET 0` ） 上。只有 6 个物理内核 （ `CORE 0..5` ）。例如，物理内核 `2` 由两个逻辑 CPU 组成： `2` 和 `8` 。

使用该 `L1d:L1i:L2:L3` 列，可以看到每对两个逻辑内核共享级别 1（L1 数据、L1 指令）和 2 （L2）的相同物理内核缓存。所有物理内核共享相同的高速缓存级别 3 （L3）。

当逻辑处理器处于空闲状态（除 C0 之外的 C 状态）时，其频率通常为 0 （HALT）。

## 驱动 acpi-cpufreq

`acpi-cpufreq` CPU frequency scaling driver

CPU 频率缩放使操作系统可以向上或向下缩放 CPU 频率，以节省电量。可以根据系统负载，响应 ACPI 事件自动缩放 CPU 频率，也可以通过用户空间程序手动缩放 CPU 频率

CPU 频率缩放在 Linux 内核中实现，基础架构称为 cpufreq。从内核 3.4 开始，必需的模块会自动加载，并且默认情况下会启用推荐的按需调控器。但是，为您的桌面环境提供的用户空间工具（如 cpupower，acpid，笔记本电脑模式工具或 GUI 工具）仍可用于高级配置。

CPU 动态节能技术用于降低服务器功耗，通过选择系统空闲状态不同的电源管理策略，可以实现不同程度降低服务器功耗，更低的功耗策略意味着 CPU 唤醒更慢对性能 影响更大。对于对时延和性能要求高的应用，建议关闭 CPU 的动态调节功能，禁止 CPU 休眠，并把 CPU 频率固定到最高。通常建议在服务器 BIOS 中修改电源管理为 Performance，如果发现 CPU 模式为 conservative 或者 powersave，可以使用 cpupower 设置 CPU Performance 模式，效果也是相当显著的。


## cpupower 设置 performance

```bash
# CentOS 安装 kernel-tools
yum install kernel-tools

# Ubuntu 安装 CPU 模式无图形化切换器
apt install cpufrequtils

# cpupower设置performance
cpupower frequency-set -g performance

# 查看当前的调节器
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
performance
```

开机自动调整

```bash
# 创建cpupower配置文件
cat << EOF | sudo tee /etc/sysconfig/cpupower
# See 'cpupower help' and cpupower(1) for more info
CPUPOWER_START_OPTS="frequency-set -g performance"
CPUPOWER_STOP_OPTS="frequency-set -g ondemand"
EOF

# 创建自启动服务
cat << EOF | sudo tee /usr/lib/systemd/system/cpupower.service
[Unit]
Description=Configure CPU power related settings
After=syslog.target

[Service]
Type=oneshot
RemainAfterExit=yes
EnvironmentFile=/etc/sysconfig/cpupower
ExecStart=/usr/bin/cpupower $CPUPOWER_START_OPTS
ExecStop=/usr/bin/cpupower $CPUPOWER_STOP_OPTS

[Install]
WantedBy=multi-user.target
EOF

# 刷新system服务
systemctl daemon-reload
systemctl enable cpupower.service
```

## cpufreq_driver

<https://wiki.archlinux.org/title/CPU_frequency_scaling>

|                  |                                                              |
| :--------------: | :----------------------------------------------------------: |
|      Driver      |                         Description                          |
|  `acpi_cpufreq`  | CPUFreq driver which utilizes the ACPI Processor Performance States. This driver also supports the Intel Enhanced SpeedStep (previously supported by the deprecated `speedstep_centrino` module). For AMD Ryzen it only provides 3 frequency states. |
|   `amd_pstate`   | This driver has three modes corresponding to different degrees of autonomy from the CPU hardware: active, passive, and guided. The `amd_pstate` CPU power scaling driver is used automatically in "active mode" on supported CPUs (Zen 2 and newer) since kernel version 6.5. See [#amd_pstate](https://wiki.archlinux.org/title/CPU_frequency_scaling#amd_pstate) for details. |
| `amd_pstate_epp` | This driver implements a scaling driver selected by `amd_pstate=active` with an internal governor for AMD Ryzen (some Zen 2 and newer) processors. |
|  `cppc_cpufreq`  | CPUFreq driver based on ACPI CPPC system (see [#Collaborative processor performance control](https://wiki.archlinux.org/title/CPU_frequency_scaling#Collaborative_processor_performance_control)). Common default on AArch64 systems. Works on modern x86 too, but the `intel_pstate` and `amd_pstate` drivers are better. |
| `intel_cpufreq`  | Starting with kernel 5.7, the intel_pstate scaling driver selects "passive mode" aka `intel_cpufreq` for CPUs that do not support hardware-managed P-states (HWP), i.e. Intel Core i 5th generation or older. This "passive" driver acts similar to the ACPI driver on Intel CPUs, except that it does not have the 16-pstate limit of ACPI. |
|  `intel_pstate`  | This driver implements a scaling driver with an internal governor for Intel Core (Sandy Bridge and newer) processors. It is used automatically for these processors instead of the other drivers below. This driver takes priority over other drivers and is built-in as opposed to being a module. `intel_pstate` may run in "passive mode" via the `intel_cpufreq` driver for older CPUs. If you encounter a problem while using this driver, add `intel_pstate=disable` to your kernel line in order to revert to using the `acpi_cpufreq` driver. |
|  `p4_clockmod`   | CPUFreq driver for Intel Pentium 4/Xeon/Celeron processors which lowers the CPU temperature by skipping clocks. (You probably want to use `speedstep_lib` instead.) |
|  `pcc_cpufreq`   | This driver supports Processor Clocking Control interface by Hewlett-Packard and Microsoft Corporation which is useful on some ProLiant servers. |
|  `powernow_k8`   | CPUFreq driver for K8/K10 Athlon 64/Opteron/Phenom processors. Since Linux 3.7, 'acpi_cpufreq' will automatically be used for more modern AMD CPUs. |
| `speedstep_lib`  | CPUFreq driver for Intel SpeedStep-enabled processors (mostly Atoms and older Pentiums) |

查看当前的 driver

```bash
cpupower frequency-info | grep driver
```

## intel_pstate

```bash
ls /sys/devices/system/cpu/intel_pstate
```

- max_perf_pct 最大频率的百分比，例如4GHZ 4GHZ*90% = 3.6GHZ,可利用此控制最大频率
- min_perf_pct 最小比列，p-state 能够调到最小频率的比例，这个比例最节能
- no_turbo 是否开启睿频， 0：开启 1： 关闭
- num_pstates : 处理器支持的 p 状态数量。这个值在 0-255 之前。包含了睿频和非睿频 p-state。此属性是只读的，这是所支持的 p 状态与 cpu 电压和频率的一张表

intel 关闭睿频

```shell
echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo
```
