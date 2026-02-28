---
description: 审查代码的质量、安全性和可维护性
agent: code-reviewer
subtask: true
---

# 代码审查命令

审查代码变更的质量、安全性和可维护性：$ARGUMENTS

## 检测审查场景

根据参数自动检测审查场景（优先级：PR > Commit > Branch > Local）：

!`if echo "$ARGUMENTS" | grep -qE "^[0-9]+$"; then echo "PR";
   elif echo "$ARGUMENTS" | grep -qE "^[0-9a-f]{7,}$"; then echo "COMMIT";
   elif [ -n "$ARGUMENTS" ]; then echo "BRANCH";
   else echo "LOCAL"; fi`

当前检测到的场景：**[根据命令输出显示场景]**

## 上下文收集

审查代码时，按照以下步骤收集上下文：

1. **识别修改的文件** — 根据检测到的场景运行相应的 Git 命令：
   - **PR**: 运行 `gh pr view $PR_NUMBER` 和 `gh pr diff $PR_NUMBER`
   - **Commit**: 运行 `git show $COMMIT_HASH`
   - **Branch**: 运行 `git diff $BRANCH...HEAD`
   - **Local**: 运行 `git diff`、`git diff --cached` 和 `git status --short`

2. **检查约定文件** — 查看是否存在以下文件，并理解项目约定：
   - AGENTS.md
   - .editorconfig
   - 任何其他约定文件（如 CONVENTIONS.md）

3. **读取完整文件** — 对于每个修改的文件：
   - 读取完整文件内容，而不仅仅是 diff
   - 理解导入、依赖和调用点
   - 识别现有的模式、控制流和错误处理

4. **理解关联** — 识别这些修改如何与其他代码交互

**重要**："Diffs alone are not enough." 必须读取完整文件理解上下文。

## 你的任务

1. **根据检测到的场景获取代码变更**
   - PR 场景：获取 PR 的 diff 和相关信息，如果 gh CLI 不可用，使用 `git ls-remote origin "refs/pull/$PR_NUMBER/head"` 获取 PR refs
   - Commit 场景：获取指定 commit 的变更
   - Branch 场景：获取与指定分支的差异
   - Local 场景：获取未提交和已暂存的变更

2. **收集上下文** — 按照上述步骤收集项目约定和完整文件内容

3. **将上下文传递给 code-reviewer agent** — agent 将使用以下审查清单分析代码

## 场景处理说明

### PR 场景

如果检测到纯数字参数，将其视为 PR 编号：

- 尝试使用 `gh pr view $PR_NUMBER` 获取 PR 信息
- 使用 `gh pr diff $PR_NUMBER` 获取代码变更
- 如果 gh CLI 不可用，使用 `git ls-remote` 方法获取 PR refs

### Commit 场景

如果检测到 7+ 位十六进制，将其视为 commit hash：

- 运行 `git show $COMMIT_HASH` 查看提交详情和变更
- 如果 commit 不存在，提示用户使用 `git log --oneline` 查看可用的 commit

### Branch 场景

如果检测到其他非空文本，将其视为分支名：

- 运行 `git diff $BRANCH...HEAD` 获取与当前分支的差异
- 如果分支不存在，提示用户使用 `git branch -a` 查看可用的分支

### Local 场景

如果没有参数，审查本地未提交的变更：

- 运行 `git diff` 获取未暂存的变更
- 运行 `git diff --cached` 获取已暂存的变更
- 运行 `git status --short` 识别未跟踪的文件
- 如果没有变更，提示用户审查特定 commit 或分支

---

**重要**：永远不要批准有安全漏洞的代码！
