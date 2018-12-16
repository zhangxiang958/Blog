title: Git 小手记
date: 2018-01-27 21:24:24
categories: git

---
记录一下日常的 git 使用与我平时用的 git 小窍门.
<!--more-->
## 关于 rebase
### 为什么不能在 master 上做 rebase 操作?
rebase 与 merge 操作是同一类的操作, 都是用于合并的, 但是对比起 merge, rebase 会显得更危险, 同时还有个优点就是可以保持线性的干净的历史提交记录.
使用 rebase 的一条黄金法则就是不要在公共分支上做 rebase 操作, 为什么呢?
核心的原因在于 rebase 会将需要移动的 commit hash 重新生成一遍. 
rebase 的本质是将需要衍合分支上的 commit 从与当前分支最近祖先 commit 起的所有 commit 重新在当前分支做一遍.
既然是重新做一遍, 那么 commit hash 肯定会改变的.这个操作就很不妙, 因为比如说你是在 master 分支上做 rebase, 
通俗地说, 你是在将 master 上的 commit 移到 feature 分支的后面, 也就是说祖先元素之后的 master 分支上的 commit 都会重新生成一遍, 虽然做的事情, 提交时间, 作者相同, 但是 commit 
hash 值是不同的. 这样的操作在公共分支上是非常危险的, 比如原本 master 上的 A commit 很多人基于这个 commit 开展了很多工作, 然后你在 master 分支上做了 rebase, 
A 就变成了 A', 之前的 A 就不见了, 这样别人在开发的时候就会非常困惑.
另外, 由于 commit hash 值不同了, 但是别人或者远程仓库都保留着之前的 A commit, 改变的只是你自己本地 master A commit, 变成了 A', 所以在你提交的时候, 因为 commit hash 已经
不同, git 会认为这是一个新的提交, 然后像新的 commit 一样提交到仓库里面, 这样可以说 master 分支已经分叉了, 因为别人的 master 分支和你的已经不一样了, 而且即使没有冲突发生,
当你查看 git 历史的时候会发现很困扰就是 A 和 A' 做的操作, 内容, 时间, 操作者都是一样的, 但是就是 commit hash 不同被看作两个 commit, 这样 git 历史其实已经混乱了, 而且后续
别人基于这样的历史进行开发并不能担保不会出现问题, 因为本身历史就是乱套的.所以这就是为什么不要在公共分支上做 rebase 操作.
那么问题又来了, 所谓的公共分支是指什么, 多人协作就是公共分支吗? 
并不是, 我觉得公共分支是指共同的主分支, 会有很多协作分支的主分支才是公共分支, 假如你有一个 feature 分支并在上面开发,
你还有其他同事一起在这个分支上开发, 这个时候 feature 并不能算公共分支, 而相反在这样的 feature 分支上做 rebase 是被提倡的, 比如同事新提交了几个 commit, 你也有几个分支需要提交,
那么你只需要 git fetch, 然后 git rebasse, 意思就是将你的 commit 移到远程的同事已经提交的 commit 后面, 对没错, 我说的就是使用 git fetch + git rebase 代替 git fetch + 
git merge, 也就是常用的 git pull. 这样做的好处是保证分支的历史的整洁性, 保持线性提交记录, 保持清爽.

### rebase --onto
在实际的协同开发中, 我们可能会面对的并不是一个分支, 很可能是很多并行的分支. 例如我们在开发中正在开发的一个新的功能, 但是突然发现线上代码有 bug,
所以需要为线上的 master 分支切出一个 bugfix 分支, 但是由于粗心, 我们并没有切换到 master 分支再切出一个新的分支, 而是从当前的 feature 分支上
切出了一个分支, 
### rebase -i
rebase 后面加上 -i 参数, 其实是交互式的 rebase 命令.它可以可以修改 commit 信息, 顺序, 合并 commit.

## cherry-pick
开发中常会有这样的场景, 比如你在开发 3.10 版本的功能, 但是同时 3.9 版本正在发布, 但是有几个小的简单的功能产品需要提前上线(可能并不会这么荒谬), 
这个时候, 也就是说, 需要将 3.10 分支上的代码上的某些修改提前交到 3.9 分支上.这时你需要 cherry-pick.
cherry-pick 会重演某些 commit, 就是将某些 commit 重新在某些分支上执行一遍.就像这样, 你先需要在需要上线的分支切出一个分支出来, 这里是 dev-3.9:
```
git checkout dev-3.9
git checkout -b dev-3.9.1
```
然后你可以 checkout 到你需要查找的 commit 所在的分支上:
```
git checkout dev-3.10
```
然后通过 git log 来查看你需要的一个或者是多个 commit 的 hash 值, 并把他们记录下来, 然后通过 cherry-pick 来重演:
```
git cherry-pick commitHash1 commitHash2 commitHash3 ....
```
多个 commit hash 之间用空格隔开, 然后 commit 的顺序最好是按照时间顺序进行排列, 最先提交的也就是按照时间先后进行排列.
因为每一个 commit hash 是特殊的, 所以你不用担心另一个分支的 commit 能不能在这个分支上被 pick 过去, git 会根据 hash 找到这个对应的 commit 进行 pick.
而且不仅可以在不同分支做 pick, 同一个分支也可以进行 pick.比如你曾经 reset 了某个 commit, 你想把它找回来, 可以使用 pick, 至于这个 commit 怎么找回来就需要 git reflog 了.

## merge --squash
git merge --squash 是合并 merge 操作.遇到的场景是你需要在 deve 分支上修复某个 bug, 你需要切出一个 bugfix 分支, 在 bugfix 分支上修复这个 bug, 但是这个 bug 你会在分支上提交
多个 commit(保持 commit 的原子性), 但是到最后合并到 deve 分支上的时候, 为了保持清爽的提交历史, 你可能会需要 git merge --squash, 你可能会有:
```
添加文件
添加文件2
添加文件3
```
这样的 bugfix 提交, 如果将这些合并到 deve 会显得有点乱, 所以使用 git merge --squash 让 bugfix 分支上的提交合并为一个提交(fix xxx bug)然后合并到 deve.
## git add -p
所谓 -p 其实是 --patch, 也就是块级, 补丁的意思.git add -p 可以交互式地, 单文件内选择性提交.我们会经常遇到这样的场景, 也就是我们在单个文件里面一连修复了很多个 bug, 但是我们
忘记逐个 bug 进行提交记录, 但是如果直接将整个文件进行提交, 我们将不能保持 commit 的原子性, 这个时候就需要 git add -p:
```
git add -p filename
```
输入这个命令就会进入一个交互式界面, 可以看到下面有命令的选择:
```
Stage this hunk [y,n,q,a,d,/,s,e,?]?
```
上面的命令就是为了对文件的修改区域进行交互式选择提交的:
```
y: 缓存该块
n: 不缓存该块
q: 退出
a: 缓存当前块与其之后所有块
d: 不缓存当前块与其之后所有块
/: 搜索与某个正则匹配的块
s: 将块进行粒度更细的划分
e: 进入编辑模式
?: 显示帮助信息
```
如果对当前块进行 y 命令操作, 你可以看到文件已经进行了一次加入到缓存区的操作了.
## 区分 reset 的几个参数
reset 有三个参数:
```
1. soft, 将历史库回退, 但是缓存区与工作区不回退.
2. hard 历史记录, 缓存区, 工作区全部回退.
3. mixed 将历史记录与缓存区回退, 但是工作区不回退, mixed 是默认参数.
```
## git reflog
git reflog 堪称 git 协同工作的最后一根救命稻草. git reflog 是保存在本地的, 记录各个分支 HEAD 指针指向情况的.而 git log 是记录当前 HEAD 指针的指向与其的祖先, 是一个递归的结构.
git reflog 是记录 HEAD 指针曾经指向过的历史, 它是撤销操作历史库, 它并不会包括在仓库中, 它只存在于本地.也就是说, 一旦你的修改被提交到缓存区, 即 commit 过, 那么就可以通过 git reflog
找回来.


参考资料:
1. https://git-scm.com/docs/git-rebase
2. http://weblog.avp-ptr.de/20120928/git-how-to-copy-a-range-of-commits-from-one-branch-to-another/
3. https://git-scm.com/docs/git-cherry-pick
4. https://stackoverflow.com/questions/10605405/what-does-each-of-the-y-n-q-a-d-k-j-j-g-e-stand-for-in-context-of-git-p
5. https://stackoverflow.com/questions/17857723/whats-the-difference-between-git-reflog-and-log