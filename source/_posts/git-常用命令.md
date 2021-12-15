---
title: git 常用命令
date: 2019-10-19 21:11:21
author: "BerniLin21"
header-img: /images/bg1.png
tags:
  - git
---

这里稍稍对一些常用的 git 的命令进行简单的说明。。。

<!-- more -->

## 初始化创建 git 仓库

```
git init
```

## 展示帮助信息

```
git help -g
```

## 克隆

```
git clone git@github.com:.....git // clone git 仓库
git clone -b <branch-name> --single-branch git@github.com:.....git  // clone 下来指定的单一
分支
git clone --depth=1 git@github.com:.....git //  clone 最新一次提交
```

## 暂存

```
git add
// git add . 暂存全部修改
```

## 提交到本地仓库

```
git commit -m 'some remark'// 带描述提交
git commit --amend  // 修改上一个 commit 的描述
git commit --amend --author='Author Name email@address.com>'  修改作者名
```

重设第一个 commit

```
git update-ref -d HEAD
```

## 查看 `commit` 记录

```
git log
git log --pretty=oneline // 精简显示
git log --graph //  查看分支合并图
git log Branch1 ^Branch2  commit 历史中显示 Branch1 有的，但是 Branch2 没有 commit
```

## 拉取代码

```
git pull
git fetch //  是从远程获取最新版本到本地，不会自动merge
git fetch origin pull/<id>/head:<branch-name> //  从远程仓库根据 ID，拉下某一状态，到本地分支
```

## 展示内容

```
git show <branch-name>:<file-name>  //  展示任意分支某一文件的内容
git show <tag-name> //  展示某一tag内容
```

## 显示文件

```
git ls-files -t  展示所有 tracked 的文件
git ls-files --others  展示所有 untracked 的文件
```

## 查看文件状态

```
git status  //  查看所有文件状态
git status [filename] //  查看指定文件状态
git status --ignored  //  展示忽略的文件
git reflog  显示本地更新过 HEAD 的 git 命令记录
```

## 展示不同

```
git diff  // 输出工作区和暂存区的 different
git diff <commit-id> <commit-id>  //  展示本地仓库中任意两个 commit 之间的文件变动
git diff --cached //  暂存区和本地最近的版本 (commit) 的 different
git diff HEAD //  工作区、暂存区 和本地最近的版本 (commit) 的 different
git diff --word-diff  //  详细展示一行中的修改
```

## 以新增一个 `commit` 的方式还原某一个 `commit` 的修改

```
git revert <commit-id>
```

## 回退

```
git reset --hard HEAD^  //  回退上一版本，上上版本 HEAD^^
git reset –mixed HEAD^  //  回退至上个版本，它将重置 HEAD 到另外一个 commit ,并且重置暂存区以便和HEAD相匹配，但是也到此为止。工作区不会被更改。
git reset –soft HEAD~3  //  回退至三个版本之前，只回退了 commit 的信息，暂存区和工作区与回退之前保持一致。如果还要提交，直接commit即可
git reset <commit-id> //  回退到指定 commit-id 的状态
git reset –hard <commit-id> //  彻底回退到指定 commit-id 的状态，暂存区和工作区也会变为指定commit-id版本的内容

```

抛弃本地所有的修改，回到远程仓库的状态。

```
git fetch --all && git reset --hard origin/master
```

## 切换

```
git checkout <branch-name>  //  切换分支
git checkout -  //  快速切换到上一个分支
git checkout -b <branch-name>  //  创建并切换到本地分支
git checkout -b <branch-name> origin/<branch-name>  //  从远程分支中创建并切换到本地分支
git checkout --orphan <branch-name> //  新建并切换到新分支上，同时这个分支没有任何 commit
git checkout -b branch_name tag_name  //  切回到某个标签
```

## 放弃修改

```
git checkout <file-name>  //  放弃工作区的修改，.放弃所有修改
```

恢复删除的文件

```
git rev-list -n 1 HEAD -- <file_path> #得到 deleting_commit
git checkout <deleting_commit>^ -- <file_path> #回到删除文件 deleting_commit 之前的状态
```

## 把 `A` 分支的某一个 `commit`，放到 `B` 分支上

```
git checkout <branch-name> && git cherry-pick <commit-id>
```

## 删除文件

```
git rm test.txt
```

## 本地分支

```
git branch  //  看当前分支（本地）
git branch <branch>  //  创建分支（本地）
git branch -d <branch>会在删除前检查merge状态（其与上游分支或者与head）
git branch -D <branch>是git branch --delete --force的简写，它会直接删除
git branch -a //  列出本地和远程分支
git branch -r //  列出所有远程分支  -r 参数相当于：remote
git branch -vv  //  展示本地分支关联远程仓库的情况
git branch -m | -M <old-branch-name> <new-branch-name>  // 重命名分支，如果newbranch名字分支已经存在，则需要使用-M强制重命名，否则，使用-m进行重命名
git branch -u origin/mybranch //  关联远程分支
git branch --set-upstream branch-name origin/branch-name  //  建立本地分支和远程分支的关联
```

## 远程仓库

```
git remote  //  列出所有远程仓库
git remote add origin git@github.....git  //  添加远程仓库
git remote set-url origin <URL> //  修改远程仓库的 url
git remote show origin  //  查看远程分支和本地分支的对应关系
git remote rm origin  // 删除远程关联关系
git remote prune origin //  远程删除了分支本地也想删除

```

## 推送

```
git push
git push -u origin master //  推送同时关联远程
git push -f <remote-name> <branch-name> //  强制推送
git push origin --delete <remote-branchname>  //  删除远程分支
git push origin :<remote-branchname>  //  删除远程分支
```

## 合并

```
git merge <branch> //  将目标分支合并到当前分支
```

## 存储

```
git stash //  存储当前的修改
git stash -u  //  保存当前状态，包括 untracked 的文件
git stash list  //  展示所有 stash
git stash apply <stash@{n}> //  回到某个 stash 的状态
git stash pop  //  回到最后一个 stash 的状态，并删除这个 stash
git stash clear //  删除所有的 stash
git checkout <stash@{n}> -- <file-path> //  从 stash 中拿出某个文件的修改
```

## 标签

```
git tag //  查看所有标签
git describe --tags --abbrev=0  //
git tag -ln //  查看标签详细信息
git tag <tag-name>  //  创建标签，默认打在最近一次 commit 上
git tag -a <tag-name> -m "v1.0 发布(描述)" <commit-id>  //  创建标签并打在指定 commit 上
git push origin <tag-name>  //  推送标签到远程仓库
git push origin --tags  //  一次性推送所有标签，同步到远程仓库
git tag -d <tag-name> //  删除本地标签
git push origin :refs/tags/<tag-name> //  删除远程标签，需先删除本地标签
git checkout -b <branch-name> <tag-name>  //  切回到某个标签
```

## 查看某段代码是谁写的

```
git blame <file-name>
```

## 给 `git` 命令起别名

```
git config --global alias.<handle> <command>
```

## 清除 gitignore 文件中记录的文件

```
git clean -X -f
```

## 删除全局设置

```
git config --global --unset <entry-name>
```

## 变基

`rebase` 可以对某一段线性提交历史进行编辑、删除、复制、粘贴。 因此，合理使用 rebase 命令可以使我们的提交历史干净、简洁！

_参考官方文档：https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA_

```
git pull --rebase

git pull --rebase 代替 git pull // git pull --rebase = git fetch + git rebase

git config --global pull.rebase true
// 更改 pull.rebase 的默认配置，后续使用 git pull 就是使用 git pull --rebase 了

// 使用 git pull -- rebase 发生冲突的时候解决所有冲突的文件
git add .
git rebase --continue

// 若是中间毫无冲突，变基则一步到位，否则需要逐步调整。
git rebase --continue // 提交变更后继续变基下一步
git rebase --skip // 引起冲突的commits会被丢弃，continue提示没有需要改动的也可以用这个跳过
git rebase --abort // 若是变基改残废了，但是走到一半，可以彻底回滚变基之前的状态

```

```
// 变基与合并
(dev): git rebase master // 将当前分(dev)支变基为 master 分支
(master): git merge dev // 回到 master 分支，进行一次快进合并
```

```
git rebase -i // 合并多次 commit
```

_参考：https://github.com/zuopf769/how_to_use_git/blob/master/%E4%BD%BF%E7%94%A8git%20rebase%E5%90%88%E5%B9%B6%E5%A4%9A%E6%AC%A1commit.md_

## 参考文档

- 官方： https://git-scm.com/
- 廖雪峰 https://www.liaoxuefeng.com/wiki/896043488029600
- 简易教程 https://www.bootcss.com/p/git-guide/
