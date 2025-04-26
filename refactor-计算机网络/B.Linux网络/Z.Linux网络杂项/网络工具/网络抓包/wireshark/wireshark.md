## Wireshark

- CaptureFilters <https://wiki.wireshark.org/CaptureFilters>

- DisplayFilters <https://wiki.wireshark.org/DisplayFilters>

## 命令行 tshark

```bash
//打印http协议流相关信息
tshark -s 512 -i eth0 -n -f 'tcp dst port 80' -R 'http.host and http.request.uri' -T fields -e http.host -e http.request.uri -l | tr -d '\t'
　　注释：
　　　　-s: 只抓取前512字节；
　　　　-i: 捕获eth0网卡；
　　　　-n: 禁止网络对象名称解析;
　　　　-f: 只捕获协议为tcp,目的端口为80;
　　　　-R: 过滤出http.host和http.request.uri;
　　　　-T,-e: 指的是打印这两个字段;
　　　　-I: 输出到命令行界面; 
//实时打印当前mysql查询语句
tshark -s 512 -i eth0 -n -f 'tcp dst port 3306' -R 'mysql.query' -T fields -e mysql.query
　　　注释:
　　　　-R: 过滤出mysql的查询语句;
//导出smpp协议header和value的例子
tshark -r test.cap -R '(smpp.command_id==0x80000004) and (smpp.command_status==0x0)' -e smpp.message_id -e frame.time -T fields -E header=y >test.txt
　　　注释:
　　　　-r: 读取本地文件，可以先抓包存下来之后再进行分析;
　　　　-R: smpp...可以在wireshark的过滤表达式里面找到，后面会详细介绍;
　　　　-E: 当-T字段指定时，设置输出选项，header=y意思是头部要打印;
　　　　-e: 当-T字段指定时，设置输出哪些字段;
　　　　 >: 重定向;
//统计http状态
tshark -n -q -z http,stat, -z http,tree
　　　注释:
　　　　-q: 只在结束捕获时输出数据，针对于统计类的命令非常有用;
　　　　-z: 各类统计选项，具体的参考文档，后面会介绍，可以使用tshark -z help命令来查看所有支持的字段;
　　　　　　 http,stat: 计算HTTP统计信息，显示的值是HTTP状态代码和HTTP请求方法。
　　　　　　 http,tree: 计算HTTP包分布。 显示的值是HTTP请求模式和HTTP状态代码。
//抓取500个包提取访问的网址打印出来
tshark -s 0 -i eth0 -n -f 'tcp dst port 80' -R 'http.host and http.request.uri' -T fields -e http.host -e http.request.uri -l -c 500
　　　注释: 
　　　　-f: 抓包前过滤；
　　　　-R: 抓包后过滤；
　　　　-l: 在打印结果之前清空缓存;
　　　　-c: 在抓500个包之后结束;
//显示ssl data数据
tshark -n -t a -R ssl -T fields -e "ip.src" -e "ssl.app_data"

//读取指定报文,按照ssl过滤显示内容
tshark -r temp.cap -R "ssl" -V -T text
　　注释: 
　　　　-T text: 格式化输出，默认就是text;
　　　　-V: 增加包的输出;//-q 过滤tcp流13，获取data内容
tshark -r temp.cap -z "follow,tcp,ascii,13"

//按照指定格式显示-e
tshark -r temp.cap -R ssl -Tfields -e "ip.src" -e tcp.srcport -e ip.dst -e tcp.dstport

//输出数据
tshark -r vmx.cap -q -n -t ad -z follow,tcp,ascii,10.1.8.130:56087,10.195.4.41:446 | more
　　注释:
　　　　-t ad: 输出格式化时间戳;
//过滤包的时间和rtp.seq
tshark  -i eth0 -f "udp port 5004"  -T fields -e frame.time_epoch -e rtp.seq -o rtp.heuristic_rtp:true 1>test.txt
　　注释:
　　　　-o: 覆盖属性文件设置的一些值;

//提取各协议数据部分
tshark -r H:/httpsession.pcap -q -n -t ad -z follow,tcp,ascii,71.6.167.142:27017,101.201.42.120:59381 | more
```



## 参考资料

- <https://www.cnblogs.com/liun1994/p/6142505.html>

- <https://cloud.tencent.com/developer/article/2462400>

- <https://www.cnblogs.com/hyacinthLJP/p/18078481>