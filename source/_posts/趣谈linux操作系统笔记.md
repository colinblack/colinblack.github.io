---
title: 趣谈linux操作系统笔记
date: 2020-03-06 16:09:34
categories:
    - linux内核
tags: 
---

## 核心原理篇
### 综述
* 操作系统内核体系结构图   
![p1](https://i.loli.net/2020/11/15/pbBWAYXISkgerhP.png)
  
* linux 命令图谱 
![p2](https://i.loli.net/2020/11/15/TurwAvLhRlDn2mN.png)

* 系统调用图谱
![p3](https://i.loli.net/2020/11/15/HXYI9Bvx4Ff7lJO.png)

### 系统初始化  
#### X86架构  
* 计算机工作模式图   
![p4](https://i.loli.net/2020/11/15/Nox2ZbL5W9QqXB7.png)    


* CPU组成  
运算单元: 加法、位移  
控制单元: CPU内部缓存和寄存器组   
数据单元: 取指令， 根据指令取数据， 计算结果后存放数据  


* X86架构   
![image-20201115153051673](https://i.loli.net/2020/11/15/olItSPWQ4s9vMzu.png)  


* 程序执行过程图  
![p6](https://i.loli.net/2020/11/15/iyR6zdnIXJgtDsM.png)


* 总线  
地址总线: 位数决定寻址的范围  
数据总线: 位数决定次能拿多少数据  

* 8086
结构图:     
![p7](https://i.loli.net/2020/11/15/8xHsFyU1KAgBc6S.png)  

寄存器:  
8个16位通用寄存器，用于计算过程中暂存数据， AX、BX、CX、DX、SP、BP、SI、DI, 其中 AX、BX、CX、DX 可以分成两个 8 位的寄存器来使用，分别是 AH、AL、BH、BL、CH、CL、DH、DL  
16位IP(Instruction Pointer Register), 指令指针寄存器，存放下一条指令在内存中的地址   
4个16位段寄存器， CS(代码段寄存器)、DS(数据段的寄存器)、SS(栈寄存器)、ES   

实模式：  
CS和DS中都存放这段的起始地址， IP中存放代码段偏移地址， 通用寄存器中存放数据段的偏移地址  
8086的地址总线位20位，寻址方式：起始地址*16+偏移量， 每个段的最大寻址64k  

* 80386 
寄存器:  
8个32位通用寄存器， 低16位保留了16位和八位的使用方式， 高16位没有分的原因是不兼容之前的架构(8080和8086最大寻址分别是2^16和2^20, 通用寄存器保存地址的偏移量,高16位地址已经大于2^16)  
32位指令寄存器EIP  
4个16位段寄存器    
![p8](https://i.loli.net/2020/11/15/gBIZs18zRj5Et2p.png)  

保护模式： 
段的起始地址存放在内存的某个地方，这个地方是一个表格，表格里面一项项是段描述符， 这里面才是段的起始地址， 段寄存器里面存放的是表格中的哪一项，即选择子。 CPU从段寄存器中找到表格中的选择子， 然后简介找到段起始地址， 为了更快速的拿到段起始地址， 段寄存器会从内存中拿到 CPU 的描述符高速缓存器中    
![p9](https://i.loli.net/2020/11/15/EBncydGJ17zAo2X.png)  

















