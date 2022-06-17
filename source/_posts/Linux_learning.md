---
title: Linux_learning
date: 2022-06-16 16:04:03
tags: Linux
description: 'linux learning these days'
cover: https://pic1.zhimg.com/v2-dc98064e8bc40d83d91362d4ce1dd9d3_1440w.jpg?source=172ae18b
top_img: https://picsum.photos/id/477/4919/3258
---

# Linux学习重点

## 1. Linux常用指令

### 1.1 文件转移指令

查看文件夹详细信息  `ls -al`  和 `ls -la` 以及 `ll`

- `cd / `   返回根目录
- `cd ~  `  直接返回家目录
- `cd  `    直接返回家目录
- `cd -`返回上一次使用的目录
- 其中`pwd`返回当前所在路径
- 家目录在LINUX中就是`/home`下的文件通常为`/home/guest`



### 1.2 管道 ｜

__概念__
管道类似于文件重定向，可以将前一个命令的`stdout`重定向到下一个命令的`stdin`。
**注意**只针对命令，不针对文件 
如果命令需要的是参数而不是stdin的输入，那么就需要在该命令之前用到xargs将stdin转化为参数
比如`find . -name '*.cpp' | wc -l > ans.txt`


### 1.3 环境变量

Linux系统中会用很多环境变量来记录**配置信息**。
环境变量类似于全局变量，可以被各个进程访问到。我们可以通过修改环境变量来方便地修改系统配置。

```bash
env  # 显示当前用户的变量
set  # 显示当前shell的变量，包括当前用户的变量;
export  # 显示当前导出成用户变量的shell变量
```

输出某个环境变量的值：`echo $PATH`
为了将对环境变量的修改应用到未来所有环境下，可以将修改命令放到`~/.bashrc`文件中。
修改完`~/.bashrc`文件后，记得执行`source ~/.bashrc`，来将修改应用到当前的`bash`环境下。

#### 常见环境变量
1. `home`：用户的家目录
2. `PATH`：可执行文件（命令）的存储路径。路径与路径之间用:分隔。当某个可执行文件同时出现在多个路径中时，会选择从左到右数第一个路径中的执行。所有存储路径的环境变量，均采用从左到右的优先顺序。

如果想要将自己的命令在任何路径下直接执行时，就需要将该命令放到`PATH`中，具体方法为：
1. 在~/.bashrc文件末尾添加一行：`export PATH=/home/acs/homework/lesson_7/homework_0:$PATH`
2. 运行`source ~/.bashrc`
即可将该可执行文件放到PATH中也就可以在任意路径下执行该命令了


### 1.4 系统状况
1. `top`：查看所有进程的信息（Linux的任务管理器)
    打开后，输入`M`：按使用内存排序
    打开后，输入`P`：按使用CPU排序
    打开后，输入`q`：退出
2. `df -h`：查看硬盘使用情况
3. `free -h`：查看内存使用情况
4. `du -sh`：查看当前目录占用的硬盘空间
5. `ps aux`：查看所有进程 `ps aux`通常和`grep`一起来使用查找进程
6. `kill -9 pid`：杀死编号为pid的进程 传递某个具体的信号：`kill -s SIGTERM pid`
7. `netstat -nt`：查看所有网络连接
8. `w`：列出当前登陆的用户
9. `ping www.baidu.com`：检查是否连网

### 1.5文件权限

`chmod`：修改文件权限

* `chmod +x xxx`：给xxx添加可执行权限
* `chmod -x xxx`：去掉xxx的可执行权限
* `chmod 777 xxx`：将xxx的权限改成777
* `chmod 777 xxx -R`：递归修改整个文件夹的权限

### 1.6 文件检索

1. `find /path/to/directory/ -name '*.py'`：搜索某个文件路径下的所有*.py文件
2. `grep xxx`：从stdin中读入若干行数据，如果某行中包含xxx，则输出该行；否则忽略该行。
3. `wc`：统计行数、单词数、字节数.既可以从`stdin`中直接读入内容；也可以在命令行参数中传入文件名列表；
   - `wc -l`：统计行数
   - `wc -w`：统计单词数
   - `wc -c`：统计字节数
4. `tree`：展示当前目录的文件结构
   - `tree /path/to/directory/`：展示某个目录的文件结构
   - `tree -a`：展示隐藏文件
5. `ag xxx`：搜索当前目录下的所有文件，检索xxx字符串
6. `cut`：分割一行内容 __从stdin中读入多行数据__
    - `echo $PATH | cut -d ':' -f 3,5`：输出PATH用:分割后第3、5列数据
    - `echo $PATH | cut -d ':' -f 3-5`：输出PATH用:分割后第3-5列数据
    - `echo $PATH | cut -c 3,5`：输出PATH的第3、5个字符
    - `echo $PATH | cut -c 3-5`：输出PATH的第3-5个字符
7. `sort`：将每行内容按字典序排序
    可以从stdin中读取多行数据
    可以从命令行参数中读取文件名列表
8. `xargs`：将stdin中的数据用空格或回车分割成命令行参数
`find . -name '*.py' | xargs cat | wc -l`：统计当前目录下所有python文件的总行数

### 1.7 查看文件内容

1. `head -x xxx`：展示xxx的前x行内容 同时支持从stdin读入内容
2. `tail -x xxx`：展示xxx末尾x行内容 同时支持从stdin读入内容

### 1.8 压缩解压缩文件
+ `tar`：压缩文件
+ `tar -zcvf xxx.tar.gz /path/to/file/*`：压缩
+ `tar -zxvf xxx.tar.gz`：解压缩


#### 注意

​	`cp xxx xxx`
注意后面是复制到的地方，如果两个地方都是普通文件，则直接将前一个复制过去，
如果前面是普通文件后面的是文件夹dir，则将文件复制到文件夹里面去
如果前面后面都是文件夹，则需要加-r强制将文件夹复制到另一个文件夹里面去
如果前面是文件夹后面是普通文件，那么错误
此前操作如果使用没有创建的文件或者文件夹会直接创建一个新的
 **注意，xxx都可以表示文件地址，如果没有指明路径（绝对还是相对），都是当前的相对路径** 
 **凡是有文件路径的地方都可以写绝对路径，没有写明的话都默认是相对路径**		

​	`mv xxx xxx`
将前面一个地方的文件转移到后一个地方去，
如果后一个文件没有创建，那么就是重命名，将前一个地方重命名为后一个地方的名字
如果两个都是普通文件,或者都是文件夹，那么也是重命名
如果前一个是普通文件，后一个是文件夹，那么就将前一个普通文件移到文件夹的目录下
如果前一个是文件夹，后一个是普通文件，那么错误

在涉及到文件匹配问题中经常用到管道，`xargs`，`find`，`cut`，`grep`，以及重定向，需要灵活使用
比如

``` bash
find . -name '*.cpp' | wc -l > ans.txt  统计当前目录下cpp文件数目
find . -name '*.cpp' | xargs cat | wc -l > ans1.txt 统计当前目录下的所有cpp文件的总行数
find . -name '*.py' | xargs cat | grep thrift | wc -l > ans2.txt 统计所有py文件中包含thrift字符串的总行数
find . -name '*.py' | xargs rm  删除当前目录下的所有py文件
cat scores.txt | cut -d ' ' -f 1 > names.txt    以空格隔开每一列将第一、二、三列保存在文件中
cat scores.txt | cut -d ' ' -f 2 > mathematics.txt
cat scores.txt | cut -d ' ' -f 3 > algorithm.txt
cat scores.txt | cut -d ' ' -f 1 | sort > names.txt 将第一列排序保存在文件中

灵活运用xargs将stdin转化为参数，grep进行匹配，wc -l计数行，find查找
```

## 2. tmux使用讲解

**注意**
在tmux中要选择并进行复制使用

1. 按下`Ctrl + a`后松开手指，然后按`[`
2. 用鼠标选中文本，被选中的文本会被自动复制到tmux的剪贴板
3. 按下`Ctrl + a`后松开手指，然后按`]`，会将剪贴板中的内容粘贴到光标处
   

只有这样才能从tmux中复制并保存在系统剪贴板中；
其余情况，单独终端中复制粘贴命令是shift+Ctrl+c、shift+Ctrl+v

在我的tmux.conf中已经配置可以使用鼠标以及修改默认`prefix`为`Ctrl+a`
tmux中`Ctrl+a+%`横向创建新的pane，`Ctrl+a+"`纵向创建新的pane
`Ctrl+a+z`全屏pane，常用于复制文本时
`Ctrl+d`关闭当前pane
`Ctrl+a+d`挂起该tmux
`tmxu a`重连上一个挂起的tmux

关于为什么要使用和建议使用tmux

> 因为tmux可以在终端莫名关闭或者连接中断之后挂起而不会导致编写的过程丢失，使用tmux a可以直接重连上一个挂起的tmux。

如果需要修改tmux的配置可以修改`~/.tmux.conf` 配置文件

## 3. vim常用命令

vim有以下三种模式
1. 一般命令模式
默认模式。命令输入方式：
1. 编辑模式
在一般命令模式里按下`i`，会进入编辑模式。按下ESC会退出编辑模式，返回到一般命令模式。
3. 命令行模式
在一般命令模式里按下`:` `/` `?`三个字母中的任意一个，会进入命令行模式。命令行在最下面。可以查找、替换、保存、退出、配置编辑器等。


- `:set nonu` 取消行号，用于复制文本
- `:set paste` 设置成粘贴模式，取消代码自动缩进
  + 在复制外部文字到vim里面的时候需要先开启`:set paste`然后复制完之后再`:set no paste`
- `gg` 到第一行 
- `G` 到最后一行 __可以用G表示多少行，比如12G为12行__
- `v` 用于选中文本，__可以为xxvxx意为xx行到xx行 比如12Gv24G__
- `0` 将光标移到行首，`$`将光标移至行末 
- `/word` word查找 __继续按n可以移动光标__
- `n<Enter>`：n为数字，光标向下移动n行
- `n<Space>`：n表示数字，按下数字后再按空格，光标会向右移动这一行的n个字符 __常用来寻找第几行第几个字符__
- `d` 删除选中的文本（剪切）dd删除当前行 **也可以使用指定行删除`12Gd24G`删除12行到24行的内容**
- `y` 复制选中的文本。     yy复制当前行 **同理也可以复制指定行的内容12Gy24G**
- `p` 将复制的文本粘贴到下一行/下一个位置 
- `u` 撤销
- `:noh` 关闭高亮
- `gg=G` 将全文代码格式化
- `:w` 保存 `w！`强制保存
- `:q` 退出 `q!` 强制退出
- `wq` 保存并退出
- `:n1,n2s/word1/word2/g：` n1与n2为数字，在第n1行与n2行之间寻找word1这个字符串，并将该字符串替换为word2
- `:1,$s/word1/word2/g：` 将全文的word1替换为word2
- `:1,$s/word1/word2/gc：` 将全文的word1替换为word2，且在替换前要求用户确认。


如果需要修改vim的配置可以修改`~/.vimrc vim`配置文件

## 3. ssh
### 3.1 ssh远程登录

一般情况可以直接使用`ssh user@hostname` 
然后输入密码即可连接远程
其中hostname一般为域名或者IP地址

如果想连接自己的服务器或经常性远程登录可以采用如下方式
1. 在配置文件`~/.ssh/config`中配置服务器相关信息（没有这个文件直接创建）相当于在本地给服务器取了个别名，并保存了user和hostname
   
``` bash
Host myserver
    Hostname xxx 
    User xxx
```
配置之后可以直接 `ssh myserver1` ，不过还是需要输入密码，要想免密登录还需要配置密钥

2. 创建密钥
   

`ssh-keygen` 创建之后会在`~/.ssh`下多两个文件
`id_rsa` 私钥
`id_rsa.pub` 公钥

想要免密登陆ssh可以将公钥`id_rsa.pub`中的内容复制到服务器的`~/.ssh/authorized_keys`中
或者直接使用`ssh-copy-id Hostname`也可以

如果需要退出服务器，按下`Control+D`或者终端输入`logout`

### 3.2 ssh命令
在本地操作服务器使用命令 `ssh user@hostname command` 或者如果是在config里面配置了相关信息的服务器可以直接 `ssh Host command`

**注意**
单引号中的`$i`可以求值
`ssh myserver 'for ((i = 0; i < 10; i ++ )) do echo $i; done' `
双引号中的`$i`不可以求值
` ssh myserver "for ((i = 0; i < 10; i ++ )) do echo $i; done" `

这是因为ssh操作时会在本地先解析命令，双引号直接解析掉了，那么传过去的i就没有了，但是单引号不会被解析，直接就穿过去了，所以能正确输出0到9

### 3.3 scp 传文件
`scp source:path destination:path` 默认是在`home`地址
可以传文件夹需要加`-r`，且加在地址之前
比如`scp -r ~/tmp myserver:/home/acs/`
注意服务器是不能`ssh`本地的，所以不要在服务器上使用`scp`往本地传文件（没配置相关信息连不上）
但是可以在本地使用`scp`实现互传，比如可以用`scp`从本地往服务器传也可以实现从服务器往本地传
注意在这里shell语言中的`$()`和` `` `也是可以用的，用`pwd`比较方便求地址

当传入的名字可能含有空格的时候需要注意
` ssh server mkdir homework/lesson_4/homework_4/\"$1\"  `正确
` ssh server mkdir homework/lesson_4/homework_4/"'$1'"  `正确
` ssh server mkdir homework/lesson_4/homework_4/'"$1"'  `错误
原理都是在本地解析一遍再传到服务器
第一种本地解析的时候不解析，直接传过去
第二种本地解析了还有单引号里的$1，可以得到正确的值
第三种直接把双引号$1传过去，没有意义
