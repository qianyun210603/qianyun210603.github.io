---
layout: "post"
title: "Git Forked Repository强制与Fork源同步"
date: "2022-11-14 16:04"
categories:
  - Programming
  - Git
---

首先将Fork源增加到分支的remote url里面，
```
git remote add upstream git@github.com:microsoft/qlib.git
```
然后`pull --rebase`
```
git pull --rebase upstream main
```
再`reset hard`。
```
git reset --hard upstream/main
```
最后`force push`
```
git push --force origin main
```
