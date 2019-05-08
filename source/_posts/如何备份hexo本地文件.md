---
title: 如何备份hexo本地文件
date: 2019-05-08 19:44:37
tags:
---

# Hexo+github搭建博客：备份博客本地文件到github

环境：**ubuntu 18.04**（win10环境下操作也差不多）

由于种种原因，~~比如sudo rm -rf命令没敲完误触回车~~，你所创建的博客的本地文件有很大可能被损坏甚至删除。
且被删除的本地文件是无法通过github上的文件恢复的：hexo d提交的只有生成好页面文件（就是hexo clean清除的那些）。

但是我们可以把本地文件提交到github中。**你可能已经注意到，hexo init生成的博客文件夹里面有.gitignore文件。**

对于经常使用Git的朋友来说，.gitignore配置一定不会陌生。这种方式通过在项目的某个文件夹下定义.gitignore文件，
在该文件中定义相应的忽略规则，来管理当前文件夹下的文件的Git提交行为。
.gitingore 文件中，应该遵循相应语法，在每一行指定一个忽略规则。

比如：（这里内容表示忽略此目录下log、temp后缀文件和vendor文件夹）

>>>
	*.log
	*.temp
	/vendor
>>>

**表明官方是希望我们把本地文件通过git保存到github等保管服务器的。**

下面介绍在博客仓库里面备份本地文件的方法。

---

## 前言：

### 0. 不要随便sudo，真的会有很多麻烦。~~不要问我怎么知道的。~~
	比如：一般默认是无法以root用户直接登录图形界面的，意味着用sudo创建的文件只能用命令行sudo权限来编辑，且用root登录图形界面，只会变得更麻烦。
### 1. 最好安装oh-my-zsh，方便看目前所在的分支，因为我们会在两个分支内操作。具体看官方github的readme。
### 2. 最好用nvm安装node和npm，nodejs是node的发行版本，各个包管理器上都不一样，有可能有npm有可能没有，下了还要升级什么的很麻烦还可能有比如包版本的和依赖的各种错误
### 3. 注意安装好node和npm后最好只cd在你的博客文件夹里操作，如npm install hexo-cli和npm install hexo-deployer-git，除了node和npm本体不要随便-g（全局安装），不要sudo（root用户操作）。
### 4. ~~如果哪里有node报错的话npm init -y（-y即全默认）一下生成packge.json文件可能会有用~~（超脱范畴了好像）
### 5. 如果你不是同时需要三台或者更多的系统的话，最好找一台真正的电脑安装linux，虚拟机是真的不方便，笔记本双系统兼容性又看脸。
	主流虚拟机应用1.Hyper-v装windows效率可以，装图形界面的linux卡的一批而且1803之后不再支持remoteFX就是gpu虚化。2.vm station一堆小毛病，而且卡顿也是有的。完全不能跟直接装系统比。

---

## 步骤：

**（自定义博客的操作推荐在备份推送到github相应分支后确认无误后再做~~方便报错重来~~）**

## 0. **准备**

=>百度一篇合适的hexo+github建博客文章并且完全读懂，安装nvm然后用nvm安装node和npm，安装好git hexo hexo-deployer-git，申请好github账号建立博客仓库等等，并且一定记得把ssh密钥生成了然后挂到github

## 1. hexo init ```<a name>```
	
下载hexo模板博客，名字叫```<a name>```。
>>>
	//最好不要加sudo前缀，会导致无法直接在图形界面操作。
>>>
## 2. 下载主题，如next

下载next到theme目录，进入theme/next文件夹，删除此目录下的.git .github .gitignore三个文件
>>>
	//留下这些文件会导致嵌套git的问题，且博客主题文件的修改大都是在这个文件夹下完成的，这里是我们备份的主要目标
>>>
## 3. cd ```<a name>```
>>>
	//进入目录
>>>

## 4. git init

>>>
	//这是在此文件夹里面初始化git，生成各种git文件文件夹，生成master分支，master分支内容为当前文件夹中除去.gitignore标记的内容的内容
>>>

## 5. （可选）在```<a name>```目录里面检查/更改.gitignore文件
	
## 6. 在```<a name>```目录git branch查看分支，注意：使用oh-my-zsh的话git branch操作出来的可能是用vi打开的的一个文件
>>>
	//康康有没有master这个分支，如果什么都没有就
	//git add . 
	//将工作区的文件提交至暂存区(提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
	//git commit -m "<这里写提交说明>"
	//暂存区提交到所属分支
	//如果还没有请删除重做，这步成功后应该显示当前仅有且位于master分支
>>>
## 7. 照着0步找的其他博客配置，然后hexo g hexo s，都没有错误后关闭终端上的本地服务，hexo d，确保可以成功上传

## 8. 回到```<a name>```目录，创建新分支back-up：git branch back-up
>>>
	//不一定必须叫back-up，只是比较好认
>>>
## 9. 转到back-up分支：git checkout back-up
>>>
	//这两个命令可以合并为：git checkout -b back-up ，不过还是建议一开始能详细就详细一点分开写
>>>
## 10. 检查分支：git branch
>>>
	//显示两个分支master和back-up且当前在back-up分支
>>>
## 11. 让git链接到github上的远程仓库：git remote add ```<name> <your url>```
>>>
	//<your url>是你的仓库的git链接，下面例子用的是https地址。<name>为仓库git地址的别名。
	//这步之后以后就可以用<name>代替<your url>了。
	//使用git remote查看已链接的仓库
	//这里格式如：git remote add origin https://github.com/xxx/xxx.github.io.git
>>>
## 12. 推送back-up分支：git push -u ```<name>``` back-up
>>>
	//<name>即11步创建的别名
	//此命令将本地的back-up分支推送到<name>链接的主机，并且新建back-up分支（如果原来仓库没有的话）
	//同时指定<name>为默认主机，后面就可以不加任何参数使用git push了
	//不带任何参数的git push，默认只推送当前分支，称为simple方式，而且如果没有执行12命令之前git push会提示没有推送目标
	//此步执行后需要你输入github用户名（上一步里面的xxx）和密码
>>>
## 13. 到此和github仓库的链接备份初始处理结束，以后备份时到back-up分支执行git push就可以
>>>
	//当然也可以git add . 、git commit -m "" 、git push <name> back-up来提交
	//注意：不要在master分支提交，master应该是博客页面文件存放的地方
	//如果是失误在master分支打入git push命令没有关系，因为我们并没有在master分支git push -u过，git只会提示没有推送目标
>>>
