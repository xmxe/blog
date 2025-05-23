---
title: Git
categories: 技术栈
index_img: /assert/git.jpg
img: https://pica.zhimg.com/v2-6baae9baaef9c2a3188f9fc301833fba_1440w.jpg

---

## 常用命令

### git diff

```shell
# 可以查看当前没有add的内容修改（不在缓冲区的文件变化）
git diff
# 查看已经add但没有commit的改动（在缓冲区的文件变化）
git diff --cached
# 是上面两条命令的合并
git diff HEAD
# 查看文件和另一个分支的区别
git diff branch-name file-name
# 查看两次提交的区别
git diff commit-id commit-id
git show commit-id
```

### git reset

```shell
# 将未commit的文件移出Staging区(撤销git add)
git reset HEAD
# 撤销git add操作(已经将文件添加到暂存区)不加filename就是上一次add里面的全部撤销了
git reset HEAD filename
# 重置代码和远程分支代码一样
git reset --hard origin/master
# 回退到上个commit 删除工作空间改动代码，撤销commit，撤销git add
git reset --hard HEAD^
# 回退到前3次提交之前，以此类推，回退到n次提交之前
git reset --hard HEAD~3
# 回退到指定版本 --hard强制将暂存区和工作目录都同步到你指定的提交
git reset --hard commitid
# 不删除工作空间改动代码，撤销commit，不撤销git add
git reset --soft HEAD^
# 不删除工作空间改动代码，撤销commit，并且撤销git add操作。这个为默认参数,git reset --mixed HEAD^和git reset HEAD^效果是一样的。
git reset --mixed HEAD^
```

### git revert

> 应用场景：有一天测试突然跟你说，你开发上线的功能有问题，需要马上撤回，否则会影响到系统使用。这时可能会想到用reset回退，可是你看了看分支上最新的提交还有其他同事的代码，用reset会把这部分代码也撤回了。由于情况紧急，又想不到好方法，还是任性的使用reset，然后再让同事把他的代码合一遍（同事听到想打人），于是你的技术形象在同事眼里一落千丈

```shell
# 回退到指定版本。与git reset --hard commitid相比区别是在我们确认了在需要回退的版本之后的提交都可以不需要的时候，我们可以直接使用git reset命令，但是当我们只是需要撤销某个版本的时候，后面的提交还需要保留，我们就可以使用git revert.revert会生成一条新的提交记录，这时会让你编辑提交信息，编辑完后:wq保存退出就好了。-n/--no edit：这个选项不会打开文本编辑器。它将直接恢复上次的提交。
git revert -n commitid
# 撤销前一次commit ,此次操作之前和之后的commit都会被保留，并且会把这次撤销作为一次最新的提交
git revert HEAD
# 撤销前前一次commit
git revert HEAD^
```

### git commit

```shell
# (git add -u + git commit -m ""组合)
git commit -am ''
# 修改commit信息
git commit --amend
# 使用新的一次commit，来覆盖上一次commit
git commit --amend -m "message"
# 提交Staging中在指定文件到本地仓库区
git commit file1 file2 -m "message
# 修改上次提交的用户名和邮箱
git commit --amend --author="name <email>" --no-edit
```

### git add

```shell
git add -A # 保存所有的修改
git add . # 保存新的添加和修改，但是不包括删除
git add -u # 保存修改和删除，但是不包括新建文件。
```

### git branch

```shell
# 查看本地分支(不包括远程分支)
git branch
# 查看远程分支
git branch -a
# 列出本地所有分支 并显示最后一次提交的哈希值
git branch -v
# 在-v的基础上 并且显示上游分支的名字
git branch -vv
# 列出上游所有分支
git branch -r
# 创建分支(未切换)
git branch <name>
# 删除分支，不能删除当前所在分支
git branch -d <name>
# 设置分支上游
git branch --set-upstream-to origin/master
# 下载指定分支
git clone -b 分支名称 git地址
# 为了节省磁盘空间，您可以使用以下命令克隆仅导致单个分支的历史记录,如果--single-branch未添加到命令中，则所有分支的历史记录将被克隆。大型存储库可能会出现此问题。
git clone -b <branch_name> --single-branch <url>
```

### git checkout

```shell
# 切换分支
git checkout <name>
# 创建+切换分支
git checkout -b <name>
# 本地没有分支的情况下拉取远程分支,此命令意思为创建本地分支并关联远程分支
git checkout -b local-branch origin/remote-branch
# 丢弃某个文件工作区的修改（还原修改过的文件）
git checkout -- file
# 放弃本地所有修改，没有提交的可以回到未修改前版本，不包括新增删除的文件
git checkout .
# 放弃本地所有修改，包括新增删除的文件
git checkout . && git clean -df
#（先切换到develop分支然后把feature分支合并到develop分支）
git checkout develop && git merge feature
# 将某个文件回滚到某个版本
git checkout commitid [file]
```

### git switch

> `git switch`命令是在Git 2.23.0版本中引入的，以解决`git checkout`命令职责过重的问题，并使得Git的命令更加直观和易于理解。在Git 2.23之前，`git checkout`既用于切换分支，也用于还原文件内容，很容易引起混淆。通过将`git checkout`的功能拆分，Git团队创建了两个新的、更专业的命令：
> `git switch`：专门用于在分支之间进行切换。
> `git restore`：专门用于还原文件内容。

```shell
# 切换到已存在的分支
git switch <branch-name>
# 创建并切换到新分支
git switch -c <new-branch-name>
# 从远程仓库创建并跟踪一个本地分支
git switch -c <new-branch-name> --track <remote>/<branch-name>
# 返回到上一个分支
git switch -
```

### git restore

```shell
# git restore命令在工作区是不会起作用的
# 取消暂存区的更改（类似于git reset HEAD <file>）.此命令将暂存区中的文件恢复到最后一次提交的状态，并将其从暂存区中移除，但保留工作目录中的修改。这意味着该文件的修改将不会包含在下一次的提交
git restore --staged file
# 此命令将工作目录中的文件恢复到最后一次提交的状态，并且不会对暂存区进行任何操作。它将丢弃工作目录中对文件的修改，恢复到最后提交的版本。
git restore file
# 从指定的提交中恢复文件到工作目录
git restore --source=<commit> <file>
# 恢复所有文件到指定的提交状态
git restore --source=<commit> .
# 恢复所有已删除的文件
git restore -w -- *
# 丢弃暂存区和工作目录中的更改（即恢复到指定的提交状态）
git restore --staged --worktree <file>
```

### git worktree

> `git worktree`命令是在Git 2.5版本中引入的，它允许在同一个仓库中创建多个工作目录（worktrees），每个工作目录可以检出不同的分支或提交。这为开发者提供了同时处理多个任务的能力，比如在不同的分支上进行开发、测试，而不需要来回切换分支

```shell
# 添加一个新的工作目录，并检出指定分支
git worktree add <path> [<branch>]
# 列出所有的工作目录
git worktree list
# 移除一个工作目录（必须先确保该目录没有未提交的更改）
git worktree remove <path>
# 移动一个工作目录到新的位置
git worktree move <current-path> <new-path>
# 例如，如果想要添加一个新的工作目录来检出名为feature-branch的分支，可以这样做：
git worktree add ../my-feature-worktree feature-branch
# 这将在../my-feature-worktree目录下创建一个新的工作目录，并检出feature-branch分支。
```

### git sparse-checkout

> `git sparse-checkout`是在Git 2.25.0版本中引入的，这个功能是对之前存在的稀疏检出机制的一个重大改进。通过`git sparse-checkout`，开发者可以更高效地克隆大型仓库，只检出部分文件或目录，而不是整个项目

```shell
# 要启用sparse-checkout，首先需要设置仓库以使用稀疏检出模式：
# 启用sparse-checkout模式
git sparse-checkout init
# 设置你想要包括的模式或路径
git sparse-checkout set <pattern>...
# 例如，如果只想检出src目录及其子目录中的文件，可以这样做：
git sparse-checkout set src/
# 如果想添加多个模式或路径，可以在set命令后列出所有路径，或者分多次调用该命令。除了set命令，还可以使用add和list来管理稀疏检出模式：
# 添加额外的路径到稀疏检出模式
git sparse-checkout add <pattern>...
# 列出现有的稀疏检出模式
git sparse-checkout list
# 如果不再需要稀疏检出模式，可以通过以下命令禁用它，并恢复完整的检出状态：
# 禁用sparse-checkout模式并恢复完整检出
git sparse-checkout disable
```

### git range-diff

> `git range-diff`是在Git 2.19.0版本中引入的，用于比较两个提交范围之间的差异。它可以帮助开发者理解在一次变基（rebase）、合并（merge）或历史改写操作后，一系列提交发生了哪些变化

```shell
# 比较两个分支上的最近n个提交
git range-diff A~n..A B~n..B
# 或者更常见的用法是直接指定两个范围
git range-diff A..B C..D
# 假设origin/feature是变基之前的远程分支状态,而feature是变基之后的本地分支状态
git range-diff origin/feature..feature~n feature~n..feature
```

### git maintenance

> `git maintenance`是在Git 2.30.0版本中引入的，用于管理和自动化各种维护任务的命令。这个命令旨在简化和优化仓库的维护工作，通过提供一组预定义的任务来帮助保持仓库的健康状态和高效性能。常见的维护任务包括：
> gc：运行完整的垃圾收集，包括压缩对象数据库。
> commit-graph：构建或更新提交图文件以加速提交历史查询。
> loose-objects：清理松散对象并将其打包。
> incremental-repack：逐步重新打包对象以优化存储。
> prefetch：预先获取远程分支的新数据，以加速未来的克隆和拉取操作。

```shell
# 启用自动维护
git maintenance start
# 禁用自动维护
git maintenance stop
# 执行所有配置的维护任务
git maintenance run
# 执行特定类型的维护任务
git maintenance run --task=<task>
```
配置自动维护计划：可以通过配置文件设置哪些任务应该被定期执行以及它们的执行频率。例如，在`.git/config`文件中添加如下内容：

```config
[maintenance "daily"]
    task = prefetch
    task = loose-objects
[maintenance "hourly"]
    task = commit-graph
[maintenance "weekly"]
    task = incremental-repack
[maintenance "monthly"]
    task = gc
```
然后启用这些计划：

```shell
git maintenance start --schedule=daily
git maintenance start --schedule=hourly
git maintenance start --schedule=weekly
git maintenance start --schedule=monthly
```

### git merge & git rebase
```shell
# 合并指定分支到当前分支
git merge <name>
# 将指定分支合并到当前分支
git rebase branch-name
# 执行commit id将rebase停留在指定commit处
git rebase -i commit-id
# 执行commit id将rebase停留在 项目首次commit处
git rebase -i --root
```

> **git rebase和git merge区别**
> 采用merge和rebase后，git log的区别，merge命令不会保留merge的分支的commit，rebase会保留所有的commit：rebase会把你当前分支的commit放到公共分支的最后面,所以叫变基。就好像你从公共分支又重新拉出来这个分支一样。举例:如果你从master拉了个feature分支出来,然后你提交了几个commit,这个时候刚好有人把他开发的东西合并到master了,这个时候master就比你拉分支的时候多了几个commit,如果这个时候你rebase master的话，就会把你当前的几个commit，放到那个人commit的后面。merge会把公共分支和你当前的commit合并在一起，形成一个新的commit提交
> 处理冲突的方式：（一股脑）使用merge命令合并分支，解决完冲突，执行git add .和git commit -m 'fix conflict'。这个时候会产生一个commit。（交互式）使用rebase命令合并分支，解决完冲突，执行git add .和git rebase --continue，不会产生额外的commit。这样的好处是干净，分支上不会有无意义的解决分支的commit；坏处，如果合并的分支中存在多个commit，需要重复处理多次冲突。

> **git pull和git pull --rebase区别**
> git pull做了两个操作分别是‘获取’和合并。所以加了rebase就是以rebase的方式进行合并分支得到一条干净的分支流。

> **git merge和git merge --no-ff的区别**
> 自己尝试merge命令后，发现merge时并没有产生一个commit。不是说merge时会产生一个merge commit吗？注意：只有在冲突的时候，解决完冲突才会自动产生一个commit。如果想在没有冲突的情况下也自动生成一个commit，记录此次合并就可以用：git merge --no-ff命令,如果不加--no-ff则被合并的分支之前的commit都会被抹去，只会保留一个解决冲突后的merge commit。

> [合并代码还在用git merge？我们都用git rebase！](https://mp.weixin.qq.com/s/T_8bkWI-JSP5ixdVIvVAGQ)
> [新来个技术总监，禁止我们用Git的rebase](https://mp.weixin.qq.com/s/CBz0ea6m623GtuTX5UkeQQ)

### git tag

```shell
# 查看所有标签，可以知道历史版本的tag
git tag
# 打标签，默认为HEAD。比如git tag v1.0
git tag <tagName>
# 根据版本号打上标签
git tag <tagName> commit_id
# 创建带说明的标签。-a指定标签名，-m指定说明文字
git tag -a <tagName> -m "<说明>"
# 查看标签信息
git show <tagName>
# 删除标签
git tag -d <tagName>
# 推送某个标签到远程
git push origin <tagname>
# 一次性推送全部尚未推送到远程的本地标签
git push origin --tags
# 这会将空引用推送到远程仓库，从而删除名为<tag-name>的远程标签。
git push origin :refs/tags/<tag-name>
```

### git stash

> 应用场景：某一天你正在feature分支开发新需求，突然产品经理跑过来说线上有bug，必须马上修复。而此时你的功能开发到一半，于是你急忙想切到master分支，然后你就会看到以下报错：Your local changes to the following...因为当前有文件更改了，需要提交commit保持工作区干净才能切分支。由于情况紧急，你只有急忙commit上去，commit信息也随便写了个“暂存代码”，于是该分支提交记录就留了一条黑历史，如果你学会stash，就不用那么狼狈了。你只需要：git stash就这么简单，代码就被存起来了。当你修复完线上问题，切回feature分支，想恢复代码也只需要：git stash apply,但是恢复后，stash内容并不删除，需要用git stash drop来删除

```shell
# 保存当前未commit的代码
git stash
# 保存当前未commit的代码并添加备注
git stash save "备注的内容"
# 列出stash的所有记录
git stash list
# 删除stash的所有记录
git stash clear
# 应用最近一次的stash
git stash apply
# 应用最近一次的stash，随后删除该记录
git stash pop
# 删除最近的一次stash
git stash drop

# 当有多条stash，可以指定操作stash，首先使用stash list列出所有记录：
git stash list 
# stash@{0}: WIP on ...
# stash@{1}: WIP on ...
# stash@{2}: On ...

# 应用第二条记录：
git stash apply stash@{1}
# 删除stash@{1}存储的内容
git stash drop stash@{1}
# pop，drop同理。
```

### git config

```shell
git config --gloabl http.postBuffer 524288000
git config --gloabl http.sslVerify "false"
git config --global user.name "your name"
git config --global user.email "your email"
# 保存密码
git config --global credential.helper store
# 配置git图形界面编码为utf-8
git config --global gui.encoding=utf-8
# 设置当前项目提交代码的用户名 
git config user.name name
```

### git cherry-pick

```shell
# 把a分支的一个提交复制到b分支
# 首先复制a分支的commitid然后切换到b分支执行
git cherry-pick commitid
# 一次转移多个提交：将commit1和commit2两个提交应用到当前分支。
git cherry-pick commit1 commit2
# 多个连续的commit，也可区间复制：将commit1到commit2这个区间的commit都应用到当前分支（包含commit1、commit2），commit1是最早的提交。
git cherry-pick commit1^..commit2
# 在cherry-pick多个commit时，可能会遇到代码冲突，这时cherry-pick会停下来，让用户决定如何继续操作,这时需要解决代码冲突，重新提交到暂存区,然后使用cherry-pick --continue让cherry-pick继续进行下去。
git cherry-pick --continue
# 但有时候可能需要在代码冲突后，放弃或者退出流程：放弃cherry-pick：
git cherry-pick --abort
# 回到操作前的样子，就像什么都没发生过。
git cherry-pick --quit
```

### git push

```shell
# 删除远程分支
git push origin :beanch-name
# 使用git push命令并加上--delete 选项来删除指定的远程分支
git push origin --delete branch-name
#  删除远程标签
git push origin --delete tag-name
# 删除远程标签（需要先删除本地标签）
git push origin :refs/tags/tag-name
# 上传本地仓库到远程分支
git push remote branch-name
# 强行推送当前分支到远程分支
git push remote branch-name --force
# 推送所有分支到远程仓库
git push remote --all
# 推送所有标签
git push --tags
# 推送指定标签
git push origin tag-name
# 将本地dev分支push到远程master分支
git push origin dev:master
```

### git fetch

```shell
# 更新所有跟踪的远程分支,这将会从默认的远程仓库(通常是origin)拉取所有新的数据
git fetch
# 指定一个不同的远程仓库
git fetch <远程仓库名>
# 只获取某个特定的分支而不是所有分支的更新
git fetch <远程仓库名> <分支名>
# 当你在远程删除了某些分支后，本地仍然可能保留着对这些已删除分支的引用。使用`--prune` 选项可以在获取最新数据的同时移除那些不再存在于远程仓库中的本地远程跟踪分支
git fetch --prune
# 或者针对特定远程仓库
git fetch <远程仓库名> --prune
# 获取更新后，你可以选择将这些更改合并到你的当前分支中，使用`git merge`或者`git rebase`。例如，假设你已经从origin拉取了更新，现在想要将这些更新合并到当前分支
git merge origin/目标分支 || git rebase origin/目标分支
```

## 常用操作

### 删除全部历史提交记录

```shell
# 创建孤立分支,没有以前的提交记录
git checkout --orphan <name>
# 切换到一个脱离主分支的另外一条全新主分支，不用太在意叫什么，因为后面还会修改分支名称
git checkout --orphan latest_branch
# 暂存所有改动过的文件，内容为当前旧分支的最新版本所有文件
git add -A
#提交更改
git commit -am "commit message"
#删除原始主分支
git branch -D main
#将当前分支重命名为 main
git branch -m main
#最后，强制更新您的存储库
git push -f origin main
```

### 删除部分历史提交记录

1. 运行`git log`来查找你要删除的提交记录的hash值(commit id)，记录下来。
2. 执行交互式rebase命令：`git rebase -i commit_id`
    > 如果想要删除最近3次历史提交: `git rebase -i HEAD~3`
3. 进入编辑模式，将要删除的commit_id前的pick修改为drop。保存并退出编辑模式。
4. 推送更新到远程仓库：`git push --force(或git p)`

### 远程分支重命名

假设你当前已经将该分支推送到远程了，这种情况修改起来要稍微多几步

```shell
# 方案1:先重命名本地分支，然后推送到远程分支
# 1.先重命名本地分支
git branch -m 旧分支名称 新分支名称
# 2.删除远程分支(如果删除的是默认分支的话会失败，需要先更改默认分支)
git push --delete origin 旧分支名称
# 3.上传新修改名称的本地分支
git push origin 新分支名称
# 4.修改后的本地分支关联远程分支
git branch --set-upstream-to origin/新分支名称

# 方案2(未测试):先修改远程仓库分支，然后与本地仓库同步
git branch -m master 2021.x
# 获取源分支
git fetch origin
# 切换源分支为远程分支
git branch -u origin/2021.x 2021.x
# 设置远程分支
git remote set-head origin -a
# 方案2另一种实现(未测试):先修改远程仓库分支，然后与本地仓库同步
# 首先，在本地仓库中切换到需要同步的分支上：这里<branch-name>是需要同步的分支名称
git checkout <branch-name>
# 接下来，从远程仓库中获取最新的分支列表和分支状态信息：这会更新本地仓库中的远程分支信息。
git fetch
# 然后，使用以下命令来重置本地分支到远程分支的最新状态：这将强制将本地分支指向远程分支的最新状态。
git reset --hard origin/<branch-name>
```

### 下载所有分支

```shell
# git clone下载的是默认分支,分支较少的话可以使用git branch -a查看所有远程分支然后使用'git checkout 分支名'来下载其他分支。如果分支较多的话使用--bare,裸仓库(bare repository)指的是除了git仓库不包含其他工作文件的仓库，可以通过git clone --bare来生成。
git clone --bare https://github.com/xx/project.git .git
# 或者git config --unset core.bare
git config --bool core.bare false
# 上面的命令执行完,再执行该命令,就可以看到仓库里面的内容了
git reset --hard
# 或者切换到指定分支
git checkout 指定分支名
```

### 上传git仓库

```shell
# 创建.git目录
git init
git add -A
git commit -m "init"
git remote add origin https://github.com/xmxe/project.git
# 断开连接
git remote remove origin
# 增加一个新的远程仓库
git remote add name url
# 获取指定远程仓库的详细信息
git remote show origin

git pull --rebase origin master
# git push --set-upstream origin master可以简写成git push -u
git push -u origin master

# 本地生成ssh密钥
ssh-keygen -t rsa -C "你的邮箱"生成sshkey
git remote set-url origin git@github.com:xmxe/springcloud.git
```

### git删除github文件夹但不删除本地的 以.idea为例

```shell
git rm -r --cached .idea # --cached不会把本地的.idea删除
git commit -m 'delete .idea dir'
git push -u origin master
```

### git删除大文件

```shell
# 显示10个最大的文件id列表
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -n | tail -10
# 根据文件id查询文件路径
git rev-list --objects --all | grep 08a7475
# 删除文件的历史记录
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 文件名' --prune-empty --tag-name-filter cat -- --all

git filter-branch --index-filter # 让每个提交的文件都复制到索引(.git/index)中 然后运行过滤器命令：git rm --cached --ignore-unmatch文件名，让每个提交都删除掉“文件名”文件,然后--prune-empty 把空的提交“修剪”掉,然后--tag-name-filter cat把每个tag保持原名字，指向修改后的对应提交,最后-- --all将所有ref（包括branch、tag）都执行上面的重写

# 删除缓存下来的ref和git操作记录
git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin
git reflog expire --expire=now --all

# 垃圾回收
# 上面2步把大文件的索引都切断了，这个时候进行垃圾回收，就可以很明显看到效果了
git gc --prune=now
# 把.git里面的修改推上去,这个时候普通的push是不行的，需要强推
git push --force

```

### 忽略已经纳入git版本控制的文件更改

```shell
# 忽略单个文件的更改
git update-index --assume-unchanged <file>
# 恢复跟踪更改
git update-index --no-assume-unchanged <file>
# 更彻底的忽略
git update-index --skip-worktree <file>
git update-index --no-skip-worktree <file>
# Linux查看被忽略更改的文件
git ls-files -v | grep '^[a-z]'
# 小写字母表示被--assume-unchanged忽略的文件，S表示被--skip-worktree忽略的文件。选择哪种方法取决于你的具体需求，--skip-worktree通常比--assume-unchanged更可靠。
# PowerShell版
# 查看所有被assume-unchanged的文件（小写标记）
git ls-files -v | Where-Object { $_ -match '^[a-z]' }
# 查看所有被skip-worktree的文件（S标记）
git ls-files -v | Where-Object { $_ -match '^S' }

# --assume-unchanged：告诉Git假设文件未更改，性能优化用
# --skip-worktree：告诉Git完全忽略文件的更改，即使文件确实有变化
# .gitignore:只对未跟踪文件有效，对已跟踪文件需要先git rm --cached.或对于目录：git rm --cached -r <directory>
```

### 设置Git短命令

```shell
# 方式一
git config --global alias.ps push
# 方式二 打开全局配置文件
vim ~/.gitconfig
# 写入内容
[alias] 
        co = checkout
        ps = push
        pl = pull
        mer = merge --no-ff
        cp = cherry-pick
# 使用
# 等同于 git cherry-pick <commitHash>
git cp <commitHash>
```

## 相关文章

| [Git不要只会pull和push，试试这5条提高效率的命令](https://mp.weixin.qq.com/s/ct6GWiE_hzoXUNeriLAnng) | [Git各指令的本质，真是通俗易懂啊！](https://mp.weixin.qq.com/s/MM7sQiFPh2vIuGvg1-813Q) | [Git科普文，Git基本原理&各种骚操作](https://mp.weixin.qq.com/s/csEgAjJwH75_IvAnFBIuvw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [一文快速掌握Git用法](https://mp.weixin.qq.com/s/xoyQ4TzVKLQb2VjZJLUqFQ) | [精心整理：Git从入门到精通、包教包会、收藏一下、随时学习](https://mp.weixin.qq.com/s/wtPizZ3RlwY3Ex1Lc3Pf4A) | [大牛总结的Git使用技巧，写得太好了！](https://mp.weixin.qq.com/s/OchvVMGoBzSFWhou4WhrWw) |
| [通过.git目录深入理解Git！](https://mp.weixin.qq.com/s/q6tI0qctvciJhNz_5KLx-w) | [Git命令全方位学习](https://mp.weixin.qq.com/s/KwTsWyFh07iYdfINqo9UTg) | [图解Git，一目了然！](https://mp.weixin.qq.com/s/TW_qUWRVEnberle5q0ws9Q) |
| [Git代码防丢指南，再也不怕丢失代码了！](https://mp.weixin.qq.com/s/dYiWQQ5PsSS7F7z7cDLEuw) | [20个最常用的Git命令，你都会用吗？](https://mp.weixin.qq.com/s/XB_G7TZqBX8r3CJlvqA0Cg) | [45个GIT经典操作场景，专治不会合代码](https://mp.weixin.qq.com/s/Fa8mmQpNZ1S80Kg9Oyocbw) |
|  [Git的奇技淫巧](https://github.com/521xueweihan/git-tips)   | [如何配置SSH管理多个Git仓库和以及多个Github账号](https://mp.weixin.qq.com/s/ADzad6e6uTF0vpZ903SJ-A) | [那些年我们错过的Git指南](https://mp.weixin.qq.com/s/L5NEAOgoee8TQ0bOpp6PgQ) |
