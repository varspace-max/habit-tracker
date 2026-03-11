---
description: 根据 GitHub issue 的 RCA 文档实施修复
argument-hint: [github-issue-id]
allowed-tools: Read, Write, Edit, Bash(ruff:*), Bash(mypy:*), Bash(pytest:*), Bash(npm:*), Bash(bun:*)
---

# 实施修复：GitHub Issue #$ARGUMENTS

## 前提条件

**此命令根据 RCA 文档实施 GitHub issues 的修复：**
- 在具有 GitHub origin 的本地 Git 仓库中工作
- RCA 文档存在于 `docs/rca/issue-$ARGUMENTS.md`
- GitHub CLI 已安装并通过身份验证（可选，用于状态更新）

## 要参考的 RCA 文档

读取 RCA：`docs/rca/issue-$ARGUMENTS.md`

**可选 - 查看 GitHub issue 以获取上下文：**
```bash
gh issue view $ARGUMENTS
```

## 实施说明

### 1. 阅读并理解 RCA

- 彻底阅读整个 RCA 文档
- 查看 GitHub issue 详情（issue #$ARGUMENTS）
- 理解根本原因
- 审查建议的修复策略
- 记下所有要修改的文件
- 审查测试要求

### 2. 验证当前状态

在做出更改之前：
- 确认问题仍然存在
- 检查受影响文件的当前状态
- 审查这些文件的任何最近更改

### 3. 实施修复

按照 RCA 的"建议修复"部分：

**对于每个要修改的文件：**

#### a. 阅读现有文件
- 理解当前实现
- 找到 RCA 中提到的具体代码

#### b. 进行修复
- 按照 RCA 中的描述进行更改
- 严格遵循修复策略
- 保持代码风格和约定
- 如果修复不明显，添加注释

#### c. 处理相关更改
- 更新受修复影响的任何相关代码
- 确保整个代码库的一致性
- 如需要，更新导入

### 4. 添加/更新测试

按照 RCA 中的"测试要求"：

**创建测试用例以：**
1. 验证修复解决了问题
2. 测试与 bug 相关的边界情况
3. 确保相关功能没有回归
4. 测试任何引入的新代码路径

**测试文件位置：**
- 遵循项目的测试结构
- 镜像源文件位置
- 使用描述性测试名称

**测试实现：**
```python
def test_issue_$ARGUMENTS_fix():
    """测试 issue #$ARGUMENTS 已修复。"""
    # Arrange - 设置导致 bug 的场景
    # Act - 执行之前失败的代码
    # Assert - 验证现在正确工作
```

### 5. 运行验证

执行 RCA 中的验证命令：

```bash
# 运行 linter
[来自 RCA 的验证命令]

# 运行类型检查
[来自 RCA 的验证命令]

# 运行测试
[来自 RCA 的验证命令]
```

**如果验证失败：**
- 修复问题
- 重新运行验证
- 在全部通过之前不要继续

### 6. 验证修复

**手动验证：**
- 按照 RCA 中的复现步骤
- 确认问题不再发生
- 测试边界情况
- 检查是否有意外的副作用

### 7. 更新文档

如需要：
- 更新代码注释
- 更新 API 文档
- 更新 README（如果面向用户）
- 添加关于修复的说明

## 输出报告

### 修复实施摘要

**GitHub Issue #$ARGUMENTS**：[简要标题]

**Issue URL**：[GitHub issue URL]

**根本原因**（来自 RCA）：
[根本原因的一行总结]

### 所做的更改

**修改的文件：**
1. **[文件路径]**
   - 更改：[更改了什么]
   - 行号：[行号]

2. **[文件路径]**
   - 更改：[更改了什么]
   - 行号：[行号]

### 添加的测试

**创建/修改的测试文件：**
1. **[测试文件路径]**
   - 测试用例：[添加的测试函数列表]

**测试覆盖：**
- ✅ 修复验证测试
- ✅ 边界情况测试
- ✅ 回归预防测试

### 验证结果

```bash
# Linter 输出
[显示 lint 结果]

# 类型检查输出
[显示类型检查结果]

# 测试输出
[显示测试结果 - 全部通过]
```

### 验证

**手动测试：**
- ✅ 遵循复现步骤 - 问题已解决
- ✅ 测试边界情况 - 全部通过
- ✅ 没有引入新问题
- ✅ 原始功能保留

### 文件摘要

**总更改：**
- X 个文件已修改
- Y 个文件已创建（测试）
- Z 行已添加
- W 行已删除

### 准备提交

所有更改已完成并验证。准备使用：
```bash
/commit
```

**建议的提交消息：**
```
fix(scope): 解决 GitHub issue #$ARGUMENTS - [简要描述]

[修复内容和方式的总结]

Fixes #$ARGUMENTS
```

**注意：** 在提交消息中使用 `Fixes #$ARGUMENTS` 将在合并到默认分支时自动关闭 GitHub issue。

### 可选：更新 GitHub Issue

**向 issue 添加实施评论：**
```bash
gh issue comment $ARGUMENTS --body "已在提交 [commit-hash] 中实施修复。准备审查。"
```

**更新 issue 标签（如需要）：**
```bash
gh issue edit $ARGUMENTS --add-label "fixed" --remove-label "bug"
```

**关闭 issue（如不使用通过提交消息自动关闭）：**
```bash
gh issue close $ARGUMENTS --comment "已修复并合并。"
```

## 说明

- 如果 RCA 文档缺失或不完整，请求先使用 `/rca $ARGUMENTS` 创建
- 如果发现 RCA 分析不正确，记录发现并更新 RCA
- 如果在实施过程中发现其他问题，为单独的 GitHub issues 和 RCAs 记录它们
- 严格遵循项目编码标准
- 确保所有验证在声明完成之前通过
- 提交消息 `Fixes #$ARGUMENTS` 将把提交链接到 GitHub issue
