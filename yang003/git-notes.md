#git笔记

标签（空格分隔）： 未分类

---
##新建仓库

1.新建版本库
```python
$ mkdir xxxx 
```
2.初始化
```python
$ git init 
```
3.把文件从工作区发送到缓存区
```python
$ git add readme.txt 
```
![此处输入图片的描述][1]

4.把文件提交到当前分支，-m后带提示信息
```python
$ git commit -m "wrote a readme file" 
```
![此处输入图片的描述][2]

5.获取当前仓库状态，查看哪些文件被修改过
```python
$ git status
```
6.查看修改内容
```python
$ git diff 
```
7.一行查看版本信息
```python
$ git log --pretty=oneline
```

----------


##版本回溯

转到当前版本
```python
$ git reset --hard HEAD^ 
```
回退到指定版本
```python
$ git reset --hard commit-id
```
重启git bash后查看之前的历史工作日志（主要是为了查看commit-id）
```python
$ git reflog
```

----------


##删除文件
1.要从版本库中删除该文件，那就用命令
```python 
$ git rm
```
删掉，并且
```python
$ git commit：
```
2.删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：


----------


##远程仓库

###一，新建SSH秘钥 
```python
1.$ ssh-keygen -t rsa -C "youremail@example.com"
```
2.在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。
3.登陆GitHub，打开“Account settings”，“SSH Keys”页面，然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容确认即可

需要SSH秘钥的原因
因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。（一台电脑一个秘钥）

###二，添加远程库
1，在Github上新建一个与本地git bash同名的仓库（如test）
2，
```python
$ git remote add origin git@github.com:yang003/test.git    
```
将本地仓库与Github关联，远程库的名字默认为origin

3,把本地库的所有内容推送到远程库上：
```python
$ git push -u origin master
```

用git push命令，实际上是把当前分支master推送到远程，由于远程库是空的，我们第一次推送master分支时，加上了-u参数
此后作提交只需：
```python
$ git push origin master
```
【注意】第一次使用Git的clone或者push命令连接GitHub时，会得到一个警告：
```python
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```
这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。
Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：
```python
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```
这个警告只会出现一次，后面的操作就不会有任何警告了。

###三，克隆远程库
1，在Github上新建仓库（如gitskills）
2，
```python
$ git clone git@github.com:yang003/gitskills.git
```

[备注]Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。如：：
```python
$ git clone http://github.com/yang003/test1.git （贼慢）
```

----------


##分支管理
###一.常用命令
当前分支指向当前提交，HEAD指向当前分支
1.创建分支：
```python
$ git branch <name>
```

2.切换分支：
```python
$ git checkout <name>
```
3.创建并切换分支（如dev）,-b参数表示创建并切换:
```python
$ git checkout -b dev
```


4.查看当前分支，带*表示当前分支
```python
$ git branch 
```

5.合并某分支到当前分支
```python
$ git merge dev 
```

6.删除分支：
```python
$ git branch -d dev
```

###二. 解决冲突
当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。
用$ git log --graph命令可以看到分支合并图。
```python
$ git log --graph --pretty=oneline --abbrev-commit
```
###三. 分支策略
合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
```python
$ git merge --no-ff -m "merge with no-ff" dev
```

**实际开发中的几个基本分支管理原则：**

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

####四. bug分支
当工作没完成时用来储存当前工场
```python
$ git stash：
```
获取储存的工场列表
```python
$ git stash list 
```
恢复原工场
```python
$ git stash apply 
```
清除工场缓存
```python
$ git stash drop 或 git stash pop 
```

修复bug时，我们会通过创建新的bug分支（如master下有bug，在工作分支如dev转到master分支，新建一个issue-101的bug分支,把bug改好后add和commit,之后转会master合并，然后转会工作分支dev，此时要恢复工场则要输命令git stash apply后清除缓存 git stash drop)


####五. feature分支
开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并过的分支，可以通过以下命令强行删除：
```python
$ git branch -D <name>
```


####六. 多人协作
1. 获取远程仓库的详细信息
```python
$ git remote -v 
```

2. 分支名 推送分支
```python
$ git push origin 
```

【备注】
master分支是主分支，因此要时刻与远程同步；
dev分支是开发分支，团队所有成员都需要在上面

当你从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin。

要查看远程库的信息，用git remote或者用git remote -v显示更详细的信息：
```python
$ git remote
origin
```
```python
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```
上面显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。

####七. 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上
```python：
$ git push origin master
```

如果要推送其他分支，比如dev，就改成：
```python
$ git push origin dev
```
但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

master分支是主分支，因此要时刻与远程同步；

dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；

bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；

feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！


####八. 抓取分支
当A君对一个文件进行修改并push到远程仓库后，碰巧B君也对这个文件进行修改并试图将之push到远程仓库，这时会遭到仓库“拒绝”，解决方法是：将A君修改后的文件抓取下来
```python
$ git pull 
```
解决后指定与远程分支的关联
```python
$ git branch --set-upstream dev origin/dev
```
然后再push


----------


##标签管理
tag与commit id 的比较:
tag往往是具有意义的标号如v1.1，而commit id只是一串无意义的数字

###一. 基本命令
1.打标签：git tag 默认标签是打在最新提交的commit上的
2.给指定的commit打标签,先通过以下命令查看commit id ：
```python
$ git log --pretty=omeline
```
  
  
  然后
```python
$ git tag v1.9 commit id 
```
  
3.查看标签信息:
```python
$ git show v1.0
```

4.创建带有说明的标签，用-a指定标签名，-m指定说明文字：
```python
$ git tag -a v0.1 -m "version 0.1 released" 3628164
```

5.还可通过-s用私钥签名一个标签：
```python
$ git tag -s v0.2 -m "signed version 0.2 released" fec145a
```

###二. 操作标签
1. 删除本地标签:
```python
$ git tag -d <tagname> 
```
创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

若标签已推送到远程，则先删除本地标签，后删除远程标签
```python
$ git push origin :refs/tags/v0.9
```

2. 推送标签
推送某个标签到远程
```python
$ git push origin <tagname>
```
推送全部未推送标签
```python
$ git push origin --tags
```

----------


##Github操作

###一. Fork repostory
可fork别人的仓库（如bootstrap），然后通过命令
```python
$ git clone git@github.com:yang003/bootstrap.git
```

将项目克隆到本地，自己更改后可推送到远程，此时也可pull request 请求作者采纳更改

###二. 自定义git
通过命令对git bash的界面进行个性化配置

###三. 配置别名
在敲命令时可能有人嫌某个单词难拼难记，这时可用别名简化
```python
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
......
```

###四. 搭建git服务器
1，下载搭建环境：Ubuntu或Debian
......


----------


  [1]: http://www.liaoxuefeng.com/files/attachments/001384907720458e56751df1c474485b697575073c40ae9000/0
  [2]: http://www.liaoxuefeng.com/files/attachments/0013849077337835a877df2d26742b88dd7f56a6ace3ecf000/0