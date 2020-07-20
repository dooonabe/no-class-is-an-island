# Git

## 查看每次提交的统计信息(statistics)
git log --stat

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

## 变基
git rebase <branch>
  
## 缓存验证信息
git config --global credential.helper cache

## git merge vs git rebase vs git cherry-pick
```
      A---B---C topic
     /
D---E---F---G master
```
- git merge topic 
```
      A---B---C topic
     /         \  
D---E---F---G---H master
```
- git rebase master topic
```
              A'--B'--C' topic
             /
D---E---F---G master
```
- git cherry-pick a b c
```
      A---B---C topic
     /
D---E---F---G---A'--B'--C' master
```