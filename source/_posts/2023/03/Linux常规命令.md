---
title: Linux常规命令
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-01 20:25:53
img:
coverImg:
password:
summary:
tags:
- command
categories: [Development & Operate,Linux & Unit]
---

# 1、删除0字节文件  

```bash
find -type f -size 0 -exec rm -rf {} \;
```

# 2、查看进程

按内存从大到小排列

```bash
PS -e -o "%C : %p : %z : %a"|sort -k5 -nr
```

# 3、按 CPU 利用率从大到小排列

```bash
ps -e -o "%C : %p : %z : %a"|sort -nr
```

# 4、打印 cache 里的URL

```bash
grep -r -a jpg /data/cache/* | strings | grep "http:" | awk -F'http:' '{print "http:"$2;}'
```

# 5、查看 http 的并发请求数及其 TCP 连接状态：

```bash
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

# 6、sed 在这个文里 Root 的一行，匹配 Root 一行，将 no 替换成 yes。
 `sed -i '/Root/s/no/yes/' /etc/ssh/sshd_config` 

# 7、如何杀掉 MySQL 进程

```bash
ps aux |grep mysql |grep -v grep  |awk '{print $2}' |xargs kill -9 (从中了解到awk的用途)
```

# 8、显示运行 3 级别开启的服务:

```bash
ls /etc/rc3.d/S* |cut -c 15-   (从中了解到cut的用途，截取数据)
```

# 9、如何在编写 SHELL 显示多个信息，用 EOF

```bash
cat << EOF
```

# 10、for 的巧用（如给 MySQL 建软链接）

```bash
cd /usr/local/mysql/bin
```

# 11、取 IP 地址

```bash
ifconfig eth0 |grep "inet addr:" |awk '{print $2}'| cut -c 6-  
```

# 12、内存的大小:

```bash
free -m |grep "Mem" | awk '{print $2}'
```

# 13、获取端口
```bash
netstat -an -t | grep ":80" | grep ESTABLISHED | awk '{printf "%s %s\n",$5,$6}' | sort
```

# 14、查看 Apache 的并发请求数及其 TCP 连接状态：

```bash
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

# 15、因为同事要统计一下服务器下面所有的 jpg 的文件的大小，写了个 SHELL 给他来统计。原来用 xargs 实现，但他一次处理一部分。搞的有多个总和……，下面的命令就能解决。

```bash
find / -name *.jpg -exec wc -c {} \;|awk '{print $1}'|awk '{a+=$1}END{print a}'
```



# 16、CPU负载 CPU 的数量（多核算多个CPU，`cat /proc/cpuinfo |grep -c processor`）越多，系统负载越低，每秒能处理的请求数也越多。

```bash
cat /proc/loadavg
```

检查前三个输出值是否超过了系统逻辑 CPU 的4倍。

# 17、 CPU负载

```bash
mpstat 1 1
```

检查 %idle 是否过低（比如小于5%）。

# 18、内存空间

```bash
free
```

检查 free 值是否过低，也可以用 `# cat /proc/meminfo`

# 19、SWAP 空间 

```bash
free
```

检查 swap used 值是否过高，如果 swap used 值过高，进一步检查 swap 动作是否频繁：  

```bash
vmstat 1 5
```

观察 si 和 so 值是否较大

# 20、磁盘空间 

```bash
df -h
```

检查是否有分区使用率（Use%）过高（比如超过90%）如发现某个分区空间接近用尽，可以进入该分区的挂载点，用以下命令找出占用空间最多的文件或目录：

```bash
du -cks * | sort -rn | head -n 10
```

# 21、磁盘 I/O 负载

```bash
iostat -x 1 2
```

检查I/O使用率（%util）是否超过 100%

# 22、网络负载

```bash
sar -n DEV
```

检查网络流量（rxbyt/s, txbyt/s）是否过高

# 23、网络错误

```bash
netstat -i
```

检查是否有网络错误（drop fifo colls carrier），也可以用命令：# cat /proc/net/dev

# 24、网络连接数目

```bash
netstat -an | grep -E “^(tcp)” | cut -c 68- | sort | uniq -c | sort -n
```

# 25、进程总数 

```bash
ps aux | wc -l
```

检查进程个数是否正常 (比如超过250)  

# 26、可运行进程数目 

```bash
vmwtat 1 5
```

列给出的是可运行进程的数目，检查其是否超过系统逻辑 CPU 的 4 倍  

# 27、进程   

```bash
top -id 1
```

观察是否有异常进程出现。  

# 28、网络状态，检查DNS，网关等是否可以正常连通

# 29、用户

```bash
who | wc -l
```

检查登录用户是否过多 (比如超过50个)   也可以用命令：# uptime。  

# 30、系统日志

```bash
# cat /var/log/rflogview/*errors
```

检查是否有异常错误记录   也可以搜寻一些异常关键字，例如：  

```bash
grep -i error /var/log/messages
```

# 31、核心日志 

```bash
dmesg
```

检查是否有异常错误记录。  

# 32、系统时间  

```bash
date
```

检查系统时间是否正确。  

# 33、打开文件数目 

```bash
lsof | wc -l
```

检查打开文件总数是否过多。  

# 34、日志 

```bash
# logwatch –print
```

配置 /etc/log.d/logwatch.conf，将 Mailto 设置为自己的 email 地址，启动 mail 服务(sendmail或者postfix)，这样就可以每天收到日志报告了。

缺省 logwatch 只报告昨天的日志，可以用 # logwatch –print –range all 获得所有的日志分析结果。

可以用 # logwatch –print –detail high 获得更具体的日志分析结果(而不仅仅是出错日志)。  

# 35、杀掉80端口相关的进程  

```bash
lsof -i :80|grep -v “ID”|awk ‘{print “kill -9”,$2}’|sh
```

# 36、清除僵死进程

```bash
ps -eal | awk '{ if ($2 == "Z") {print $4}}' | kill -9
```

# 37、tcpdump 抓包，用来防止80端口被人攻击时可以分析数据

```bash
tcpdump -c 10000 -i eth0 -n dst port 80 > /root/pkts
```

# 38、然后检查IP的重复数并从小到大排序 注意 “-t\ +0”   中间是两个空格

```bash
# less pkts | awk {'printf $3"\n"'} | cut -d. -f 1-4 | sort | uniq -c | awk {'printf $1" "$2"\n"'} | sort -n -t\ +0
```

# 39、查看有多少个活动的 php-cgi 进程

```bash
netstat -anp | grep php-cgi | grep ^tcp | wc -l
```

# 40、查看系统自启动的服务

```bash
chkconfig --list | awk '{if ($5=="3:on") print $1}'
```

# 41、kudzu 查看网卡型号

```bash
kudzu --probe --class=network
```

