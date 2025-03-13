## wireshark-cli

Wireshark，是最受欢迎的 GUI 嗅探工具，实际上它带了一套非常有用的命令行工具集。其中包括 editcap 与 mergecap。

- editcap 是一个万能的 pcap 编辑器，它可以过滤并且能以多种方式来分割 pcap 文件。

- mergecap 可以将多个 pcap 文件合并为一个。

除此之外，还有其它的相关工具，如 [reordercap](https://www.wireshark.org/docs/man-pages/reordercap.html) 用于将数据包重新排序，[text2pcap](https://www.wireshark.org/docs/man-pages/text2pcap.html) 用于将 pcap 文件转换为文本格式， [pcap-diff](https://github.com/isginf/pcap-diff) 用于比较 pcap 文件的异同，等等

Debian 系列：

```bash
apt install wireshark-common
```

RHEL 系列：

```bash
dnf install wireshark-cli
```

## editcap

editcap 是一个万能的 pcap 编辑器，它可以过滤并且能以多种方式来分割 pcap 文件

- <https://www.wireshark.org/docs/man-pages/editcap.html>

editcap 是一个从文件读取部分或所有捕获数据包的程序，可选地以各种方式转换它们，并将结果数据包写入输出文件。缺省情况下，它从输入文件中读取所有数据包，并以 pcapng 文件格式写进输出文件。

editcap 的几个常见功能：

- 可以按时间、长度等截取数据包。
- 可以用来删除重复的数据包，可用来控制用于重复比较的包窗口或相对时间窗口。
- 可以用来编辑数据帧的描述。
- 可以检测、读取和写入 Wireshark 支持的相同捕获文件。
- 可以用几种输出格式编写文件。

### pcap 文件过滤

通过 editcap， 能以很多不同的规则来过滤 pcap 文件中的内容，并且将过滤结果保存到新文件中。

以“起止时间”来过滤 pcap 文件。 -A 和 -B 选项可以过滤出在这个时间段到达的数据包（如，从 2:30 ～ 2:35）。时间的格式为 "YYYY-MM-DD HH:MM:SS"。

```bash
editcap -A '2014-12-10 10:11:01' -B '2014-12-10 10:21:01' input.pcap output.pcap
```

也可以从某个文件中提取指定的 N 个包。下面的命令行从 input.pcap 文件中提取100个包（从 401 到 500）并将它们保存到 output.pcap 中：

```bash
editcap input.pcap output.pcap 401-500
```

使用 "`-D <dup-window>`" （dup-window 可以看成是对比的窗口大小，仅与此范围内的包进行对比）选项可以提取出重复包。每个包都依次与它之前的 `<dup-window> - 1` 个包对比长度与 MD5 值，如果有匹配的则丢弃。

```bash
editcap -D 10 input.pcap output.pcap
```

也可以将 `<dup-window>` 定义成时间间隔。使用"`-w <dup-time-window>`"选项，对比 `<dup-time-window>` 时间内到达的包。

```bash
editcap -w 0.5 input.pcap output.pcap 
```

### 分割 pcap 文件

当需要将一个大的 pcap 文件分割成多个小文件时，editcap 也能起很大的作用。

将一个 pcap 文件分割成数据包数目相同的多个文件

```bash
editcap -c <packets-per-file> <input-pcap-file> <output-prefix>
```

输出的每个文件有相同的包数量，以 `<output-prefix >-NNNN` 的形式命名。

以时间间隔分割 pcap 文件

```bash
editcap -i <seconds-per-file> <input-pcap-file> <output-prefix>
```

## mergecap

### 合并 pcap 文件

如果想要将多个文件合并成一个，用 mergecap 就很方便。

当合并多个文件时，mergecap 默认将内部的数据包以时间先后来排序。

```bash
mergecap -w output.pcap input.pcap input2.pcap [input3.pcap ...]
```

如果要忽略时间戳，仅仅想以命令行中的顺序来合并文件，那么使用 `-a` 选项即可。

例如，下列命令会将 `input.pcap` 文件的内容写入到 `output.pcap`, 并且将 `input2.pcap` 的内容追加在后面。

```bash
mergecap -a -w output.pcap input.pcap input2.pcap
```

## 参考资料

- <https://juejin.cn/post/7222141882515030071>
