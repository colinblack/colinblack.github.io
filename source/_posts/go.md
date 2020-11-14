---
title: go
date: 2020-07-04 15:26:14
tags:
categories:
	- go
---


<!-- more -->  
## 项目构建
* 个人  
![p1](/images/go_20200706_p1.png)

* 公司  
![p2](/images/go_20200706_p2.png)


## 工具
```go
    go fmt -w *.go   //格式化代码 
    go fmt github.com/hello   
    go run  src/xx/xx.go  //编译+执行
    go build -o /bin/test /src/test  //编译生成二进制文件， -o指定目录
    go build  github.com/xx   //注意src目录不用写
    go install  github.com/xx  //生成可执行文件到bin目录下
    go env //显示环境变量
    go get //安装第三方包  
```



## 标识符  
标识符以字母或下划线开头，大小写敏感  

* 保留关键字  
```go
    break default func interface select case defer go map struct chan else goto package switch  
    const fallthough if range type continue for import return var   
```

## 基本结构
* 可执行程序的包名必须为main, 且包含一个main函数 
* go build; go install 对于非main包会在GOPATH pkg 中生成静态库文件 



* 包的别名  
```go
    import(  
        a  "fmt"  //a是fmt的别名  
    )  
```

* 访问控制规则    
大写函数/变量可导出      
小写函数变量是私有的，外部包不能访问    

* 万能格式输出%v   

## 初始化    

* init函数    
每个包里都有0个或多个init函数， 在main函数调用之前会执行init
```go
    //无参数和返回值 
    func init(){

    }

```
* 包初始化顺序 
全局变量 -> init -> main

* 有import的情况 
![go_20200723_1](/images/go_20200723_p1.jpg)


```go
import(  
     "test"  //仅初始化， 不使用包里面的东西  
)  
```

嵌套包的初始化顺序， 最里层包先初始化     
![p3](/images/go_20200408_1.png)  

## 数据类型  

```go
    int int8 int16 int32 int64 uint8 uint16 uint32 uint64 float32 float64
```

### 标识符  
以字母或_开头, 后面跟着字母,_或数字  


### 关键字 
```go
   break default func interface select case defer go map struct 
   chan else goto package switch const fallthough  if range type 
   continue for import return var 
```
### 常量  
编译时已经确定, const修饰，只读    
常量必须要赋值   
```go  
const 只能修饰boolean, number(int相关， 浮点， complex)和string  
const b string = "hello word"  
const b = "hello word"  
const c = getValue()  //错误，与c++不同  


//优雅的写法  
const(  
    a = 0  
    b = 1  
    c = 2 
    d     //2 
)  

//更加专业的写法 
const(  
    a = iota //0  
    b  //1  
    c   //2  
)  

const(  
    a = 1 << iota //1
    b  //2 
    c   //4  
)  

const(
    a  = iota  //0
    b          //1 
    c          //2 
    d  = 8    //8  
    e         //8 
    f  = iota //5
    g         //6
)
```  
### 变量  
```go  
//第一种写法
var identifier type  
var a int  
var b string  
var c bool  
d int = 8  
e string = "hello world"  

//第二种写法
Var(  
    a int  //默认0  
    b string //默认""  
    c bool //默认false  
    d = 8  
    e = "hello world"  
)  
```  
### 值类型和引用类型  
值类型： 基本数据类型int, float, bool, string以及数组和struct，内存在栈中分配  
引用类型： 指针, slice, map, chan, interface等， 内存在对上分配，GC回收  

变量的作用域  
局部变量--生命周期在函数内或语句块内  
全局变量--生命周期在整个包，大写的可以外部访问  
 a :=1 实际是两条语句: var a int a = 1 go中不能在函数外赋值，所以:= 不能用于函数外   


### 指针类型   
var p *int = &a  
```go
//传递数组指针和c数组指针类似 
    func modify(arr *[3]int) { 
        (*arr)[0] = 90
    }
    
    func main() { 
        a := [3]int{89, 90, 91}
        modify(&a)
        fmt.Println(a) 
    }

//切片是引用类型 
func modify(sls []int) { 
     sls[0] = 90
}
func main() { 
     a := [3]int{89, 90, 91}
     modify(a[:])
     fmt.Println(a)
}

```




### 类型转化  
type(var)  
int 和 int32是不同类型， 不能直接运算  

### 字符串  
* 两种表示方式  
1."" 允许转义  
2.`` 原生字符串，允许换行  
```go  
    == //比较
    len(str)  //长度
    +, fmt.Sprintf //拼接
    strings
    strings.Contains  //包含  
    strings.HasPrefix(s string, prefix string) bool //判断字符串s是否以prefix开头  
    strings.HasSuffix(s string, suffix string) bool //判断字符串s是否以suffix结尾。  
    strings.Index(s string, str string) int  //判断str在s中首次出现的位置，如果没有出现，则返回-1  
    strings.LastIndex(s string, str string) int //判断str在s中最后出现的位置，如果没有出现，则返回-1  
    strings.Replace(str string, old string, new string, n int) //字符串替换  
    strings.Count(str string, substr string)int //字符串计数  
    strings.ToLower(str string)string //转为小写  
    strings.ToUpper(str string)string //转为大写  
    strings.TrimSpace(str string) //去掉字符串首尾空白字符  
    strings.Trim(str string, cut string) //去掉字符串首尾cut字符  
    strings.TrimLeft(str string, cut string) //去掉字符串首cut字符  
    strings.TrimRight(str string, cut string) //去掉字符串尾cut字符  
    strings.Field(str string) //返回str空格分隔的所有子串的slice  
    strings.Split(str string, split string) //返回str split分隔的所有子串的slice  
    strings.Join(s1 []string, sep string) //用sep把s1中的所有元素链接起来  
    strconv.Itoa(i int) //把一个整数i转成字符串  
    strconv.Atoi(str string)(int, error) //把一个字符串转成整数  
```  
* 字符串底层是byte数组, 可以和[]byte类型互相转换   
```go
//修改字符串  
    var str="hello"
    var byteSlice []byte
    byteSlice = []byte(str)
    byteSlice[0] = '0'
    str = string(byteSlice
```
* rune表示utf8的字符  
```go
    var str="哈"
	var runeSlice []rune
	runeSlice = []rune(str)
	fmt.Printf("str 长度:%d, len(str)=%d\n", len(runeSlice), len(str) //1, 3
```


### 时间和日期  
```go  
    //获取当前时间
    now := time.Now()  

    //获取时间戳
    timestamp := time.Now().Unix()

    //时间戳转time类型  
    timeobj := time.Unix(timestamp, 0)

    time.Now().Day()，time.Now().Minute()，time.Now().Month()，time.Now().Year()  

    //格式化
    fmt.Printf(“%02d/%02d%02d %02d:%02d:%02d”, now.Year()......)  
    //time.Duration用来表示纳秒  
    //一些常量  
    const (  
        Nanosecond  Duration = 1  
        Microsecond          = 1000 * Nanosecond  
        Millisecond          = 1000 * Microsecond  
        Second               = 1000 * Millisecond  
        Minute               = 60 * Second  
        Hour                 = 60 * Minute  
    )  
    //格式化
    now := time.Now()  
    fmt.Println(now.Format(“02/1/2006 15:04”))  
    fmt.Println(now.Format(“2006/1/02 15:04”))  
    fmt.Println(now.Format(“2006/1/02”))  
```  
* 定时器  
```go
	ticker := time.Tick(1*time.Second)
	for i := range ticker {
		fmt.Printf("%v\n", i) //打印出当前时间 
	}
```


## 流程控制  
```go  
    //if  
    if condition {  
    }  
    
    if condition {  
    } else {  
    }  
    
    if condition1 {  
    } else if condition2 {  
    } else if condition3 {  
    } else {  
    }  
    //错误代码
    if condition {  
    }  
    else{     //else必须和上一个}在一行否则编译不过去  
    }  
    //switch  
    switch var {  
        case var1:  
        fallthrough //继续往下执行  
        case var2:  
        case var3:  
        default:  
    }  
    //注意没有break  
    （2）switch var {  
            case var1, var2: //多条件在一行  
            case var 3:  
            default:  
            }  
     (3) switch{  //没有变量  
                condition1:  
                condition2:  
                default:  
            }  
     (4) switch 语句块 {  
         }  
     //for  
     (1) for i := 0; i < 100;  i++{  //注意for没有()  
        }  

     (2)  for 条件{  
        }  

     (3) fro range  
        str := “hello world,中国”  
        for i, v := range str {  
            fmt.Printf(“index[%d] val[%c] len[%d]\n”, i, v, len([]byte(v)))  
        }  
```  

## 函数
```go
//其中参数列表和返回值列表是可选
func functionname([parametername type]) [returntype] { 
//function body
}

```

不支持重载，一个包不能有两个名字一样的函数  
函数是一等公民，函数也是一种类型，一个函数可以赋值给变量  
匿名函数  
多返回值  
```go
    func calc(a, b int) (sum int, sub int) {
        sum = a + b
        sub = a - b
        return  //返回sum, sub
    }

    sum, _ := calc(100, 200); //忽略返回值  

```

可变参
```go
    func calc_v2(a int, b ...int) int {
        sum := a
        for i := 0; i < len(b);i++ {
            sum = sum + b[i]
        }
        return sum
    }

    sum := calc_v2(10);
```

### defer  
当函数返回时，执行defer语句。因此，可以用来做资源清理  
多个defer语句，按先进后出的方式执行  
defer语句中的变量，在defer声明时就决定了   
```go  
    //关闭文件句柄 
    func read() {  
        file := open(filename)  
        defer file.Close()  
    }  
    //锁资源释放  
    func read() {  
        mc.Lock()  
        defer mc.Unlock()  
        //其他操作  
    }    
    //数据库连接释放  
    func read() {  
        conn := openDatabase()  
        defer conn.Close()  
        //其他操作  
    }  

    func testDefer3() {
	    var i int = 0
	    defer fmt.Printf("defer i=%d\n", i)  //打印结果i=0
	    i= 1000
	    fmt.Printf("i=%d\n", i)
    }
```  

### 内置函数  
```go  
    不需要导入包  
    close：主要用来关闭channel  
    len：用来求长度，比如string、array、slice、map、channel  
    new：用来对各种类型分配内存，主要用来分配值类型，比如int、struct。返回的是指针  
    make：用来对内建类型分配内存，主要用来分配引用类型，比如channel map slice  
    append：用来追加元素到数组、slice中  
    panic和recover：用来做错误处理  

    var b *[]int = new([]int) 
    //*b[0] = 100 error, 需要用make初始化 
    (*b) = make([]int, 5, 100)

```  



* new和make的区别  
![p4](/images/go_20200408_2.png)  


### 匿名函数  
```go
    func testFunc2() {
        f1 := func (a, b int) int {  //匿名函数  
            return a + b
        }

        fmt.Printf("type of f1=%T\n", f1) //%T类型 
        sum := f1(2, 5)
        fmt.Printf("sum=%d\n", sum)
    }

    //defer和匿名函数  
    func testFunc4() {
        var i int = 0
        defer func() {
            fmt.Printf("defer i=%d\n", i) //i=100
        }()

        i = 100
        fmt.Printf("i=%d\n", i)
        return
    }
```
### 函数类型作参数 
```go
    func calc(a , b int, op func(int, int)int) int {
        return op(a, b)
    }

```

### 闭包
```go
    //eg1
    func add(base int) func(int) int {
    return func(i int) int {
    base += i
    return base
    }
    }

    tmp1 := add(10)
    fmt.Println(tmp(1), tmp(2))  //11, 13
    tmp2 := add(100) //101, 103
    fmt.Println(tmp2(1), tmp2(2))

    //eg2
    func makeSuffixFunc(suffix string) func(string) string { 
    return func(name string) string { 
    if !strings.HasSuffix(name, suffix) { 
    return name + suffix 
    } 
    return name 
    } 
    }

    func1 := makeSuffixFunc(".bmp") 
    func2 := makeSuffixFunc(".jpg") 
    fmt.Println(func1("test"))  //test.bmp
    fmt.Println(func2("test")) //test.jpg


    //eg3
    func calc(base int) (func(int) int, func(int) int) {
    add := func(i int) int {
    base += i
    return base
    }
    sub := func(i int) int {
    base -= i
    return base
    }
    return add, sub
    }


    f1, f2 := calc(10)
    fmt.Println(f1(1), f2(2)) //11, 9
    fmt.Println(f1(3), f2(4)) //12, 8
    fmt.Println(f1(5), f2(6)) //13, 7
    fmt.Println(f1(7), f2(8)) //14, 6

    //eg4
    for i:=0; i<5; i++ {
    go func(){
        fmt.Println(i) //i全=5
    }()

    go func1(i int){
        fmt.Println(i)
    }(i)    //1,2,3,4,5

    }
    time.Sleep(time.Second)

```

## 数组   
* 定义
```go
    var a [len]int
    //eg
    var a[5]int 
```
长度是数组类型的一部分，因此，var a[5] int和var a[10]int是不同的类型, 不能a=b  
数组可以通过下标进行访问，下标是从0开始，最后一个元素下标是：len-1  
访问越界，如果下标在数组合法范围之外，则触发访问越界，会panic  
数组是值类型，因此改变副本的值，不会改变本身的值 

* 初始化  
整数数组元素默认初始化是0, 字符串数组默认初始化"", 浮点型是0.0
```go
var age0 [5] int = [5] int{1,2,3} 
var age1 = [5]int{1,2,3,4,5}  
var age2 = [...]int{1,2,3,4,5,6}  
age3 := [5]int{1,2,3} //1 2 3 0 0  
var str = [5]string{3:”hello world”, 4:”tom”} //指定索引赋值  
```  

* 遍历  
```go
    
    for i := 0; i < len(a); i++ {
		fmt.Printf("a[%d]=%d\n", i, a[i])
	}
    
    for _, value := range a {
		fmt.Printf("%d\n", value)
    }
```

* 多维数组 
```go 
    var age [5][3] int  
    var age [5][3] int = [...][3]int{{1, 2 , 3}, {4, 5, 6}}  

	for i := 0; i < 3; i++ {
		for j := 0; j < 2; j++ {
			fmt.Printf("%d ", a[i][j])
		}
		fmt.Println()
	}

	for i, val := range a {
		fmt.Printf("row[%d]=%v\n", i, val)
		for j, val2 := range val {
			fmt.Printf("(%d,%d)=%d ",i, j, val2)
		}
		fmt.Println()
    }
```

* 数组拷贝
数组是值类型
```go
    //b初始化时是深拷贝    
	a := [3]int{10, 20, 30}
	b := a
	b[0] = 1000
	fmt.Printf("a=%v\n", a) //10, 20, 30
	fmt.Printf("b=%v\n", b) //1000, 20, 30
```
* 数组传参 
```go
    //数组在传参的时候会深拷贝   
    func modify(b [3]int) {
	    b[0] = 1000
    }

    func modify(b [3][3]int){
        b[2][1] = 0
    }
```


## 切片
基于数组类型做的一层封装，可以自动扩容  
切片是数组的一个引用，因此切片是引用类型    
切片的长度可以改变，因此，切片是一个可变的数组  

### 定义 
```go
    var 变量名 []类型
    //eg
    var str []string  
    var arr []int

```

### 初始化
```go
    //方法1 基于数组  
    //a[start:end]创建一个包括从start到end-1的切片  
    a := [5]int{1,2,3,4,5}
    var b[]int = a[1:4] //2 3 4 

    //eg
    a := [...]string{"a", "b", "c", "d", "d", "f", "g", "h"}
    b := a[1:3]  //b的长度是2, cap是7
    //方法二
    c := []int{6,7,8} //写长度就是数组  
```

### 操作
```go
    //包含start到end之间的元素，但不包含end  
    var slice []int = arr[start:end]  
    var slice []int = arr[:end]  //var slice []int = arr[0:end]
    var slice[]int = arr[start:] //var slice []int = arr[start:len(arr)]
    var slice[]int = arr[:] //var slice []int = arr[0, len(arr)]

```

```go  
    //创建
    var slice []type = make([]type, len)  
    slice  := make([]type, len)  
    slice  := make([]type, len, cap)  //如果使用[]访问超过len的空间，需要使用append插入元素, 否则会panic    
```  
![p6](/images/go_20200408_4.png) 

```go
    //将切片追加到另一个切片末尾  
    var a []int = []int{1, 3, 4}
	var b []int = []int{4, 5, 6}
	a = append(a, 23, 34, 45)
	a = append(a, b...) //将b展开
```
* 拷贝
```go
	var a []int = []int{1}
	var b []int = []int{4, 5, 6}
	copy(a, b) //不会对被拷贝的切片扩容  
```

* cap
cap可以求出slice最大的容量，0 <= len(slice) <= cap(array)，其中array  
如果要切片最后一个元素去掉，可以这么写: slice = slice[:len(slice)-1]  

* 切片再切片
```go
	a := [...]string{"a", "b", "c", "d", "d", "f", "g", "h"}
	b := a[1:3] //len2, cap 7
	b = b[:cap(b)] //len 7 cap 7
```

* 空切片
```go
    var a []int //直接操作会panic
    if a == nil { //判空 
		fmt.Printf("a is nil\n")
	}

    a = append(a, 100) //对空切片扩容  
```

* 传参  
```go
    func sumArray(a []int) int {
        var sum int = 0
        for _, v := range a {
            sum = sum + v
        }
        return sum
    }

    var a [10]int = [10]int{1, 3, 3, 4, 5, 5, 8}
    sum := sumArray(a[:]
    
```



### 切片的内存布局    
![p5](/images/go_20200408_3.png)  


### string与slice  
string底层就是一个byte的数组，因此，也可以进行切片操作  
```go  
str := “hello world”  
s1 := str[0:5]  
fmt.Println(s1)  
s2 := str[5:]  
fmt.Println(s2)  
```  

### 切片示例
[生成密码](https://github.com/colinblack/go_devel/blob/master/tools/passwd/passwd.go)




## map  
```go  
//key-value的数据结构，又叫字典或关联数组  
//声明是不会分配内存的，初始化需要make  
// map是引用类型  
var map1 map[keytype]valuetype  
var a map[string]string  
var a map[string]int  
var a map[int]string  
var a map[string]map[string]string  

//申明时初始化
var a map[string]string = map[string]string{“hello”: “world”}  
//make初始化
a := make(map[string]string, 10)  

```
* 操作  
```go

    a[“hello”] = “world”            //插入和更新  
    Val, ok := a[“hello”]           //查找  
    for k, v := range a {           //遍历  
        fmt.Println(k,v)  
    }  
    delete(a, “hello”)              //删除  
    len(a)                          //长度  

    //slice of map  
    Items := make([]map[int][int], 5)  
    For I := 0; I < 5; i++ {  
            items[i] = make(map[int][int])  
    }  

```  




* 排序  
map中key值是无序的  
a. 先获取所有key，把key进行排序  
b. 按照排序好的key，进行遍历  

* 翻转   
初始化另外一个map，把key、value互换即可  


## 并发  
* 线程同步   
a. import(“sync”)  
b. 互斥锁, var mu sync.Mutex  
c. 读写锁, var mu sync.RWMutex  


## struct  
* 用来自定义复杂数据结构  
* struct里面可以包含多个字段（属性）  
* struct类型可以定义方法，注意和函数的区分  
* struct类型是值类型  
* struct类型可以嵌套  
* Go没有class类型，只有struct类型  
* 结构体内字段地址连续
* struct没有构造函数，一般可以使用工厂模式来解决这个问题  

```go  
    type student struct {  
        Name stirng  
        Age int  
    }  

    //初始化 
    user := student{
        Name : "user",
        Age  : 18,
    }

    func NewStudent(name string, age int) *student {  
        return &student{  
            Name:name,  
            Age:age,  
        }  
    }  


    S := new (student)  
    S := model.NewStudent(“tony”, 20)  
```  

* 指向结构体的指针 
```go
    var user *student //nil
    var user01 *student = &student{}
    user01.Name="user01" //指针使用.操作，是编译器简化了

    var user02 *student = &student{
        Name : user02,
        Age  : 18,
    }

    var user03 *student = new(student)
```

* 结构体嵌套 
```go
    type Address struct {
        Province string
        City     string
    }

    type User struct {
        Username string
        Sex      string
        address  *Address
    }

	user := &User{
		Username: "user01",
		Sex:      "man",
		address: &Address{
			Province: "beijing",
			City:     "beijing",
		},
	}
```


* 匿名字段   
```go
    //匿名字段默认采用类型名作为字段名 
    type User struct { 
        Username string
        Sex string
        Age int
        AvatarUrl string
        int
        string
    }

    var s User
    s.int = 100
    s.string="hello"

    //嵌套结构体匿名字段 
    type Address struct {
	    Province   string
    	City       string
    	CreateTime string
    }
    
    type User struct {
    	Username string
    	Sex      string
    	*Address
    }

    //方法1
    var user User
    user.Address = &Address{
		Province: "bj",
		City:     "bj",
	}

    //方法2
    user.Province = "bj01"
	user.City = "bj01"


    //字段冲突解决 
    type User01 struct {
        City     string
        Username string
        Sex      string
        *Address
        *Email
    }

    var user01 User01
    user01.City = "bj" //此时因为User01中有City被有限房屋， 如果没有再访问Address中City
    user.Address.CreateTime = "001" //Address和Email中都有CreateTime需要指明 

```

* tag
结构体的元信息，可以在运⾏的时候通过反射的机制读取出来
```go
    type User struct { 
        Username string `json:”username”,db:”user_name”`
        Sex string `json:”sex”`
        Age int `json:”age”`
        avatarUrl string
        CreateTime string
    }

    var user User

    data, _ := json.Marshal(user)


```
* 结构体与json序列化  




## 方法  
Golang中的方法是作用在特定类型的变量上，因此自定义类型，都可以有方法，而不仅仅是struct  
```go
    //定义
    func (recevier type) methodName(参数列表)(返回值列表){}  


    type People struct {
        Name    string
        Country string
    }

    func (p People) Print() {
        fmt.Printf("name=%s country=%s\n", p.Name, p.Country)
    }

    func (p People) Set(name string, country string) {
        p.Name = name
        p.Country = country
    }

    func (p *People) SetV2(name string, country string) {
        p.Country = country
        p.Name = name
    }

	var p1 People = People{
		Name:    "people01",
		Country: "china",
	}
    p1.Set("people02", "enligsh")  //传值不能改变p1的成员
    (&p1).SetV2("people02", "english") //要传指针 
    p1.SetV2("people02", "english")

    //可以为这个包中的任意类型增加方法  
    type Integer int
    func (i Integer) Print() {
	fmt.Println(i)
}


```



## 链表  
```go  
type Student struct{  
    Name string  
    Next *Student  
}  
```  


## 继承   
通过匿名字段来实现   
```go
    type Animal struct {
        Name string
        Sex  string
    }

    func (a *Animal) Talk() {
        fmt.Printf("i'talk, i'm %s\n", a.Name)
    }

    type PuruAnimal struct {
    }

    func (p *PuruAnimal) Talk() {
        fmt.Println("buru dongwu talk")
    }

    type Dog struct {
        Feet string
        //Animal
        *Animal   
        *PuruAnimal
    }

    func (d *Dog) Eat() {
        fmt.Println("dog is eat")
    }

    /*
    func (d *Dog) Talk() {
        fmt.Println("dog is talking")
    }*/

    func main() {
        var d *Dog = &Dog{
            Feet: "four feet",
            Animal: &Animal{
                Name: "dog",
                Sex:  "xiong",
            },
        }

        d.Eat()
        d.Animal.Talk() 
        d.PuruAnimal.Talk()
    }

```




## 接口  
Interface类型可以定义一组方法，但是这些不需要实现。并且interface不能包含任何变量  
如果一个变量含有了多个interface类型的方法，那么这个变量就实现了多个接口  
如果一个变量只含有了1个interface的方部分方法，那么这个变量没有实现这个接口  

## 类型断言  
类型断言，由于接口是一般类型，不知道具体类型，如果要转成具体类型可以采用以下方法进行转换  
```go  
var t int  
var x interface{}  
x = t  
y, ok = x.(int)   //转成int，带检查  
```  

## 反射  
```go  
可以在运行时动态获取变量的相关信息  
import (“reflect”)  
reflect.TypeOf，获取变量的类型，返回reflect.Type类型  
reflect.ValueOf，获取变量的值，返回reflect.Value类型  
reflect.Value.Kind，获取变量的类别，返回一个常量  
reflect.Value.Interface()，转换成interface{}类型  
reflect.Value.Kind()方法返回的常量  
```  
![p7](/images/go_20200408_5.png)  

## IO  
```go  
终端读写  
os.Stdin：标准输入  
os.Stdout：标准输出  
os.Stderr：标准错误输出  

文件写入  
os.OpenFile(“output.dat”,  os.O_WRONLY|os.O_CREATE, 0666)  
第二个参数：文件打开模式：  
1. os.O_WRONLY：只写  
2. os.O_CREATE：创建文件  
3. os.O_RDONLY：只读  
4.  os.O_RDWR：读写  
5.  os.O_TRUNC ：清空  
第三个参数：权限控制：  
r ——> 004  
w——> 002  
x——> 001  

命令行参数  
os.Args是一个string的切片，用来存储所有的命令行参数  
```  
## 序列化  
### Json  
```go  
导入包：import “encoding/json”  
序列化: json.Marshal(data interface{})  
反序列化: json.UnMarshal(data []byte,  v  interface{})  
```  

## goroutine  
不同goroutine之间如何通信  
* 全局变量和锁同步  
* Channel  
  类似unix中的管道  
  先进先出  
  线程安全, 多个goroutine同时访问, 不需要加锁  
  channel是有类型的, 一个整数的channel只能存放整数  
  channel带缓冲区  

## 单元测试  
文件名必须以_test.go结尾
函数名必须以Test开头



