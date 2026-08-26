---
title: Git 从初始化到日常全流程
tags: [工具链, git, 版本管理]
---

# Git 从初始化到日常全流程

## 一、初始化
```bash
# 1. 进到你的代码目录
cd 你的工程目录

# 2. 初始化 git（如果还没有）
git init

# 3. 关联远程仓库
git remote add origin <仓库地址>

# 4. 第一次提交
git add .
git commit -m "初始化项目"
git push -u origin master
```

> 日常开发就三步：`git add .`（暂存）→ `git commit -m "改了什么"`（提交）→ `git push`（推送）
> 远程仓库为空直接走上面流程；若远程已有 README 之类，第一次 push 前先 `git pull origin master --allow-unrelated-histories` 合并

## 二、分支操作
```bash
git checkout -b feature/xxx   # 新建并切换到新分支
git checkout master           # 切换回已有分支
git branch -a                 # 看所有分支（本地+远程）
git branch -d feature/xxx     # 删本地分支（已合并）
git branch -D feature/xxx     # 强制删（没合并）
```
命名习惯：`feature/功能名` `fix/修什么` `refactor/改什么`

## 三、提交代码
```bash
git status                     # 看改了啥
git diff
git add src/main.c src/gpio.c  # 只加想提交的（别无脑 git add .）
git commit -m "feat: 添加串口驱动"
git push
git push -u origin feature/xxx # 第一次推新分支
```
提交信息习惯：`feat: xxx` / `fix: xxx` / `refactor: xxx`

## 四、拉取代码
```bash
git pull              # 拉最新代码（等价于 fetch + merge）
git pull --rebase     # 拉但用 rebase（历史更干净，推荐日常用）
```
pull 之前先把本地改完的提交了，或者先 `git stash` 暂存

## 五、合并冲突
```bash
git checkout feature/xxx
git merge master
```
冲突文件标记：
```
<<<<<<< HEAD
你的代码
=======
master 上的代码
>>>>>>> master
```
手动改成最终版本、删掉标记、保存，然后：
```bash
git add 冲突文件
git commit -m "merge: 合并 master，解决 xxx 冲突"
# 放弃本次合并
git merge --abort
```
> 防坑：合并前先 `git status` 确认工作区干净。不确定就别在凌晨合并。

## 六、回退版本
```bash
git checkout -- .          # 情况1：丢弃未暂存的改动
git reset --hard HEAD      #       全丢，回到最近一次提交

git reset --soft HEAD~1    # 情况2：提交了后悔，撤销上一次（改动保留在工作区）

git reset --hard HEAD~1    # 情况3：提交了后悔，连改动一起丢掉

git revert HEAD            # 情况4：已推送到远程，生成「撤销上一个提交」的新提交
git push

git log --oneline -10      # 看历史找版本号
```
> 铁律：`reset --hard` 之前先确认没东西要留。推过的提交用 `revert` 别用 `reset`，否则坑队友。

## 七、暂存
```bash
git stash          # 正写着东西突然要切分支
git stash pop      # 切回来之后恢复
git stash list     # 看暂存列表
```

## 八、日常完整流程
```bash
# 早上开工
git checkout master
git pull --rebase
git checkout -b feature/xxx

# 写代码...

git add src/xxx.c
git commit -m "feat: 搞定了xxx"

# 写更多代码...

git add src/yyy.c
git commit -m "feat: 加了yyy"

# 推之前先同步 master（免得合的时候冲突一堆）
git fetch origin master
git rebase origin/master
# 有冲突就解决，然后 git add + git rebase --continue

# 推到远程
git push -u origin feature/xxx

# 去 GitLab/GitHub 提 PR/MR
# 合完之后切回 master
git checkout master
git pull --rebase
git branch -d feature/xxx
```

## 九、紧急救火
```bash
# 线上炸了，手头工作先暂存
git stash

# 切到 master 开修复分支
git checkout master
git pull --rebase
git checkout -b hotfix/crash

# 改完提交推送，提 PR

# 合完之后切回来继续干活
git checkout feature/xxx
git stash pop
```

## 三条铁律
1. `push --force` 别用，除非你完全清楚后果
2. `reset --hard` 之前看一眼 `git status`
3. 不确定的时候先 `git log --oneline` 看一眼自己在哪
