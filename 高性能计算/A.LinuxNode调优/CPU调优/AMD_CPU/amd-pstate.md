## amd-pstate

`amd-pstate` 是 AMD CPU 性能调频驱动，它在 Linux 内核中的现代 AMD APU 和 CPU 系列上引入了新的 CPU 频率控制机制。新机制基于协作处理器性能控制 (CPPC)，它提供比传统 ACPI 硬件 P-state 更精细的频率管理。当前的 AMD CPU/APU 平台使用 ACPI P-State 驱动程序来管理 CPU 频率和时钟，仅在 3 个 P-state 下进行切换。 CPPC 取代了 ACPI P-state 控制，并为 Linux 内核提供了灵活、低延迟的接口，以直接向硬件传达性能提示。

`amd_pstate` CPPC 有 3 种操作模式：自主（active）模式、非自主（passive）模式和引导自主（guided）模式。通过不同的内核参数可以选择 active/passive/guided 模式。

- 在自主模式下，平台会忽略所需的性能级别请求，并仅考虑设置为最小、最大和能量性能首选项寄存器的值。
- 在非自主模式下，平台直接通过所需性能寄存器从操作系统获取所需的性能水平。
- 在引导自主模式下，平台根据当前工作负载并在操作系统通过最小和最大性能寄存器设置的限制内自主设置操作性能级别。

`amd_pstate=active` 是低级固件控制模式，由 amd_pstate_epp 驱动程序实现，并在命令行中将 amd_pstate=active 传递给内核。在此模式下，如果软件想要偏向 CPPC 固件的性能 (0x0) 或能效 (0xff)，amd_pstate_epp 驱动程序会向硬件提供提示。然后CPPC功耗算法将计算运行时工作负载，并根据电源和热量、核心电压和其他一些硬件条件调整实时核心频率。

## AMD P-State EPP 缩放驱动程序

EPP 是 Energy Performance Preference（EPP）的简称

## 启用 CPPC

P-State EPP 依赖于 CPPC（Collaborative Processor Performance Control），所以首先要在 BIOS 中启用 CPPC。

如果没有，需要使用 [Smokeless_UMAF](https://github.com/DavidS95/Smokeless_UMAF) 访问 BIOS 中的隐藏选项，步骤如下

- 下载仓库中的 `UMAF_BETA.zip`，解压到一个空的 FAT32 格式的 U 盘中
- 用此 U 盘引导启动
- 进入 Device Manager - AMD CBS - NBIO - SMU

在这里应该就可以看到 CPPC 选项。即使已经显示 Auto 或者 Default，也需要手动 Enable。设置完成后按保存重启即可。

## 启用 P-State EPP

可以直接运行上图中的 `cpupower` 工具来查看当前的调频驱动。如果你和我一样已经在运行 Linux 6.5，那么可能什么都不用做了。

不同内核版本对 P-State EPP 的支持如下：

- 6.5 起，默认启用 P-State EPP[1](https://cyp0633.icu/post/amd-pstate-epp/#fn:1)
- 6.4 起，支持 Guided Autonomous 模式[4](https://cyp0633.icu/post/amd-pstate-epp/#fn:4)
- 6.3 起，支持 EPP，但默认不启用[5](https://cyp0633.icu/post/amd-pstate-epp/#fn:5)[6](https://cyp0633.icu/post/amd-pstate-epp/#fn:6)
- 5.17 起，支持 P-State

## 参考资料

- <https://cyp0633.icu/post/amd-pstate-epp/>

- <https://www.bilibili.com/opus/803895260811886601>

- <https://www.reddit.com/r/linux/comments/15p4bfs/amd_pstate_and_amd_pstate_epp_scaling_driver/?tl=zh-hans>