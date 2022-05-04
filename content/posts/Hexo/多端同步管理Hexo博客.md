---
title: "多端同步管理Hexo博客"
tags:
  - note
  - git
  - HEXO
categories:
  - [Note]
  - [Git]
  - [HEXO]
date: 2019-05-18 11:37:35
draft: false
toc: false
images:
math: true
---


多端管理其实很简单。

<!--more-->

hexo 其实是帮我们编译好放在`博客根目录\public`目录下的，然后把`public目录`推送到github上的，所以说github.io上面都是编译过的文件，他们是光鲜亮丽的演员，但离不开默默付出的后台人员。这些默默付出的后台人员就是除了public之外的所有的文件。

我们的博客是发布到master主分支的，那就建另一个分支，把所有文件都传上去，在别的电脑上都拉下去就可以了。



### 1. 首先填写忽略声明文件`.gitignore`

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_config.yml
```

- public  每次都会编译覆盖，所以不需要它
- .deploy*   编译产生的文件，也不需要
- _config.yml  配置文件里会有一些id/key的重要信息，所以不上传



### 2. 初始化仓库和提交

之前我以为hexo就是帮我们操作git，所以不明白为什么还能再初始化git

后来才知道不是那么回事。但是hexo是怎么操作我还是没明白。这里不管

像往常一样，在根目录右键 git bash here

> 注意！！！大坑！！！
>
> 如果你用的是第三方的主题theme，是使用git clone下来的话，要把主题文件夹下面把.git文件夹删除掉，不然主题无法push到远程仓库，导致你发布的博客是一片空白。所以先去检查你使用的主题有没有.git这个目录

```shell
git init  //初始化本地仓库
git add . //添加本地所有文件到仓库        
git commit -m "blog源文件" //添加commit
git branch backup //添加本地仓库分支hexo
git remote add origin <server>  //添加远程仓库 <server> 是指在线仓库的地址 origin是本地分支,remote add操作会将本地仓库映射到云端
git push origin backup //将本地仓库的源文件推送到远程仓库hexo分支
```



### 在另一台电脑的操作

首先肯定要搭建环境啦（Node 和 Git）

完了后用这个命令

```shell
git clone <server> hexo //<server> 是指在线仓库的地址
cd hexo 
npm install
```

npm install的时候会根据package.json中的插件列表自动加载相应插件。
 本机的同步完成。

> 因为在上传博客源文件的时候忽略了配置文件（_config.yml这是站点的配置文件）的上传，也就是没有上传配置文件的，在克隆下来的时候记得把配置文件拿过来，不然会报错。主题里面的配置文件也要（themes/next/_config.yml这是主题配置文件）



这里贴一张常用Git命令

\>_

![git常用命令](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Git-basic-operation/git常用.png)
