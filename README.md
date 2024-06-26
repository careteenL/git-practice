## git-practice
git-practice


### 分布式 VCS 的优缺点：

#### 分布式 VCS 的优点：

- 大多数的操作可以在本地进行，所以速度更快，而且由于无需联网，所以即使不在公司甚至没有在联网，你也可以提交代码、查看历史，从而极大地减小了开发者的网络条件和物理位置的限制（例如，你可以在飞机上提交代码、切换分支等等）；
- 由于可以提交到本地，所以你可以分步提交代码，把代码提交做得更细，而不是一个提交包含很多代码，难以 review 也难以回溯。

#### 分布式 VCS 的缺点：

- 由于每一个机器都有完整的本地仓库，所以初次获取项目（Git 术语：clone）的时候会比较耗时；
- 由于每个机器都有完整的本地仓库，所以本地占用的存储比中央式 VCS 要高。

### HEAD、master、branch

- HEAD 是指向当前 commit 的引用，它具有唯一性，每个仓库中只有一个 HEAD。在每次提交时它都会自动向前移动到最新的 commit 。
- branch 是一类引用。HEAD 除了直接指向 commit，也可以通过指向某个 branch 来间接指向 commit。当 HEAD 指向一个 branch 时，commit 发生时，HEAD 会带着它所指向的 branch 一起移动。
- master 是 Git 中的默认 branch，它和其它 branch 的区别在于：
新建的仓库中的第一个 commit 会被 master 自动指向；
在 git clone 时，会自动 checkout 出 master。
- branch 的创建、切换和删除：
    - 创建 branch 的方式是 git branch 名称 或 git checkout -b 名称（创建后自动切换）；
    - 切换的方式是 git checkout 名称；
    - 删除的方式是 git branch -d 名称，不能删除当前分支
        出于安全考虑，会报错`error: the branch 'xx' is not fully merged`
    - 强制删除的方式是 git branch -D 名称。

### push本质

- push 是把当前的分支上传到远程仓库，并把这个 branch 的路径上的所有 commits 也一并上传。
- push 的时候，如果当前分支是一个本地创建的分支，需要指定远程仓库名和分支名，用 git push origin branch_name 的格式，而不能只用 git push；或者可以通过 git config 修改 push.default 来改变 push 时的行为逻辑。
- push 的时候之后上传当前分支，并不会上传 HEAD；远程仓库的 HEAD 是永远指向默认分支（即 master）的。

### merge:合并commits

- merge 的含义：从两个 commit「分叉」的位置起，把目标 commit 的内容应用到当前 commit（HEAD 所指向的 commit），并生成一个新的 commit；取消merge：`git merge --abort`
- merge 的适用场景：
    - 单独开发的 branch 用完了以后，合并回原先的 branch；
    - git pull 的内部自动操作。
- merge 的三种特殊情况：
    - 冲突
        - 原因：当前分支和目标分支修改了同一部分内容，Git 无法确定应该怎样合并；
        - 应对方法：解决冲突后手动 commit。
    - HEAD 领先于目标 commit：Git 什么也不做，空操作；
    - HEAD 落后于目标 commit：fast-forward。

### feature branch :最流行的工作流

- 每个新功能都新建一个 branch 来写；
- 写完以后，把代码分享给同事看；写的过程中，也可以分享给同事讨论。另外，借助 GitHub 等服务提供方的 Pull Request 功能，可以让代码分享变得更加方便；
- 分支确定可以合并后，把分支合并到 master ，并删除分支。

- [合并特定commits到另一个分支](https://blog.csdn.net/ybdesire/article/details/42145597)
    - 合并特定commit：`git cherry-pick specialhash`
    - 合并多个commit：
        - 切换到一个指明最后一次commit的新分支`git checkout -b new-branch special-hash`
        - 然后rebase这个新分支到master，special-hash^可以指明从某个特定的commit开始`git rebase --onto master special-hash^`


### add
```bash
git add -A //stages All
git add . //stages new and modified, without deleted
git add -u //stages modified and deleted, without new
git reset HEAD <file>... //to unstage
git checkout -- <file>... //to discard changes in working directory
```

### 查看自己的修改

- 查看详细历史`git log -p` -p为--patch缩写
- 查看简要统计`git log --stat`
- 查看某个commit的具体修改`git show special-hash`
- 查看某个commit某个文件的具体修改`git show special-hash file-name`
- 比对暂存区和上一条commit的区别`git diff --staged`或者--cached
- 比对工作目录和暂存区的区别`git diff`
- 比对工作目录和上一条commit的区别`git diff HEAD`,新建的文件没有被追踪，所以是看不到工作目录新建文件和commit的区别

### rebase 变基？
```shell
git rebase master
```
```shell
careteenL modified

ketingwang modified agin
# 第一次修改
# 第二次修改
```

### 刚刚提交的代码，发现写错了怎么办？

利用`--amend`参数进行修正

注：并不是修改上一次commit，而是生成新的commit取代上一次commit。
```shell
git commit --amend
```
![git_commit--amend](./imgs/git_commit--amend.png)

### 写错的不是最新的提交，而是倒数第二次提交？

- 使用方式是 git rebase -i 目标commit；
- 在编辑界面中指定需要操作的 commits 以及操作类型；
- 操作完成之后用 git rebase --continue 来继续 rebase 过程。
![git_rebase-i](./imgs/git_rebase-i.png)


### 比错还错，想直接取消刚才的提交？
```shell
git reset --hard 目标commit
git reset --hard HEAD^
```

### 想丢弃的不是最新的提交？

> 方式一：

```shell
// git rebase -i 目标commit
git rebase -i HEAD^^
// 进入交互页面编辑删除 想丢弃的commit即可

// 然后继续操作
git rebase --continue
```
![git_rebase-i-another](./imgs/git_rebase-i-another.png)

> 方式二：

```shell
// git rebase --onto 目标commit 起点commit 终点commit
git rebase --onto HEAD^^ HEAD^ master
```
![git_rebase--onto](./imgs/git_rebase--onto.png)

### 代码push上去了才发现写错了？

> 场景一：出错的提交在自己的分支

```shell
// git rebase -i 目标commit
git rebase -i HEAD^^
// 进入交互页面编辑删除 想丢弃的commit即可

// 然后继续操作
git rebase --continue

//git push origin 当前分支 -f -f为--force简写，不解决冲突强制提交。
git push origin branch1 -f

```

> 场景二：出错的提交在master

此时不能像场景一强制提交，因为master分支可能存在同事的push，强制提交会将他们的提交内容抹掉

使用revert将错误撤销 ........
```shell
// git revert 目标commit
git revert HEAD^
git add .
git commit -m 'xxx'
git push
```

### 线上紧急热修复可使用 worktree ？

> 场景：当你正在 feature 分支开发新功能，线上版本紧急错误需要你基于 master 分支做修复上线

#### 解法 1

先将本地代码提交至 feature 分支，然后再切到 master 分支做热修复

```shell
# 当前在 feature
git add .
git commit -m '暂存 feature 代码'
git push
git checkout master
# 修改master 代码并 push
# 再切回来继续开发
git checkout feature
```

> 缺点：若当前 feature 分支正在运行耗时比较长的功能测试，需要等待热修复后再切换重新运行

#### 解法 2

先将本地代码 stash，然后切到 master 热修复后，再回来 pop

```shell
# 当前在 feature
git stash
git checkout master
# 修改master 代码并 push
# 再切回来继续开发
git checkout feature
git stash pop
```

> 缺点：同解法 1

#### 解法 3

clone 一份 master 代码到本地其他目录下

```shell
git clone https:xxx
# 修改master 代码并 push
```

> 缺点：若仓库比较大，耗时久

#### 解法 4

使用 worktree 另开一个独立工作区

> 实例代码在 ./worktree.js 以及 worktree-feature 分支

```shell
# 当前在 feature
# 再当前项目同目录下基于 master 分支新开 hotfix 目录
git worktree add ../hotfix master
cd ../hotfix
# 修改master 代码并 push

# 然后再回到当前目录下继续 feature 分支开发
cd ../current-project
```

> 优点：hotfix 与 feature 分支互不干扰
