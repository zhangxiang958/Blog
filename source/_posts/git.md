title: git 小结
date: 2017-11-25 00:00:00
categories: git

---
git 的使用小结. 我写别的东西去了, Blog 还是要更的.
<!--more-->
## git
常用的 git 命令:
```
git clone addr 克隆项目到本地
git init 初始化 git 仓库
git add  添加修改文件到工作区
git commit 将工作区的修改移到缓存区, 并打上信息
git push 将缓存区的代码修改提交到远程仓库中
```
添加文件到工作区:
```
git add .
git add -A/--all
git add --patch filename
```
git add . 与 git add -A 的区别在于作用的目录不同, git add -A 是将整个仓库的所有修改与新文件提交到暂存区, git add . 是将当前命令行所在的目录下修改的文件提交到工作区,
父目录的修改则不会提交.
git add --patch 是交互式命令.可以使用命令, 将一些修改, 分成小部分地提交到缓存区.

查看 commit:
```
git show commit_hash 查看 commit 的详细信息
```

提交 commit:
```
git status 查看当前 git 的状态
```

分支管理:
```
git branch 列出所有分支
git checkout 切换分支
git checkout -b branch_name 新建分支并切换到新的分支上
git push --set-upstream remote_name branch 将本地分支的修改上传到远程对应仓库的分支上.
git merge branch2 将 branch2 的修改合并到当前分支.
git push --delete remote_name branch 将远程分支移除
git branch --delete branch_name 将本地分支移除
```

远程仓库:
```
git remote add remote_name addr 为当前仓库添加远程仓库地址.
git pull 将远程仓库的 commit 拉取下来, 并自动 merge 当前的分支, 可以说是 git fetch 与 git merge 的合体
git fetch 拉取远程的 commit
git merge 合并 commit
```
日志:
```
git reflog
git log
```
两者区别在于 git log 是记录仓库从 git init 开始的所有记录, git reflog 是从你 git clone 开始记录的, 相对来说 git reflog 会个性化一点.

对于下面的操作, 画图会更容易理解:
撤销提交:
```
git reset --hard commit_hash  将 head 指针回退到指定的 commit 上, 然后删除之前提交的修改
git reset --soft commit
git reset (--mixed) commit
```
三者的区别在于他们对于目录的重置. --hard 最危险, 会让历史, 暂存区, 工作区全部重置并且极有可能找不回.
--soft 是重置历史库, 不重置暂存区与工作目录

## git tip
为什么不用 git add . ? 而是用那种改一个添加哪一个的方式
一般来说, 在多人协作的时候, 很多人会不推荐使用 git add ., 而是针对修改过哪个文件去添加那个文件. 其实如果熟悉 git 来说 git add .是没有问题的.
问题就是出现问题 git add . 会很难恢复.而且万一项目中有子模块, 就会将子模块的 commit 提交上去了.保持 commit 的粒度很细是一个好习惯.
需要保持在 git add 之前用 git status 查看一下状态是好事.
为什么提交的时候需要 git submodule update ?
因为如果你不小心 git add . 了, 然后项目中又有子模块, 这个时候是很容易将子模块的 commit 记录弄错了, 所以提交之前, update 一下, 保证子模块的记录最新就没问题了. 
什么是 revert, 为什么 revert 会有坑? 会有什么坑?
revert 是用一个 commit 来回退到上一个 comit 的情况.目前还没有研究到 revert 会有哪些坑, 但是会有麻烦.
什么是 cherry-pick ?
cherry-pick 是将某个 commit 在重演一遍, cherry-pick 后面可以接 1 个到多个 commit hash, 最先提交的 hash 放在前面.
团队合作，用好以下几条命令 
rebase 
cherry-pick 
还有一根救命稻草 reflog