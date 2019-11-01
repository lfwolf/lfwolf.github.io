---
title: DNS域名污染
date: 2019-10-31 22:43:51
tags:
---

## 问题发现

贪图便宜用了长城宽带，真是垃圾的要命。经常上网抽风，实在受不了了。不定时的替换域名。看下图
![](https://ftp.bmp.ovh/imgs/2019/10/4e6958f3cf13b383.png)

>[什么是域名污染?](https://zh.wikipedia.org/wiki/域名服务器缓存污染)

## 解决问题

先看看别人是怎么做的: 

### 01 方案1

[长城宽带劫持DNS以及绕过](https://xuanxuanblingbling.github.io/life/study/2018/12/23/DNS/)

#### 快速检测

在网上找到一个办法快速检测是否你家的网络的dns遭到劫持

```zsh
➜  ~ nslookup whether.114dns.com 114.114.114.114
```

>这是114家自己的黑科技，如果真的是通过114.114.114.114这台dns服务器来查询whether.114dns.com的结果是会返回给你一个公网地址的，否则会给你个回环地址，如下：

```zsh
➜ ~ nslookup whether.114dns.com 114.114.114.114
Server:		114.114.114.114
Address:	114.114.114.114#53

Non-authoritative answer:
Name:	whether.114dns.com
Address: 127.0.0.1   //----->不正常
```

>所以估计是长城宽带把所有的dns请求都拦掉了，然后换成他们的查询结果，MMP！！！也就是说你修改路由器的配置一点卵用没有

#### 绕过1

>经过热心的网友提示，发现运营商们大部分都是劫持的udp53的dns查询流量，而没有动tcp的流量，所以尝试一下：nslookup -vc 可以强制使用tcp解析域名

本机测试如下，看来长宽升级了，**REFUSED**。

```zsh
➜ ~ nslookup -vc whether.114dns.com 114.114.114.114
Server:		114.114.114.114
Address:	114.114.114.114#53

** server can't find whether.114dns.com: REFUSED
```

#### 01结论

不能通过强制tcp解析域名。

### 02 方案2

[[ChinaDNS]无污染的智能路由 DNS 折腾记](https://moe.best/tutorial/chinadns.html).

>目前我自己已知的最容易判断的污染是(www.pixiv.net)，即P站域名

```zsh
➜ ~ dig www.pixiv.net @114.114.114.114

; <<>> DiG 9.10.6 <<>> www.pixiv.net @114.114.114.114
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38259
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;www.pixiv.net.			IN	A

;; ANSWER SECTION:
www.pixiv.net.		165	IN	A	69.171.239.11

;; AUTHORITY SECTION:
pixiv.net.		139688	IN	NS	ns2.pixiv.net.
pixiv.net.		139688	IN	NS	ns1.pixiv.net.

;; ADDITIONAL SECTION:
ns1.pixiv.net.		96	IN	A	31.13.79.1

;; Query time: 10 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Thu Oct 31 23:34:45 CST 2019
;; MSG SIZE  rcvd: 99
```

>国外的 DNS 是无污染的，其中比较有名的就是 OpenDNS 和 GoogleDNS，不过因为谷歌被特殊照顾，你根本无法在大陆内使用8.8.8.8（自己在本机测试下路由追踪就懂了），但是 OpenDNS 是没问题的

通过 OpenDNS 208.67.222.222进行查询

```zsh
➜ ~ dig www.pixiv.net @208.67.222.222

; <<>> DiG 9.10.6 <<>> www.pixiv.net @208.67.222.222
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53361
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2

;; QUESTION SECTION:
;www.pixiv.net.			IN	A

;; ANSWER SECTION:
www.pixiv.net.		216	IN	A	69.171.239.11

;; AUTHORITY SECTION:
pixiv.net.		140409	IN	NS	ns1.pixiv.net.
pixiv.net.		140409	IN	NS	ns2.pixiv.net.

;; ADDITIONAL SECTION:
ns1.pixiv.net.		117	IN	A	31.13.79.17
ns2.pixiv.net.		6	IN	A	69.171.232.21

;; Query time: 5 msec
;; SERVER: 208.67.222.222#53(208.67.222.222)
;; WHEN: Thu Oct 31 23:22:55 CST 2019
;; MSG SIZE  rcvd: 115
```
结果依然是被污染的

>有一个突破口就在于，DNS 的端口不一定只能用53

>幸运的是，GFW 确实只会检测53端口的 DNS 数据包，而且 OpenDNS 除了53，还提供了443和5353端口的 DNS 服务

```zsh
➜ ~ dig www.pixiv.net @208.67.222.222 -p 443

; <<>> DiG 9.10.6 <<>> www.pixiv.net @208.67.222.222 -p 443
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30822
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.pixiv.net.			IN	A

;; ANSWER SECTION:
www.pixiv.net.		193	IN	CNAME	pixiv.net.
pixiv.net.		193	IN	A	210.140.131.219
pixiv.net.		193	IN	A	210.140.131.223
pixiv.net.		193	IN	A	210.140.131.226

;; Query time: 242 msec
;; SERVER: 208.67.222.222#443(208.67.222.222)
;; WHEN: Thu Oct 31 23:36:24 CST 2019
;; MSG SIZE  rcvd: 104
```

终于返回正确ip了。

#### 为什么使用 ChinaDNS 而不是直接使用 OpenDNS

>1.我们必须使用非53端口去查询国外 DNS 才能得到没有被 GFW 篡改的正确解析结果，ChinaDNS 可以自定义使用的 DNS 的端口，而 Windows 系统的 DNS 设置定死了使用53端口
>2.如果通过 OpenDNS 去解析国内网站，那么很可能会得到一个海外 IP（很多大公司都会配备有海外服务器供海外华人使用，例如京东啦淘宝啦B站啦），这样会导致访问国内网站访问速度很慢
而 ChinaDNS 可以根据 chnrouter 来判断，如果从国内 DNS 里解析到国内 IP 的话就使用，对于国外网站会过滤掉从国内 DNS 解析得到的被污染的结果，十分完美的解决了这个问题

### 腾讯云linux上部署ChinaDNS

先用tmux创建一个会话，然后执行下面脚本。[十分钟学会tmux](https://www.cnblogs.com/kaiye/p/6275207.html)

```zsh
# 如果 Linux 上没有安装 make 和 gcc 的话要先安装
# Ubuntu / Debian
apt-get install -y make gcc
# CentOS
yum install -y make gcc

# 下载 ChinaDNS 源码并解压编译
wget --no-check-certificate  https://github.com/shadowsocks/ChinaDNS/releases/download/1.3.2/chinadns-1.3.2.tar.gz
tar -zxvf chinadns-1.3.2.tar.gz
cd chinadns-1.3.2
./configure && make

# 更新 chnrouter（必要，因为源码中自带的 chnrouter 已经很旧了）
curl 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | grep ipv4 | grep CN | awk -F\| '{ printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > chnroute.txt

# 更新黑名单IP列表（推荐）
rm -f iplist.txt && wget --no-check-certificate  https://raw.githubusercontent.com/Tsuk1ko/ChinaDNS/master/iplist.txt

# 启动 ChinaDNS（编译好的程序在 src 目录中）
src/chinadns -m -c chnroute.txt -s 114.114.114.114,208.67.222.222:443
```

本地机器验证下

```
➜ ~ dig www.pixiv.net @wxapp02

; <<>> DiG 9.10.6 <<>> www.pixiv.net @wxapp02
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33448
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.pixiv.net.			IN	A

;; ANSWER SECTION:
www.pixiv.net.		64	IN	CNAME	pixiv.net.
pixiv.net.		140	IN	A	210.140.131.226
pixiv.net.		140	IN	A	210.140.131.219
pixiv.net.		140	IN	A	210.140.131.223

;; Query time: 352 msec
;; SERVER: 119.29.233.14#53(119.29.233.14)
;; WHEN: Fri Nov 01 00:14:13 CST 2019
;; MSG SIZE  rcvd: 104
```

最后，设置路由器的DNS: [荣耀路由器DNS设置](https://club.huawei.com/thread-3710741-1-1.html)

#### 02结论

方案2解决了我的问题，不过前提是要有台**云主机**。
