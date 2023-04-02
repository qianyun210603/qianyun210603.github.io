---
layout: "post"
title: "gitignore文件的顺序"
date: "2022-11-22 17:26"
categories:
- Programming
- Git
---

发现`.gitignore`里面条目顺序是有意义的。
```
*.c
*.cpp
!skts.c
!np_datetime.c
!np_datetime_strings.c
```
则`!`的几项生效
```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .gitignore

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        __init__.pxd
        pandas_helper/_libs/tslibs/src/datetime/np_datetime.c
        pandas_helper/_libs/tslibs/src/datetime/np_datetime_strings.c

no changes added to commit (use "git add" and/or "git commit -a")
```

如果反过来
```
!skts.c
!np_datetime.c
!np_datetime_strings.c
*.c
*.cpp
```
