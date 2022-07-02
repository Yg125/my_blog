---
title: xv6实验汇总
date: 2022-06-25 11:21:11
tags: [汇编, 操作系统, C语言]
categories: OS
description: 'xv6实验'
cover: https://miro.medium.com/max/700/1*9LESgcIZN-iF3yIw5kxsaQ.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35464/preview/madebyvadim_npreviews_a576.jpg 
---

xv6实验中不像平时的C语言程序，走到`}`就结束了，xv6中进程必须显式地调用`exit`来结束，否则进程就会一直在那里
使用`make qemu`来编译程序，每一次实验都会有自动评测脚本，可以使用`make grade`来测试所有任务得分，也可以使用`./grade-lab-util xxx`来评测某个任务是否完成

---
# 实验目录

[util](https://yg125.github.io/2022/06/30/xv6_lab1/)

