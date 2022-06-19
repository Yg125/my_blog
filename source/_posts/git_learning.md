---
title: Git_learning
date: 2022-06-16 20:18:21
tags: Git
description: 'Git principles and common operations'
cover: http://cdn.admxj.com/wp-content/uploads/2017/07/git.jpg
top_img: https://picsum.photos/id/477/4919/3258
---



# git基本知识及指令
**git即管理本地代码及与云端代码互存**
### 1 git原理
git建立仓库之后将代码划分了三个区域
本地工作区work，暂存区stage，版本库commit <u>连接远程仓库还可以有origin</u>
work可以修改代码，修改后的代码需要使用`git add xxx`加入暂存区stage
暂存区在合适的时候可以把代码`git commit -m "xx"`推送到版本库，
这就是一个版本（由某个hash值确定），可以使用git操作进行选择版本
branch可以理解为在这个HEAD下创建另一个方向，做得任何改动都是这个branch的
可以`git merge branch`将这个branch与当前branch合并，也就是HEAD指向它的最后一个版本，
然后将branch归于当前branch
可能会在merge之前两个/多个branch都做了改动，那么会有冲突，手动选择改变的版本即可
画图很容易理解，一条branch只有一条路，中间分路的话可以分别走，也可以合并，合并就要到最终的版本
如果两/多条路都走了那么在合并时会冲突，需要手动选择

### 2 git命令分类整理

----------


全局设置
1. `git config --global user.name xxx`：设置全局用户名，信息记录在~/.gitconfig文件中
2. `git config --global user.email xxx@xxx.com`：设置全局邮箱地址，信息记录在~/.gitconfig文件中
3. `git init`：将当前目录配置成git仓库，信息记录在隐藏的.git文件夹中

常用命令
1. `git add XX` ：将XX文件添加到暂存区
2. `git add .`：将所有修改的文件加到暂存区
2. `git commit -m "给自己看的备注信息"`：将暂存区的内容提交到当前分支
3. `git status`：查看仓库状态
4. `git log`：查看当前分支的所有版本
5. `git diff`:查看更改的内容，只能查看显示暂存区和工作区的差异
6. `git clone git@git.acwing.com:xxx/XXX.git`：将远程仓库XXX下载到当前目录下
7. `git reflog`:查看HEAD指针的移动历史（包括被回滚的版本）
8. `git log --pretty=oneline`：用一行来显示
9. `git restore --staged xx`：将xx从暂存区里移除，即还是修改文件，只不过没有在stage中了
10. `git restore xx`：将xx尚未加入暂存区stage的修改撤销，即撤销改动，如果stage为空，则撤销到上一个commit
11. `git reset --hard HEAD^` 或`git reset --hard HEAD~ `：将代码库回滚到上一个版本
12. `git reset --hard 版本号`：回滚到某一特定版本 其中版本号可以通过`git reflog`得到想要的版本号
13. `git checkout -b branch_name`：创建并切换到branch_name这个分支
14. `git branch branch_name`:创建branch_name的分支
15. `git branch`：查看所有分支和当前所处分支
16. `git checkout branch_name`：切换到branch_name这个分支
17. `git branch -d branch_name`：删除本地仓库的branch_name分支
18. `git merge branch_name`：将分支branch_name合并到当前分支上
19. `git remote add origin git@git.acwing.com:xxx/XXX.git`：将本地仓库关联到远程仓库
20. `git push -u` (第一次需要-u以后不需要) ：将当前分支推送到远程仓库
21. `git push --set-upstream origin branch_name`:设置本地的branch_name分支对应远程仓库的branch_name分支
22. `git push -d origin branch_name`：删除远程仓库的branch_name分支
23. `git pull`：将远程仓库的当前分支与本地仓库的**当前分支**合并
24. `git pull origin branch_name`：将远程仓库的branch_name分支与本地仓库的**当前分支**合并
25. `git branch --set-upstream-to=origin/branch_name1 branch_name2`：将远程的branch_name1分支与本地的branch_name2分支对应,通常这样来建立远程和本地的分支联系，然后在从远程拉取到本地指定branch
26. `git checkout -t origin/branch_name`:将远程的branch_name分支拉取到本地

### 3. .gitignore
------
通过配置.gitignore文件git的时候可以直接忽略某些自动生成的文件不上传
``` bash
.vscode/   #忽略.vscode文件夹的文件更新
*.log       #忽略所有.log后缀的文件
config.ini  #忽略config.ini文件
```