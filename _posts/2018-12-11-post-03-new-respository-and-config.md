---
title: "[GitLearn-03] 新建Git仓库并配置"
search: false
author: "Shadow"
---

## 新建Git仓库
* 在现有目录中初始化仓库  
    进入该目录，然后  
    ` $ git init  `  
或者在该目录的上一级目录执行  
`git init <目录名>`  

&emsp;&emsp;该命令将创建一个名为 .git 的子目录，
这个子目录含有你初始化的Git仓库中所有的必须文件，这些文件是 Git 仓库的骨干。 
但是，在这个时候，我们仅仅是做了一个初始化的操作，你的项目里的文件还没有被跟踪。
如果想让Git跟踪该目录下的所有文件  
首先使用  
`git add .`  添加所有未跟踪的文件到暂存区  
然后使用  
`git commit -m '说明信息'`  将暂存区内容提交
* 创建空的Git仓库  
`$ git init <仓库名>`  
## 配置Git仓库
* 配置该计算机的所有用户的所有Git仓库信息(系统级)  
    `git config --system user.name 'your name'`  
    `git config --system user.email 'your email'`
* 配置该计算机的当前用户的所有Git仓库信息(全局级)  
    `git config --global user.name 'your name'`  
    `git config --global user.email 'your email'`
* 配置该计算机的当前用户的当前Git仓库信息(仓库级)  
    `git config --local user.name 'your name'`  
    `git config --local user.name 'your name'`  
    或者  
    `git config user.name 'your name'`  
    `git config user.name 'your name'`  
不加区域范围则表示配置当前Git仓库  
Git会优先使用仓库级配置，其次是全局级，最后是系统级

## 查看Git配置信息

* 查看某一级别的所有配置信息  
    `git config --<级别> --list`  
级别可选  
`--local` ，`--system`，`--global`  
  
    例如:`git config --global --list` 查看全局配置所有信息

* 查看某一级别的某个配置信息
    `git config --<级别> <具体项>`  
    例如:`git config --global user.name`  
    查看全局配置信息中的user.name
