---
name: daily-ship
description: 每日收尾流程——检查代码→同步进度文档→本地提交代码（排除SQL/文档）→拉取合并线上→推送
argument-hint: "[仓库路径，可多个，默认当前目录所在仓库]"
allowed-tools:
  - read
  - grep
  - glob
  - exec
  - edit
  - write
---

对涉及改动的每个 git 仓库执行以下四步收尾流程。默认作用于当前工作目录所在的仓库；若用户指定了多个仓库路径则逐个执行。全程用中文向用户汇报，每仓一段。

## 第 1 步：检查代码

- `git status --short` 列出全部改动；`git diff` 和 `git diff --staged` 审阅内容。
- 检查项：
  - 调试残留：`console.log`、`System.out.println`、`debugger`、临时注释掉的代码、TODO 标记
  - 敏感信息：密码、密钥、token、内网地址硬编码
  - 环境指向改动：如 `application-dev.yml` 指向 `127.0.0.1` 这类本机配置——标记出来提醒用户，默认不提交
  - 明显的逻辑错误、吞掉的异常、未关闭的资源
- 汇报发现的问题清单。若有严重问题（疑似 bug、敏感信息），停下来问用户是否继续，不要直接进下一步。

## 第 2 步：同步进度文档

- 定位进度文档：默认取工作区根目录 `project progress/` 下最新的 `*-最新工作计划-*.txt`；用户另行指定时按指定路径。
- 按文件既有格式（日期分节登记）追加今日条目：完成了什么、关键提交 hash、遗留事项。
- 该文件不入 git——只修改文件，不要暂存或提交它。

## 第 3 步：本地提交代码（SQL 和文档不上传）

- 逐仓库提交，**只暂存代码文件**。排除规则：
  - `*.sql`、`sql/` 目录下的一切
  - `*.md`、`docs/` 目录
  - `project progress/`、进度文档
  - 第 1 步标记的本机环境配置改动（除非用户明确要提交）
- 排除用 `git add` 精确列文件或 `git add . && git reset <排除路径>` 实现，不要改 `.gitignore`。
- 提交信息遵循该仓 `git log --oneline -10` 的既有风格，并附加 trailer：

  ```
  Generated with [Devin](https://devin.ai)

  Co-Authored-By: Devin <158243242+devin-ai-integration[bot]@users.noreply.github.com>
  ```
- 改动跨多个关注点时拆成多个语义化提交。
- 工作区干净（没有可提交的代码改动）则跳过此步并说明。

## 第 4 步：拉取线上、合并、推送

- `git pull` 拉取当前分支的线上更新（默认 merge；仓库若惯例 rebase 则 `git pull --rebase`）。
- 有冲突时：逐个打开冲突文件，理解双方意图后合并，保留双方合理改动；解决后 `git add` 并完成 merge/rebase。
- 冲突复杂、涉及他人较大改动或取舍不明确时，列出冲突点和建议方案请用户裁决，不要擅自选边丢弃改动。
- 确认本地干净后 `git push` 到 origin 当前分支。
- push 被 protected-branch 拒绝时停下来告诉用户，不要 force push。

## 汇报格式

每个仓库输出一段：

- **检查**：发现的问题（或"无问题"）
- **文档**：进度文档追加的内容摘要
- **提交**：本次产生的 commit hash + 标题列表
- **同步**：pull 结果（已最新/合并 N 个提交/冲突已解决）、push 结果
