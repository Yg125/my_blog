---
title: Django_learning_5
date: 2022-06-21 18:36:1
tags: Django
categories: Django Learning
description: '用户名密码登录'
cover: https://miro.medium.com/max/1200/1*xJCYi2kk1HXBBJJVItq31w.png
top_img: https://d29fhpw069ctt2.cloudfront.net/photo/35185/preview/foggy-read_npreviews_414c.jpg
---

## 摘要
创建了用户注册登录系统

## 后台表创建

Django虽然自带了用户系统，但我们希望能够让用户信息再加一个头像，这时候需要自行利用Django自带的库进行扩充。以下是信息扩充的步骤
`acapp/game/models`是用于储存信息的，所以我们可以在这里面储存用户信息。
我们首先要定义一个数据表，首先进入`acapp/game/models/`创建一个`player`文件夹并进入，再创建py文件`__init__.py`和`player.py`，然后开始写`player.py`，用来存储`Player`这个数据表的信息。就相当于定义了一个SQL表，只不过是python中的类
`player.py`示例

```python
# 用 Django 自带的基类扩充
from django.db import models # 从django的数据库中引入models
from django.contrib.auth.models import User # 从django中引入这个基本的User类

class Player(models.Model): # 要从models.Model这个类来继承
    user = models.OneToOneField(User, on_delete = models.CASCADE) # Player从User扩充，这里是定义一个扩充关系，代表Player都有唯一对应的User，User就是代表User数据表，on_delete代表删除User的时候，把他对应的Player也删掉
    photo = models.URLField(max_length = 256, blank = True) # 代表头像的URL，max_length是链接最大长度，blank是是否可空

    def __str__(self): # 返回一个对象的描述信息
        return str(self.user)
```
如果我们希望要把这个数据表出现在我们后台的页面，我们就需要**将这个数据表注册到管理员页面**
这时候进入到`acapp/game/`，在`admin.py`上加上两行

```python
from django.contrib import admin

# Register your models here.
from game.models.player.player import Player # 引入自己定义的数据表 +
admin.site.register(Player) # 注册这个数据表 +
```

我们在定义完一个数据表之后，**需要将其更新到数据库**，这时候我们需要执行两句话：
`python3 manage.py makemigrations`
`python3 manage.py migrate`
<font color = red size = '3'>完成之后进入管理员页面，会发现更新了一个新的数据表，然后可以自己创建这个新类型的对象了</font>

## 登录

流程大概是这样：

用户(Client)每次一刷新，就会先给后台服务器(Server)发送一个请求(Request):`getinfo`获取用户信息，然后后台就会返回一个响应(Response)，表示用户信息(用户名，头像)或者登录失败的响应

> ----------  Request   ----------
> |        |----------->|        |
> | Client |  Response  | Server |
> |        |<-----------|        |
> ----------            ----------
<font color = red>每一次写操作的时候都要考虑views，url，js</font>


```python
from django.http import JsonResponse
from game.models.player.player import Player

def getinfo(request): # 每个处理请求的函数都要有这个参数'request' 
    user = request.user
    if not user.is_authenticated: # 没有登录
        return JsonResponse({
            'result': "未登录",
        })
    else:
        player = Player.objects.all()[0] # Player.objects是一个数组，代表Player数据表的所有元素的数组，这里为了测试就直接返回第一个人的信息，后面再实现找出这个用户
        return JsonResponse({ # 返回Json
        'result': "success",
        'username': player.user.username, # 用户名
        'photo': player.photo, # 头像URL
    })
```

接下来写路由，以调用这些逻辑。进入`game/urls/settings/index.py`  修改如下

```py
from django.urls import path
from game.views.settings.getinfo import getinfo # 引入上面写的getinfo函数

urlpatterns = [
    path("getinfo/", getinfo, name = "settings_getinfo"), # 增加这个路由
]
```

现在我们访问网址`/settings/getinfo`，成功的话会显示有用户名和头像URL的信息

下面实现前端部分。我们进入`game/static/js/src/settings/zbase.js`，创建一个`class Settings`
前端的代码非常多，主要是包括

```js
class Settings {
    constructor(root) {
        this.root = root;
        this.username = "";  // 储存用户名
        this.photo = "";     // 储存头像

        this.$settings = $(`
<div class="ac-game-settings">
    <div class="ac-game-settings-login">            //登录界面
        <div class="ac-game-settings-title">        //标题
            登录
        </div>
        <div class="ac-game-settings-username">     //用户名输入框
            <div class="ac-game-settings-item">
                <input type="text" placeholder="用户名">    //输入处
            </div>
        </div>
        <div class="ac-game-settings-password">     //密码输入框
            <div class="ac-game-settings-item">
                <input type="password" placeholder="密码">   //密码输入处
            </div>
        </div>
        <div class="ac-game-settings-submit">      //按钮
            <div class="ac-game-settings-item">
                <button>登录</button>
            </div>
        </div>
        <div class="ac-game-settings-error-message">    //错误信息
        </div>
        <div class="ac-game-settings-option">
            注册
        </div>
        <br>    //这里一定要加上，不然一键登录图标不会居中，前面两行是inline的样式，可能会有bug
        <div class="ac-game-settings-acwing">
            <img width="30" src="https://app165.acapp.acwing.com.cn/static/image/settings/acwing_logo.png">
            <br>
            <div>
                AcWing一键登录
            </div>
        </div>
    </div>
    
    <div class="ac-game-settings-register">
        <div class="ac-game-settings-title">
            注册
        </div>
        <div class="ac-game-settings-username">
            <div class="ac-game-settings-item">
                <input type="text" placeholder="用户名">
            </div>
        </div>
        <div class="ac-game-settings-password ac-game-settings-password-first">
            <div class="ac-game-settings-item">
                <input type="password" placeholder="密码">
            </div>
        </div>
        <div class="ac-game-settings-password ac-game-settings-password-second">
            <div class="ac-game-settings-item">
                <input type="password" placeholder="确认密码">
            </div>
        </div>
        <div class="ac-game-settings-submit">
            <div class="ac-game-settings-item">
                <button>注册</button>
            </div>
        </div>
        <div class="ac-game-settings-error-message">
        </div>
        <div class="ac-game-settings-option">
            登录
        </div>
        <br>
        <div class="ac-game-settings-acwing">
            <img width="30" src="https://app165.acapp.acwing.com.cn/static/image/settings/acwing_logo.png">
            <br>
            <div>
                AcWing一键登录
            </div>
        </div>
    </div>
</div>
`);          // 生成settings的html对象
        this.$login = this.$settings.find(".ac-game-settings-login");  // 登录界面
        this.$login_username = this.$login.find(".ac-game-settings-username input");   //用户名输入框
        this.$login_password = this.$login.find(".ac-game-settings-password input");  // 密码输入框
        this.$login_submit = this.$login.find(".ac-game-settings-submit button");     // 提交按钮
        this.$login_error_message = this.$login.find(".ac-game-settings-error-message");// 错误信息
        this.$login_register = this.$login.find(".ac-game-settings-option");          // 注册选项

        this.$login.hide();     //隐藏登录界面

        this.$register = this.$settings.find(".ac-game-settings-register");   //注册界面
        this.$register_username = this.$register.find(".ac-game-settings-username input");
        this.$register_password = this.$register.find(".ac-game-settings-password-first input");
        this.$register_password_confirm = this.$register.find(".ac-game-settings-password-second input");
        this.$register_submit = this.$register.find(".ac-game-settings-submit button");
        this.$register_error_message = this.$register.find(".ac-game-settings-error-message");
        this.$register_login = this.$register.find(".ac-game-settings-option");

        this.$register.hide();   //隐藏注册界面

        this.root.$ac_game.append(this.$settings);    // 将这个html对象加入到ac_game    定义了这些html对象还需要修改game.css文件中的样式，这里不再赘述

        this.start();
    }

    start() {
        this.getinfo();   // 一进入网页就要先获取信息
        this.add_listening_events();     //绑定监听函数
    } 

    add_listening_events() {
        this.add_listening_events_login();   // 注册页面的监听
        this.add_listening_events_register();   // 登陆页面的监听
    }  

    add_listening_events_login() {
        let outer = this;

        this.$login_register.click(function() {  // 在登录页面点击注册选项就打开注册界面
            outer.register();
        });
        this.$login_submit.click(function() {
            outer.login_on_remote();
        });
    }

    add_listening_events_register() {
        let outer = this;
        this.$register_login.click(function() {    // 在注册页面点击登录选项就打开登录界面
            outer.login();
        });
        this.$register_submit.click(function() {
            outer.register_on_remote();
        });
    }

    login_on_remote() {  // 在远程服务器上登录
        let outer = this;
        let username = this.$login_username.val();  // 获取输入框中的用户名和密码
        let password = this.$login_password.val();
        this.$login_error_message.empty();

        $.ajax({
            url: "https://app165.acapp.acwing.com.cn/settings/login/",
            type: "GET",
            data: {
                username: username,   //向login传输数据
                password: password,
            },
            success: function(resp) { //从login返回的resp
                console.log(resp);
                if (resp.result === "success") {  //如果成功了就刷新
                    location.reload();
                } else {
                    outer.$login_error_message.html(resp.result);  // 如果失败了就显示错误信息
                }
            }
        });
    }

    register_on_remote() {  // 在远程服务器上注册
        let outer = this;
        let username = this.$register_username.val();
        let password = this.$register_password.val();
        let password_confirm = this.$register_password_confirm.val();
        this.$register_error_message.empty();

        $.ajax({
            url: "https://app165.acapp.acwing.com.cn/settings/register/",
            type: "GET",
            data: {
                username: username,
                password: password,
                password_confirm: password_confirm,
            },
            success: function(resp) {
                console.log(resp);
                if (resp.result === "success") {
                    location.reload();  // 刷新页面
                } else {
                    outer.$register_error_message.html(resp.result);
                }
            }
        });
    }

    logout_on_remote() {  // 在远程服务器上登出
        $.ajax({
            url: "https://app165.acapp.acwing.com.cn/settings/logout/",
            type: "GET",
            success: function(resp) {
                console.log(resp);  //测试输出
                if (resp.result === "success") {
                    location.reload();    // 如果成功了就直接刷新
                }
            }
        });
    }

    register() {  // 打开注册界面
        this.$login.hide();
        this.$register.show();
    }

    login() {  // 打开登录界面
        this.$register.hide();
        this.$login.show();
    }

    getinfo() { // 获取信息
        let outer = this;

        $.ajax({  // 发送一个请求
            url: "https://app165.acapp.acwing.com.cn/settings/getinfo/",// 获取信息的URL
            type: "GET",
            data: {
                platform: outer.platform,
            },
            success: function(resp) {
                if (resp.result === "success") {
                    outer.username = resp.username;
                    outer.photo = resp.photo;
                    outer.hide(); //如果成功登录，隐藏登录界面
                    outer.root.menu.show();  // 并显示游戏菜单
                } else {
                    outer.login();// 如果没有登录就打开这个登录页面
                }
            }
        });
    }

    hide() {
        this.$settings.hide();
    }

    show() {
        this.$settings.show();
    }
}

```

在实现了登陆页面和注册页面之后，处理**后端的登录和注册逻辑**
首先实现登录
创建`game/views/settings/login.py`，实现登录，代码如下

```py
from django.http import JsonResponse
from django.contrib.auth import authenticate, login

def signin(request):
    data = request.GET      # 获取请求的信息
    username = data.get('username')
    password = data.get('password')
    user = authenticate(username=username, password=password)  # 从数据库中查找这个用户
    if not user:
        return JsonResponse({
            'result': "用户名或密码不正确"
        })
    login(request, user)    # 找到了就登录，python内置函数
    return JsonResponse({
        'result': "success"
    })
```

创建`game/views/settings/logout.py`，实现登出，代码如下

```py
from django.http import JsonResponse
from django.contrib.auth import logout

def signout(request):
    user = request.user
    if not user.is_authenticated:
        return JsonResponse({
            'result': "success",
        })
    logout(request)
    return JsonResponse({
        'result': "success",
    })
```

由于登出操作是在菜单`menu`界面中执行，所以需要在`menu/zbase.js`中修改监听函数

```js
class AcGameMenu {
    this.$menu.hide();     // +
    ...
    this.$settings.click(function() {
            console.log("click settings");
            outer.root.settings.logout_on_remote();  // +
        });
    ...
}
```

实现注册,创建`game/views/settings/register.py`并进去，代码如下

```js
from django.http import JsonResponse
from django.contrib.auth import login
from django.contrib.auth.models import User
from game.models.player.player import Player

def register(request):
    data = request.GET
    username = data.get("username", "").strip()
    password = data.get("password", "").strip()
    password_confirm = data.get("password_confirm", "").strip()
    if not username or not password:
        return JsonResponse({
            'result': "用户名和密码不能为空"
        })
    if password != password_confirm:
        return JsonResponse({
            'result': "两个密码不一致",
        })
    if User.objects.filter(username=username).exists():
        return JsonResponse({
            'result': "用户名已存在"
        })
    user = User(username=username)
    user.set_password(password)
    user.save()
    Player.objects.create(user=user, photo="默认头像URL")
    login(request, user)
    return JsonResponse({
        'result': "success",
    })
```

将实现的四个视图函数加入到`url`中，在`game/urls/settings/index.py`中添加代码如下

```py
from django.urls import path
from game.views.settings.getinfo import getinfo
from game.views.settings.login import signin
from game.views.settings.logout import signout
from game.views.settings.register import register

urlpatterns = [
    path("getinfo/", getinfo, name="settings_getinfo"),
    path("login/", signin, name="settings_login"),
    path("logout/", signout, name="settings_logout"),
    path("register/", register, name="settings_register"),
]
```

注意，创建settings模块之前一定要修改主模块AcGame，加入其中

```js 
export class AcGame {
    this.settings = new Settings(this); // +
    
}
```

