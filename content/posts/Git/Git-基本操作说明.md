---
title: "Git-基本操作说明"
tags:
  - git
  - 命令
categories: [Git]
date: 2019-07-18 08:09:37
draft: false
toc: false
images:
math: true
---

Git Yes!

<!--more-->



![git](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/git说明.png)

经典git关系图

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

## 初始化

刚新建一个项目的时候需要来几条初始化命令

##### 生成本地仓库

git init 

##### 把工作区的文件提交到暂存区

git add . 
或者
git add 文件名

##### 把暂存区的文件提交到本地仓库

git commit -m "描述"
这里的描述就是到时候看到的下面的这些

![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/1.png)


##### 先给你要提交的远程仓库起个别名

git remote add 仓库别名 Git地址
Git地址如下图所示

![2](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/2.png)


##### 把本地仓库的文件提交到远程仓库（就是github上能看到的那种）

git push -u 仓库别名 分支名
分支就是....算了这里是傻瓜备忘不想解释

![3](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/3.png)


到这就可以上去github看看了。文件内容都在里面

## 第二次提交

第二次提交的时候一般不是整个项目都有变动对吧？
没事git会自动识别修改过的和没修改过的文件

##### 可以看看git识别到哪些

git status

![gitstatus](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/gitstatus.png)

红色的，Umerged，表示工作区有改动的文件，还没提交到暂存区
git文件的四种状态

- untracked   未被追踪的。就是还没添加过的
- unmodified 工作区里已经被追踪了，还没修改
- modified     工作区的文件修改了但还没提交到暂存区
- staged        添加到了暂存区倒是还没提交到远程仓库

##### 把它提交到暂存区去

git add .

##### 把它提交到本地仓库去

git commit -m "描述"

##### 把它提交到远程仓库去

git push -u 远程仓库别名 分支名

搞定！

##### 拉取远程仓库的文件到本地

git pull 远程仓库别名 分支名

多用户共同开发的时候，新用户可能会在本地新建一个文件夹，然后git init, 接着pull
可能会出现拉取失败，因为git认为这是两个不同的项目，所以拒绝拉取合并，可以加上`--allow-unrelated-histories`

```shell
git pull 仓库别名 分支名 --allow-unrelated-histories
```

### 总结

要把代码写完放在github上就等于 你要从山旮旯里寄东西到北京

工作区 => 就是你项目的文件夹，你经营的这家客栈
暂存区 => 就是你这个项目的git索引，你这个村里的驿站
本地仓库 => 就是你电脑里的一个存储库，你市里的驿站
远程仓库 => 就是github服务器上面，北京

你东西收拾好了的时候  得找村里的快递站帮你保管和寄送到市里的快递站
`git add .`
等你说commit的时候    市里的快递站 就给你 送到省里的快递站
`git commit -m "描述"`
等你说push的时候        省里的快递站 就给你 送到北京
`git push -u 远程仓库别名 分支名`

\>_

![GIT常用命令](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/git常用.png)
