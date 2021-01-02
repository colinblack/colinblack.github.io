---
title: rust实战课
date: 2020-12-20 13:58:30
categories:
    - Linux
tags: 
	- rust
---

### **Rust词法结构**
#### **Rust编译过程**

![编译过程](https://i.loli.net/2020/12/20/DV7NEZxKSvlW8n5.png)

#### **六大词法结构**
* **关键字(Keywords)**
  **严格关键字**:
  ![关键字](https://i.loli.net/2020/12/20/YyaGKU6XPxgvj1M.png)
  **保留字**:
  ![保留字](https://i.loli.net/2020/12/20/zglahqPXjfs2xUS.png)
  被保留的关键字不代表将来一定会使用

  **弱关键字**:
  ![弱关键字](/home/cp/blog/colinblack.github.io/source/_posts/image-20201220141457554.png)
  
  


* **标识符(Identifier)**

  以下划线或字母开头的, 将来支持非ASCII标示符
  ![标识符](https://i.loli.net/2020/12/20/B5rJkmPKMYwsNhC.png)
  
* 注释(Comment) 

* 空白(Whitespace)

  ![image-20201220142904808](/home/cp/blog/colinblack.github.io/source/_posts/image-20201220142904808.png)

* 词条(Tokens)

* 路径(Path)

### **面向表达式**
 分号表达式返回值是永远为自身的单元类型()
 分号表达式只有在块表达式最后一行才会进行求值, 其他时候只作为'连接符'存在
 块表达式只对其最后一行表达式进行求值

#### **编译期计算** 
* **过程宏 + Build脚本(build.rs)**

* **类似于Cpp中constexpr的CTFE功能**
  	常量表达式与常量上下文
    ![常量函数](https://i.loli.net/2020/12/20/3FQ8worYRxuz5Ot.png)

    常量上下文(const context) 包含:
     1. 常量值初始化位置
     2. 静态数组的长度表达式, [T;N]
     3. 重复的长度表达式, 类似于[0;10]
     4. 静态变量, 枚举判别式的初始化位置

    常量传播和编译期计算是不同的:
     1. 常量传播是编译器的一种优化
     2. 常量传播并不能改变程序的任何行为, 并且对开发者是影藏的
     3. 编译期计算是指编译是执行的代码, 必须知道其结果, 才能继续执行  
   
    常量安全(Const Safe)    
     1. Rust里的大部分表达式都可用作常量表达式
     2. 并不是所有常量表达式都可以用在常量上下文
     3. 编译期求值必须得到一个确定性的结果 

    常量函数(const fn)
    ![](https://i.loli.net/2020/12/20/huRA9swCbzktKyG.png)

	支持内部嵌入式定义和递归
    ![image-20201220151852450](/home/cp/blog/colinblack.github.io/source/_posts/image-20201220151852450.png)
	
	常量安全子系统类型系统
	1. 普通的fn关键字定义的函数,是safe rust主类型系统保证安全
	2. const fn定义的函数, 是Safe Rust 主类型系统下有一个专门用于常量计算的子类型系统来    		保证常量安全的
	
	常量上下文可接受的常量表达式
	1. const fn 函数
	2. 元组结构体
	3. 元组的值
	4. 反例
	   (1)
	   ![](https://i.loli.net/2020/12/20/WrMZ2b4oFeIaCUT.png)
	   (2)
	   ![](https://i.loli.net/2020/12/20/WGJmBpxHflM4ukD.png)

    常量泛型(const generic)
	Rust中以下定义数组属于不同类型
	![](https://i.loli.net/2020/12/20/lvbWZG3aJouLDBd.png)
	常量泛型
	![](https://i.loli.net/2020/12/20/J9XVqK3MrahfHxZ.png)
	
	#### **所有权**
	Rust中表达式分类:
	![](https://i.loli.net/2020/12/20/Aupbcf6rEKaIZBz.png)
	
	表达式背后的内存管理:
	![](https://i.loli.net/2020/12/20/1OCGTJNLHEKQVA7.png)
   
    位置表达式:
    1. 静态变脸初始化, 比如static mut LEVELS:u32 = 0;
    2. 解引用表达式, *expr
    3. 数组索引表达式, expr[expr]
    4. 字段表达式, expr.field
    5. 加上括号的表达式, (expr)
    6. 除此之外, 都是值表达式
    
   
      