---
title: Git
sticky: 49
tags: git
index_img: /assert/git.jpg
img: http://qungz.photo.store.qq.com/qun-qungz/V51g1j7W00cGjy3O92fU33gLE919gaa6/V5bCQAyOTM0MDQ3NTjHu0Rj.BxYNg!!/800?w5=1600&h5=800&rf=viewer_421
---

##### Git常用命令

###### 上传项目到git远程仓库 

```shell
git init
git add -A
git commit -m "init" 
git remote add origin https://github.com/xmxe/mySpringBoot.git
# 断开连接
git remote remove origin
git pull --rebase origin master 
git push origin master

# 本地生成ssh密钥
ssh-keygen -t rsa -C "你的邮箱"生成sshkey
git remote set-url origin git@github.com:xmxe/springcloud.git

```
###### diff

```shell
git diff # 可以查看当前没有add 的内容修改（不在缓冲区的文件变化） 
git diff --cached # 查看已经add但没有commit 的改动（在缓冲区的文件变化） 
git diff HEAD # 是上面两条命令的合并

```

###### reset

```shell
git reset HEAD filename # 撤销git add操作（已经将文件添加到暂存区）如果后面什么都不跟的话 就是上一次add 里面的全部撤销了 
git reset --hard HEAD^ # 回退到上一个版本，删除工作空间改动代码，撤销commit，撤销git add . 
git reset --hard commitid # 回退到指定版本 --hard 强制将暂存区和工作目录都同步到你指定的提交
git reset --soft HEAD^ # 不删除工作空间改动代码，撤销commit，不撤销git add 
git reset --mixed HEAD^ # 不删除工作空间改动代码，撤销commit，并且撤销git add . 操作。这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
git revert -n commitid # 回退到指定版本。 与git reset --hard commitid相比 区别是在我们确认了在需要回退的版本之后的提交都可以不需要的时候，我们可以直接使用git reset 命令，但是当我们只是需要撤销某个版本的时候，后面的提交还需要保留，我们就可以使用git revert
git revert HEAD # 撤销前一次commit ,此次操作之前和之后的commit都会被保留，并且会把这次撤销作为一次最新的提交 

```

###### commit

```shell
git commit -am '' # (git add -u + git commit -m ""组合) 
git commit --amend # 修改注释
```
###### add

```shell
git add -A # 保存所有的修改 
git add . # 保存新的添加和修改，但是不包括删除 
git add -u # 保存修改和删除，但是不包括新建文件。 

```

###### branch

```shell
git branch # 查看分支
git branch <name> # 创建分支(未切换)
git branch -d <name> # 删除分支，不能删除当前所在分支 
git branch --set-upstream branch-name origin/branch-name # 建立本地分支和远程分支的关联

```

###### checkout

```shell
git checkout <name> # 切换分支
git checkout -b <name> # 创建+切换分支
git checkout -- file # 丢弃某个文件工作区的修改（还原修改过的文件）
git checkout . # 放弃本地所有修改，没有提交的可以回到未修改前版本，不包括新增删除的文件 
git checkout . && git clean -df # 放弃本地所有修改，包括新增删除的文件
git checkout develop && git merge feature #（先切换到develop分支然后把feature分支合并到develop分支） 
```
###### merge

```shell
git merge <name> # 合并指定分支到当前分支
```
###### git rebase 和 git merge 有啥区别？

rebase会把你当前分支的 commit 放到公共分支的最后面,所以叫变基。就好像你从公共分支又重新拉出来这个分支一样。
举例:如果你从 master 拉了个feature分支出来,然后你提交了几个 commit,这个时候刚好有人把他开发的东西合并到 master 了,这个时候 master 就比你拉分支的时候多了几个 commit,如果这个时候你 rebase master 的话，就会把你当前的几个 commit，放到那个人 commit 的后面。
merge 会把公共分支和你当前的commit 合并在一起，形成一个新的 commit 提交

###### tag

```shell
git tag # 查看所有标签，可以知道历史版本的tag 
git tag <tagName> # 打标签，默认为HEAD。比如git tag v1.0
git tag <tagName> commit_id # 根据版本号打上标签
git tag -a <tagName> -m "<说明>" # 创建带说明的标签。-a指定标签名，-m指定说明文字 
git show <tagName> # 查看标签信息 
git tag -d <tagName> # 删除标签 
git push origin <tagname> # 推送某个标签到远程 
git push origin --tags # 一次性推送全部尚未推送到远程的本地标签 

```
###### stash

> 应用场景：某一天你正在 feature 分支开发新需求，突然产品经理跑过来说线上有bug，必须马上修复。而此时你的功能开发到一半，于是你急忙想切到 master 分支，然后你就会看到以下报错：Your local changes to the following...因为当前有文件更改了，需要提交commit保持工作区干净才能切分支。由于情况紧急，你只有急忙 commit 上去，commit 信息也随便写了个“暂存代码”，于是该分支提交记录就留了一条黑历史，如果你学会 stash，就不用那么狼狈了。你只需要： git stash就这么简单，代码就被存起来了。 当你修复完线上问题，切回 feature 分支，想恢复代码也只需要： git stash apply,但是恢复后，stash 内容并不删除，需要用 git stash drop 来删除

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

# 当有多条 stash，可以指定操作stash，首先使用stash list 列出所有记录：
git stash list 
# stash@{0}: WIP on ...
# stash@{1}: WIP on ...
# stash@{2}: On ...

# 应用第二条记录：
git stash apply stash@{1}
# pop，drop 同理。
```

###### config

```shell
git config --gloabl http.postBuffer 524288000
git config --gloabl http.sslVerify "false"
git config --global user.name "your name"
git config --global user.email "your email"
git config --global credential.helper store
```

###### git删除github文件夹但不删除本地的 以.idea为例

```shell
git rm -r --cached .idea # --cached不会把本地的.idea删除
git commit -m 'delete .idea dir'
git push -u origin master
```
###### git删除大文件

```shell
# 显示10个最大的文件id列表
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -n | tail -10
# 根据文件id查询文件路径
git rev-list --objects --all | grep 08a7475
# 删除文件的历史记录
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 文件名' --prune-empty --tag-name-filter cat -- --all

git filter-branch --index-filter # 让每个提交的文件都复制到索引(.git/index)中 然后运行过滤器命令：git rm --cached --ignore-unmatch文件名 ，让每个提交都删除掉“文件名”文件 然后--prune-empty 把空的提交“修剪”掉 然后--tag-name-filter cat 把每个tag保持原名字，指向修改后的对应提交 最后-- --all 将所有ref（包括branch、tag）都执行上面的重写

# 删除缓存下来的ref和git操作记录
git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin
git reflog expire --expire=now --all

# 垃圾回收
# 上面2步把大文件的索引都切断了，这个时候进行垃圾回收，就可以很明显看到效果了
git gc --prune=now
# 把.git里面的修改推上去  这个时候普通的push是不行的，需要强推 
git push --force

```
##### 相关文章

- [Git 不要只会 pull 和 push，试试这 5 条提高效率的命令](https://mp.weixin.qq.com/s/ct6GWiE_hzoXUNeriLAnng)
- [Git 各指令的本质，真是通俗易懂啊！](https://mp.weixin.qq.com/s/MM7sQiFPh2vIuGvg1-813Q)
- [Git科普文，Git基本原理&各种骚操作](https://mp.weixin.qq.com/s/csEgAjJwH75_IvAnFBIuvw)
- [一文快速掌握 Git 用法](https://mp.weixin.qq.com/s/xoyQ4TzVKLQb2VjZJLUqFQ)
- [精心整理 ：Git 从入门到精通、包教包会、收藏一下、随时学习](https://mp.weixin.qq.com/s/wtPizZ3RlwY3Ex1Lc3Pf4A)
- [大牛总结的 Git 使用技巧，写得太好了！](https://mp.weixin.qq.com/s/OchvVMGoBzSFWhou4WhrWw)
- [通过 .git 目录深入理解 Git！](https://mp.weixin.qq.com/s/q6tI0qctvciJhNz_5KLx-w)
- [Git 命令全方位学习](https://mp.weixin.qq.com/s/KwTsWyFh07iYdfINqo9UTg)
- [图解 Git，一目了然！](https://mp.weixin.qq.com/s/TW_qUWRVEnberle5q0ws9Q)
- [Git 代码防丢指南，再也不怕丢失代码了！](https://mp.weixin.qq.com/s/dYiWQQ5PsSS7F7z7cDLEuw)
- [20 个最常用的 Git 命令，你都会用吗？](https://mp.weixin.qq.com/s/XB_G7TZqBX8r3CJlvqA0Cg)
- [合并代码还在用 git merge？我们都用 git rebase！](https://mp.weixin.qq.com/s/T_8bkWI-JSP5ixdVIvVAGQ)
- [45个 GIT 经典操作场景，专治不会合代码](https://mp.weixin.qq.com/s/Fa8mmQpNZ1S80Kg9Oyocbw)

##### 一图以概览
![](/images/git.png)