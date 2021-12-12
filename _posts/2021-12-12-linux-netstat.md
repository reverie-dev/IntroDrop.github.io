---
layout:     post
title:      "Linux运维工具netstat"
subtitle:   "netstat"
date:       2021-12-12 23:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- linux 
---

### 安装netstat

```shell
# yum install net-tools     [On CentOS/RHEL]
# apt install net-tools     [On Debian/Ubuntu]
# zypper install net-tools  [On OpenSuse]
# pacman -S netstat-nat     [On Arch Linux]
```

### netstat命令选项一览

         -r, --route              display routing table
         -i, --interfaces         display interface table
         -g, --groups             display multicast group memberships
         -s, --statistics         display networking statistics (like SNMP)
         -M, --masquerade         display masqueraded connections
        
        -v, --verbose            be verbose
        -W, --wide               don't truncate IP addresses
        -n, --numeric            don't resolve names
        --numeric-hosts          don't resolve host names
        --numeric-ports          don't resolve port names
        --numeric-users          don't resolve user names
        -N, --symbolic           resolve hardware names
        -e, --extend             display other/more information
        -p, --programs           display PID/Program name for sockets
        -o, --timers             display timers
        -c, --continuous         continuous listing
    
        -l, --listening          display listening server sockets
        -a, --all                display all sockets (default: connected)
        -F, --fib                display Forwarding Information Base (default)
        -C, --cache              display routing cache instead of FIB
        -Z, --context            display SELinux security context for sockets
        
        



### 常用命令选项

1. 显示所有连接。
   **-a** 选项会列出 tcp, udp 和 unix 协议下所有套接字的所有连接。

2. 只列出 TCP 或 UDP 协议的连接

   使用 **-t** 选项列出 TCP 协议的连接，可和 **-a** 选项配合使用

   使用 **-u** 选项列出 UDP 协议的连接

3. 禁用反向域名解析，加快查询速度
   默认情况下 netstat 会通过**反向域名解析**查找每个 IP 地址对应的主机名，会降低查找速度。**n** 选项可以禁用此行为，并且用户 ID 和端口号也优先使用数字显示。
4. 只列出监听中的连接
   **-l** 选项可以只列出正在监听的连接（不能和 **a** 选项同时使用）
5. 获取进程名、进程号以及用户 ID
   **-p** 选项可以查看进程信息（此时 **netstat** 应尽量运行在 **root** 权限之下，否则不能得到运行在 root 权限下的进程名）
6. 显示路由信息
   使用 **-r** 选项打印内核路由信息，与 **route** 命令输出一样。
7. 网络接口信息
   **-i** 选项可以输出网络接口设备的统计信息，结合上 **-e** 选项，等于 **ifconfig** 命令的输出
8. 获取网络协议的统计信息
   **-s** 选项可以输出针对不同网络协议的统计信息，包括 **Ip**、**Icmp**、**Tcp** 和 **Udp** 等。

### 常用运维

netstat -anp | grep ...