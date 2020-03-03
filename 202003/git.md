# git

### 命令


`.gitkeep` 在目录下可以跟踪空目录，与`.gitignore`相对

`git add`:

    `git add -u`：表示修改和删除后添加到缓存区，不包括新增。

    `git add -i`: 表示经过修改或者删除，但是没有被提交的文件

    `git add .`: 所有修改与新增文件到暂存区，但是不包删除

    `git add -p`: 进入交互模式

`git rm`:

    `git rm -cache`：只是停止追踪文件

`git blame`: 明确责任

`git branch`:

    `git branch -d`: 删除

    `git branch -D`: 强制删除

    `git branch -m` 修改分支名字

`git checkout`:

    `git checkout --` 丢弃更改

    `git checkout HEAD -- <filename>` 某个提交中恢复，会同时修改暂存区与工作区

    `git checkout tag/12` 恢复到某个tag

`git cherry-pick XXX` 复制某个提交节点在当前分支做一次完全一样的提交

`git commit`:

    `git commit -amend`:撤销上一次提交，生成新的commit。

    `git commit --fixup <commit>` 当前的提交是某一次提交的修正

`git diff`:

    `git diff file.txt` 比较工作区与暂存区

    `git diff --cached` 暂存区与仓库区别

    `git diff <commit>`工作区与某次提交的区别

    `git diff HEAD -- ./test`就test文件，上次提交与暂存区区别

    `git diff test master` 分支间的区别

    `git diff test...master` test分支建立后，master的变化

    `git format-patch master -- stdout>test.patch`

`git merge develop` 此处是把develop分支合并到当前分支，能保留每次commit，但是merge过程会额外产生commit节点

`git rebase -i <commit/master>` 当前分支master移植到当前分支或者提交之上。就是commit按照顺序转移。commit 信息清晰，但是不容易定位信息。

`git reset`:

    `git reset <commit>` 改变指针同时改变暂存区

    `git reset --soft <commit>` 改变指针时不改变暂存区

    `git reset --hard` 改变指针同时改变工作区

`git stash`:

    `git stash pop`： 取出暂存

    `git stash apply <>` 应用某次提交


### .git目录

`git add`相当于把文件转成二进制文件（`git hash-object`），放到`.git/objects`中，并计算内容的hash值，40位的hash值，前两位为文件夹名，文件在文件夹内名字是后38位。然后通知git文件的变动，保存到暂存区（`git update-index`）

`git commit`相当于把当前目录结果保存为二进制文件(`git write-tree`)，同样放到`.git/objects`中，然后加上必要信息提交(`git commit-tree`)

`objects`: 目录存放文件，每个文件是一个。包括`blob/tree/commit`

`.git/refs/heads/` 存放指针快照。
