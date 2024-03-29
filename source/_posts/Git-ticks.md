---
title: Git tricks
date: 2018-06-26 21:02:14
tags:
categories:
- Programming
- Git
---
#### Caching GitHub password in Git

In Terminal, enter the following:
```bash
git config --global credential.helper cache
# Set git to use the credential memory cache
```
To change the default password cache timeout, enter the following:
```bash
git config --global credential.helper 'cache --timeout=3600'
# Set the cache to timeout after 1 hour (setting is in seconds)
```

Or alternatively, store forever:
```bash
git config --global credential.helper store
```

#### Revert local change to previous commit
```bash
git checkout -- readme.txt # revert readme.txt to previous commit
```
or
```bash
git fetch --all
git reset --hard origin/master
git pull // can be ignored
```

#### Revert to status in another branch

```bash
git checkout <other-branch-name> -- <path to your file_or_folder>
```
