---
name: merge-branches
description: 安全、可审计地把一个 Git 源分支合并到目标分支，包含分支关系检查、工作区保护、冲突分类与解决、验证和结果报告。用户要求 merge 分支、把功能分支合入 main/develop/release、同步分支后整合代码，或需要处理 merge conflict 时使用。
---

# Merge Branches

将源分支的提交整合到目标分支，并把过程推进到一个可验证的结果。合并是有状态操作：先确认边界和仓库状态，再改变目标分支；任何无法证明安全的情况都停下来报告，不用破坏性命令“强行修好”。

## 先确定意图

从用户请求中提取：

- `source`：要被合入的分支。
- `target`：接收改动的分支。
- 是否需要从远程刷新（默认不擅自 fetch；如果用户说“最新远程代码”，才刷新对应 remote）。
- 合并策略：`--ff-only`、默认 fast-forward/普通 merge，或 `--no-ff`。未指定时先根据仓库历史和现有提交惯例判断；不能猜时询问。
- 冲突偏好和受保护路径。默认逐文件人工判断，禁止默认全选 ours/theirs。

若 `source` 或 `target` 缺失，先询问；不要把当前分支或 `origin/HEAD` 当成隐含答案。明确记录方向：`source -> target`。禁止把目标分支 checkout 错后再 merge。

## 工作流

### 1. 只读预检

在仓库根目录执行并记录：

```bash
git rev-parse --show-toplevel
git status --short --branch
git branch --show-current
git log --oneline --decorate -n 8
git remote -v
```

拒绝在以下状态直接合并：

- 工作区或暂存区有未提交改动（包括与本次任务无关的改动）。不要 stash、reset、clean 或覆盖它们；先让用户提交、另开 worktree，或明确授权安全的暂存方案。
- 已存在 merge、rebase、cherry-pick 或 revert 进行中状态。先报告状态，不能嵌套操作。
- `source` 与 `target` 相同、分支不存在、或解析到的 ref 与用户意图不一致。

确认 refs 和关系：

```bash
git rev-parse --verify <source>^{commit}
git rev-parse --verify <target>^{commit}
git merge-base <source> <target>
git log --graph --oneline --decorate --left-right <target>...<source>
git diff --stat <target>...<source>
```

先判断结果类型：

- `source` 已是 `target` 的祖先：无需 merge，报告“已经包含”。
- `target` 是 `source` 的祖先：可以 fast-forward；只有用户或仓库惯例要求保留合并节点时才使用 `--no-ff`。
- 两边均有独有提交：是真正的三方 merge，预期可能冲突。
- 无共同祖先：这是 unrelated histories；禁止自动加 `--allow-unrelated-histories`，先向用户确认这确实是目标。

### 2. 必要时刷新远程

只有用户要求最新远程状态，或明确允许同步时才执行：

```bash
git fetch <remote> --prune
```

fetch 后重新解析并记录实际使用的 commit SHA。不要把远程分支名的旧本地 tracking ref 当成最新状态。

### 3. 切换并合并

在预检确认干净后切换到目标分支：

```bash
git switch <target>
```

再次检查 `git status --short --branch`，然后执行用户确认过的策略：

```bash
# 仅在可证明为纯快进时
git merge --ff-only <source>

# 普通合并（未指定策略且需要创建合并提交时）
git merge --no-edit <source>

# 仓库明确要求保留合并节点时
git merge --no-ff --no-edit <source>
```

不要加 `--no-verify`、`-X theirs`、`-X ours` 或 `--allow-unrelated-histories` 来绕过检查。若仓库有 hooks、生成步骤或提交模板，保留其默认行为。

### 4. 处理冲突

发生冲突后不要继续执行无关命令，先收集：

```bash
git status --short
git diff --name-only --diff-filter=U
git diff --merge
```

对每个冲突文件分类并单独处理：

- 内容冲突：阅读冲突上下文、共同祖先、ours 和 theirs，按业务不变量合并。
- 删除/修改、重命名/重命名：明确决定保留、删除或重新命名，不能靠批量 checkout 猜测。
- 二进制或生成文件：依据仓库生成规则和源文件决定版本；能重新生成就重新生成并验证。
- 测试、配置、迁移文件：优先检查兼容性、默认值、回滚路径和环境差异。

可用以下只读命令比较三方版本：

```bash
git show :1:<path>   # common ancestor
git show :2:<path>   # ours: target
git show :3:<path>   # theirs: source
```

解决后检查没有残留标记，逐文件暂存：

```bash
git diff --check
git diff --cached --check
git add <resolved-path>
git status --short
```

在继续前运行与改动相关的最小测试或静态检查。确认所有冲突已暂存后使用：

```bash
git merge --continue
```

如果 Git 版本或 hook 不接受 `--continue`，使用 `git commit`，但先确认提交信息明确说明这是一次 merge。不要自动 `git merge --abort`；只有用户明确要求撤销当前未完成合并时才执行。

### 5. 完成验证

合并完成后必须检查：

```bash
git status --short --branch
git diff --check HEAD^1 HEAD
git log -1 --summary --decorate
git merge-base --is-ancestor <source> <target>
```

然后按项目类型执行仓库已有的聚焦测试、lint、build 或生成步骤。至少验证本次冲突文件和跨分支公共接口；不要把“命令退出成功”误报成完整运行时验证。若是 fast-forward，使用合并前记录的 target SHA 对比：`git diff <old-target-sha> <target>`。

## 安全边界

- 不使用 `git reset --hard`、`git clean -fd`、`git checkout -- .`、批量 `git checkout --ours/--theirs` 或隐式 stash。
- 不删除分支、不强推远程、不改写已发布历史；这些是独立请求。
- 不把冲突数量当成失败原因；报告具体文件、冲突类型、选择依据和未验证风险。
- 不因测试失败擅自回滚或修改无关代码。先区分合并引入的失败、既有失败和环境/依赖失败。
- 发现 merge 结果不符合意图时，保留现场并停止；给出恢复选项。已提交的普通 merge 优先用 `git revert -m 1 <merge-commit>`，不要建议无依据 reset。

## 交付报告

用简短但可审计的格式汇报：

1. 方向：`<source> -> <target>`，实际 commit SHA 和使用的 merge 策略。
2. 结果：已包含、fast-forward、创建 merge commit，或停在冲突/预检阶段。
3. 冲突：按文件列出解决方式；无冲突则明确写“无冲突”。
4. 验证：执行的命令及成功/失败；标明静态、聚焦测试、构建或运行时验证边界。
5. 后续动作：仅列出必要且未执行的动作，不隐藏风险。
