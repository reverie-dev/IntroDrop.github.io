---
layout:     post
title:      "Linux的awk和sed简易教程 "
subtitle:   "awk sed"
date:       2021-11-14 20:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- tutorial

---



## awk

**AWK是贝尔实验室1977年搞出来的文本出现神器**

### 基本用法示例

我从netstat命令中提取了如下信息作为用例：

```bash
$ cat netstat.txt

Proto Recv-Q Send-Q Local-Address          Foreign-Address             State
tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
tcp        0      0 coolshell.cn:80        124.205.5.146:18245         TIME_WAIT
tcp        0      0 coolshell.cn:80        61.140.101.185:37538        FIN_WAIT2
tcp        0      0 coolshell.cn:80        110.194.134.189:1032        ESTABLISHED
tcp        0      0 coolshell.cn:80        123.169.124.111:49809       ESTABLISHED
tcp        0      0 coolshell.cn:80        116.234.127.77:11502        FIN_WAIT2
tcp        0      0 coolshell.cn:80        123.169.124.111:49829       ESTABLISHED
tcp        0      0 coolshell.cn:80        183.60.215.36:36970         TIME_WAIT
tcp        0   4166 coolshell.cn:80        61.148.242.38:30901         ESTABLISHED
tcp        0      1 coolshell.cn:80        124.152.181.209:26825       FIN_WAIT1
tcp        0      0 coolshell.cn:80        110.194.134.189:4796        ESTABLISHED
tcp        0      0 coolshell.cn:80        183.60.212.163:51082        TIME_WAIT
tcp        0      1 coolshell.cn:80        208.115.113.92:50601        LAST_ACK
tcp        0      0 coolshell.cn:80        123.169.124.111:49840       ESTABLISHED
tcp        0      0 coolshell.cn:80        117.136.20.85:50025         FIN_WAIT2
tcp        0      0 :::22                  :::*                        LISTEN
```



下面是最简单最常用的awk示例，其输出第1列和第4例，

`awk '{print $1, $4}' netstat.txt`

- 其中单引号中的被大括号括着的就是awk的语句，注意，其只能被单引号包含。
- 其中的$1..$n表示第几例。注：$0表示整个行。

输出：

```bash
Proto Local-Address
tcp 0.0.0.0:3306
tcp 0.0.0.0:80
tcp 127.0.0.1:9000
tcp coolshell.cn:80
tcp coolshell.cn:80
tcp coolshell.cn:80
```

然后是格式化输出：

`awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt`

跟c语言的printf一样。

### 过滤记录

下面过滤条件为：第三列的值为0 && 第6列的值为LISTEN

`awk '$3==0 && $6=="LISTEN"' netstat.txt`

输出

```bash
tcp        0      0 0.0.0.0:3306               0.0.0.0:*              LISTEN
tcp        0      0 0.0.0.0:80                 0.0.0.0:*              LISTEN
tcp        0      0 127.0.0.1:9000             0.0.0.0:*              LISTEN
tcp        0      0 :::22                      :::*                   LISTEN 
```

其中的“==”为比较运算符。其他比较运算符：!=, >, <, >=, <=

示例2

```bash
$ awk '$3 > 0 {printf "%-20s %-20s %s\n",$4,$5,$6}' netstat.txt  
// 过滤出第3列大于0的行 并格式化输出
Local-Address        Foreign-Address      State
coolshell.cn:80      61.148.242.38:30901  ESTABLISHED
coolshell.cn:80      124.152.181.209:26825 FIN_WAIT1
coolshell.cn:80      208.115.113.92:50601 LAST_ACK
```

### 内建变量

说到了内建变量，我们可以来看看awk的一些内建变量：

| $0       | 当前记录（这个变量中存放着整个行的内容）                     |
| -------- | ------------------------------------------------------------ |
| \$1~\$n  | 当前记录的第n个字段，字段间由FS分隔                          |
| FS       | 输入字段分隔符 默认是空格或Tab                               |
| NF       | 当前记录中的字段个数，就是有多少列                           |
| NR       | 已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。 |
| FNR      | 当前记录数，与NR不同的是，这个值会是各个文件自己的行号       |
| RS       | 输入的记录分隔符， 默认为换行符                              |
| OFS      | 输出字段分隔符， 默认也是空格                                |
| ORS      | 输出的记录分隔符，默认为换行符                               |
| FILENAME | 当前输入文件的名字                                           |

怎么使用呢，比如：我们如果要输出行号：

```bash
$ awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s\n",NR, FNR, $4,$5,$6}' netstat.txt
01 1 Local-Address        Foreign-Address      State
07 7 coolshell.cn:80      110.194.134.189:1032 ESTABLISHED
08 8 coolshell.cn:80      123.169.124.111:49809 ESTABLISHED
10 10 coolshell.cn:80      123.169.124.111:49829 ESTABLISHED
14 14 coolshell.cn:80      110.194.134.189:4796 ESTABLISHED
17 17 coolshell.cn:80      123.169.124.111:49840 ESTABLISHED
```

### 指定分隔符

-F就是指定分隔符，文本是有usr:root:root:这样的，然后指定分隔符就成了usr root root这样

指定分隔符为":"，然后输出的字符分隔符，替换成"\\t" 即制表符

`awk -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd`

```bash
root    0       /root
daemon  1       /usr/sbin
bin     2       /bin
sys     3       /dev
sync    4       /bin
games   5       /usr/games
```

可以写：

`awk  'BEGIN{FS=":"} {print $1,$3,$6}' /etc/passwd`

指定多个分隔符：`awk -F '[;:]'`

### 字符串匹配

示例：前面的 '$6 ~ /FIN/' 即 匹配第6列长的像FIN或者TIME的字段。

~ 表示模式开始。/ /中是模式。这就是一个正则表达式的匹配。

```bash
$ awk '$6 ~ /FIN|TIME/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt
1       Local-Address   Foreign-Address State
5       coolshell.cn:80 124.205.5.146:18245     TIME_WAIT
6       coolshell.cn:80 61.140.101.185:37538    FIN_WAIT2
9       coolshell.cn:80 116.234.127.77:11502    FIN_WAIT2
11      coolshell.cn:80 183.60.215.36:36970     TIME_WAIT
13      coolshell.cn:80 124.152.181.209:26825   FIN_WAIT1
15      coolshell.cn:80 183.60.212.163:51082    TIME_WAIT
18      coolshell.cn:80 117.136.20.85:50025     FIN_WAIT2
```

awk可以像grep一样的去匹配第一行

```bash
$ awk '/LISTEN/' netstat.txt
tcp        0      0 0.0.0.0:3306           0.0.0.0:*                   LISTEN
tcp        0      0 0.0.0.0:80             0.0.0.0:*                   LISTEN
tcp        0      0 127.0.0.1:9000         0.0.0.0:*                   LISTEN
tcp        0      0 :::22                  :::*                        LISTEN
```

取反：` awk '$6  !~ /WAIT/ {print NR,$4,$5,$6}' OFS="\t" netstat.txt' `

或者 `awk '!/WAIT/' netstat.txt`

#### 拆分文件

