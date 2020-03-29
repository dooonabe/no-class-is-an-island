# Git

## 移除跟踪的文件
git rm --cached submit.log

## 修正最后一次提交
git commit --amend

## 远程仓库分支列表
git remote show <remote>

## 创建远程跟踪分支
git checkout -b <branch> <remote>/<branch>
git checkout --track <remote>/<branch>

## 更改本地分支跟踪的远程分支
git branch -u <remote>/<branch>

## 查看本地分支与远程分支关系
git fetch --all
git branch -vv
