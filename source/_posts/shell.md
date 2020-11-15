---
title: shell
date: 2020-07-04 19:10:37
categories:
    - Linux
tags:
---

### 语法   
```sh
#! /bin/bash                                                                    
                                                                               
if [ $# -lt 1 ];then                                                            
   echo "arg count must > 1"                                                    
   echo "Uage bash -x example_1.sh [args...]"                                  
   exit                                                                        
fi                                                                              
                                                                               
arg=$1                                                                          
if [ $arg -gt 10 ];then                                                          
   echo "$arg > 10"                                                            
else                                                                            
   echo "$arg <= 10"                                                            
fi
```
* if [[...]] 表示模式匹配   
```sh
if [[ $trunk_path = *$dir* ]]; then                     
    trunk_path=`echo $${trunk_path} | sed "s/\/$$dir.*//"`  
fi 
```

* 数组，函数传参数，for循环
```sh
#! /bin/bash


if [ $# -lt 1 ]; then
   echo "args must > 1"
   echo "Usage bash +x example_2.sh [args...]"
fi


args=$@
for arg in $args;
do
   echo $arg
done


function func1(){
   echo $1 #$1是该函数的第一个参数
}


func1 "hello everybody" #函数传参


#第二种写法
func2(){
   echo "bash shell function defination"
}


func2 #调用
```

```sh
#! /bin/bash

array=("hello"  "gg" 1 "shit")

echo "first element is ${array[0]}"
echo "second element is ${array[1]}"
echo "third element is ${array[2]}"
echo "fourth element is ${array[3]}"
echo "all element are ${array[@]}" #所有的元素
echo "size of array is ${#array[@]}" #数组的元素个数

for arg in ${array[@]};
do
   echo $arg
done
```
* While循环以及其它几种循环、case、表达式expr  
```sh
#! /bin/bash
if [ $# -lt 1 ]; then
       echo "args must > 1"
           echo "Usage bash +x example_2.sh [args...]"
fi


case $1 in
   "install")
       echo "operation is install"
   ;;
   
   "uninstall")
       echo "operation is uninstall"
   ;;
   *)
       echo "operation is not support"
   ;;
esac


for ((i=0; i<10; i++))
do
   if ((i==1));then
       continue
   fi
   echo $i
done


for i in 'seq 5'; do
   echo "loop $i"
done
```
* 脚本之间的引用  
/lib/lsb/init-functions  
```sh
first.sh
#! /bin/bash

function fun(){
   echo "execute first script"
}

file=first

second.sh
#! /bin/bash
. first.sh  ##引用脚本
echo $file
```

* 错误处理  
$?  
set -o errexit  //遇到报错不会往下执行  
command > file 2>&1 //将stdout和stderr合并后重定向到文件  

* 字典
```sh
#! /bin/bash


set -o errexit


#方式1
hput(){
   eval "hkey_$1"="$2"
}


hget(){
   eval echo '${'"hkey_$1"'}'
}


hput k1 value1
hget k1




#方式2
declare -A dic
dic=([key1]="value1" [key2]="value2" [key3]="value3")
echo ${dic["key1"]}
echo ${!dic[@]} #打印所有的key
echo ${dic[@]} #打印所有的值


for key in ${!dic[@]}
do
   echo $key: ${dic[$key]}""
done
```



### 调试
* trap  
shell脚本执行时产生三个伪信号，可使用trap捕获 ，并输出调试信息   
```sh
  信号名              产生原因
  EXIT               从一个函数中退出或整个脚本执行完毕
  ERR                从一条命令返回非零状态时（代表命令执行不成功） 
  DEBUG              脚本中每一条命令执行之前
```
* tee　　
* 调试钩子　
* shell参数  
man sh 查看  
-n 只读取， 不执行， 例如 sh -n *.sh  好像只能检查语法是否有问题  
-x 进入跟踪方式，打印执行的每一条指令  
输出的默认值$PS4是"+"，可以设置环境变量export PS4='+{$LINENO:${FUNCNAME[0]}}';  echo $PS4 



### 命令　 　
#### ls     
```sh
    ls file | xargs rm  //查看文件并删除
    ls -l  *.cpp *.c *.h | awk '{sum+=$5} END {print sum}' // 统计文件大小总和
    ls -lt //按时间排列，时间又近及远
```

#### cp  
```sh 
    cp s1/conf/ActivityTime.json ./s[2-3]/conf/ -r  //递归拷贝文件
```

#### ps  
```sh
    ps -aux --sort=lstart    //进程按时间排序 
```
#### pstree
```sh
    pstree -p | wcl -c  //查看系统线程数
```
#### bash
```sh
 ps aux | grep Cgi | grep cgi | grep sg17|awk '{printf("kill -9 %s\n", $2);}' | bash 
```
#### ping 
```sh
    ping //是否可达
    ping -l src -c times dst //多ip情况下指定ip，times表示次数   
```
#### telnet
```sh
    telnet //端口是否开放
```
#### bg
```sh
    bg  //将任务放置后台
```

#### fg   
```sh
    fg //将任务放置前台
```

#### jobs
```sh
    jobs -l  //列出任务pid
```
#### history
```sh
    history | grep rm //查看删除文件历史
    /root/.bash_history //root用户所有历史记录（不同终端）
```
#### getconf  
```sh
    getconf WORD_BIT //获取cpu字长
```

#### ldd
```sh
    ldd //查看动态库依赖
```
#### whereis which locate  
```sh
    查看软件安装路径
```

#### date 
```sh
   date -d "00:00:00 2018-06-01" +%s //通过日期显示时戳
   date -d @1501570226  //通过时间戳显示日期
```
#### netstat  
```sh
    netstat -tunlp | grep 8061 查看端口被哪个进程占用
```

#### gcc
```sh
   -I: 第一个寻找头文件目录
   -L: 第一个寻找库文件目录
   -l: 寻找动态库或静态库
   -c: 只编译，不连接
```

#### objdump  
```sh
    objdump -s -d xx.o   //反汇编
```

#### find
```sh
   find . -name "*.cpp" | xargs grep -w "CLog"                  #全词匹配查找
   find -name file                                              #查找文件名
   find -iname file                                             #忽略大小写
   find  -maxdepth file                                         #最大递归深度
   find  -mindepth file                                         #对消递归深度
   find  -not  file                                             #相反匹配
   find  . -empty                                               #查找空文件
   find  -name file -exec cmd {} \;                             #在找到的文件上执行命令
   find . -type f                                               #查找文件
   find ~ -size +100M/-100M                                     #查找满足尺寸文件
   find / -type f -name *.zip -size +100M -exec rm -i {} \;     #查找删除
```

#### grep
```sh
   grep -v "match" file                               #出除含有匹配字符串之外的所有行
   grep  "match" file --color=auto                    #匹配字符串标颜色
   grep -E "[1-9]+" file; egrep "[1-9]+" file         #正则匹配
   grep -o "match" file                               #只输出匹配的部分
   grep -c "match" file                               #输出匹配到的行数
   grep -b "match" file                               #匹配字符的偏移量
   grep -n "match" file                               #输出匹配到字符的行数
   grep -l "match" file                               #列出匹配到字符的文件名
   grep -r "match" file                               #递归搜索匹配项
   grep -i "MATCH" file                               #忽略大小写
   grep -w "match" file                               #全词匹配
   grep -e "match1" -e "match2" file                  #多重匹配项
   grep "match" --include *.{php,html}                #只在目录中所有的.php和.html文件中递归搜索字符"main()"
   grep "main()" . -r --exclude "README"              #在搜索结果中排除所有README文件
   grep 170501686 coins_20180417.log | grep 'change pay log' |grep ',cash=-' |awk -F '[=,]' '{a+=$12} END{print a}'
   grep -E 'act=new|act=add' equipment_20180705.log | grep 'code=get_world_battle_Kill_rewards' #找出包含 get_world_battle_Kill_rewards 的行，其中act=new 或act=add
   grep get_default_error_string /* -rFn --binary-files=without-match  #只递归匹配文本文件，不匹配二进制文件中的内容

```

#### awk
```sh
   awk '{print $1, $4}' netstat.txt                                                   #打印1,4列
   awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}'  netstat.txt #格式化输出
   awk '$3==0 && $6=="LISTEN"' netstat.txt                                            #第三列的值为0 && 第6列的值为LISTEN
   awk '$3 > 0 {print $0}' netstat.txt                                                #第三列不为0的所有项
   awk '$3==0 && $6=="LISTEN" || NR==1 ' netstat.txt                                  #包括第一行
   awk '$3==0 && $6=="LISTEN" || NR==1 {printf "%-20s %-20s %s\n",$4,$5,$6}' netstat.txt   #加上格式化输出
   awk 'BEGIN{FS=":"}{print $1, $2, $5}' /etc/passwd 或 awk -F: '{print $1, $2, $5}' /etc/passwd  #指定分隔符
   awk -F'[::]' '{print $1, $2, $5}' /etc/passwd                                 #指定多个分割符
   awk '$6 ~ /FIN/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt       #匹配FIN状态
   awk 'NR!=1{print > $6}' netstat.txt                                                #拆分文件
   awk '{for(i=1;i<=NF;i++)printf "%-22s", $i "  ";printf"\n"}' netstat.txt    #打印出除开第一行的所有行   
   grep appid $file | awk -F "=" '{print $3}' | sed 's/["/>]//g' | sed 's/ //g' #过滤掉"/>和空格

```
#### sed  
```sh
   sed -i "s/my/cp/g" pet.txt                                          #将文件中所有的my替换成cp
   sed -i "s/^/#/g" pet.txt                                            #在每行最前面加#
   sed -i "s/$/---/g" pet.txt                                          #在每行最有面加---
   sed -i 's/<[^>]*>//g' html.txt                                      #去掉某html中的tags
   sed -i "3s/my/your/g" pets.txt                                      #替换第3行以后的my
   sed -i "3,6s/my/your/g" pets.txt                                    #只替换3到6行的my
   sed -i 's/s/S/1' my.txt                                             #只替换每一行的第一个s
   sed -i -e '1,3s/my/your/g' -e '3,$s/This/That/g' my.txt             #1到3行换成your, 3行以后换成That 
   sed -i s/1527696000/1528560000/g ./s*/conf/ActivityTime.json
```
#### lsof
```sh
    lsof  -p PID  | wc -l  #查看进程打开的文件数量
    lsof  -p PID  |awk '{print $2}'|sort | uniq -c | awk '{sum += $1} END{print sum}' #查看进程打开的文件数量
    lsof -i :port  #查看某个端口的连接情况
    watch "losf -p PID | wc -l"  #查看进程打开文件数量的变化情况
```

#### ip
```sh
    ip addr  //查看ip地址（可显示mac地址和网卡名称）
```
#### 通配符  
```sh
    "?"  匹配单个字符串
    "*"  匹配字符串序列
```

#### 正则　　
```sh
    ERE BRE
    "."  任意单个字符
    "*"  匹配任一字符的任意长度
    "^" 以xx开始字符
    "$" 以xx结尾字符
    "[]" 字符集合
```  


### 实战　  
终端遍历目录导入数据库  
终端中执行　　
```sh
for i in $(ls sg17_s*); do $(mysql -u root -p $(basename $i .sql) < $i);done
```
将所有db.conf 中的密码替换　　　
```sh
find ./ -name db.conf | xargs sed -i 's/password\ =\ 1234/password\ =\ Ujg5-bc@0520/g'
```



