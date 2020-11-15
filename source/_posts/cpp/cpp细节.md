---
title: cpp细节
date: 2020-01-08 19:59:12
categories:
    - c++
---

## 移动构造函数中如果有拷贝动态内存要注意将指针赋空
```cpp
class A{
public:
    A() : m_ptr(new int(0)){std::cout << "construct" << std::endl;}
    A(const A& a) : m_ptr(a.m_ptr){std::cout << "copy construct" << std::endl;}
    A(A&& a) : m_ptr(a.m_ptr){a.m_ptr = nullptr; std::cout << "move construct" << std::endl;}
    ~A(){delete m_ptr; std::cout << "destruct" << std::endl;}
public:
    int* m_ptr;
};
```

<!-- more -->
## 重载决议
```cpp
#include <iostream>

struct foo{
int a_;
template <typename T>
foo(T&& v) : a_{static_cast<int>(v)}
{
}

foo(const foo& rhs) : a_{rhs.a_}
{
}

operator std::uint8_t(){return a_;}};


int main()
{
    foo bar1{256}, bar2{bar1};

    if(bar1.a_ == bar2.a_)
        std::cout << "hello!" << std::endl;
    return 0;
}
```
这段代码中，编译器会选择会选择移动构造函数而不是拷贝构造，应为如果调用拷贝构造bar1会存在隐式转换.但是如果是默认的拷贝构造则会优先被调用
[Template constructor and copy constructor](https://stackoverflow.com/questions/53902448/template-constructor-and-copy-constructor)


## vector删除性能对比
[vector_pop_back.cc](https://sourcegraph.com/github.com/colinblack/language/-/blob/c++/stl/vector/vector_pop_back.cc#L14)


## vector 构造函数
vector<inv> v(10)　构造一个size(不是capacity)为10的vector


## 关联容器的构造函数
如map set unordered_map 中 使用[]操作符，首先会调无参构造函数，插入元素正确的方式是使用insert, 但是这会调用两次拷贝构造，需使用std::move。使用[]操作符复制是调用的是拷贝操作符，使用insert会调拷贝构造
```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <utility>  

using std::vector;
struct Item
{
    int i;
    Item(int a): i(a)
    {
    }
};

struct Demo
{
    int a_;
    int b_;
    vector<Item> v_;
    Demo()
    {
        a_ = 0;    
        b_ = 0;
        std::cout << "no param " << std::endl;
    }

    Demo(int a, int b, vector<Item>& v)
    {
        a_ = a;    
        b_ = b;
        v_ = v;
        std::cout << "param " << std::endl;
    }
    Demo(const Demo& obj)
    {
        a_ = obj.a_;    
        b_ = obj.b_;
        v_ = std::move(obj.v_);
        std::cout << "copy construct " << std::endl;
    }
    Demo& operator= (const Demo& obj)
    {
        a_ = obj.a_;
        b_ = obj.b_;
        v_ = std::move(obj.v_);
        std::cout << "copy operator " << std::endl;
    }

};

int main()
{
    vector<Item> v;
    v.push_back(Item(1));
    std::unordered_map<unsigned, Demo> m;
    m.insert({1, Demo(1, 2, v)});
    m[1] = Demo(1, 2, v);
    std::cout << m[1].a_ << " " << m[1].b_ << " " << m[1].v_.size() << " " << v.size() << std::endl;    
    return 0;
}

```
## C中的结构体
```c
fd_set rd;
fd_set all;
ll = rd; //合法
fd_set 是个结构体, 里面有一个整型数组， c++中则会调用拷贝操作符
```

## STL 中的类型
SIZE_TYPE：不同平台下长度不一样， 64位下是8字节，unsigned log
![size_type](/images/f1a84799-5f3b-41a6-a52d-d75a92fdea4d.png)

这里返回值不能用unsigned去接， 否则找不到会返回-1, 实际find找不到只会返回sting::npos, 显然是类型溢出

## 初始化列表 
函数中返回变参的初始化列表    
```cpp
class A{
};


//内置类型 
template <class... Args>
std::array<int, sizeof...(Args)>  make_A(Args... args){
    return { args... };
}

//自定义类型
template <class... Args>
std::array<A, sizeof...(Args)>  make_A(Args... args){
    return { (args, A())... };
}

int main(){
    std::array<int, 3> a = make_A(1, 2, 3);
    std::cout << a[0] << std::endl;
    return 0;
}

```

## 惯用法  
### 命名构造器(Named Constructor)  
解决构造函数较多时， 且函数之间差别细微时产生的歧义问题 
```cpp

//https://isocpp.org/wiki/faq/ctors#named-ctor-idiom
#include <cmath>               // To get std::sin() and std::cos()
class Point {
public:
  static Point rectangular(float x, float y);      // Rectangular coord's
  static Point polar(float radius, float angle);   // Polar coordinates
  // These static methods are the so-called "named constructors"
  // ...
private:
  Point(float x, float y);     // Rectangular coordinates
  float x_, y_;
};
inline Point::Point(float x, float y)
  : x_(x), y_(y) { }
inline Point Point::rectangular(float x, float y)
{ return Point(x, y); }
inline Point Point::polar(float radius, float angle)
{ return Point(radius*std::cos(angle), radius*std::sin(angle)); }


```

## 宏  

"#" 把一个符号直接转换为字符串
```cpp
    #define STRING(x) #x
    const char *str = STRING( test_string ) //"test_string"
```

"##" 符号会连接两个符号，从而产生新的符号  
```cpp
   #define SIGN( x ) INT_##x
   int SIGN( 1 ); 宏被展开后将成为：int INT_1;
```

__VA_ARGS__ 变参宏
```cpp
   #define LOG( format, ... ) printf( format, __VA_ARGS__ )
   LOG( "%s %d", str, count );
   //__VA_ARGS__是系统预定义宏，被自动替换为参数列表。
```

宏递归 
```cpp
   #define TEST( x ) ( x + TEST( x ) )
   //为了防止无限制递归展开，语法规定，当一个宏遇到自己时，就停止展开
   //当对TEST( 1 )进行展开时，展开过程中又发现了一个TEST，那么就将这个TEST当作一般的符号。TEST(1)
   //最终被展开为：1 + TEST( 1) 
```
prescan  
当一个宏参数被放进宏体时，这个宏参数会首先被全部展开,当展开后的宏参数被放进宏体时，预处理器对新展开的宏体进行第二次扫描，并继续展开
```cpp
    // 因为ADDPARAM( 1 ) 是作为PARAM的宏参数，所以先将ADDPARAM( 1 )展开为INT_1，然后再将INT_1放进PARAM
   #define PARAM( x ) x
   #define ADDPARAM( x ) INT_##x
   PARAM( ADDPARAM( 1 ) );
```   

例外情况是，如果PARAM宏里对宏参数使用了#或##，那么宏参数不会被展开
```cpp
   //将被展开为"ADDPARAM( 1 )"
   #define PARAM( x ) #x
   #define ADDPARAM( x ) INT_##x
   PARAM( ADDPARAM( 1 ) );
```

打印出一个宏被展开后的样子  
```cpp
   //TO_STRING首先会将x全部展开(如果x也是一个宏的话)，然后再传给TO_STRING1转换为字符串，现在你可以这样
   #define TO_STRING( x ) TO_STRING1( x )
   #define TO_STRING1( x ) #x
```  
如果一个像函数的宏在使用时没有出现括号，那么预处理器只是将这个宏作为一般的符号处理(那就是不处理)

## do...while(false)实现类似goto功能　　
```cpp
    do
    {
        if( A==false )
            break;　　//跳出循环　
        if( B==false )
            break;
        if( C==false )
            break;
        if( D==false )
            break;
        //实现功能代码
    }while(false);

//错误处理代码
```
## void * 
### delete void* 
```
//使用delete　void* 仅仅释放内存而不执行析构，如果析构函数里释放内存则会存在内存泄露　
//解决　
//使用shared_ptr
class A{
public:
    A(){
       std::cout << "A()" << std::endl;
    }
    ~A(){
       std::cout << "~A()" << std::endl;
    }
};

void func(std::shared_ptr<void> var){
}

int main(){
    func(std::shared_ptr<A>(new A));
    return 0;
}
```
[std::shared_ptr<void>的工作原理](https://www.cnblogs.com/imjustice/p/how_shared_ptr_void_works.html)


## 模板
* 模板实现写在源文件里链接时会找不到定义　

## 判断类中是否存在指定的方法, 变量 
[C++11模板:如何判断类中是否有指定名称的成员变量](https://cloud.tencent.com/developer/article/1433752)   
[C++11 SFINEA规则_判断类是否存在某个成员函数](https://blog.csdn.net/qq_31175231/article/details/77479692)     
[C++：如何判断类中是否存在特定的成员函数](https://blog.csdn.net/netyeaxi/article/details/83479646)     

##　判断返回对象为空　
[C++定义一个函数，返回值为一个对象，如何想办法返回一个逻辑上的空值](https://www.zhihu.com/question/51158804)
 






















