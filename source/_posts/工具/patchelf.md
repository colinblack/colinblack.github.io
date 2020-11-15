---
title: patchelf
date: 2020-03-26 21:09:29
tags:
categories:
    - tools
---


<!-- more -->
## 简介   
PatchELF 是一个用来修改elf格式的动态库和可执行程序的小工具，可以修改动态链接库的库名字，以及链接库的rpath。  

## 安装   
wget https://nixos.org/releases/patchelf/patchelf-0.10/patchelf-0.10.tar.gz
tar -zxvf patchelf-0.10.tar.gz
cd patchelf-0.10
./configure --prefix=/usr/local
make && make install

## 使用   
###  修改动态库的rpath   
rpath全称是run-time search path。Linux下所有elf格式的文件都包含它，特别是可执行文件。它规定了可执行文件在寻找.so文件时的第一优先位置. linux下在链接共享库的时候可以通过rpath选项来指定运行时共享库加载路径。通过这个选项指定的路径会写到ELF文件dynamic段的RPATH里, 运行时链接器会在此路径下搜索ELF文件所依赖的共享库   

编译代码时如果不指定所依赖动态库的路径很可能会编译报错  
有如下解决方法:
* 将所依赖的动态库路径添加到LD_LIBRARY_PATH环境变量中
* 链接时直接将搜索路径写到rpath中，如:
  gcc -L. -larith main.c -Wl,-rpath=依赖动态库所在路径 -o main 
  readelf -d main | grep PATH   //查看rpath
  如果我们拿到的时已经编译好的可执行文件，就不能像方法2一样指定rpath,
  这时可以使用pathelf, 它可以修改elf文件，如:
  patchelf  --set-rpath 依赖动态库路径 main 或
  patchelf  --set-rpath '$ORIGIN/lib' main //ORIGIN会在运行时解析成程序所在路径
* ldd main //查看可执行程序依赖动态库

--set-rpath 实际上是修改runpath, 关于rpath和runpath的区别，前者不会覆盖LD_LIBARAY_PATH, 后者会推荐使用后者

--force-rpath pathelf的默认动作是修改runpath, 如果要修改rpath就要加上该字段

```
patchelf [--force-rpath] --set-rpath "<desired-rpath>" <path-to-elf>

```
<desired-path> 是用逗号分隔的目录列表，例如： /my/libs:/my/other/libs   
如果指定--force-rpath，则设置RPATH，否则设置RUNPATH    


### 修改动态库的名称
如果有这样的场景: 某一个库依赖了一个第三库库的版本1，另外一个库依赖了同样的这个第三库的版本2，两个库的名字是一样的，程序运行时两个动态库都要加载，但是加载的时候由于第三库版本不一致导致程序运行过程中崩溃。这种情况下就可以通关patchelf修改其中一个的依赖库的名字，从而达到同时加载不同版本的动态库的效果，解决运行中的崩溃问题。
相关命令:
patchelf --print-needed test.so //查看依赖动态库，假设test.so依赖a.so
patchelf --replace-needed a.so b.so test.so //将a.so改成b.so

### 删除rpath中未使用的路径
    patchelf --shrink-rpath 可执行文件
    patchelf --remove-rpath <path-to-elf> //同时删除RPATH和RUNPATH   

### 修改动态链接器(dynamic linker)
在实际运行程序时，经常会遇到一个问题，电脑上安装的glibc版本不符合要求，由于glibc是最底层的库，不可能重新编译glibc，这样会导致系统崩溃。两种方法修改：
有代码的情况下，gcc main.c -o main - -Wl --dynamic-linker 动态链接库路径
二进制文件情况下，patchelf --set-interpreter 动态链接库路径 可执行文件


## 兼容性 
PatchELF已在i386-linux， x86_64-linux和powerpc-linux上进行了测试。它能在所有32位或64位，大端或小端   Linux平台上运行。稍作修改，它也可以在其他ELF平台上运行   

关于动态库的搜索：
https://blog.csdn.net/FoxBryant/article/details/53389804
https://blog.csdn.net/dbzhang800/article/details/6918413#commentBox
https://en.wikipedia.org/wiki/Rpath


                                                                                                                             https://nixos.org/patchelf.html
                                                                                                                              https://www.dazhuanlan.com/2019/09/24/5d89371175726/
                                                                                                                             https://blog.csdn.net/force_eagle/article/details/48263365
                                                                                                                             https://www.jianshu.com/p/505a32ccdc91
                                                                                                                             http://shibing.github.io/2016/08/20/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E4%B8%8Erpath/
                                                                                                                             https://blog.csdn.net/farmwang/article/details/73195799                                                                                                                     https://cqlin1995.github.io/2018/06/26/%E8%BF%90%E8%A1%8C%E4%B8%8D%E5%90%8Cglibc%E7%89%88%E6%9C%AC%E7%9A%84%E6%96%87%E4%BB%B6/

ORIGIN
https://medium.com/@nehckl0/creating-relocatable-linux-executables-by-setting-rpath-with-origin-45de573a2e98