---
title: xv6实验1 
date: 2022-06-30 20:06:24
tags: 
- 汇编
- 操作系统
- 计算机组成原理
- C语言
categories: OS
description: xv6实验1 Util心路历程
cover: https://miro.medium.com/max/700/1*9LESgcIZN-iF3yIw5kxsaQ.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35464/preview/madebyvadim_npreviews_a576.jpg
---
# 实验一

## Sleep
这个任务简单，只需要参考其他指令的实现方式比如`rm.c`，就可以写出来
系统调用函数在`user/user.h`中，完成sleep.c之后需要将其加入大`Makefile`中 `$U/_sleep\`
```C
#include "kernel/types.h"
#include "user/user.h"
int main(int argc, char const *argv[])
{
  if (argc != 2) { //参数错误
    printf("Sleep needs one argument!\n");
    exit(1);
  }
    int ticks=atoi(argv[1]);//将字符串参数转为整数
    sleep(ticks);
    printf("nothing happens for a little while\n");
    exit(0);//确保进程退出
}
```
## pingpong

这个任务需要实现父子进程的信息传递
1. 通过`pipe(fd)`来创建一个管道，通常用到的管道都是单向的，于是如果要实现双向传递，则需要两个管道
2. 使用`fork()`得到的子进程会继承管道，让父子进程分别关闭一端，只使用一端进行`read`和`write`即可实现单向传递信息
3. 父进程要等待子进程结束需要调用`wait`
```C
#include "kernel/types.h"
#include "user/user.h"

#define R 0
#define W 1
char buffer[10];
int main()
{
    int pid;
    int fd1[2],fd2[2];
    if(pipe(fd1) == -1 || pipe(fd2) == -1){
        printf("Pipe failed\n");
        exit(-1);
    }
    pid = fork();
    if(pid < 0)
    {
        printf("fork fail\n");
        exit(-1);
    }
    else if(pid == 0)  //child process
    {
        close(fd1[W]);
        close(fd2[R]);
        read(fd1[R],buffer,sizeof(buffer));
        int pid = getpid();
        printf("%d: received %s\n",pid,buffer);
        close(fd1[R]);
        write(fd2[W],"pong",sizeof("pong"));
        close(fd2[W]);
        exit(0);
    }
    else{   //parent process
        close(fd1[R]);
        close(fd2[W]);
        write(fd1[W],"ping",sizeof("ping"));
        close(fd1[W]);
        wait(0);
        read(fd2[R],buffer,sizeof(buffer));
        int pid = getpid();
        printf("%d: received %s\n",pid,buffer);
        close(fd2[R]);
        exit(0);
    }
    return 0;
}
```

## primes

这个任务比较难
1. `main`函数中，创建管道和相应文件描述符，并`fork()`创建子进程，父进程中关闭读端，将2到35依次以四个字节的`int`写入管道中，关闭写端，等待子进程结束再退出。
2. 子进程中以继承得来的文件描述符作为参数调用函数 `Child_Process(int fd[2])`，退出。
3. `Child_Process(int fd[])`用来完成质数筛选，其过程大致为:
    + 关闭 `fd[1]`，从 `fd[0]`中**读出第一个数字作为质数**，并打印它的信息，如果`read()`返回`0`那么就返回
    + 在函数中再创建管道用于向下一个子进程中传入筛选后的数据，得到相应文件描述符`fd_child[2]`,创建子进程，子进程递归调用 `Child_Process(fd_child)`，退出
    + 在函数父进程中，关闭`fd_child`的读端，将管道`fd`中的数据一个一个读出并判断能不能整除刚得到的质数，直到没有数据为止，把不能整除的数据写入下一个管道`fd_child`，否则直接丢弃。关闭`fd`的读端，关闭`fd_child`的写端。等待子进程结束，返回
```C
#include "kernel/types.h"
#include "user/user.h"

#define R 0
#define W 1

void Pipe(int fd[2])  //include the false detection
{
    if (pipe(fd) == -1)
    {
        printf("pipe fail\n");
        exit(-1);
    }
}
int Fork()          //include the false detection
{
    int pid = fork();
    if (pid < 0)
    {
        printf("fork fail\n");
        exit(-1);
    }
    return pid;
}
void Child_proc(int fd[2])
{
    close(fd[W]);
    int fd_child[2];
    pipe(fd_child);
    int prime;
    int testnum;
    if (read(fd[R], &prime, sizeof(int)) == 0)    //if there is number in the pipe
        return;
    printf("prime %d\n", prime);

    int pid = Fork();
    if (pid == 0)
    {
        Child_proc(fd_child);
        exit(0);
    }
    else
    {
        while (read(fd[R], &testnum, sizeof(int)) != 0)
        {
            if (testnum % prime != 0)
                write(fd_child[W], &testnum, sizeof(int)); 
        }
        close(fd[R]);
        close(fd_child[W]);
        wait(0);
        return;
    }
}
int main()
{
    int fd[2];
    Pipe(fd);
    int pid = Fork();
    if (pid == 0)
    {
        Child_proc(fd);
        exit(0);
    }
    else
    {

        close(fd[R]);
        for (int i = 2; i < 36; i++)
            write(fd[W], &i, sizeof(int)); //write every elements into pipe
        close(fd[W]); // after finishing write ,close it
        wait(0);      // wait all the child process exit
        exit(0);
    }
    return 0;
}
```

## find

这个任务没什么好说的，仿照`ls.c`做基本就可以了
有几个点有点不同，一个是函数`fmtname`获取文件名的最后一步不能扩充，只需得到实际文件名再在字符串末尾添加`0`即可
还有就是在目录情况下，需要择去`.`和`..`目录，然后在剩余的文件/夹中递归`find`即可
```C
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char *fmtname(char *path)
{
    static char buf[DIRSIZ + 1];
    char *p;

    // Find first character after last slash.
    for (p = path + strlen(path); p >= path && *p != '/'; p--)
        ;
    p++;

    // Return blank-padded name.
    if (strlen(p) >= DIRSIZ)
        return p;
    memmove(buf, p, strlen(p));
    buf[strlen(p)] = 0;
    //memset(buf + strlen(p), ' ', DIRSIZ - strlen(p));
    return buf;
}

void find(char *path, char *filename)
{
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path, 0)) < 0)
    {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }
    if (fstat(fd, &st) < 0)
    {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }

    switch (st.type)
    {
    case T_FILE:
        if(strcmp(fmtname(path),filename) == 0)
            printf("%s\n",path);
        break;

    case T_DIR:
        if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf)
        {
            printf("ls: path too long\n");
            break;
        }
        strcpy(buf, path);
        p = buf + strlen(buf);
        *p++ = '/'; //p就是path/，之后该目录下的文件可以表示为在p之后加文件名
        while (read(fd, &de, sizeof(de)) == sizeof(de))
        {
            if (de.inum == 0)
                continue;
            else if (strcmp(de.name,".") == 0 || strcmp(de.name,"..") == 0)
                continue;
            memmove(p, de.name, DIRSIZ);  //抛开. 和 .. 得到的目录中的文件路径
            p[DIRSIZ] = 0;
            
            find(buf,filename); //buf是完整路径
        } 
        break;
    }
    close(fd);
    return;
}
 
int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        printf("find needs at least three arguments\n");
        exit(-1);
    }
    find(argv[1], argv[2]);
    exit(0);
}
```

## xargs

这个任务其实不难，但是不好理解
首先`xargs`在之前学过的Linux命令中将将标准输入中的数据用空格或回车分割成命令行参数
有几点需要注意
1. 要读取单个输入行，一次读取一个字符，直到出现换行符`'\n'`
2. 需要循环读入，因为可能有多行，每一行相当于一次命令可以执行，直到没有输入
3. 通过`fork`和`exec`来执行相应命令，传入`exec`的命令行参数的最后一项必须是0
4. 原始的`argv`第一个参数就是`xargs`，之后的作为新的命令行参数再和标准输入的一起作为整个命令行参数传递至`exec`中执行

```C
#include "kernel/param.h" 
#include "kernel/types.h"
#include "user/user.h"

int main(int argc,char *argv[])
{
    char buf[1000];
    char *argv_exec[MAXARG];
    int i;
    if((argc < 2) || (argc + 1 > MAXARG))
    {
        printf("arguments fail\n");
        exit(-1);
    }
    for (int j = 1; j < argc ;j++)
        argv_exec[j-1] = argv[j];
    while(1)
    {
        i = 0;
        while(1)
        {
            if(read(0, buf + i,sizeof(char)) < 0)
                break;
            if(buf[i] == '\n')
                break;
            i++;
        }
        if(i == 0)
            break;
        buf[i] = 0;
        argv_exec[argc - 1] = buf;
        argv_exec[argc] = 0;
        if(fork() == 0)
        {
            exec(argv_exec[0],argv_exec);
            exit(0);
        }
        else{
            wait(0);
        }
    }
    exit(0);
}

```
