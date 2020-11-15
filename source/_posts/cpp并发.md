---
title: cpp并发
date: 2020-02-20 22:20:11
categories:
    - c++
tags:
    - 并发
---

<!-- more -->
## c++11中创建线程

创建线程的几种方式
* 使用全局函数或类静态函数
```cpp
void threadFunc(){
    std::cout << "subThread" << std::endl;
}
std::thread t1(threadFunc);
```
* 使用函数对象
```cpp
class Obj{
public:
    void operator ()(){
        std::cout << "subThread" << std::endl;
    }
};
Obj b;
std::thread t2(b);
```
* 使用lambda
```cpp
auto l = []{
    std::cout << "subThread" << std::endl;
};
std::thread t3(l);
```
join  
阻塞等待线程结束

detach  
线程分离，一般不使用此函数主进程结束即使子线程还在运行仍然会被终止，使用此函数，线程会被c＋＋运行时库接管，成为守护线程.


joinable  
是否可以join或detach

## 线程传参
* 使用detach带来的陷阱
线程函数中无论是引用传递还是值传递编译器都会拷贝副本，所以detach下传递int&安全，传递char*不安全(主线程退出内存释放掉了)
```cpp
void myprint(const int& i, char* pmybuf){
    cout << i << " " << pmybuf << endl;
}
char mybuf[] = "hello world";
thread mytobj(myprint, mvar, mybuf);
mytobj.detach();
```

避免隐式转换
从char* 到 string转换时机不确定，可能主线程退出后再发生转换此时char*指向内存已销毁
```cpp
void myprint(const int i, const string& pmybuf){
    cout << i << " " << pmybuf.c_str() << endl;
}
char mybuf[] = "hello world"; 
thread mytobj(myprint, mvar, mybuf); //改为thread mytobj(myprint, mvar, string(mybuf))可以看到内存拷贝发生在主线程退出前
mytobj.detach();

```
* 传递类对象、智能指针、函数对象, 成员函数作为线程参数 
线程函数传递引用类型参数前需加const 
需要使用std::ref才能传递myobj的引用 
```cpp
class A{
public:    
    int m_i;
    A(int a) : m_i(a){std::cout <<"A()" << std::endl;}
    A(const A& a):m_i(a.m_i){std::cout<< A(constA& ) << std::endl;}
    ~A(){std::cout << "~A()" << std::endl;}
};

void myprint(const A& a){
    cout << i << a.m_i << endl;
}
A myobj(10);
//thread mytobj(myprint, myobj); 
thread mytobj(myprint, std::ref(myobj)); 
mytobj.join();

```

传递std::unique_ptr使用std::move

```cpp
void myprint(unique_ptr<int> pzn){
}

unique_ptr<int> p(new int(100));
//thread mytobj(myprint, myobj); 这样操作有问题unique_ptr是独占指针 
thread mytobj(myprint, std::move(myobj)); 
mytobj.join();

```
函数对象
```cpp
class A{
public:    
    int m_i;
    void operator()(int num){
    }
}

A myobj(10);
thread mytobj(myobj, 15); //会有拷贝构造
thread mytobj(std::ref(myobj), 15); //没有拷贝构造
mytobj.join();
```

用成员函数指针做线程函数
```cpp
class A{
public:    
    int m_i;
    void thread_work(int num){
        std::cout << num << " " << std::this_thread::get_id() << std::endl;
    }

};
A myobj(10);
thread mytobj(&A::thread_work, myobj, 15); //会有拷贝构造
// thread mytobj(&A::thread_work, std::ref(myobj), 15); //没有拷贝构造
// thread mytobj(&A::thread_work, &myobj, 15); //没有拷贝构造
mytobj.join();

```


 






