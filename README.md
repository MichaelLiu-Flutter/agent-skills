# Agent Skills

可复用的 AI 编程技能。当前包含 `code-design`（代码设计规范）和 `merge-branches`（Git 分支合并）。

## 安装

需要 Node.js 和 npm。运行下列命令，按提示选择使用的 AI 编程工具和安装范围：

```bash
npx skills add MichaelLiu-Flutter/agent-skills --skill code-design
npx skills add MichaelLiu-Flutter/agent-skills --skill merge-branches
```

安装工具支持 Codex、Claude Code、Cursor 等工具；具体兼容性以 [skills CLI](https://github.com/vercel-labs/skills) 当前说明为准。安装后检查目标工具是否识别技能；必要时重新打开会话。

## code-design 能做什么

- 基于代码证据，检查职责、依赖、状态、业务规则和外部接口。
- 使用 SOLID、KISS、YAGNI、DRY 等原则判断设计，而不是机械套用。
- 根据实际问题选择设计模式，比较简单替代并解释新增成本。
- 检查异步结果归属、资源释放、错误处理和接口兼容性。
- 区分方案、只读审查和实施任务，避免扩大修改范围。

适用于多种语言与架构，不强制使用类、固定层数或所有设计模式。普通文案、格式和简单修复无需启动完整设计流程。

技能默认使用中文说明；项目规则和用户明确要求优先。

## 使用示例

Codex 中可以通过 `$code-design` 明确调用：

```text
使用 $code-design 审查这个模块的职责和依赖。
先给出有代码证据的问题与改进方案，不修改代码。
```

```text
使用 $code-design 实现这个功能。
遵守当前项目架构，优先复用现有能力，验证关键行为。
```

```text
使用 $code-design 重构这段代码，保持业务行为不变。
重点解决职责混杂和重复规则，不增加没有实际收益的抽象。
```

其他工具的调用语法可能不同，请使用各自的技能入口。

## merge-branches 能做什么

- 明确源分支和目标分支，检查提交关系和工作区状态。
- 根据分支关系选择快进或普通合并，逐文件处理冲突。
- 保护未提交改动，禁止用强推、清空工作区等方式绕过问题。
- 完成相关验证，并报告合并结果、冲突处理和未验证风险。

Codex 中可以通过 `$merge-branches` 明确调用，例如：

```text
使用 $merge-branches，把 feature/login 合并到 develop。
先检查工作区和分支关系，处理冲突后运行相关检查。
```

未说明源分支或目标分支时，技能会先询问。推送、删除分支和改写历史需要单独授权。

## 内容

- [分支合并技能入口](skills/merge-branches/SKILL.md)
- [代码设计技能入口](skills/code-design/SKILL.md)
- [设计目标与原则](skills/code-design/references/goals-and-principles.md)
- [设计模式决策](skills/code-design/references/pattern-decisions.md)
- [工作流程与验证](skills/code-design/references/workflow-and-validation.md)
- [案例与自检](skills/code-design/references/examples-and-self-check.md)

## 更新

```bash
npx skills update code-design
npx skills update merge-branches
```

## 当前验证范围

初始版本已通过技能结构校验和本地参考链接检查。安装验证结果见发行说明。尚未完成跨工具的独立行为评测；技能不保证每次模型决策正确，也不代替项目测试与代码审查。

问题与建议请通过 GitHub Issues 提交，并说明任务、预期结果和实际结果。

## 许可证

[MIT](LICENSE)。
