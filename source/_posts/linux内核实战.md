---
title: linux内核实战
date: 2020-05-07 16:13:54
categories:
    - Linux
tags:
    - Linux内核
---

<!-- more -->  

### 内核C中结构体初始化  

```c
struct A{
  int a;
  int b;
};

# define __percpu       __attribute__((noderef, address_space(3)))
struct As{ 
   struct A gdt[2];
};
#define PER_CPU_DEF_ATTRIBUTES
#define DEFINE_AS \
        __percpu PER_CPU_DEF_ATTRIBUTES \
        struct As 

int main(){
    //struct As ss ={.gdt={[0]={.a= 1, .b = 2}, [1]={.a=3, .b=4}}};
    DEFINE_AS ss ={.gdt={[0]={.a= 1, .b = 2}, [1]={.a=3, .b=4}}};
    printf("%d\n", ss.gdt[0].a);
    return 0;
}

```
