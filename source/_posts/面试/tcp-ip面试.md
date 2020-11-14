---
title: tcp-ip面试
date: 2020-02-16 13:31:11
categories:
    - 协议 
tags:
    - 面试
---
## TCP和UDP的区别  
tcp连接是永久的只要没断开  
tcp连接是指tcp协议栈维护的状态  
tcp 流式协议，数据是有序的。面向连接可靠，慢，开销大，容易受攻击  
udp 数据报协议，数据不保证有序。不需要连接， 不可靠，快，开销小

<!-- more -->  
## 三次握手与四次挥手  
三次握手：  
 &nbsp;&nbsp;&nbsp;1.客户端发起连接请求，SYN=1，seq=y, 客户端进入SYN_SENT状态  
 &nbsp;&nbsp;&nbsp;2.服务端收到连接，SYN=1,ack=y+1, seq=q, 服务端进入SYN_RCVD状态  
 &nbsp;&nbsp;&nbsp;3.客户端收到服务端的回应, ACK=1，ack=q+1, 客户端进入ESTABLISHED，服务端收到后也进入ESTABLISHED   

四次挥手：  
 &nbsp;&nbsp;&nbsp;&nbsp;主动关闭：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.主动关闭方close FIN=1, 进入FIN_WAIT_1状态  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.被动关闭方收到请求, 发送 ACK=1确认后进入CLOSE_WAIT状态，主动关闭方收到后进入FIN_WAIT_2  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.被动关闭close 发送FIN=1, 进入LAST_ACK状态  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.主动关闭方收到后，发送ACK=1，进入TIME_WAIT状态  
 &nbsp;&nbsp;&nbsp;&nbsp;同时关闭：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;双方发送ACK后进入CLOSING，不会有FIN_WAIT_2,LAST_ACK, CLOSEWAIT    

序号：  
 &nbsp;&nbsp;&nbsp;每个字节的数据都有自己的序号，对端接收的数据是乱序时用来确定顺序  

ISN:      
 &nbsp;&nbsp;&nbsp;三次握手第一个连接时的序列号     
 
判断syn flood攻击：  
```  
    netstat -nap | grep SYN_RECV        
```  

防止Syn Flood方法：  
 &nbsp;&nbsp;&nbsp;1.清除所有的半连接  
 &nbsp;&nbsp;&nbsp;2.syn cookie(sysctl.conf tcp_syncookies选项)  

半关闭状态：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主动关闭端发送FIN，被动关闭端回ACK后  
 
三次握手中握手包丢了会怎么样    
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在第三次握手中丢包，服务端会有一个定时器，负责重发，会重发5次，第一次1s，以后时间在上次基础上翻倍，如果客户端还是回ack服务端会发rst  


## 三次握手，四次挥手的原因  
三次握手:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.服务端的seq(序号)和ack(确认号)是一起发给客户端的  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.客户端如果不发ack包， 服务端一直会超时重传（防止syn flood攻击）  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.三次握手的本质是为了确认连接双方的初始序列号  
四次挥手：            
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.ACK 和 FIN 是分开的  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.TCP是全双工协议，一端可读可写，半关闭只关闭了写端，还可以读数据  

## TIME_WAIT状态是什么，为什么会有，那一方会有  
time_wait:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主动关闭的一端发出ACK包后进入的状态，等待2MSL时间后进入CLOSE状态  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主动发送 RST段的一方，不会进入TIME_WAIT 状态   
原因:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.保证TCP全双工通信可靠性  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果有TIME_WAIT，对端没收到ACK包会重发FIN包，在2MSL内会重发ACK包  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果没有TIME_WAIT，主动关闭方直接进入CLOSE状态，收到FIN包时会返回RST错误(ECONNRESET)  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果主动关闭方挂掉，对端会一直重发FIN，知道重置连接  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 保证建立新连接时，旧连接重传的包在网络中小时而不影响新连接  
TIME_WAIT引发的问题:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.进程退出后，端口不能再次绑定(解决:SO_REUSEADDR)  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.出现大量的TIME_WAIT  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般在高并发短连接，服务器主动关闭socket, 出现大量TIME_WAIT，导致部分用户连不上  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决办法：修改内核相关参数(/etc/sysctl.conf, 重用与快速回收)       


## TCP相关操作  
查看系统支持端口号范围：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cat /proc/sys/net/ipv4/ip_local_port_range  
扩大可用端口范围:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;echo 15000 65000 > /proc/sys/net/ipv4/ip_local_port_range  
查看tcp连接默认的timeout时长:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cat /proc/sys/net/ipv4/tcp_fin_timeout  
缩短timeout的时间:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout  
统计tcp各个状态数:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;netstat -ant|awk '/^tcp/ {++S[$NF]} END {for(a in S) print (a,S[a])}'  
知名服务器端口定义：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/etc/services  

## TCP头部  
固定20 + 可选项40 = 最大60  
mss在可选项中  
mss和MTU：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mss: 最大报文段长度，tcp能发的数据长度，只在三次握手的第一次握手发送给对端（一般1460）  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mtu: 最大传输单元， 网卡一次发送数据的大小（一般是1500）  
修改mss iptable  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看mtu netstat -i  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改mtu vim /etc/network/interfaces /etc/init.d/networking restart  
PUSH:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发送方最后发送的TCP段将会被标记为push  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接收方收到push标志到段，将接收缓冲区的内容拷贝到应用进程  

URG与紧急指针:  
紧急指针只在URG=1的时候有用  
紧急指针实际是序列号的偏移量  
TCP只支持一个字节的紧急数据  
紧急数据及linux中的带外数据（MSG_OOB）  
可以在select设置exceptfds，在epoll中EPOLLPRI监听  
seq:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决乱序  
ack:   
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决丢包  
window:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;流控  
tcpflag:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tcp状态机  
tcp头中的窗口是指真个滑动窗口大小吗？  


## 流量控制  
延时ack：  
 &nbsp;&nbsp;&nbsp;接收数据的一方会根据需要延时发送ack(linux 中动态调节算法在快速ack和延时ack中切换)  
 &nbsp;&nbsp;&nbsp;一般情况下：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.当接收方有数据发送，ack会和数据一起发送  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.如果没有数据会延时200ms，在这期间有数据会和数据一起发送，如果过了时间没有数据，ack会被发送  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.如果延时期间又有第二个数据到来会立即发送ack  
 &nbsp;&nbsp;&nbsp;优点：减少了发送的分段，提高了网络利用率，节省带宽  
 &nbsp;&nbsp;&nbsp;缺点：增大网络延迟  
 &nbsp;&nbsp;&nbsp;关闭延时发送：TCP_QUICKACK  
 &nbsp;&nbsp;&nbsp;注意：每次recv后要重新设置  
nagle算法:  
 &nbsp;&nbsp;&nbsp;发送数据的一方会累积数据直到接收方发来ack包后才将数据一起发送出去（最多累积到一个mss的大小），为了尽可能的发送大块数据  
 &nbsp;&nbsp;&nbsp;优点：提高了吞吐量  
 &nbsp;&nbsp;&nbsp;缺点：增大网络延时  
 &nbsp;&nbsp;&nbsp;nagle和延时确认都是为了减少小包   
   
滑动窗口:  
 &nbsp;&nbsp;&nbsp;目的为了做流量控制，防止对方发送过快导致缓冲区被塞满, 在tcp头的窗口字段确定其大小  
 &nbsp;&nbsp;&nbsp;持续定时器：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当对端窗口为0是，为防止本端死等，会定时发送长度为1的探测报文段（时间层指数增长）  
 &nbsp;&nbsp;&nbsp;问题：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;糊涂窗口综合征：  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;滑动窗口变小导致发送的数据量表小，久之导致网络传输效率变低  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解决方法:  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nagle算法  
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Clark算法和延时ACK  

## 拥塞控制  
网络拥塞:  
 &nbsp;&nbsp;&nbsp;路由器等网络设备来不及处理高数率到来的流量出现的丢包现象  
 &nbsp;&nbsp;&nbsp;判断：1.ack超时 2.连续收到同样的ack  

慢启动：  
 &nbsp;&nbsp;&nbsp;tcp会将大的数据分成若干小的分组，分为若干次发送，而不是一次全部发出去  
 &nbsp;&nbsp;&nbsp;cwdn按照两倍大小增加，大于慢启动门限是进入拥塞避免算法（指数增长）  
 &nbsp;&nbsp;&nbsp;拥塞窗口（cwdn）  
 &nbsp;&nbsp;&nbsp;慢启动门限（ssthresh）  

拥塞避免算法  
 &nbsp;&nbsp;&nbsp;cwdn 每次大小加1（线性增长）, 出现拥塞时cwdn和ssthresh被重置（一般ssthresh=cwdn/2, cwdn=1）， 然后进入慢启动算法  
 &nbsp;&nbsp;&nbsp;加法增大，乘法减小  

快重传：  
 &nbsp;&nbsp;&nbsp;接收方收到一个失序的报文段， 会不断重发相同的ACK（没收到包的序号）  
 &nbsp;&nbsp;&nbsp;发送方连续收到三个相同的Ack, 会立即发送对方Ack的包  
 &nbsp;&nbsp;&nbsp;解决了超时的问题， 但是没有解决重传哪些包的问题（需要sack算法）                            

快恢复：  
 &nbsp;&nbsp;&nbsp;出现超时重传或快重传（连续三个重复的ACK包)时:  
 &nbsp;&nbsp;&nbsp;ssthresh &nbsp;= cwdn/2  
 &nbsp;&nbsp;&nbsp;cwdn = ssthresh  

## 重传机制  
快速重传与超时重传  
超时重传:  
 &nbsp;&nbsp;&nbsp;RTO:(超时重传时间，发送方发出一个包，会启动定时器，定时器超时未收到包，则会重发，这个时间成为重传超时)， 由RTT（数据往返时间）根据公式计算出  
 &nbsp;&nbsp;&nbsp;RTO时间指数方式增加  
 &nbsp;&nbsp;&nbsp;定时器超时后，在滑动窗口内没有收到ACK的包都会重传（sack算法）  
设置重发次数：  
 &nbsp;&nbsp;&nbsp;/proc/sys/net/ipv4/tcp_retries1  
 &nbsp;&nbsp;&nbsp;/proc/sys/net/ipv4/tcp_retries2  

## 保活机制  
保活定时器  
服务端探测死连机制（探测客户端宕机或重启, &nbsp;如果是主动关闭连接）  
缺点：无法识别客户端宕机和还是报文不可达  
设置：SO_KEEPALIVE  

## 数据在不同层的叫法  
数据链路层 帧  
网络层 包（tcp）, 报（udp）  
传输层 段  

## arp是什么，实现，怎么找到MAC, arp欺骗  
利用arp缓存中ip与mac的映射，如果有对方mac可以直接通信，没有发给rap广播， 当对方收到后会将自己的MAC填到包里面返回  
同时将发送方的ip和mac映射放在arp缓存中  
arp欺骗: 主要方式是中间人攻击，达到获取通信双方数据又不被发现的效果  
预防：使用arp网关  


## TCP粘包处理  
利用包体长度，bodylen  
 &nbsp;&nbsp;&nbsp;1. 如果缓冲区里面的消息长度小于消息头长度，不处理  
 &nbsp;&nbsp;&nbsp;2. 如果缓冲区长度大于等于bodylen + headlen, 处理  


## HTTP 请求的过程  
建立TCP连接->发送请求行->发送请求头->（到达服务器）发送状态行->发送响应头->发送响应数据->断TCP连接  




参考：  
[TCP的TIME_WAIT快速回收与重用](https://blog.csdn.net/dog250/article/details/13760985)  
[解决TIME_WAIT过多造成的问题](https://www.cnblogs.com/dadonggg/p/8778318.html)  
[TCP/IP TIME_WAIT状态原理](https://elf8848.iteye.com/blog/1739571)  
[tcp状态介绍最详细--没有之一](https://blog.csdn.net/wuji0447/article/details/78356875)  
[“三次握手，四次挥手”你真的懂吗？](https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=2666543559&idx=1&sn=83cf0e9367511d6b311909a5b3dfc81e&chksm=80dcfd6cb7ab747af19259cce70621b269c5fae25582af7c57f5be904bc18e216625cf6f4157&mpshare=1&scene=24&srcid=0110n9eggIf8eEIoZnl2Loe5&key=02793bf78abe30c4c571e7a6695d4645e0c3637a779c7915ba755677395083d39ee60f9a9d204e28b6d06d279fff9d0a25171eac0cb3e8f126cf46a027d6459f015c13308729af2f9d27c88c38e270c4&ascene=14&uin=ODI0NTI3MTg0&devicetype=Windows+7&version=62060728&lang=zh_CN&pass_ticket=wXb/sgBYyv1S7izs0CuJXuDgNxtBHPyjdhjnyZMw7twBDQnbBA0IyQV1EnfMvwsI)  
[TCP-IP详解：Delay ACK](https://blog.csdn.net/wdscq1234/article/details/52430382)  
[TCP Nagle算法&&延迟确认机制](https://my.oschina.net/xinxingegeya/blog/485643)  
[TCP中的Nagle算法](https://blog.csdn.net/ce123_zhouwei/article/details/9050797)  
[TCP-IP详解：滑动窗口（Sliding Window)](https://blog.csdn.net/wdscq1234/article/details/52444277)  
[TCP协议的滑动窗口具体是怎样控制流量的](https://www.zhihu.com/question/32255109)  
[TCP带外数据（URG，MSG_OOB](https://blog.csdn.net/ordeder/article/details/43243425)  
[带外数据和TCP紧急指针](https://blog.csdn.net/gbasp2008/article/details/47666421)  
[TCP紧急数据](https://blog.csdn.net/u012122743/article/details/46484675)  
[TCP-IP详解：超时重传机制](https://blog.csdn.net/wdscq1234/article/details/52476231)  
[TCP-IP详解：SACK选项（Selective Acknowledgment）](https://blog.csdn.net/wdscq1234/article/details/52503315)  
[27-TCP 协议（快重传与快恢复）](https://blog.csdn.net/q1007729991/article/details/70185266)  
[聊一聊重传次数](https://perthcharles.github.io/2015/09/07/wiki-tcp-retries/)  
[TCP/IP重传超时--RTO](http://www.orczhou.com/index.php/2011/10/tcpip-protocol-start-rto/)  
[0-Linux 网络编程修炼指南——内功心法](https://blog.csdn.net/q1007729991/article/details/69091877)  
[TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)  
[从TCP三次握手说起--浅析TCP协议中的疑难杂症（1)](https://zhuanlan.zhihu.com/p/25569676)  
[从TCP三次握手说起--浅析TCP协议中的疑难杂症（2）](https://zhuanlan.zhihu.com/p/25596865)  
[中间人攻击——ARP欺骗的原理、实战及防御](http://netsecurity.51cto.com/art/201303/386031.htm)  
[ARP欺骗的两种方式](https://blog.csdn.net/qq_37969433/article/details/79587026)  
[浅析TCP之头部可选项](https://blog.csdn.net/Mary19920410/article/details/72857764)  
[TCP报文格式详解](https://blog.csdn.net/mary19920410/article/details/58030147)  
[TCP和UDP的优缺点及区别](https://www.cnblogs.com/xiaomayizoe/p/5258754.html)  
[muduo库chat server对TCP粘包问题的处理](https://blog.csdn.net/freeelinux/article/details/53823731)  
[TCP分包方法 && 粘包处理策略](https://blog.csdn.net/yusiguyuan/article/details/17270679)  
[Linux 网络编程——TCP 粘包及其解决方案](https://blog.csdn.net/lu_embedded/article/details/77430050)  
[HTTP面试题都在这里](https://juejin.im/post/5a8102e0f265da4e710f5910)  
[HTTP是一个无状态的协议。这句话里的无状态是什么意思](https://www.zhihu.com/question/23202402)  
[服务端是如何主动推送信息到客户端的？](https://www.zhihu.com/question/24938934)  
[HTTP 协议入门](http://www.ruanyifeng.com/blog/2016/08/http.html)  
[程序员过关斩将--面试官再问你Http请求过程，怼回去！](https://mp.weixin.qq.com/s?__biz=MzIwNTc3OTAxOA==&mid=2247484323&idx=1&sn=5513334623b73352034a828badfe985d&chksm=972afa86a05d7390fe96222718fd9ce6c21fe700af7e7f9396449815a5323e7315d96681bcac&scene=0&xtrack=1&key=a539a6045067ac190e5c71cc4876a35ec2cab39ecc57c3618a3c22861a71c8bfa13cb8c4e1190e7ca56b6fa54c00b894874dabfdb24eb7aa3f45382bd07dd271a25982549dfd70f959b71db7aeb5d371&ascene=14&uin=ODI0NTI3MTg0&devicetype=Windows+7&version=62060739&lang=zh_CN&pass_ticket=UG6jIZiPy0h82EWKj8fu5ZN0dIyjwygUpMCiLLSskgXdEE9mhfeXBtYPbOR2lLEE)  






