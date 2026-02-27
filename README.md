## 日常开发流程

### 日常“先同步，再开发”的基本操作

```bash
# 0) 拉取所有远端分支信息
git fetch origin

# 1) 切到公共开发分支（比如 develop），更新到最新
git switch develop
git pull --ff-only
```

### 新功能/修Bug/推送代码从这里开始操作

```python
# 2) 基于最新 develop 新建自己的分支
git switch -c feature/xxx

# 3) 开发 -> 提交（可多次小步提交）
git add .
git add setup.py pyproject.toml # 部分add

git commit -m "实现：xxx；修复：yyy"

git push -u origin feature/xxx
```

### 期间需要同步团队最新进展（避免分叉）

> 在自己的分支上用 rebase 对齐最新 develop（历史更干净）
> 

```bash
git fetch origin
git rebase origin/develop      # 把你的提交“搬到”最新 develop 之上

# 如果有冲突：编辑文件解决 → 保存
git add <冲突文件>
git rebase --continue          # 直到完成

# rebase 改写了历史，已推送过就需要安全地强推
git push --force-with-lease
```

### 合并完成后清理分支

```python
# 目标分支（develop）里 PR 合并完：
git switch develop
git pull --ff-only
git branch -d feat/xxx          # 本地删分支
git push origin --delete feat/xxx  # 远端也删（可选）
```

## 其他

### git新建分支（branch）命名

分支类型与示例

```python
# 前缀标识
# 分支类型：feature/、fix/、hotfix/、chore/、docs/、perf/、refactor/、test/、release/

# 可选带任务单号：紧接前缀放工单/需求号，便于追踪（Jira/Tapd/Trello等）

# 名称力求“动词 + 作用域 + 简述”：一眼看懂要做什么、改哪里
# 不要空格、不要中文、不要大写、不要特殊符号（_也尽量不用，统一-

#分支类型与示例
新功能：feature/<issue-key>-<scope>-<short-desc>
例：feature/MTCHJX-22923-yolov5-dann-multi-scale
例：feature/clip-reid-export-api

缺陷修复（非线上紧急）：fix/<issue-key>-<scope>-<bug-desc>
例：fix/PJ-1045-dataloader-empty-labels

线上紧急修复：hotfix/<issue-key>-<scope>-<bug-desc>
例：hotfix/PROD-777-memory-leak-yolov8

性能优化：perf/<scope>-<short-desc>
例：perf/nms-int8-quant

重构（无功能变化）：refactor/<scope>-<short-desc>
例：refactor/trainer-logging

测试相关：test/<scope>-<short-desc>
例：test/augmentations-unit-cases

文档：docs/<scope>-<short-desc>
例：docs/avi_split-readme

杂务/构建/脚本：chore/<scope>-<short-desc>
例：chore/packaging-cython-wheel

预发布/发布分支：release/<version>
例：release/2.1.0

scope 建议使用模块名/子系统名，如：dataloader、trainer、inference、deploy、ui、docs、infra、scripts。

```

快速命名清单

```python
新功能：feature/<任务号>-<模块>-<动词-简述>
修 Bug：fix/<任务号>-<模块>-<问题简述>
紧急修：hotfix/<任务号>-<模块>-<问题简述>
重 构：refactor/<模块>-<动词-简述>
性 能：perf/<模块>-<动词-简述>
文 档：docs/<模块>-<动词-简述>
发布线：release/<语义化版本号>
```

### 其他常用操作

```python
# 克隆分支代码
git clone -b branchname http://xxxxx

# 查看当前所在分支
git status

# 查看远端仓库地址（查看某个远端（比如 origin）的详细配置）
git remote show origin

# 查看远端仓库地址（所有远端及其 URL）
git remote -v

# 查看提交历史记录
'''
q：退出查看
↑/↓：上下滚动
'''
git log
```

### 常用指令速查表

| 目的 | 命令 | 说明 |
| --- | --- | --- |
| 查看状态 | `git status` | 哪些文件改了、将提交到哪里 |
| 看提交历史 | `git log --oneline --graph --decorate` | 清爽图示 |
| 暂存未完成的改动 | `git stash -u` | 临时放一边（含未跟踪文件） |
| 取回暂存 | `git stash pop` | 恢复刚才那份 |
| 撤销工作区改动 | `git checkout -- <文件>` | 回到上次提交版本 |
| 撤销暂存区 | `git reset <文件>` | 从暂存区拿回工作区 |
| 修改最近一次提交信息/内容 | `git commit --amend` | 还没推就可以改 |
| 回到某个提交（保留改动到工作区） | `git reset --soft <commit>` | 常用于合并提交 |
| 丢弃到某提交（危险） | `git reset --hard <commit>` | 小心！会丢改动 |
|  |  |  |

### Haitao教学版本

```python
git branch
git stash list
git stash drop 0
git stash list
git stash push -m "[bev]compress 20251103" # 把当前工作区的改动（默认包括已跟踪文件）压进一个新的 stash，备注为该信息；工作区恢复到“干净状态”。
git checkout develop
git rebase develop
```

### 可能的意图 vs 正确做法

### 想把本地改动先收起来，再对齐主干

```bash
git stash push -u -m "[bev] compress 20251103"   # -u 连未跟踪文件也一起收
git switch develop
git pull --ff-only                               # 把 develop 更新到远端最新
# 回到你的功能分支并对齐主干（线性历史）
git switch feature/your-branch
git rebase origin/develop
# 若有冲突：解决 → git add 改过的文件 → git rebase --continue
```

### 想把 stash 的改动继续用在功能分支上

```bash
git switch feature/your-branch
git stash apply stash@{0}   # 或直接 git stash pop （用完即删）
# 继续开发 → 提交
git add .
git commit -m "xxx"
```

---
