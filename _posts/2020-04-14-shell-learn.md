---
title: "[Shell] Shell学习记录"
search: false
author: "Shadow"
---

- 

# 常用操作

### 日期格式化

```
finish_time=$(date --date='0 days ago' "+%Y-%m-%d %H:%M:%S")
```

###  删除当前目录下行数为1的文件

```
wc -l *.txt | awk -F" " '$1=="1"{ print $2}' | xargs -n1 rm -f
```



### 统计目录下最大的10个文件

```
du -sb * | sort -rn |head -10 | awk -F"\t" '{print $2}' | xargs -n1  du -sh 
```

### 获得文件行数

- 第一种：

  ```
  n_line=`sed -n '$=' file_name`
  echo $n_line
  ```

- 第二种：

  ```
  n_line=`wc -l file_name | awk '{print $1}'`
  echo $n_line
  ```

### 计算时间差

- 利用Unix时间戳计算

```
start=$(date +%s)
end=$(date +%s)
#计算时间间隔
time=$(( $end - $start )) # 秒级别
echo $time
```

- 利用时间日期来计算

```
start_time=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`

程序代码...

finish_time=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`
duration=$(($(($(date +%s -d "$finish_time")-$(date +%s -d "$start_time")))))
echo "time cost: ${duration}s"
```

- 时间戳和时间字符串互换

```
 date -d "2015-08-04 00:00:00" +%s     输出：1438617600
  date -d @1438617600  "+%Y-%m-%d %H:%M:%S"    输出：2015-08-04 00:00:00
```



### 数字运算

```
#!/bin/bash
i=0
j=0
k=0
while (($i < 1))
do
    echo "\n=========================";
    ((i += 1))    # 第一种:使用(())
    echo "i=$i";

    j=`expr $j + 1`   # 第二种:使用expr 
    echo "j=$j";

    let "k+=1"    # 第三种:使用let 不可以有空格
    echo "k=$k";
    let k+=1
    echo "k=$k";
    let k=k+1
    echo "k=$k";
    
    echo "=========================";
done

```

### 切割文件

`split`命令参数

- `-a`   设置切割后文件名的可变范围（默认为2）即xaa-xzz
- `-b`   设置按大小分割文件(可以使用K, M, G, T, P, E, Z, Y (powers of 1024) or KB, MB, ... (powers of 1000))
- `-C`   设置每一行的最大bytes
- `-d`   将输出文件名可变改为数字模式（未改变-a参数的话，文件名：x00-x99）
- `-e`   省略空文件输出
- `-l`   设置按文件行数分割文件
- `指定输出文件的前缀`  命令最后写上文件前缀即可

使用示例

- 将文件分割成n个100MB的文件

### 删除空行

-  使用sed

  ```
  sed -i '/^$/d' test.txt
  ```

- 使用grep

  ```
  grep -v "^$" test.txt > test_new.txt
  ```

### 去除重复行

- 使用 sort + uniq 命令

  ```
  sort test.txt | uniq > test_uniq.txt
  ```

### tcpdump抓包统计来源方IP

- 使用`tcpdump`+`perl`

  ```
  sudo tcpdump -i eth0 -Nnn port 11668  2>/dev/null | perl -alne 'BEGIN{$| = 1};$ip = $1 if $F[2] =~ /([\d.]+)\.\d+/; print $ip unless $conn{$ip}++;'
  ```

- 使用`tcpdump`+`sort`+`uniq`

  ```
  tcpdump -i eth0 port 11668 -nn -c 5000 2>/dev/null | awk '{print $3}' | awk -F '.' '{print $1"."$2"."$3"."$4}' | sort | uniq -c | grep -v "`ifconfig | grep -A3 "eth0" | grep "inet" | awk '{print $2}'`"
  ```

- 统计请求某个端口次数最多的`top N`

  ```
  # 先统计所有的IP
  sudo tcpdump -i eth1 'dst port 13793'  | awk '{print $3}' | awk -F '.' '{print $1"."$2"."$3"."$4}' > all_ip.txt
  # 然后计算访问次数并输出top N
  awk '{sum[$1]+=1} END {for(k in sum) print k "\t" sum[k]}' all_ip.txt | sort -k 2 -rn | head -10
  ```
  


###  查看进程运行路径

`ls -l /proc/{进程PID}/cwd`

# cut命令

>  作用:从文件中的每一行截取一部分并输出

基本语法: `cut [参数列表]  [文件名] `

参数：

- `-d`:分隔符,默认为Tab,例如:`cut -d':' xxx`
- `-f`:截取的字段 ,例如:`cut -f1 xxx`
- 截取字符串,例如：`echo "abc_1234" | cut -c5- `，结果为:`1234`

# sed命令

### sed替换文件中的字符串

- 将文件中的 `"null" `替换为 `null  `

```
# -i 会更改文件  不适用-i不会更改文件 而是输出替换后的内容
sed -i 's/\"null\"/null/g' test.txt
```



# awk命令

## 使用awk处理多文件时,NR与FNR变量

NR：表示awk开始执行程序后所读取的数据行数。
FNR：awk当前文件读取的记录数，其变量值小于等于NR（比如当读取第二个文件时，FNR是从0开始重新计数，而NR不会）。

NR==FNR：正在读取第一个文件

NR>=FNR：正在读取第二个文件

- eg-1：以两个文件的第一列为标识，将`file2`的第二列补充`file1`的对应行后面

  file1内容

  ```
  jack 20
  jay 30
  john 21
  lili 22
  ```

  file2内容

  ```
  john beijing
  jack jiangsu
  ```

  执行命令：`awk -F"\t" 'NR==FNR{a[$1]=$0;next}NR>FNR{if ($1 in a) print a[$1]"\t"$2}' file1 file2`后的输出

  ```
  john 21 beijing
  jack 20 jiangsu
  ```

- eg-2：将第二列为 3 的行打印出来

  ```
  awk -F"\t" '{if ($2=="3") print $0}' file_name
  ```

- eg-3：如果 `file1`中的第一列值为 `hello` ,则打印`该行内容 + HELLO` ,否则打印`该行内容 + NO`

  ```
  awk -F " " '{if ($1=="hello") print $0"\t""HELLO";else print $0"\t""NO"}' file1
  ```

- eg-4：以$1为A、B文件关联，打印B文件在A文件中的行

  ```
  awk -F"\t" 'NR==FNR{ a[$1]=$0;next }NR>FNR{ if ($1 in a) print $0}' A.txt B.txt
  ```

- eg-4：以$1为A、B文件关联，打印B文件不在A文件中的行

  ```
  awk -F"\t" 'NR==FNR{ a[$1]=$0;next }NR>FNR{ if ($1 in a == 0) print $0}' A.txt B.txt
  ```




# uniq 命令

- uniq -c 统计重复行的次数



# free命令

## 作用: 查看内存使用情况

## 详细解释：

`free` 默认 `-k`，显示的是`KB`

```
[root@TENCENT64 /usr/local/services/cc_account_canal_admin_test-1.0/bin]# free   
              total        used        free      shared  buff/cache   available
Mem:        3917244     2208696      156112      208316     1552436     1424064
Swap:             0           0           0
```

- total：总共的物理内存

- used：已经使用的内存

- free：可使用的内存

- shared：被进程共享使用的内存

- buff/cache：磁盘缓存内存，buff是写入磁盘时所用的内存，cache是读取磁盘时缓存下来的内存。对于OS而言，buff/cache属于已经被占用的内存，不可以再用(除非被释放)，对于应用程序而言，buff/cache属于可以使用的内存，当应用程序需要使用时buff/cache会被OS很快回收。

  对于buff/cache的英文解释：

  > A buffer is something that has yet to be "written" to disk.
  > A cache is something that has been "read" from the disk and stored for later use.

- swap：交换分区，也就是虚拟内存。当可用内存少于额定值的时候，就会开始进行交换

## 结论

应用程序可使用的内存 = `free` + `buff/cache`

OS可使用的内存 = `free`



# grep命令

常用参数解释

`-An`： 向前显示n行，即将检索行的下n行也显示出来

`-Bn`：向后显示n行，即将检索行的上n行也显示出来

`-Cn`：将检索行的前后n行都显示出来

`-n`: 显示行号

`-v`：反向筛选

`-e`：实现多个筛选 or 关系 例如：`grep -e "hello" -e "hi" eg.txt`

`-E`：使用正则表达式

`-w`：显示整个单词

`-c`：统计匹配的行数

`-i`：忽略大小写

`-o`：仅显示匹配到的字符串

`-q`：安静模式，脚本里常用

`-s`：不显示错误信息

`--color`：以颜色突出显示匹配到的字符串

`-o`：

`-o`：







eg1: 把以abc开头的行显示出来 `grep "^abc" eg.txt`

eg2：查看当前目录中包含某关键字的所有文件`grep -r .`





# Linux数组

```
# 定义数组
array=()

# 计算数组长度
length=${#array[@]}

# 数组添加元素
array+=($xxx)

# 获得数组某个位置元素
i=10
echo "${array[$i]}"
```



# QA

### Q：`/var/vda1`满了

```
[root@ubuntu /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1       9.8G  9.4G     0 100% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  187M  1.7G  10% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/vdb1        99G   86G  8.0G  92% /data
tmpfs           383M     0  383M   0% /run/user/0
```

### A：

一般是`/var/log/` 下的日志太多了，清理一下就好了

```
cd /var/log
// 找出占用空间最大的五个文件
du -sb * | sort -rn |head -10 | awk -F"\t" '{print $2}' | xargs -n1  du -sh 
// 清理占用空间大的日志
rm -f xxx
// 即使清理完之后也可能还是现实100%占用。是因为有进程仍然在写被删除的文件。需要kill掉进程
lsof | grep deleted | grep "/var/log"
kill -9 xxx
```

| 服务                                  | 功能 |
| ------------------------------------- | ---- |
| goneat_qeh_account_query_and_sync_svr |      |



### Q：`删除文件后磁盘空间仍然未释放`

```
当看到某个文件占用非常大的磁盘空间时，我们使用rm -f 删除,  删除完成后 df -h 发现磁盘空间未发生变化
原因:有进程仍然在使用该文件
```

### A：

```
// 显示已经被删除但仍在使用的文件
lsof | grep deleted 

```









# 终端操作快捷键

| 快捷键   | 作用                   |
| -------- | ---------------------- |
| ctrl + a | 光标移动至行首         |
| ctrl + e | 光标移动至行尾         |
|          |                        |
| ctrl + k | 清除光标后至行尾的内容 |
| ctrl + u | 清除光标前至行首的内容 |
|          |                        |
| ctrl + w | 删除光标前一个单词     |
|          |                        |
| ctrl + f | 向右移动光标           |
| ctrl + b | 向左移动光标           |
|          |                        |
| esc + b  | 移动到当前单词的开头   |
| esc + f  | 移动到当前单词的结尾   |
|          |                        |
| ctrl + h | 退格向左删除           |
| Ctrl + d | 退格向右删除           |
|          |                        |
| ctrl + w | 剪切光标之前的一个词   |
| Alt + d  | 剪切光标之后的一个词   |


