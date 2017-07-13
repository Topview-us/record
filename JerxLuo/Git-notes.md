# 7月12号总结

## Git

### 1.版本库创建和初始化

```nginx
mkdir learngit
cd learngit
git init

```

**Git跟踪文件的改动，不能跟踪富文本格式，而只能跟踪纯文本格式的文件和图片视频**

### 2.添加文件告知仓库

1. 文件修改
2. 命令：`git add <filename>`通知仓库，文件的修改
3. 命令：`git commit -m "..."`提交仓库

### 3.版本回退

这是将当前指针（此时指向当前分支最近一次commit_id）指向其他版本，撤回也要用此命令

`git reset --hard  <commit_id/HEAD^/HEAD~数字>`

撤回时需要辅助命令
`git status`
查看当前状态

`git log --pretty=oneline --abbrev-commit`
查看提交历史（指的是整个仓库）

`git reflog`
查看commit和reset和分支切换命令历史

### 4.暂存区和工作区和最新版本库版本

| 工作区版本 |            | 暂存区版本 |                     | 最新版本库版本 |
| ----- | ---------- | ----- | ------------------- | ------- |
|       | `git diff` |       | `git diff --staged` |         |
|       |            |       |                     |         |

​                                      `git diff HEAD ---<filename>`


#### `git status`

会显示两个信息：

(1) 工作区未跟踪的
(2)暂存区已跟踪但未提交的

### 5.撤销修改

`git checkout <filename>`撤销工作区的内容，回到最近一次跟踪状态的版本（可能是暂存区版本也可能是最新版本库版本）

`git reset HEAD`撤销暂存区跟踪但未提交的内容，回到最近一次最新版本库2版本

### 6.删除文件

这是指手动删除了某个文件后

(1)告知仓库删除信息：`git rm <file>`
(2)从仓库找回这个文件（误删）`git checkout <file>/git checkout`（前提是(1)操作未执行）

### 7.分支管理

#### (1)创建分支

`git branch <branchname>`

#### (2)切换分支

`git checkout <branchname>`

#### (3)合并分支(将当前分支与指定分支合并)

`git merge <--no-ff> -m "..." <branchname>`
普通合并模式：`--no-ff`
快进模式：默认,是直接将指针指向最新版本

#### **(4)解决冲突**

分支1和分支2都修改了文件a，且都有新的提交
此时切换到分支1（分支2也类似），执行与分支2合并的命令，结果显示，文件a有冲突，而且仓库会在文件a将冲突情况解释清楚，等待你的修改然后再提交（此时是MERGING状态）
(5)临时分支的使用
常见情况下临时分支用于解决bug等，解决完之后，我们会将在**主分支与该分支合并**，然后切换到自己的工作分支继续工作。这里衍生两个问题：自己的工作分支工作现场尚不能提交，需要被另存；在临时分支工作结束后回到自己的工作分支要找回工作现场，在这之前是否要再次与主分支合并。
a.`git stash`储藏工作区（未跟踪）、暂存区（已跟踪未提交）的内容
b.从临时分支结束回到自己的分支：

- `git merge master --no-ff -m "update"`

- `git stash pop`把stash储藏区的内容都装载回工作区，并且清空储藏区，`git stash list `查看储藏区内容，`git stash apply`同pop但储藏区不会清空，需要`git stash drop`

  #### (5)多人协作

  1.查看远程库信息

  ```
  git remote -v
  ```

2. 分支推送

   ```
   git push origin <branchname>
   ```

3. fetch分支
   本地仓库没有对应分支：`git checkout -b <branchnaem> origin/<branchname>`
   本地仓库有对应分支(但是没有设置两对应分支的关联)：`git branch --set-upstream <branchname> origin/<branchname> `

   ### push and clone

   - `git pull `
   - 本地推送到远程（第一次推送需要-u参数）`git push <-u> origin <branchname>`
   - 添加远程库（远程库已经存在）`git remote add origin http://github.com/JerxLuo/learngit.git`

   ​    （第一次使用clone和push命令会有警告）

   - `git clone <address>`

   ### 标签管理

   标签添加查看

   - `git tag -a/s <tagname> -m "..." <commit_id>`
   - `git tag `
   - `git show <tagname>`
     标签删除
     - 本地标签 	`git tag -d <tagname>`	
     - 远程标签 `git push origin :refs/tags/<tagname>`
   ###忽略特殊文件
   有些文件需要被忽略
   创建.gitignore文件（每行写上一个文件名，要忽略的文件）
   就可以了= =
