---
title: Docker_learning
date: 2022-06-17 20:03:1
tags: Docker
categories: 毕设
description: 'Docker principles and common command'
cover: https://www.ruanyifeng.com/blogimg/asset/2018/bg2018020901.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35185/preview/foggy-read_npreviews_414c.jpg
---
## 1. 关于镜像(images)的命令
1. `docker image ls`：列出本地所有镜像
2. `docker pull ubuntu:20.04`：拉取一个ubuntu镜像
3. `docker image rm ubuntu:20.04`:删除一个ubuntu镜像
4. `docker save -o ubuntu_20_04.tar ubuntu:20.04`：将镜像ubuntu:20.04导出到本地压缩文件ubuntu_20_04.tar中
5. `docker load -i ubuntu_20_04.tar`：将镜像ubuntu:20.04从本地压缩文件ubuntu_20_04.tar中加载出来

## 2.关于容器(container)的命令
1. `docker create -it ubuntu:20.04`：利用镜像ubuntu:20.04创建一个容器，创建之后就exit了
2. `docker run -itd ubuntu:20.04`：创建并启动一个容器
3. `docker ps -a`：查看本地的所有容器，其中`docker ps`查看本地所有运行的容器
4. `docker rename CONTAINER name`：重命名容器
5. `docker start CONTAINER`：启动容器
   
    **注意 这里的CONTAINER可以是容器的names或者ID**

6. `docker stop CONTAINER`：停止容器
7. `docker restart CONTAINER`：重启容器
8. `docker attach CONTAINER`：进入容器
   + 先按`Ctrl-p`，再按`Ctrl-q`可以挂起容器,如果直接`exit`就会把容器关闭
9. `docker rm CONTAINER`：删除容器**只能删除已停止的容器**
10.  `docker container prune`：删除所有已停止的容器
11.  .  `docker export -o xxx.tar CONTAINER`：将容器CONTAINER导出到本地文件xxx.tar中
12. `docker import xxx.tar image_name:tag`：将本地文件xxx.tar导入成镜像，并将镜像命名为image_name:tag
    **注意容器导出后再导入得到的是镜像image而不是容器container**
13. `docker export/import`与`docker save/load`的区别:
    + `export/import`会丢弃历史记录和元数据信息，仅保存容器当时的快照状态
    + `save/load`会保存完整记录，体积更大
14. `docker top CONTAINER`：查看某个容器内的所有进程
15. `docker stats`：查看所有容器的统计信息，包括CPU、内存、存储、网络等信息
16. `docker cp xxx CONTAINER:xxx` 或 `docker cp CONTAINER:xxx xxx`：在本地和容器间复制文件
17. `docker update CONTAINER --memory 500MB`：修改容器限制

## 3. Docker登录容器需要注意的地方

+ 在进入一个容器之后——`docker attach container`是直接以`root`用户来操作（以ubuntu为例）
输入命令`passwd`即可为`root`用户创建密码
+ 如果想要创建一个普通用户（root用户权限太大），应该使用`adduser xxx`，这样就创建了普通用户，再`usermod -aG sudo xxx`为用户添加可sudo权限
+ 如果在`attach`进入容器之中`exit`/`Ctrl-d`的话会关闭这个容器，这个容器也就停止了，如果只是想暂时挂起而不是关闭的话使用的是先按`Ctrl-p`，再按`Ctrl-q`
+ 如果是`ssh`登录一个容器（因为容器也是一个ubuntu服务器了），那么可以直接`exit`/`Ctrl-d`退出，不影响容器本身状态，例如
  `ssh root@localhost -p 20000`
  `ssh xxxx@localhost -p 20000`
  其中localhost为服务器ip地址，见下

## 4. 涉及ssh及登录端口
我的服务器以20号端口ssh登录，那么如果想要本地或者服务器直接ssh到docker的容器的话（将容器看作服务器）需要修改端口，在我的Tencent服务器开启了20000端口，这样需要ssh到docker的容器的话就可以指明端口号为20000即可成功ssh了
``` bash
docker run -p 20000:22 --name my_server -itd ubuntu:18.04 
# 创建并运行ubuntu:18.04镜像并将20000端口映射到20号端口
```
<font color="red" size=5>
踩坑记录
</font>

如果是最原始的镜像可能没有配置ssh，需要手动配置
[参考链接](https://blog.csdn.net/weixin_43590796/article/details/109666934?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-109666934-blog-122604934.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.1&utm_relevant_index=3)
```bash
apt-get update 
apt-get install openssh-server
安装完成后使用 service ssh start 
vim /etc/ssh/sshd_config 
将PermitRootLogin的值改为yes 将PasswordAuthentication的值改为yes
最后service ssh restart
```

![我开启的20000端口](./docker_learning/docker端口截图.png)
即20000端口也可以实现20号端口ssh的功能
同时还可以在本地`./.ssh/config`中配置docker容器的登录信息
``` bash
Host Djserver
  Hostname 114.132.67.106
  User yangang
  port 20000
```
并使用`ssh-copy-id Hostname`即可直接免密登录
> 由于docker容器部署在服务器上，所以IP地址即Hostname与服务器的IP地址相同，只不过端口号不同，连接/设置时只需强调端口号，ip地址是一样的
