---
layout: post
title: GIT介绍
date: 2016-08-14
tag: GIT
---

* TOC
{:toc}

### **版本控制**

版本控制系统(Version Control System,简称VCS)是一中记录一个或多个文件的变化的系统，能够方便找回某个时期/版本的文件。

版本控制系统的发展

![image](\assets\images\git_introduce\1.png) 



>- 本地版本控制(Local Version Control System)顾名思义就是本地化的版本控制，没有网络协作等较为先进的版本控制的概念。(如RCS)
>- 集中式版本控制(Centralized Version Control System)运行一台版本控制服务器，提供存储项目中所有文件的服务。(如SVN、CVS)
>- 分布式版本控制(Distributed Version Control System)是一种不需要中心服务器的管理文件版本的方法，但是它也可以使用中心服务器。(如Git)


CVCS和DVCS差异

>- DVCS 通过本地提交支持离线工作，这是由 DVCS 的操作方式决定的。这与集中式版本控制完全不同，集中式版本控制要求通过到中心服务器的连接执行所有操作。这种灵活性让开发人员在飞机上也能够像在办公室中一样轻松地工作，可以一次又一次地进行提交。
>- DVCS 比集中式系统更灵活，因为 DVCS 支持许多不同类型的工作流，从传统的集中式工作流到纯粹的特殊工作流，再到特殊工作流和集中式工作流的组合。
>- DVCS 比集中式版本控制系统快得多，因为大多数操作在客户机上进行，速度非常快。另外，在需要进行推（push ）操作（与另一个节点通信）时，速度也更快，因为两个客户机机器上都有完整的元数据。速度差异相当显著。

### **Git基础**

Git优势

>- 直接记录快照，而非差异比较
>- 近乎所有操作都是本地执行
>- 时刻保持数据完整性
>- 多数操作仅添加数据

其他系统(如svn,每个版本中记录着各个文件的具体差异)
![image](\assets\images\git_introduce\2.png)

直接记录快照，而非差异比较 

![image](\assets\images\git_introduce\3.png)

近乎所有操作都是本地执行

>- 在 Git 中的绝大多数操作都只需要访问本地文件和资源，不用连网。但如果用 CVCS 的话，差不多所有操作都需要连接网络。因为 Git 在本地磁盘上就保存着所有当前项目的历史更新，所以处理起来速度飞快。
>- 举个例子，如果要浏览项目的历史更新摘要，Git 不用跑到外面的服务器上去取数据回来，而直接从本地数据库读取后展示给你看。所以任何时候你都可以马上翻阅，无需等待。如果想要看当前版本的文件和一个月前的版本之间有何差异，Git 会取出一个月前的快照和当前文件作一次差异运算，而不用请求远程服务器来做这件事，或是把老版本的文件拉到本地来作比较。

时刻保持数据完整性

>- 在保存到 Git 之前，所有数据都要进行内容的校验和（checksum）计算，并将此结果作为数据的唯一标识和索引。换句话说，不可能在你修改了文件或目录之后，Git 一无所知。
>- 24b9da6552252987aa493b52f8696cd6d3b00373
>- Git 使用 SHA-1 算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串。
>- Git 的工作完全依赖于这类指纹字串，所以你会经常看到这样的哈希值。实际上，所有保存在 Git 数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。

多数操作仅添加数据

>- 常用的 Git 操作大多仅仅是把数据添加到数据库。因为任何一种不可逆的操作，比如删除数据，都会使回退或重现历史版本变得困难重重。在别的 VCS 中，若还未提交更新，就有可能丢失或者混淆一些修改的内容，但在 Git 里，一旦提交快照之后就完全不用担心丢失数据，特别是养成定期推送到其他仓库的习惯的话

Git中文件的三种状态

>- 已提交(committed):表示该文件已经被安全地保存在本地库中了
>- 已修改(modified):表示修改了某个文件，但还没有提交保存
>- 已暂存(staged):表示把已修改的文件放在下次提交时要保存的清单中

Git的三个文件区域

>- Git 管理项目时，文件流转的三个工作区域：Git 的工作目录、暂存区域以及本地仓库(图1)
>- 使用git add将文件从工作区添加到暂存区域(图2)
>- 使用git commit将文件从暂存区域添加到本地仓库(图3)

![image](\assets\images\git_introduce\4.jpg)

![image](\assets\images\git_introduce\5.jpg)

![image](\assets\images\git_introduce\6.jpg)



### **Git命令**

Git基本操作

>- 检出版本：git init  、git clone
>- 提交修改：git add 、git commit 、git pull 、 git push
>- 获取信息：git help 、git status   、git diff 、  git log 、git show 、git remote
>- 分支： git branch 、git checkout
>- 回退： git reset

Git目录结构

>- HEAD：这个git项目处在当前哪个分支里
>- config：项目的配置信息，使用git config改动
>- description：项目的描述信息
>- hooks/：系统默认挂钩目录(特定情况下触发某些操作)
>- index：索引文件
>- logs/：各个refs的历史信息
>- objects/：git本地仓库的所有对象(commits,trees,blobs,tags)
>- refs/：标识你项目里的每个分支指向了哪个提交(commit)

Git命令

> 全局配置

>-     git config global user.name “matieli” #设置名称
>-     git config global user.email “ma_tl@suixingpay.com” #设置邮箱
>-     git config list #查看配置信息


> 创建一个库

>-     git init


> 克隆一个库

>-     git clone http://192.168.120.68/root/lemon-mongo.git


> 基本使用

>-    git help    #帮助
>-    git status #查看当前状态
>-    git log      #”j”向下浏览,”k”向上浏览,”q”退出
>-    git diff     #比较
>-    gitk          #图形日志

> 增删改

>-     git add readme.txt    #从工作区增加readme.txt到暂存区
>-     git commit –m “add readme.txt” #提交暂存区文件到git，内容add..
>-     git rm readme.txt      #删除readme.txt，需要git commit –m “ ..”


> 分支管理

>-    git branch -a                          #所有分支
>-    git branch dev                       #创建一个名为dev的分支
>-    git checkout dev                   #切换到dev分支
>-    git branch -m dev develop #修改分支名称由dev改成test
>-    git branch –d dev                 #删除dev分支
>-    git merge dev                       #将当前分支与dev分支合并

> 标签管理

>-    git tag                    #查看标签列表
>-    git tag v101          #建立标签
>-    git tag –a v101 –m “version v101”     #建立含批注的标签
>-    git tag –d v101     #删除标签

> 远程库

>-   git clone <url>                                #复制现有的远程库
>-   git remote add <name>  <url>    #复制现有的远程库
>-   git remote                        #显示远程库列表，加-v显示详细
>-   git push <repository>    #推送提交/修改内容到分支
>-   git fetch <repository>    #查看远程库分支的修改内容
>-   git pull <repository>      #读取远程分支的修改内容
>-   git push --delete <repository>  <branchname> #删除远程库的分支
>-   git push <repository>  <tagname>                  #建立远程库标签
>-   git push –delete <repository>  <tagname>    #删除远程库标签
>-   git remote set-url <name> <newurl>  #修改已注册的远程库的地址
>-   git remote rename <old> <new>    #修改已注册的远程库名称


> 还原撤销

>-   git checkout -- file     #恢复一个文件
>-   git reflog                    #查看HEAD的移动历史
>-   git reset --hard HEAD~ #放弃最近提交
>-   git rebase                 #重新定义分支的版本库的状态
>-   git log --grep "<pattern>“  #查找包含特定注解的提交

Git和Svn命令差别

操作| Git | Svn
---|---|---
复制数据库 | git clone | svn checkout
提交 | git commit | svn commit(※2)
查看提交的详细记录 | git show | svn cat
确认状态 | git status | svn status
确认差异 | git diff | svn diff
确认记录 | git log | svn log
添加 | git add | svn add
移动 | git mv | svn mv
删除 | git rm | svn rm
取消修改 | git checkout / git reset(消除提交) | svn revert (取消修改)
创建分支 | git branch | svn copy (※1)
切换分支 | git checkout | svn switch
合并 | git merge | svn merge
创建标签 | git tag | svn copy (※1)
更新 | git pull / git fetch | svn update
反映到远端 | git push | svn commit(※2)
忽略档案目录 | .gitignore | .svnignore
   
### **使用EGit**

安装\配置Egit插件

>- Eclipse 4.3以上已集成了Egit。window-->preferences搜索git.
>- 在Eclipse Marketplace搜索Egit安装
>- 配置个人信息：最重要的是user.name和user.email
>- Window> Preferences > Team > Git > Configurationl> New Entry

![image](\assets\images\git_introduce\7.png)

创建Git仓库

>- 新建项目
>- 在项目上右键>Team > Share Project 选择GIT,会出现下图，选中及选择仓库地址后点击Create Repository创建仓库，点击Finish。

![image](\assets\images\git_introduce\8.png)

创建Git仓库

>- 创建仓库后，在目录下的.git文件夹，就是git的仓库地址。和SVN不同，GIT不会在每一个目录下建立版本控制文件夹，仅在根目录下建立仓库
>- 同时，eclipse中的project也建立git版本控制，此时未创建分支，处于NO-HEAD状态
>- 文件夹中的符号”?”表示此文件夹处于untracked状态，这样就成功创建GIT仓库。

![image](\assets\images\git_introduce\9.png)

![image](\assets\images\git_introduce\10.png)

配置.gitignore

尝试进行一次提交Team -> Commit… 此时我们看到出现.settings文件，这些文件是我们不需要进行版本控制的这时候就需要配置.gitignore,打开.gitignore文件，写入” .settings”,再次提交已经过滤掉了.settings

![image](\assets\images\git_introduce\11.png)

首次提交

>- 首次提交后，会自动生成master分支；创建B.java，可以看到图标依然是问号，处于untracked状态，即git没有对此文件进行监控。通过Team -> Add to index可以将文件加入git索引，进行版本监控。

![image](\assets\images\git_introduce\12.png)

![image](\assets\images\git_introduce\13.png)

![image](\assets\images\git_introduce\14.png)

>- 图标显示有了变化(EGIT中只要Commit就可以默认将untracked的文件添加到索引再提交更新，不需要分开操作)，此时文件处于staged 状态；将新增的文件commit到仓库中，文件将处于committed状态；修改文件的内容，文件将处于modified状态。



查看历史记录

>- Team -> Show in history可以查看版本历史提交记录，可以选择对比模式，查看修改内容

![image](\assets\images\git_introduce\15.png)

![image](\assets\images\git_introduce\16.png)

远程GIT仓库

>- 此小结的前提是已经搭建GIT服务器，使用公司的gitLab服务器。
>- 在服务器上创建一个项目http://192.168.120.68/matieli/gitTest.git
>- 打开GIT资源库窗口，选择克隆资源库

![image](\assets\images\git_introduce\17.png)

![image](\assets\images\git_introduce\18.png)


>- 在gitTest项目上右键>Team>Remote>Push…将项目推送到远程仓库。
>- 登录gitLab服务器查看在添加远程库之前的3次commits都在远程库中有体现目前就一个分支(master)。

![image](\assets\images\git_introduce\19.png)

导入gradle项目

>- 在GitLab找到要导入已有的gradle项目： http://192.168.120.68/root/lemon-mongo.git
>- 在Git Repositories中，使用快捷键Ctrl +V，出现Clone Git Repository 窗口，根据引导选择导入的分支、项目的存放路径(workspace)等信息，最后点击Finish，结果如下图。

![image](\assets\images\git_introduce\20.png)

![image](\assets\images\git_introduce\21.png)

>- 这个时候在选择的Directory下会有lemon-mongo的项目
>- 切换到java视图，按照导入gradle的lemon-mongo项目，Import-->import…-->选择Gradle Project-->选择git导出时的项目路径-->点击Build Model-->勾选项目-->Finish。

![image](\assets\images\git_introduce\22.png)


创建新分支dev1和dev2

>- Team-->Switch To-->new branch… 分别创建dev1和dev2分支。checkout new branch的选择选中后会自动切换到你的新建分支中。注：这个操作仅仅是创建本地分支，Team-->Push branch…推送分支到远程。Switch To是切换分支的作用可以新建分支，也可以切换到git repositories视图下直接创建local和remote分支。

![image](\assets\images\git_introduce\23.png)

合并分支

>- 在dev1分支中修改项目中的某个文件并且合并到master分支上。比如修改build.gradle的maven nexus地址。并推送到远程dev1分支下。这个时候dev1的分支比master分支高了一个版本，把刚才的改动合并到master分支下。切换到master分支。选择Team--> Merge 选择dev1分支。这个时候master分支后会显示向上箭头1，意思是本地master比远端的master高一个版本。需要把本地的master改动推送到远端。Team--> Push Branch…选择远端的master分支，Finish。登录gitLab查看master分支发现已经变成dev1的地址了。

![image](\assets\images\git_introduce\24.png)

![image](\assets\images\git_introduce\25.png)

冲突解决

>- 假设在dev2分支下也修改了这个私服地址，改为：172.16.20.18。（步骤省略）
>- 使用dev2和master分支merge。这个时候肯定会冲突。
>- 很明显自动合并的冲突文件不能直接使用，我们可以手动调整，右键发生冲突的文件，选择Team -> Merge Tool，然后右键点击此冲突文件，选择Team -> Add to index再次将文件加入索引控制，此时文件已经不是冲突状态，并且可以进行提交并push到服务器端。

![image](\assets\images\git_introduce\26.png)

Merge和Rebase的区别

>- merge 会生成一个新得合并节点(历史记录)，而rebase不会。
>- 对于一个多人开发团队频繁提交更新的情况，如果使用merge会使得历史线图非常复杂，并且merge一次就会新增一个记录点，如果使用rebase就是完全的线性开发。
>- 建议：如果同一文件反复修改或提交次数比较多，预期会出现很多的冲突，那么可以使用merge合并，仅需要解决一次冲突即可(不过，大范围的修改，是不是应该事先就新开一个分支呢？)；如果修改范围小，预期冲突少，则建议使用rebase。

重置功能Reset

> Team-->Reset…

>- GIT中有三种重置功能，分别是soft、mixed、hard，区别如下：
   l  Soft - t将其他的commit重置到HEAD，就仅此而已。index和working copy中的文件都不改变。
   
![image](\assets\images\git_introduce\27.png)

>-   l  Mixed -改变HEAD和index，指向那个你要reset到的commit上。而working copy文件不被改变。当然会显示工作目录下有修改，但没有缓存到index中
   
   ![image](\assets\images\git_introduce\28.png)
   
   
>-   l  Hard -  HEAD & index & working copy同时改变到你要reset到的那个commit上。这个参数很危险，执行了它，你的本地修改可能就丢失了。

  ![image](\assets\images\git_introduce\29.png)

### **GitLab**

GitLab服务器

>- 安装步骤略，安装完效果图如下

![image](\assets\images\git_introduce\30.png)

查看相关信息

>- 登录GitLab界面后，点击一个项目，进入项目空间，可以看到项目的相关信息。在这里你可以查阅项目、文件、提交的Commits、里程碑、Bug列表（Issues）、讨论墙、维基页面、成员等等

创建项目

>- 登录GitLab界面后，你可以创建自己的项目，别的开发人员也可以加入你的项目。在右侧的Projects框里可以看到当前所有的项目、自己的项目和自己加入的项目。点击+New Project按钮可以创建新项目（顶部的+号也一样可以）。

>- a) 只需要填写一个项目名称，点击Create project按钮就可以创建项目。新项目建立在你的用户名之下,Http访问的URL格式为：http://192.168.120.68/root/lemon-mongo.git。
>- b) 新项目创建后，需要通过Git客户端进行初始化以后才能够使用，在新项目页面给出了通过Git命令行工具初始化的方法。

![image](\assets\images\git_introduce\31.png)

添加项目成员

>- 创建好项目后需要在Members中添加成员，否则就只有创建者有权限提交代码。权限有guest、reporter、developer、master。具体权限信息查看http://192.168.120.68/help/permissions/permissions


![image](\assets\images\git_introduce\32.png)


