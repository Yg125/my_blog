---
title: Django_learning_2
date: 2022-06-19 19:35
tags: Django
categories: Django Learning
description: '完成了项目基本框架的搭建和菜单界面的显示'
cover: https://miro.medium.com/max/1200/1*xJCYi2kk1HXBBJJVItq31w.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35185/preview/foggy-read_npreviews_414c.jpg
---

## 摘要
本次完成了项目的基本框架搭建以及菜单界面的前端

## 1. 项目基本框架

项目框架设计
+ `menu`：菜单页面
+ `playground`：游戏界面
+ `settings`：设置界面
由于项目被分成了三个部分，在之后的实现过程中也最好在每个目录划分这三个部分

项目文件结构
+ `templates`目录：管理html文件
+ `urls`目录：管理路由，即链接与函数的对应关系
+ `views`目录：管理http函数
+ `models`目录：管理数据库数据
+ `static`目录：管理静态文件，比如：
  - `css`：对象的格式，比如位置、长宽、颜色、背景、字体大小等
  - `js`：对象的逻辑，比如对象的创建与销毁、事件函数、移动、变色等
  - `image`：图片
  - `audio`：声音
  …
+ `consumers`目录：管理websocket函数

### 1.1 具体过程
由于项目可能包含的链接很多，函数很多所以一般以文件夹来管理
将第一次创建的game/urls.py game/views.py game/models.py删除并用文件夹表示，并创建game/static文件夹
在python文件中如果想通过import的话需要在文件夹中创建__init__.py文件
于是在game/urls game/models game/views 中创建__init__.py文件

需要将创建的app也就是game添加到settings中
> 在game/apps.py中的内容如下
> ```python
> from django.apps import AppConfig
> class GameConfig(AppConfig):
>     default_auto_field = 'django.db.models.BigAutoField'
>     name = 'game'
> ```
> 在settings.py中需要修改如下内容
> ```python
> INSTALLED_APPS = [
>   'game.apps.GameConfig',
> ```
还需要将settings.py中修改static和media路径
```python
STATIC_ROOT = os.path.join(BASE_DIR,'static')
STATIC_URL = '/static/'

MEIDA_ROOT = os.path.join(BASE_DIR,'media')
MEDIA_URL = '/media/'
```
在static文件夹下创建`css`,`js`,`image`文件夹,在image下创建`menu`,`playground`,`settings`三个文件夹，分别存储三个部分的图片
**由于设置了static路径，所以可以直接114.132.67.106:8000/static进入static目录**，也可以114.132.67.106:8000/static/image访问图片
css文件夹中通常一个css文件即可
由于使用的js文件可能很多也很混乱，所以在js文件夹下创建dist和src文件夹,dist文件夹存放最终使用的js文件，src存放所有的js文件,可以理解为dist文件夹中存放着src中的合集。
在js/src中创建`menu`,`playground`,`settings`三个文件夹,保存三个部分的js,并在js/src目录下创建总的zbase.js文件,其中zbase.js会调用`menu`,`playground`,`settings`文件中的js内容
```js
class Ac_Game {
    constructor(id) {
    }
}
```
> 此处，在总目录下创建了scripts文件夹用来存放项目的所有的脚本文件
> 首先创建了第一个简易的脚本文件`compress_game_js.sh`,用来将src中的js文件整合并放入dist文件夹下。

```bash
#! /bin/bash

JS_PATH=/home/yangang/acapp/game/static/js/
JS_PATH_DIST=${JS_PATH}dist/
JS_PATH_SRC=${JS_PATH}src/

find $JS_PATH_SRC -type f -name '*.js' | sort | xargs cat > ${JS_PATH_DIST}game.js
# 将src所有的js打包存入dist/game.js中
```

在templates文件夹中创建`menu`,`playground`,`settings`和`multiends`四个文件夹，`multiends`用于多终端显示，并在其中创建`web.html`
```html
{% load static %}
<head>
    <!-- 使用jquery渲染 -->
    <link rel="stylesheet" href="https://cdn.acwing.com/static/jquery-ui-dist/jquery-ui.min.css">
    <script src="https://cdn.acwing.com/static/jquery/js/jquery-3.3.1.min.js"></script>
    <link rel="stylesheet" href="{% static 'css/game.css' %}">
    <!-- 使用自己写的css和js文件 -->
</head>

<body style="margin: 0">
    <div id="ac_game_12345678"></div>
    <script type="module">
        import {AcGame} from "{% static 'js/dist/game.js' %}";
        $(document).ready(function(){
            let ac_game = new AcGame("ac_game_12345678");
        });
    </script>
</body>

```
在`views`文件夹中创建`menu`,`playground`,`settings`文件夹，并分别在三个文件夹中创建`__init__.py`文件，创建一个`index.py`文件,渲染得到html的信息

```python
from django.shortcuts import render 

def index(request):
    return render(request,"multiends/web.html") # render回templates下面，所以这里是从multiends开始
```

在`urls`文件夹中创建`menu`,`playground`,`settings`文件夹，并分别在三个文件夹中创建`__init__.py`和`index.py`文件，在`urls`目录下创建一个`index.py`文件,包含url路径
分别在三个字文件夹中的index.py文件中写入
```python
from django.urls import path

urlpatterns = [
]
```
然后在`urls`目录下的总`index.py`下include这些路径，并包含刚才views/index.py写的内容
```python
from django.urls import path,include
from game.views.index import index
urlpatterns = [
    path("",index,name="index"),
    path("menu/",include("game.urls.menu.index")),
    path("playground/",include("game.urls.playground.index")),
    path("settings/",include("game.urls.settings.index")),
]
```
最后再在总的urls.py中修改
```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('', include('game.urls.index')), # + 默认打开为无选项的
    path('admin/', admin.site.urls),
]
```
这样，基本的框架就搭建好了

> 完整的过程实质上是在http://114.132.67.106:8000/中，由于无选项，在`urls.py`中的第一个`path`查找
> 其中由于include了`game.urls.index`，进入`game/urls/index.py`，其中包含了无选项以及三个部分的url
> 无选项，进入`game/views/index.py`，返回渲染的`game/templates/multiends/web.html`内容。
> 其中的js内容会创建一个AcGame的class，之后所有的开发都是基于这个**AcGame**

## 2.搭建菜单界面
<font color = red size = '3'>注意，如果打开界面出现js错误或图片加载不出来等情况，请先执行浏览器缓存清理!!! </font>
### 2.1 新建界面
创建`static/js/src/menu/zbase.js`和`static/js/src/playground/zbase.js`用来保存界面逻辑
menu/zbase.js代码如下

```js
// js相关
//  当对象是时html的一个组件的时候可用一个$符号放在变量前面
//  当div里面是id时,需要加一个 ['#' + ]
```

```js
class AcGameMenu {
    constructor(root) {
        this.root = root;  //只有总的js文件才会传入id，其余都是传的root，其实就是总的js中的id
        this.$menu = $(`  
        <div class="ac-game-menu">
            <div class="ac-game-menu-field">
                <div class="ac-game-menu-field-item ac-game-menu-field-item-single-mode">
                    单人模式
                </div>
                <br>
                <div class="ac-game-menu-field-item ac-game-menu-field-item-multi-mode">
                    多人模式
                </div>
                <br>
                <div class="ac-game-menu-field-item ac-game-menu-field-item-settings">
                    设置
                </div>  
            </div>
        </div>
        `);              //menu的布局
        this.root.$ac_game.append(this.$menu);  //将menu加入到ac_game中，其中ac_game是总的js传进来的，也就是总的对象
        //用于找按钮
        this.$single_mode = this.$menu.find('.ac-game-menu-field-item-single-mode');
        this.$multi_mode = this.$menu.find('.ac-game-menu-field-item-multi-mode');
        this.$settings = this.$menu.find('.ac-game-menu-field-item-settings');

        this.start();     //一般在最开始打开界面的时候会有操作，就在constructor析构函数中调用start
    }
    start() {
        this.add_listening_events();
    }

    add_listening_events() {    //点击不同按钮的操作
        let outer = this;
        this.$single_mode.click(function() {
            outer.hide();
            outer.root.playground.show();
        });
        this.$multi_mode.click(function() {
            console.log("click multi mode");
        });
        this.$settings.click(function() {
            console.log("click settings");
        });
    }

    show() { //显示menu界面
        this.$menu.show();
    }
    hide() { //关闭menu界面
        this.$menu.hide();
    }
}
```
playground/zbase.js代码如下
```js
class AcGamePlayground {
    constructor(root) {
        this.root = root;
        this.$playground = $(`
            <div>
                游戏界面
            </div>
        `)
        this.hide();
        this.root.$ac_game.append(this.$playground);

        this.start();
    }
    start() {
    }
    show() { //打开playground界面
        this.$playground.show();
    }

    hide() { //关闭playground界面
        this.$playground.hide();
    }
}
```
修改static/js/src/zbase.js文件
```js
export class AcGame { // 由于之前web.html文件中只部分引入AcGame，所以需要export暴露出来
    constructor(id) {
        this.id = id;
        this.$ac_game = $('#' + id);
        this.menu = new AcGameMenu(this);             //创建刚才写的js文件中的对象，并会在其中加入到总的AcGame中
        this.playground = new AcGamePlayground(this);
    }
}
```

这样，在执行./compress_game_js.sh之后就会在之前提到过的static/js/dist/game.js中包含所有的js文件内容
也就意味着前端一旦返回web.html内容就会在game.js中找到所有的js信息
### 2.2 初识css文件
仅仅修改js知识在页面中显示了这些内容，对使用者不友好，需要修改/static/css/game.css文件如下
```css
这里用到的.xxx必须是之前js中定义在<div>
.ac-game-menu {
    width: 100%;
    height: 100%;
    background-image: url("/static/image/menu/background.gif");
    background-size: 100% 100%;
    user-select: none;
}

.ac-game-menu-field {
    width: 20vw;
    position: relative;
    top: 40vh;
    left: 19vw;
}

.ac-game-menu-field-item {
    color: white;
    height: 7vh;
    width: 18vw;
    font-size: 5vh;
    font-style: italic;
    padding: 2vh;
    text-align: center;
    background-color: rgba(39, 21, 28, 0.6);
    border-radius: 20px;
    letter-spacing: 0.2vw;
    cursor: pointer;
}

.ac-game-menu-field-item:hover { /*指针悬停设置 */
    transform: scale(1.2);
    transition: 100ms;
}
```