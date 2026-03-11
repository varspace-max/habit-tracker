# 初始化项目

在本地设置和启动项目。

## 1. 安装后端依赖

```bash
# Python 示例（使用 uv）：
cd backend && uv sync

# Node.js 示例：
cd backend && npm install
```

## 2. 安装前端依赖

```bash
cd frontend && npm install
```

## 3. 启动后端服务器

```bash
# Python/FastAPI 示例：
cd backend && uv run uvicorn app.main:app --reload --port 8000

# Node.js/Express 示例：
cd backend && npm run dev
```

## 4. 启动前端服务器

```bash
cd frontend && npm run dev
```

## 5. 验证设置

```bash
# 测试 API 是否响应
curl -s http://localhost:8000/health

# 或检查主端点
curl -s http://localhost:8000/api/...
```

## 访问点

- **前端**：http://localhost:5173（或您配置的端口）
- **后端 API**：http://localhost:8000（或您配置的端口）
- **API 文档**：http://localhost:8000/docs（如果使用 FastAPI）

## 说明

<!-- 在此添加项目特定的初始化说明 -->
