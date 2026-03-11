运行项目的全面验证。

按顺序执行以下命令并报告结果：

## 1. 代码检查（Linting）

```bash
# Python (ruff):
cd backend && uv run ruff check .

# JavaScript/TypeScript (eslint):
cd frontend && npm run lint
```

**预期：** 无 linting 错误

## 2. 类型检查（如适用）

```bash
# Python (mypy):
cd backend && uv run mypy app/

# TypeScript:
cd frontend && npm run typecheck
```

**预期：** 无类型错误

## 3. 单元测试

```bash
# Python (pytest):
cd backend && uv run pytest -v

# JavaScript (vitest/jest):
cd frontend && npm test
```

**预期：** 所有测试通过

## 4. 测试覆盖率

```bash
# Python:
cd backend && uv run pytest --cov=app --cov-report=term-missing

# JavaScript:
cd frontend && npm run test:coverage
```

**预期：** 覆盖率满足项目阈值

## 5. 构建

```bash
cd frontend && npm run build
```

**预期：** 构建成功完成

## 6. 总结报告

所有验证完成后，提供包含以下内容的总结报告：

- Linting 状态
- 类型检查状态（如适用）
- 测试通过/失败
- 覆盖率百分比
- 构建状态
- 遇到的任何错误或警告
- 整体健康评估（通过/失败）

**使用清晰的部分和状态指示符格式化报告**
