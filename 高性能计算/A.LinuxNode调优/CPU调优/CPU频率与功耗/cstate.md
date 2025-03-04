## C-states

C-state 用于控制 CPU 在不活动时可以进入的休眠级别，它从 C0 开始编号（CPU 处于正常工作状态），一直到 C6（最深的休眠级别，此时 CPU 进入低功耗状态）。当 CPU 进入较深的 C-state 时，其唤醒时延也会变大，在一些实时性要求较高的负载场景，会对性能产生影响。

通常从 C0 开始，但 C0 比较特殊，它正常的 CPU 工作模式，即 CPU 已 100％的开启。

C 状态是空闲的节能状态，而 P 状态是执行节能状态。

在 P 状态期间，处理器仍在执行指令，而在 C 状态（C0 除外）期间，处理器处于空闲状态，这意味着没有任何东西正在执行。

C 状态：

- C0 是运行状态，表示 CPU 正在执行有用的工作
- C1 是第一个空闲状态
- C2 是第二种空闲状态：外部 I/O 控制器集线器阻止对处理器的中断。
- ...

| mode   | Name                  | What id does                                                 | CPUs                                                         |
| ------ | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| C0     | Operating State       | CPU fully turned on, currently executing instructions.       | All CPUs                                                     |
| C1     | Operating State       | CPU fully turned on, awaiting instructions                   | All CPUs                                                     |
| C1E    | Halt                  | Stops CPU main internal clocks via software; bus interface unit and APIC are kept running at full speed | 486DX4 and above                                             |
| C1E    | Enhanced Halt         | Stops CPU main internal clocks via software and reduces CPU voltage; bus interface unit and APIC are kept running at full speed | All socket 775 CPUs                                          |
| C1E    | --                    | Stops all CPU internal clocks                                | Turion 64, 65-nm Athlon X2 and Phenom CPUs                   |
| C2     | Stop Grant            | Stops CPU main internal clocks via hardware; bus interface unit and APIC are kept running at full speed | 486DX4 and above                                             |
| C2     | Stop Clock            | Stops CPU internal and external clocks via hardware          | Only 486DX4, Pentium, Pentium MMX, K5, K6, K6-2, K6-III      |
| C2E    | Extended Stop Grant   | Stops CPU main internal clocks via hardware and reduces CPU voltage; bus interface unit and APIC are kept running at full speed | Core 2 Duo and above (Intel only)                            |
| C3     | Sleep                 | Stops all CPU internal clocks                                | Pentium II, Athlon and above, but not on Core 2 Duo E4000 and E6000 |
| C3     | Deep Sleep            | Stops all CPU internal and external clocks                   | Pentium II and above, but not on Core 2 Duo E4000 and E6000; Turion 64 |
| C3     | AltVID                | Stops all CPU internal clocks and reduces CPU voltage        | AMD Turion 64                                                |
| C4     | Deeper Sleep          | Reduces CPU voltage                                          | Pentium M and above, but not on Core 2 Duo E4000 and E6000 series; AMD Turion 64 |
| C4E/C5 | Enhanced Deeper Sleep | Reduces CPU voltage even more and turns off the memory cache | Core Solo, Core Duo and 45-nm mobile Core 2 Duo only         |
| C6     | Deep Power Down       | Reduces the CPU internal voltage to any value, including 0 V | 45-nm mobile Core 2 Duo only                                 |

Processor Power States

![image-20250304131613319](./.assets/cstate/image-20250304131613319.png)

Processor Package and Core C-States

![image-20250304131544336](./.assets/cstate/image-20250304131544336.png)

## 查看 C 状态

查看频率

```bash
watch -n1 lscpu -a -e
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

该 `cpupower idle-info` 命令列出了支持的 C 状态：

```bash
[root@test-server ~]# cpupower idle-info
CPUidle driver: intel_idle
CPUidle governor: menu
analyzing CPU 10:

Number of idle states: 2
Available idle states: POLL C1
POLL:
Flags/Description: CPUIDLE CORE POLL IDLE
Latency: 0
Usage: 3067
Duration: 29922
C1:
Flags/Description: MWAIT 0x00
Latency: 2
Usage: 72618
Duration: 4413257546
```

显示 `cpupower monitor` C 状态的统计信息：

````bash
[root@test-server ~]# cpupower monitor -m Idle_Stats
intel-rapl/intel-rapl:0
0
intel-rapl/intel-rapl:0/intel-rapl:0:0
0
    | Idle_Stats
 CPU| POLL | C1
   0|  0.00| 100.0
   6|  0.00| 102.2
   1|  0.00| 97.60
   7|  0.00|  0.00
   2|  0.00| 180.0
   8|  0.00| 206.2
   3|  0.00| 101.2
   9|  0.00| 97.59
   4|  0.00| 90.53
  10|  0.00| 99.97
   5|  0.00| 203.0
  11|  0.00| 99.98
````

## 调整 cstate

当 CPU 没有负载时，让 CPU 处于较低的 C-state 状态可以减少指令的开销，同时也能降低 CPU 的唤醒时延。但是当操作系统让 CPU 进入更深的 C-state 时，将 CPU 重新唤醒（比如网卡上有新的中断）并执行指令需要一定的时间，这个唤醒时间开销由 CPU 芯片架构决定。可以配置操作系统禁用更深的 C-state 状态，以降低 CPU 的响应延迟。

```bash
# 查看状态
# 执行以下命令显示相应的 CPUidle driver
cpupower idle-info

# 设置
cpupower idle-set -D 0
```

Grub 参数，`intel_idle.max_cstate=1` 和 `processor.max_cstate=1` 选项，将空闲 CPU 的最大 C-state 限制为 C1。

```bash
# 只对 Intel 有效
grubby --update-kernel=ALL --args=intel_idle.max_cstate=0
grubby --update-kernel=ALL --args=processor.max_cstate=0
```

- `intel_idle.max_cstate=0`

 在 intel 平台上，模式会使用 intel cpuidle drviver，intel_idle.max_cstate=0 意味着禁用 intel cpuidle driver，让其退化使用 acpi driver。

- `processor.max_cstate=0`

`processor.max_cstate=0` 用描述 acpi driver 中 cpu cstate 的最大级别，但是实际 max_cstate=0 并不能真的让 CPU 保持在 C0 态，只能让 CPU 保持在 C1 状态

重启，查看当前系统使用的 CPUidle driver 以及支持的 C-states

返回信息如下所示，说明系统仅支持 2 种 C-states 状态（POLL、C1）。

![image](./.assets/cstate/p826574.png)

## `idle=poll`

```bash
grubby --update-kernel=ALL --args=idle=poll
```



`idle=poll/halt/momwait`

- poll：禁止 CPU 进入休眠状态
- halt：使用 HALT 指令让 CPU 最多进入到 C1 休眠状态。
- nomwait：进入休眠状态时，禁止使用 MWAIT 指令。

## 参考资料

- <https://cloud.tencent.com/developer/article/2166203>

- <https://access.redhat.com/solutions/202743>

- <https://vstinner.github.io/intel-cpus.html>
- <https://blog.csdn.net/weixin_45959095/article/details/120311110>
