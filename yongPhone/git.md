### 获取帮助文档
- git help <verb>
- git <verb> --help
- man git-<verb>

### 初始化git仓库
- git init

### add和commit
1. git add + file

git add -A | git add . | git add -u
---|---|---
new/modified/deleted files | new/modified files | modified/deleted files
++git add 目录 ，会添加当前文件夹内的所有文件，并且是递归式添加（包括其子文件夹内的文件）++

2. git commit -m "说明"  
直接使用git commit会启动编辑器，输入说明内容，下面注释行会有git status的输出，可以加上-v 参数会丢掉这些注释行 

3. 跳过使用暂存区域直接commit  
git commit -a -m '说明'

### 查看当前仓库状态
- git status
    - -s 参数，状态简览
        - A ：新添加到暂存区的文件
        - MM ： 右边的是修改了还未add，左边的是修改了也add了
        - ?? ：新添加的未跟踪文件

### 查看具体被修改的内容(工作区-暂存区)
- git diff [file]  
此命令比较的是工作目录中++当前文件++和++暂存区域++快照之间的差异

### 查看具体被修改的内容(暂存区-版本区)
- git diff --staged
- git diff --cached

### 查看具体被修改的内容(工作区-版本库)
- git diff HEAD -- file

### 版本历史记录
- git log  
参数：   
    - -p 显示每次提交的内容差异  
    - --stat 在每次提交下面列出修改过的文件信息和总结  
    - --shortstat 只显示--stat中最后的行数修改添加移除统计  
    - --graph 分支合并历史图
    - --pretty=oneline（一行显示）  
    pretty还有oneline,short,full,fuller这些子选项可以用,format作为子选项可以定制要显示的记录格式  
    eg：git log --pretty=format:"%h - %an,%ar :%s"    
    - --abbrev-commit 仅显示SHA-1的前几个字符  
    - -name status 显示新增、修改、删除的文件清单  
    - --relative-date 相对时间显示
    - 限制git log输出
        - -(n),仅显示最近的n条记录
        - --since和--after,仅显示指定提交之后的提交
        - --until和--before，仅显示指定时间之前的提交
        - --author仅显示该作者的提交
        - --committer 仅显示指定提交者的提交
        - --grep 仅显示含指定关键字的提交
        - ==-S==string仅显示添加或者移除了某个字符串的提交


### 版本回退
- git reset --hard 版本号  
++版本号写前几位就好，HEAD^表示上个版本++

### 提交或者版本回退操作记录
- git reflog

### 撤销操作
- 漏了文件没有commit：把它add了之后，git commit --amend，第二次提交代替第一次提交的结果
- 撤销暂存的文件：git reset HEAD <file>  
++工作区文件还是和撤销前工作区文件内容保持一致++
- 丢弃文件在工作区的修改：git checkout -- file，相当于拷贝了暂存或仓库的文件覆盖它（拷贝暂存优先）

### 删除文件
1. 在工作去直接rm file删除
2. git rm file 可以把它的删除记录到暂存区，然后再commit完成删除
3. 如果删除之前修改过文件并放入暂存区的话，则必须加上强制删除选项-f
4. 从git仓库中删除，文件保留在当前工作目录，git rm --cached file

### 文件改名
- git mv file_from file_to  
- mv file_from file_to  
  git rm file_from  
  git add file_to

### 远程仓库
1. 创建SSH Key。（如果在用户主目录下有.ssh目录可以直接跳到下一步）  
ssh-keygen -t rsa -C "邮箱"
2.在github的Account Settings，SSH Keys，添加SSH Key，在文本框黏贴id_rsa.pub文件的内容

### 本地仓库推送到github
- git remote add origin git@github.com：yongPhone/xxx.git  
++git默认吧远程仓库叫做origin++

### 把某条分支所有内容推送到远程
- git push -u origin 分支名称  
++加上了-u参数，不但会把本地内容推送到远程，而且会把本地的某条分支和远程的对应分支关联起来，以后推送和拉取可以简化命令为git push origin master++

### 远程仓库
- 查看远程仓库：git remote
    - -v,会显示需要读写远程仓库使用的git保存的简写与其对应的url
- 添加一个新的远程git仓库：git remote add <shortname> <url>
- 获取别的仓库有但是我没有的信息：git fetch [remote-name],但是并不会自动合并当前的工作
- 查看远程仓库：git show [remote-name]
- 移除远程仓库：git remote rm [remote-name]
- 重命名远程分支：git remote rename [old-remote-name] [new-remote-name]

### 克隆远程库
- git clone git@github.com:yongPhone/xxx.git [new name]

### 新建并切换分支
- git checkout -b 分支名称（新建并切换）
- 1. ++新建分支++:git branch 分支名称
  2. ++切换分支++:git checkout 分支名称

### 查看（当前）分支
- git branch

### 查看分支合并情况
- git log --graph --pretty=oneline --abbrev-commit  
- git log --graph 可以查看分支合并图

### 删除分支
- git branch -d 分支名称
- git branch -D 分支名称 ，强制删除

### 把当前dev分支合并到master分支
（Fast forward模式，删除分支后会丢失分支信息）
1. 切换到master分支
2. git merge dev

### --no-ff方式合并分支
（会生成一个新的commit）
git merge --no-ff -m "commit的说明" + 分支名称

### 解决冲突
（两个分支都修改并add同文件，在commit的时候会有冲突）
- git status 可以查看冲突文件
- 再次修改文件后commit

### 工作现场"储藏"
- git stash 储藏现场
- git stash list 查看被储藏的现场
- 1. git stash pop 恢复到最近的现场并删除stash的内容
  2. git stash apply stash@{0} 恢复特定的现场  
     git stash drop 删除stash的内容

### 创建远程分支到本地
默认情况下，从远程clone下来的只能看到master分支，假设要在dev分支开发，就要创建远程分支到本地：  
- git checkout -b dev origin/dev

### 向远程推送失败，冲突
假设合并dev分支  
1. git branch --set-upstream dev origin/dev   
指定本地dev分支与远程origin/dev分支链接（没链接才需要）
2. git pull  
把最新的提交从origin/dev抓下来，然后在本地合并，解决冲突再推送
3. 如果git pull合并不成功，就解决冲突，在本地提交，然后推送

---
## 标签管理
- git tag 标签名，对当前分支的最新commit打标签
    1. 可以用-a tag_name参数指定标签名
    2. 可以用-m “claim”，指定说明文字
    3. 可以用-s tag_name参数私钥签名一个标签（需安装gpg）
- git tag tag_name commit_id ，对某次提交打标签
- git tag ，查看所有标签
- git show tag_name，查看标签信息
- git tag -d tag_name，删除一个标签
- git push origin tag_name，推送一个标签到远程
- git push origin --tags，推送全部尚未推送的标签到远程
- ++删除远程标签++  
  1. git tag -d tag_name，删除本地标签
  2. git push origin ：refs/tags/tag_name，从远程删除标签
- 检出分支：git checkout -b [branchname] [tagname],git标签不能像分支一样来回移动，想要工作目录与仓库中特定的标签版本完全一样，可以使用以上命令，但是如果在此之后又一次提交，之前指定的分支就会向前移动，那么就要小心他们不一样了

---
## 忽略特殊文件
工作区根目录新建.gitignore文件，并提交到git  
内容：  
\# Windows:  
Thumbs.db  
ehthumbs.db  
Desktop.ini   

\# Python:  
\*.py[cod]  
\*.so  
\*.egg  
\*.egg-info  
dist  
build  

\# My congiguartions:

---
## 配置
- 配置别名：git config --global alias.newword oldword
- 高亮：git config --global color.ui true
- NAME: git config --global user.name "my_name"
- E-MAIL: git config --global user.email "my_email"
- 外部命令别名:git  config --global alias.visual '!外部命令'
---

## 搭建git服务器


---
## ELSE
- 版本库：工作区的“.git”目录
    - stage（或叫index）暂存区
    - 第一个分支master，指向master的指针是HEAD
    
- .gitignore格式规范：
    - 空行和#的开头行都会被忽略
    - 使用标准的glob模式匹配
        - \* 0或多个任意字符
        - [abc] 方括号中任意一个字符
        - ? 一个任意字符
        - [0-9] 范围
        - a/\*\*/z 两个星号匹配任意中间目录
    - 以/开头防止递归
    - 以/结尾指定目录
    
- 对待数据的方法：直接快照记录，不同于svn的记录差异比较

- 校验和：
    - 计算机制：SHA-1散列（hash），40个16进制字符，基于git中文件内容或者目录结构计算
    - 保证数据完整性：每次存储前都计算校验和，并且以校验和来引用（索引）  

- 文件三种状态
    - modified
    - staged
    - committed

```
sequenceDiagram
工作区->>暂存区:staged修改
暂存区->>本地仓库: commit
本地仓库->>工作区: checkout
```