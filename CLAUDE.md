# 工作区模板

这是一个用于 Claude Code 工作坊的模板仓库。它包含了使用 Claude Code 构建全栈应用程序所需的脚手架和斜杠命令。

## 包含内容

- **`.claude/commands/`** - 用于规划、执行、验证和工作流自动化的斜杠命令
- **`.claude/reference/`** - 各种技术的最佳实践文档
- **`CLAUDE.md`** - 项目特定指令的模板（在构建时填写）

## 入门指南

1. **Fork 或克隆**此仓库
2. **定义您的项目** — 与您的 AI 编码助手讨论您想要构建的内容。讨论需求、功能和技术决策。
3. **创建您的 PRD** — 运行 `/create-prd` 根据您的对话生成产品需求文档
4. **构建您的 CLAUDE.md** — 与 AI 助手合作，填写 `CLAUDE.md` 模板，包括项目的技术栈、结构、规范和命令
5. **创建参考文档** — 在 `.claude/reference/` 中添加特定代码库部分的详细指南（例如 API 模式、数据库约定、部署策略）。保持您的 `CLAUDE.md` 简洁并在需要时指向这些参考文档 — 这可以防止 LLM 上下文过载，同时在处理特定区域时仍能提供详细指导
6. **开始构建** — 使用开发工作流程：
   - `/core_piv_loop:prime` — 加载项目上下文
   - `/core_piv_loop:plan-feature` — 为功能创建实施计划
   - `/core_piv_loop:execute` — 逐步执行计划

   这些命令遵循 **PIV Loop**（Prime → Implement → Validate）工作流程：

   ![PIV Loop 流程图](PIVLoopDiagram.png)

## Claude 命令

用于协助开发工作流程的 Claude Code 斜杠命令。

### 规划与执行
| 命令 | 描述 |
|---------|-------------|
| `/core_piv_loop:prime` | 加载项目上下文和代码库理解 |
| `/core_piv_loop:plan-feature` | 通过代码库分析创建全面的实施计划 |
| `/core_piv_loop:execute` | 逐步执行实施计划 |

### 验证
| 命令 | 描述 |
|---------|-------------|
| `/validation:validate` | 运行完整验证：测试、linting、覆盖率、构建（根据您的项目定制） |
| `/validation:code-review` | 对已更改的文件进行技术代码审查 |
| `/validation:code-review-fix` | 修复代码审查中发现的问题 |
| `/validation:execution-report` | 在实现功能后生成报告 |
| `/validation:system-review` | 分析实施与计划的偏差以改进流程 |

### Bug 修复
| 命令 | 描述 |
|---------|-------------|
| `/github_bug_fix:rca` | 为 GitHub issue 创建根本原因分析文档 |
| `/github_bug_fix:implement-fix` | 根据 RCA 文档实施修复 |

### 其他
| 命令 | 描述 |
|---------|-------------|
| `/commit` | 创建带有适当标签（feat、fix、docs 等）的原子提交 |
| `/init-project` | 安装依赖并启动开发服务器（根据您的项目定制） |
| `/create-prd` | 根据对话生成产品需求文档 |
