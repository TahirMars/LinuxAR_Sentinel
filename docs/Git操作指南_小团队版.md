# Git 操作指南 · 小团队版

> 适用项目：proj_name Monorepo  
> 团队规模：4 人（Dev A / B / C / D）  
> 托管平台：GitHub Organization（Free 套餐）  
> 更新日期：2026-03-13

### ⚠️ Git 版本要求

本项目要求 **git ≥ 2.32.0**（Cursor IDE 会自动向 `git commit` 注入 `--trailer` 参数，该选项在 2.32.0 中引入）。

```bash
# 检查当前版本
git --version

# 如版本低于 2.32.0（常见于 Ubuntu 20.04 默认源），请升级：
sudo add-apt-repository -y ppa:git-core/ppa
sudo apt update && sudo apt install -y git
```

---

## 目录

1. [核心概念速查](#1-核心概念速查)
2. [首次配置（每人只做一次）](#2-首次配置每人只做一次)
3. [日常开发工作流](#3-日常开发工作流)
4. [分支管理](#4-分支管理)
5. [提交规范](#5-提交规范)
6. [Pull Request 流程](#6-pull-request-流程)
7. [同步与冲突处理](#7-同步与冲突处理)
8. [撤销与回滚](#8-撤销与回滚)
9. [查看与检索](#9-查看与检索)
10. [常见问题 FAQ](#10-常见问题-faq)

---

## 1. 核心概念速查

| 术语 | 含义 |
|------|------|
| `main` | 生产分支，只接受 PR 合并，禁止直接 push |
| `develop` | 集成分支，各功能分支合并到这里联调 |
| `feature/{dev}-{module}` | 个人功能分支，日常开发在此进行 |
| `origin` | 远端仓库（GitHub 上的那份） |
| `HEAD` | 当前所在的提交位置 |
| `staged` | 已 `git add`、等待提交的状态 |
| Pull Request | 请求他人把你的分支代码 pull 并合并，简称 PR |

### 分支结构图

```
main          ← 生产环境，稳定代码
  ↑ PR 合并
develop       ← 集成环境，联调用
  ↑ PR 合并
feature/a-session-api    ← Dev A 的功能分支
feature/b-detection-01   ← Dev B 的功能分支
feature/c-agent-runner   ← Dev C 的功能分支
feature/d-landing-page   ← Dev D 的功能分支
```

---

## 2. 首次配置（每人只做一次）

### 2.1 设置个人身份

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
```

### 2.2 配置认证方式（二选一）

**方式 A：HTTPS + Personal Access Token（推荐新手）**

```bash
# 保存凭据，避免每次都输入
git config --global credential.helper store

# 首次 push 时会提示：
# Username: GitHub用户名
# Password: 粘贴 Token（不是登录密码）
```

生成 Token 路径：
```
GitHub 头像 → Settings → Developer settings
→ Personal access tokens → Fine-grained tokens
→ 选择仓库 → 勾选 Contents: Read and write / Pull requests: Read and write
```

**方式 B：SSH 密钥（推荐长期使用）**

```bash
# 生成密钥
ssh-keygen -t ed25519 -C "你的邮箱"

# 查看公钥并复制
cat ~/.ssh/id_ed25519.pub

# 粘贴到 GitHub：Settings → SSH and GPG keys → New SSH key

# 测试是否成功
ssh -T git@github.com
```

### 2.3 克隆仓库

```bash
# HTTPS 方式
git clone https://github.com/corp_name/proj_name.git

# SSH 方式
git clone git@github.com:corp_name/proj_name.git

cd proj_name
```

### 2.4 配置换行符（Windows 用户必做）

```bash
git config --global core.autocrlf true
```

---

## 3. 日常开发工作流

### 完整流程一览

```
拉取最新代码 → 创建/切换分支 → 写代码 → 查看状态
→ 添加文件 → 提交 → 推送到远端 → 发起 PR
```

### 3.1 开始新任务前，先同步代码

```bash
git checkout develop          # 切换到 develop 分支
git pull origin develop       # 拉取最新代码
```

### 3.2 创建自己的功能分支

```bash
# 命名规范：feature/{角色}-{模块简称}
git checkout -b feature/a-session-api     # Dev A 示例
git checkout -b feature/b-detection-01    # Dev B 示例
git checkout -b feature/d-landing-page    # Dev D 示例
```

### 3.3 写代码，然后提交

```bash
# 查看哪些文件有改动
git status

# 查看具体改了什么内容
git diff

# 添加所有改动文件
git add .

# 或只添加指定文件
git add backend/internal/session/handler.go

# 确认已添加哪些文件（绿色 = 已 add）
git status

# 查看将要提交的具体改动
git diff --staged

# 提交（写清楚做了什么）
git commit -m "feat(session): 实现 POST /sessions 创建匿名会话"
```

### 3.4 推送到远端

```bash
# 第一次推送新分支
git push -u origin feature/a-session-api

# 后续推送（已建立追踪关系后）
git push
```

---

## 4. 分支管理

### 4.1 查看分支

```bash
git branch                  # 查看本地所有分支
git branch -r               # 查看远端所有分支
git branch -a               # 查看全部（本地 + 远端）
```

### 4.2 切换分支

```bash
git checkout develop                      # 切换到已有分支
git checkout -b feature/a-new-module      # 创建并切换到新分支
```

### 4.3 删除分支

```bash
# PR 合并后，删除本地分支
git branch -d feature/a-session-api

# 同时删除远端分支
git push origin --delete feature/a-session-api
```

### 4.4 分支命名规范

| 类型 | 格式 | 示例 |
|------|------|------|
| 功能开发 | `feature/{dev}-{module}` | `feature/a-session-api` |
| Bug 修复 | `fix/{dev}-{描述}` | `fix/b-hmac-timeout` |
| 紧急热修 | `hotfix/{描述}` | `hotfix/c2-sink-leak` |

---

## 5. 提交规范

### 格式

```
{类型}({范围}): {简短描述}

{可选：详细说明}
```

### 类型对照表

| 类型 | 含义 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(agent): 实现心跳保活接口` |
| `fix` | Bug 修复 | `fix(hmac): 修复 nonce 重放漏洞` |
| `test` | 添加测试 | `test(session): 补充边界条件测试` |
| `refactor` | 重构（不改功能）| `refactor(sink): 提取公共日志方法` |
| `docs` | 文档更新 | `docs: 更新 API 合约说明` |
| `chore` | 构建/配置变更 | `chore: 更新 docker-compose 配置` |
| `style` | 代码格式调整 | `style(frontend): 统一缩进格式` |

### 好的提交 vs 差的提交

```bash
# ✅ 好：清楚说明做了什么
git commit -m "feat(reconcile): 实现双侧联合判定四种组合逻辑"

# ✅ 好：带详细说明
git commit -m "fix(ratelimit): 修复 Redis 滑动窗口边界计算错误

原逻辑在整点附近会出现计数重置，导致速率限制失效。
修改为基于当前时间戳的严格滑动窗口。
Fixes #42"

# ❌ 差：无法理解做了什么
git commit -m "修改"
git commit -m "update"
git commit -m "fix bug"
```

---

## 6. Pull Request 流程

### 6.1 发起 PR

```bash
# 推送分支到远端
git push -u origin feature/a-session-api

# 然后在 GitHub 网页：
# 1. 点击仓库页面出现的 "Compare & pull request" 按钮
# 2. 填写标题和描述
# 3. 选择合并目标：feature → develop（日常），develop → main（发版）
# 4. 指定 Reviewer（至少 1 人）
# 5. 点击 Create pull request
```

### 6.2 PR 描述模板（建议团队统一使用）

```markdown
## 做了什么
简述本次 PR 的改动内容

## 关联任务
- Dev A 任务 A-2：匿名会话管理

## 测试情况
- [ ] 单元测试通过
- [ ] 本地 Docker Compose 启动正常
- [ ] 已用 mock 数据验证接口响应

## 注意事项
其他人 review 时需要关注的点
```

### 6.3 Review 与合并

```
发起者推送 PR
    ↓
指定的 Reviewer 收到通知
    ↓
Reviewer 在 GitHub 上 review 代码，可以：
  - Approve（通过）
  - Request changes（要求修改）
  - Comment（只评论不决定）
    ↓
发起者根据意见修改后再次 push（自动更新 PR）
    ↓
Reviewer Approve → 发起者点击 Merge
    ↓
删除已合并的功能分支（GitHub 上有快捷按钮）
```

### 6.4 安全敏感代码必须交叉 Review

根据项目规约，以下模块必须由**非原作者**进行 Review：

- HMAC 签名验证逻辑
- C2 Sink 隔离代码（严禁出现 `http.Client`、`net.Dial`）
- Agent 沙箱与清理机制
- 速率限制实现

---

## 7. 同步与冲突处理

### 7.1 同步远端最新代码

```bash
# 方式一：fetch + merge（推荐，可以先看改动再合并）
git fetch origin
git merge origin/develop

# 方式二：pull（直接拉取并合并）
git pull origin develop
```

### 7.2 保持功能分支与 develop 同步

长时间开发时，develop 可能已有其他人的新代码，建议定期同步：

```bash
git checkout develop
git pull origin develop
git checkout feature/a-session-api
git merge develop               # 把 develop 的新内容合入你的分支
```

### 7.3 处理合并冲突

```bash
# 执行 merge 后，如果出现冲突：
git status
# 会看到 "both modified: xxx.go" 这样的提示

# 打开冲突文件，找到冲突标记：
# <<<<<<< HEAD
# 你的代码
# =======
# 别人的代码
# >>>>>>> develop

# 手动编辑，保留正确内容，删除冲突标记

# 解决后标记为已解决
git add 冲突文件名

# 完成合并提交
git commit -m "merge: 合并 develop 并解决冲突"
```

### 7.4 rebase（保持提交历史整洁，进阶用法）

```bash
# 用 rebase 替代 merge，让提交历史更线性
git checkout feature/a-session-api
git rebase develop

# 注意：rebase 后需要强制推送
git push --force-with-lease
# 使用 --force-with-lease 而非 --force，更安全
```

---

## 8. 撤销与回滚

### 8.1 撤销 git add（还没有 commit）

```bash
# 撤销某个文件的 add
git restore --staged handler.go

# 撤销所有 add
git restore --staged .
```

### 8.2 撤销工作区修改（还没有 add）

```bash
# 丢弃某个文件的修改（慎用，不可恢复）
git restore handler.go

# 丢弃所有未 add 的修改（慎用）
git restore .
```

### 8.3 撤销最后一次 commit（还没有 push）

```bash
# 保留代码改动，只撤销提交记录
git reset --soft HEAD~1

# 保留文件但撤销 add 和提交
git reset HEAD~1

# 彻底丢弃（慎用，代码也没了）
git reset --hard HEAD~1
```

### 8.4 撤销已经 push 的提交（安全方式）

```bash
# 用 revert 生成一个"反向提交"，不破坏历史记录
git revert <commit-hash>
git push
```

### 8.5 找回误删的内容

```bash
# 查看所有操作记录（包括已删除的提交）
git reflog

# 恢复到某个状态
git checkout <reflog中的hash>
```

---

## 9. 查看与检索

### 9.1 查看状态和改动

```bash
git status                    # 查看哪些文件有改动
git diff                      # 查看未 add 的改动详情
git diff --staged             # 查看已 add 但未 commit 的改动
git diff develop..feature/a-session-api  # 对比两个分支的差异
```

### 9.2 查看提交历史

```bash
git log                       # 完整历史
git log --oneline             # 每条提交一行，简洁版
git log --oneline --graph     # 带分支图形的历史
git log --oneline -10         # 只看最近 10 条
git log --author="Dev A"      # 只看某人的提交
```

### 9.3 查看某次提交的内容

```bash
git show <commit-hash>        # 查看某次提交改了什么
git show HEAD                 # 查看最新一次提交
```

### 9.4 搜索代码历史

```bash
# 搜索某个关键词在哪个提交里被引入
git log -S "HMAC_SECRET"

# 搜索提交信息中包含某关键词的提交
git log --grep="reconcile"
```

---

## 10. 常见问题 FAQ

### Q1：push 时提示 "rejected"，怎么办？

```bash
# 通常是远端有新提交，先拉取再推送
git pull origin feature/a-session-api
# 解决可能的冲突后
git push
```

### Q2：不小心在 main/develop 上直接改了代码，怎么转移到功能分支？

```bash
# 不要 commit！先把改动暂存
git stash

# 切换到正确的分支
git checkout feature/a-session-api

# 取出暂存的改动
git stash pop
```

### Q3：stash 是什么？

`git stash` 是一个临时"抽屉"，可以把当前未提交的改动暂时收起来，切换分支后再取出来继续工作。

```bash
git stash           # 收起当前改动
git stash list      # 查看所有暂存
git stash pop       # 取出最近一次暂存（并删除）
git stash apply     # 取出但不删除
```

### Q4：clone 下来的仓库怎么看有哪些分支？

```bash
git branch -a       # 查看所有分支（含远端）

# 切换到远端分支
git checkout -b feature/b-detection-01 origin/feature/b-detection-01
```

### Q5：如何查看某个文件是谁修改的？

```bash
git blame backend/internal/session/handler.go
# 每一行都会显示：提交 hash、作者、时间
```

### Q6：误删了一个文件怎么恢复？

```bash
# 如果还没有 commit
git restore 误删文件名

# 如果已经 commit 了
git show HEAD:误删文件路径 > 恢复的文件名
```

### Q7：`.gitignore` 加了规则但还是追踪了某些文件？

```bash
# 已经被追踪的文件，需要先从 Git 缓存中移除
git rm -r --cached node_modules/
git commit -m "chore: 移除已追踪的 node_modules"
# 之后 .gitignore 的规则就会生效
```

---

## 附录：高频命令速查卡

```bash
# === 每天都用 ===
git status                          # 查看状态
git pull origin develop             # 同步最新代码
git add .                           # 添加所有改动
git commit -m "feat: xxx"           # 提交
git push                            # 推送

# === 分支操作 ===
git checkout -b feature/xxx         # 创建并切换分支
git checkout develop                # 切换分支
git branch -d feature/xxx          # 删除本地分支

# === 查看 ===
git log --oneline --graph           # 图形化历史
git diff --staged                   # 检查将要提交的内容
git blame 文件名                    # 查看每行作者

# === 救急 ===
git stash                           # 临时收起改动
git stash pop                       # 取出改动
git restore --staged .              # 撤销所有 add
git reset --soft HEAD~1             # 撤销最后一次 commit（保留代码）
git reflog                          # 找回任何误操作
```

---

*本文档基于 proj_name 项目实际协作流程整理，适用于 4 人 Monorepo 开发场景。*
