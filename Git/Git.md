<h1 align="center">Git操作指南</h1>

## 目录

* [Github技巧](#Github技巧)
* [Git操作指南](#Git操作指南)

## Github技巧

* [如何正确接收 GitHub 的消息邮件](https://github.com/cssmagic/blog/issues/49)
* [如何用好 Github 中的 watch、star、fork](https://www.jianshu.com/p/6c366b53ea41)

## Git操作指南

```
git status

git add .

git commit -m "注释"

git stash # 每次 push 前

git pull --rebase

# 如果有冲突，解决冲突
git rebase --continue

# gerrit review
git push origin HEAD:refs/for/master

# or

# github
git push origin develop 

git pull

git stash pop
```
