---
title: makefile
date: 2020-07-04 21:40:24
categories:
    - Linux
tags:
---

* makefile中赋值号  
=  直接赋值   
:= 把之前的值替换掉   
?= 如果变量中没值则赋值，有值则不赋值   
+= 在变量未追加   

* makefile中的shell   
作用域   
目标项冒号后另起一行才是shell代码    
$(shell  这里是shell代码)    
```makefile
define get_trunk_path  
    trunk_path=`pwd`; \ 
    for dir in $(dirs); do \ 
      if [[ $${trunk_path} = *$$dir* ]]; then \ 
        trunk_path=`echo $${trunk_path} | sed "s/\/$$dir.*//"`; \
        break; \    
      fi \    
    done; \
    echo $${trunk_path}
endef  

PATH_PLATFORM_PRJ := $(shell $(call get_trunk_path))
```
makefile中的shell，每一行是一个进程，不同行之间变量值不能传递。所以，Makefile中的shell不管多长也要写在一行，每行后面加;\ 如上例  

makefile定义的变量使用${var}, shell定义的变量使用$${var}  
```makefile
//运行方式，直接make
PATH="data"
all:
   echo {PATH}
   PATH="555";\
   echo ${PATH};\ //data
   echo $${PATH} //555
```
makefile中赋值可以这样写var = 123, shell中不能留空格var=123  

* makefile中的模式规则  
```makefile
%:%.cpp  #冒号左边表示所有的target, 右边表示依赖的cpp文件           
  g++ $< -o $@   #"$<"表示了所有依赖目标的挨个值, "$@"表示所有的目标的挨个值
```

