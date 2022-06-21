---
title: Django_learning_4
date: 2022-06-21 16:31:0
tags: Django
categories: Django Learning
description: '部署nginx'
cover: https://miro.medium.com/max/1200/1*xJCYi2kk1HXBBJJVItq31w.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35185/preview/foggy-read_npreviews_414c.jpg
---

## 摘要
完成了nginx部署以及acapp对接

## 增加容器的映射端口：80与443
1. 登录容器，关闭所有运行中的任务
2. 登录运行容器的服务器，然后执行
    ```bash
    docker commit CONTAINER_NAME django_lesson:1.1  # 将容器保存成镜像，将CONTAINER_NAME替换成容器名称
    docker stop CONTAINER_NAME  # 关闭容器
    docker rm CONTAINER_NAME # 删除容器
    # 使用保存的镜像重新创建容器
    docker run -p 20000:22 -p 8000:8000 -p 80:80 -p 443:443 --name CONTAINER_NAME -itd django_lesson:1.1
    ```
3. 去云服务器控制台，在安全组配置中开放80和443端口（通常已经默认开发）
   
## 创建AcApp，获取域名、nginx配置文件及https证书
1. 
    将nginx.conf中的内容写入`/etc/nginx/nginx.conf`文件中。如果django项目路径与配置文件中不同，注意修改路径
    将acapp.key中的内容写入`/etc/nginx/cert/acapp.key`文件中
    将acapp.pem中的内容写入`/etc/nginx/cert/acapp.pem`文件中
    然后启动nginx服务：`sudo /etc/init.d/nginx start`

2. 修改django项目的配置
   - 打开`settings.py`文件：
        + 将分配的域名添加到`ALLOWED_HOSTS`列表中。注意只需要添加`https://` 后面的部分。
        + 令`DEBUG = False`
   - 归档`static`文件：
        + `python3 manage.py collectstatic`
3. 配置uwsgi
   + 在django项目中添加uwsgi的配置文件：`scripts/uwsgi.ini`，内容如下：
    ```
    [uwsgi]
    socket          = 127.0.0.1:8000
    chdir           = /home/yangang/acapp
    wsgi-file       = acapp/wsgi.py
    master          = true
    processes       = 2
    threads         = 5
    vacuum          = true
    ```
    启动uwsgi服务：`uwsgi --ini scripts/uwsgi.ini`
    
需要在项目根目录下执行`python3 manage.py collectstatic`命令
也就是在项目根目录下归档`static`文件夹
由于每次修改之后都需要归档，类似每次修改src中的js文件之后都需要执行脚本文件归档到dist的game.js中一样，可以在脚本文件中加入`echo yes | python3 manage.py collectstatic`
这样在执行脚本的时候就可以一起执行了（此时须在项目根目录下执行脚本命令`./scripts/compress_game_js.sh`）

由于添加域名，需要修改的文件如下

`settings.py`
```python
DEBUG = False # +

ALLOWED_HOSTS = ["47.100.238.236", "app152.acapp.acwing.com.cn"]  # +

```
`css/game.css`
```css
.ac-game-menu-field {
    width: 20vw;
    position: relative;
    top: 40%; /* + */
    left: 20%; /* + */
}
.ac-game-menu-field-item {
    color: white;
    height: 6vh; /* + */
    width: 18vw;
    font-size: 4vh;  /* + */
    font-style: italic;

```
`playground/zbase.js`开始时隐藏，计算画布大小在点击游戏界面才开始，于是交换了一下代码顺序
