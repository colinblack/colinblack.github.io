---
title: linux系统编程
date: 2020-03-30 09:02:55
tags: 
categories:
    - Linux
---

<!-- more -->  
### exec函数簇和system区别
exec是新进程复制原来的进程， pid不变， exec后的代码不会被执行   

### 守护进程  
守护进程会清理环境变量， 所以守护进程中使用exec执行系统命令时最好加上绝对路径  
