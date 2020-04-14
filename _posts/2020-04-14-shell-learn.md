---
title: "[Shell] Shell学习记录"
search: false
author: "Shadow"
---

[TOC]

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
  awk -F"\t" '{if ($2==3) print $0}' file_name
  ```

- eg-3：如果 `file1`中的第一列值 `hello` ,则打印`该行内容 + HELLO` ,否则打印`该行内容 + NO`

  ```
  awk -F " " '{if ($1=="hello") print $0"\t""HELLO";else print $0"\t""NO"}' file1
  ```

  

# 常用操作

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

#################
要执行的程序
#################

finish_time=`date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"`
duration=$(($(($(date +%s -d "$finish_time")-$(date +%s -d "$start_time")))))
echo "time cost: ${duration}s"
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

# sed命令

### sed替换文件中的字符串

- 将文件中的 `"null" `替换为 `null  `

```
sed -i 's/\"null\"/null/g' test.txt
```





