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

- Restore file change

```bash
git restore <filename>
```

- Reset all changes

```bash
git reset --hard HEAD
```

- Unstage

```bash
git restore --cached <filename>
```

- Restore accidental deletion before commit

```bash
git checkout HEAD <filename>
```

- Restore deleted file after commit

```bash
git reset --hard HEAD~1
```

## Remote

- List remotes

```bash
git remote -v
```

- Add remote

```bash
git remote add <remote_name> <remote_url>
```

- Set remote url

```bash
git remote set-url <remote_name> <remote_url>
```

## Diff

- Diff unstaged files

```bash
git diff
```

- Diff staged files

```bash
git diff --staged
```

- List staged files

```bash
git diff --staged --name-only
```

## Branch

- Clone specifying branch

```bash
git clone -b <branch_name> <remote_url>
```

- Create and checkout

```bash
git checkout -b <branch_name>
```

- Delete local branch

```bash
git branch -d <branch_name>
```

- Delete remote branch

```bash
git push -d <remote_name> <branch_name>
```

- Update local branch list

```bash
git fetch -p
```
