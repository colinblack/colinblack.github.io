---
title: c++中对象数组初始化问题
date: 2020-07-26 10:10:32
tags:
categories:
    - c++
---


如果对象数组中没有默认构造函数，要像下面这样初始化:
```cpp
#include <iostream>
class A{
public:
    A(int* p)
       : p_(p){
        std::cout << "A" << std::endl;  
    }   
private:
    int *p_;
};
int main(){
   int i;
   A a[3] = {&i, &i, &i}; //如果有100万个元素就要写100万个&i
}
```
有两种解决思路
*  使用std::array 和 std::index_sequence
*  利用宏自动生成100万个&i


### 利用 std::array 和 std::index_sequence
```cpp
#include <iostream>
#include <utility> 
#include <array>
#include <algorithm>

struct foo_t{
    foo_t(int* i)
     : i_(i){
        std::cout << "foo_t" << std::endl; 
    }
    foo_t(const foo_t& obj){
        std::cout << "foo_t copy" << std::endl; 
        i_ = obj.i_; 
    }
   int* i_;
};

foo_t make_foo(int* i)
{
    return foo_t(i);  //这里如果使用std::forward等于将右值转为左值,左值生成一个副本，多了一次拷贝
}
    

template<size_t...Is>
std::array<foo_t, sizeof...(Is)> make_foos(std::index_sequence<Is...>, int* i) {
    return { (Is, make_foo(i))... };
}

template<size_t N>
std::array<foo_t, N> make_foos(int* i) {
    return make_foos(std::make_index_sequence<N>(), i);
}
    	

int main(){
    int i = 1;
    std::array<foo_t, 10> a = make_foos<10>(&i);  //返回值不会有拷贝，详见														rvo
    return 0;
}
```
但是这种方式也有缺陷，当元素为100万时编译速度很慢, gcc异常退出 


### 使用宏展开 
参考ttl库的实现   
```cpp
#include <iostream>
#include <string>

#define TTL_LAST_REPEAT_0(m,p)
#define TTL_LAST_REPEAT_1(m,p) m(1,p)
#define TTL_LAST_REPEAT_2(m,p) m(2,p)
#define TTL_LAST_REPEAT_3(m,p) m(3,p)
#define TTL_LAST_REPEAT_4(m,p) m(4,p)
#define TTL_LAST_REPEAT_5(m,p) m(5,p)
#define TTL_LAST_REPEAT_6(m,p) m(6,p)
#define TTL_LAST_REPEAT_7(m,p) m(7,p)
#define TTL_LAST_REPEAT_8(m,p) m(8,p)
#define TTL_LAST_REPEAT_9(m,p) m(9,p)
#define TTL_LAST_REPEAT_10(m,p) m(10,p)

#define TTL_REPEAT_0(m,l,p)
#define TTL_REPEAT_1(m,l,p) TTL_REPEAT_0(m,l,p) m(1,p)
#define TTL_REPEAT_2(m,l,p) TTL_REPEAT_1(m,l,p) m(2,p)
#define TTL_REPEAT_3(m,l,p) TTL_REPEAT_2(m,l,p) m(3,p)
#define TTL_REPEAT_4(m,l,p) TTL_REPEAT_3(m,l,p) m(4,p)
#define TTL_REPEAT_5(m,l,p) TTL_REPEAT_4(m,l,p) m(5,p)
#define TTL_REPEAT_6(m,l,p) TTL_REPEAT_5(m,l,p) m(6,p)
#define TTL_REPEAT_7(m,l,p) TTL_REPEAT_6(m,l,p) m(7,p)
#define TTL_REPEAT_8(m,l,p) TTL_REPEAT_7(m,l,p) m(8,p)
#define TTL_REPEAT_9(m,l,p) TTL_REPEAT_8(m,l,p) m(9,p)
#define TTL_REPEAT_10(m,l,p) TTL_REPEAT_9(m,l,p) m(10,p)

#define TTL_CNTDEC_0 0
#define TTL_CNTDEC_1 0
#define TTL_CNTDEC_2 1
#define TTL_CNTDEC_3 2
#define TTL_CNTDEC_4 3
#define TTL_CNTDEC_5 4
#define TTL_CNTDEC_6 5
#define TTL_CNTDEC_7 6
#define TTL_CNTDEC_8 7
#define TTL_CNTDEC_9 8
#define TTL_CNTDEC_10 9

#define TTL_APPEND( x, y ) TTL_APPEND1(x,y) 
#define TTL_APPEND1( x, y ) x ## y
#define TTL_DEC(n) TTL_APPEND(TTL_CNTDEC_, n)

#define TTL_REPEAT(n, m, l, p) TTL_APPEND(TTL_REPEAT_, TTL_DEC(n))(m,l,p) TTL_APPEND(TTL_LAST_REPEAT_,n)(l,p)
#define TTL_TPARAM(n,t)  t,
#define TTL_TPARAM_END(n,t) t
#define TTL_TPARAMSX(n,t) TTL_REPEAT(n, TTL_TPARAM, TTL_TPARAM_END, t)
#define TTL_TPARAMS(n, T) TTL_TPARAMSX(n,T)

#define TO_STRING( x ) TO_STRING1( x )
#define TO_STRING1( x ) #x

class A{
public:
    A(int* i)
    :i_(i){
        std::cout << "A" << std::endl; 
    }
	
	int * i_;
};

int main(){
//  std::cout << TO_STRING(TTL_TPARAMSX(1, &i)) << std::endl;
    int i = 1;
    A a[10] = {TTL_TPARAMSX(10, &i)};

    return 0;
}


```
### 使用placement new  
```cpp
#include <iostream>
#include <type_traits>

class A{
public:
    A(int* p)
        : p_(p){
            std::cout << "A" << std::endl;
    }
private:
    int *p_;
};

int main(){
    using elemType = std::aligned_storage<sizeof(A), alignof(A)>::type;
    const size_t count = 1000000;

    int i;
    elemType a[count];
    
    for(int idx = 0; idx < count: ++idx) {
         new (&a[idx]) A(&i);
    }
    
    ...
    
    for(int idx = 0; idx < count: ++idx) {
         reinterpret_cast<A&>(a[idx]).~A();
    }
    
    return 0;
}
```

参考:   
[initialize an array of object in c++ ](https://stackoverflow.com/questions/63068079/initialize-an-array-of-object-in-c)  
[代码自动生成-宏带来的奇技淫巧](http://www.cppblog.com/kevinlynx/archive/2008/03/19/44828.aspx)  
[用C语言宏批量生成代码的思考与实现](https://zhou-yuxin.github.io/articles/2016/%E7%94%A8C%E8%AF%AD%E8%A8%80%E5%AE%8F%E6%89%B9%E9%87%8F%E7%94%9F%E6%88%90%E4%BB%A3%E7%A0%81%E7%9A%84%E6%80%9D%E8%80%83%E4%B8%8E%E5%AE%9E%E7%8E%B0/index.html)









