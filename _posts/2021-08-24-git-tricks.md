---
layout: post
title: 'Git Tricks Worth Knowing'
subtitle: '使用 Git 的「奇技淫巧」'
author: 'Will'
header-style: text
lang: en
tags:
  - Git
  - Tricks
  - 笔记
---

## Restore / Reset

```bash
#1 - restore a file change
git restore <filename>

#2 - reset all file changes
git reset --hard HEAD

#3 - unstage a file
git restore --cached <filename>

#4 - restore a deleted file (but didn't commit)
git checkout HEAD <filename>

#5 - restore a deleted file (and committed)
git reset --hard HEAD~1
```

## Remote

```bash
#1 - list existing remotes
git remote -v

#2 - add remote
git remote add <remote-name> <remote-url>

#3 - set remote url
git remote set-url <remote-name> <remote-url>
```

## Diff

```bash
#1 - diff unstaged files
git diff

#2 - diff staged files
git diff --staged

#3 - list staged files
git diff --staged --name-only
```

## Branch

```bash
#1 - clone from a specific branch
git clone -b <branch-name> <remote-url>

#2 - create a branch and check it out
git checkout -b <branch-name>

#3 - delete a local branch
git branch -d <branch-name>

#4 - delete a remote branch
git push -d <remote-name> <branch-name>

#5 - update branches
git fetch -p
```
