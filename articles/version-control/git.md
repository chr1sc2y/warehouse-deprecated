# Git Reference

![git-basic](https://raw.githubusercontent.com/ZintrulCre/warehouse/main/resources/version-control/git-basic.png)

## HEAD 用法

HEAD 最后一次 commit

HEAD^ 倒数第二次 commit

HEAD^^ 倒数第三次 commit，以此类推

HEAD~0 最后一次 commit

HEAD~1 倒数第二次 commit

HEAD^2 倒数第三次 commit，以此类推

## 回到以前的某次 commit

```shell
git reflog # Reference logs 记录了本地仓库每一次更新分支的操作
git reset HEAD@{index} # 回到某一次提交，把文件修改留在工作区
git reset hash --hard # 加上 --hard 可以忽略掉所有文件修改
```

## 在最后一次 commit 的基础上添加部分改动

```shell
git add . # 把改动添加到暂存区
git commit --amend #
git commit --amend --no-edit # 加上 --no-edit
```

如果最后一次 commit 已经 push 到 remote，那么在再次 push 的时候需要加上 -f

## 在以前的某次 commit 的基础上添加部分改动

```shell
git log # 找到要修改的 commit 的前一次
git rebase -i hash # 将 HEAD 移到需要修改的 commit 上
(vim) R edit # 将首行的 pick 改成 edit，保存退出
# 修改文件
git add .
git commit --amend # 追加改动到这次 commit 上
git rebase --continue # 恢复 HEAD
```

## 把未 commit 的修改移动到其他分支上

```shell
git reset HEAD~ --soft # 撤销最后一次 commit 操作，但是保留文件修改
git stash
git checkout another-branch
git stash pop
git add .
git commit -m "your message here"
```

## 把某一次 commit 添加到其他分支上

```shell
git log # 找到需要移动的 commit 的 hash
git checkout another-branch
git cherry-pick hash # 把对应的 commit 应用到当前的 branch
```

## 撤销某次 commit

```shell
git log # 找到需要撤销的 commit 的 hash
git revert hash # 撤销对应的改动并直接 commit
```

## 撤销某次 commit 的某个文件

```shell
git log # 找到需要撤销的 commit 的 hash
git checkout hash -- path/to/file # 把修改之前的文件添加到工作区
git commit -m "your message here"
```

## 放弃治疗

```shell
git fetch origin
git checkout master
git reset --hard origin/master
git clean -d --force # 删除工作区所有 untracked 的文件和目录
```

或者

```shell
cd ..
rm -r repo-name
git clone https://some.github.url/repo-name
cd repo-name
```

## 删除 branch

```shell
git branch -d branch-name # 删除本地分支
git push origin -d branch-name # 删除 origin 的分支
```