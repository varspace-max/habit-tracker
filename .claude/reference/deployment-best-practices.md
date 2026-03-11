# 部署最佳实践参考

部署 Python/FastAPI + React 应用程序的简洁参考指南。

---

## 目录

1. [本地开发](#1-本地开发)
2. [生产构建](#2-生产构建)
3. [后端部署](#3-后端部署)
4. [前端部署](#4-前端部署)
5. [Docker](#5-docker)
6. [反向代理（Nginx）](#6-反向代理nginx)
7. [环境与环境与配置)
配置](#7-8. [生产中的数据库](#8-生产中的数据库)
9. [监控与日志](#9-监控与日志)
10. [云平台](#10-云平台)
11. [安全](#11-安全)
12. [单二进制部署](#12-单二进制部署)

---

## 1. 本地开发

### 运行前端和后端

**双终端方法：**

```bash
# 终端 1：后端
cd backend
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# 终端 2：前端
cd frontend
npm install
npm run dev  # 运行在端口 5173
```

### Vite 代理配置

通过 Vite 代理 API 请求来避免 CORS 问题：

```javascript
// frontend/vite.config.js
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
});
```

现在前端代码可以使用相对路径：
```javascript
fetch('/api/habits')  // 代理到 http://localhost:8000/api/habits
```

### 热重载

- **后端**：`uvicorn --reload` 监视文件更改
- **前端**：Vite HMR（热模块替换）内置

### 环境变量

```bash
# backend/.env
DATABASE_URL=sqlite:///./habits.db
DEBUG=true
CORS_ORIGINS=["http://localhost:5173"]

# frontend/.env
VITE_API_URL=/api
```

---

## 2. 生产构建

### 前端构建（Vite）

```bash
cd frontend
npm run build  # 创建 dist/ 文件夹
```

**输出：**
```
dist/
├── index.html
├── assets/
│   ├── index-abc123.js
│   └── index-def456.css
```

### 构建优化

```javascript
// vite.config.js
export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          query: ['@tanstack/react-query'],
        },
      },
    },
  },
});
```

### 包分析

```bash
npm install rollup-plugin-visualizer --save-dev
```

```javascript
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true }),
  ],
});
```

---

## 3. 后端部署

### Uvicorn（推荐大多数情况）

```bash
# 开发
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 生产（多 worker）
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

**Worker 数量**：CPU 核心数用于异步 worker。

### Gunicorn + Uvicorn Workers

```bash
# 安装
pip install gunicorn uvicorn

# 运行
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

**Gunicorn 配置文件：**

```python
# gunicorn.conf.py
import multiprocessing

bind = "0.0.0.0:8000"
workers = multiprocessing.cpu_count()
worker_class = "uvicorn.workers.UvicornWorker"
timeout = 30
keepalive = 5
max_requests = 10000
max_requests_jitter = 1000
```

```bash
gunicorn app.main:app -c gunicorn.conf.py
```

### Systemd 服务

```ini
# /etc/systemd/system/habittracker.service
[Unit]
Description=Habit Tracker API
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/habit-tracker/backend
Environment="PATH=/var/www/habit-tracker/backend/.venv/bin"
ExecStart=/var/www/habit-tracker/backend/.venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable habittracker
sudo systemctl start habittracker
sudo systemctl status habittracker
```

---

## 4. 前端部署

### 选项 1：FastAPI 提供静态文件

```python
# app/main.py
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from starlette.responses import FileResponse
import os

app = FastAPI()

# API 路由优先
@app.get("/api/habits")
async def list_habits():
    pass

# 提供 React 应用
frontend_path = os.path.join(os.path.dirname(__file__), "..", "..", "frontend", "dist")

@app.get("/")
async def serve_react_app():
    return FileResponse(os.path.join(frontend_path, "index.html"))

# 处理客户端路由
@app.exception_handler(404)
async def custom_404_handler(request, exc):
    if not request.url.path.startswith("/api"):
        return FileResponse(os.path.join(frontend_path, "index.html"))
    raise exc

# 在路由之后挂载静态文件
app.mount("/", StaticFiles(directory=frontend_path, html=True), name="static")
```

**优势**：单一部署，无 CORS 问题，基础设施更简单。

### 选项 2：Nginx 提供静态文件

静态资源更好的性能：

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/habit-tracker/frontend/dist;

    # 提供静态文件
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 代理 API 到 FastAPI
    location /api {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 选项 3：CDN/静态托管

将前端部署到 Vercel、Netlify 或 Cloudflare Pages：

```bash
# Vercel
npm install -g vercel
vercel --prod

# Netlify
npm install -g netlify-cli
netlify deploy --prod
```

为独立后端配置 API URL：
```javascript
// frontend/.env.production
VITE_API_URL=https://api.yourdomain.com
```

---

## 5. Docker

### Dockerfile（多阶段构建）

```dockerfile
# backend/Dockerfile
# 阶段 1：构建
FROM python:3.11-slim AS builder

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# 阶段 2：运行时
FROM python:3.11-slim

WORKDIR /app

# 创建非 root 用户
RUN groupadd -r appuser && useradd -r -g appuser appuser

# 从 builder 复制依赖
COPY --from=builder /root/.local /home/appuser/.local
ENV PATH=/home/appuser/.local/bin:$PATH

# 复制应用程序
COPY . .

# 设置所有权
RUN chown -R appuser:appuser /app

# 切换到非 root 用户
USER appuser

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 前端 Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./data/habits.db
    volumes:
      - ./data:/app/data  # 持久化 SQLite 数据库
    restart: unless-stopped

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: unless-stopped
```

### Docker 命令

```bash
# 构建并运行
docker-compose up --build

# 后台运行
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止
docker-compose down

# 重建单个服务
docker-compose up --build backend
```

### 镜像优化技巧

| 技巧 | 影响 |
|-----|--------|
| 使用 slim 基础镜像 | `python:3.11-slim` 是 45MB vs 125MB |
| 多阶段构建 | 镜像小 70%+ |
| 使用 `.dockerignore` | 构建更快 |
| 按更改频率排序层 | 更好的缓存 |
| 合并 RUN 命令 | 更少的层 |

**.dockerignore:**
```
__pycache__
*.pyc
.git
.env
.venv
node_modules
dist
*.md
```

---

## 6. 反向代理（Nginx）

### 基本配置

```nginx
# /etc/nginx/sites-available/habittracker
server {
    listen 80;
    server_name yourdomain.com;

    # 提供 React 静态文件
    root /var/www/habit-tracker/frontend/dist;
    index index.html;

    # 处理 React Router（客户端路由）
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 代理 API 请求到 FastAPI
    location /api {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 缓存静态资源
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 启用站点

```bash
sudo ln -s /etc/nginx/sites-available/habittracker /etc/nginx/sites-enabled/
sudo nginx -t  # 测试配置
sudo systemctl reload nginx
```

### SSL 与 Let's Encrypt

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d yourdomain.com

# 自动续期（已由 certbot 配置）
sudo certbot renew --dry-run
```

**结果**（由 certbot 自动生成）：

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # ... 其余配置
}

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

---

## 7. 环境与配置

### 12-Factor App 原则

| 因素 | 应用 |
|--------|-------------|
| 配置 | 环境变量 |
| 依赖 | requirements.txt / package.json |
| 进程 | 无状态应用 |
| 端口绑定 | 应用绑定到端口 |
| 日志 | 流式输出到 stdout |
| 开发/生产 parity | 使用 Docker |

### Pydantic 设置

```python
# app/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache

class Settings(BaseSettings):
    app_name: str = "Habit Tracker"
    database_url: str = "sqlite:///./habits.db"
    debug: bool = False
    cors_origins: list[str] = ["http://localhost:5173"]

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
    )

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### 环境文件

```bash
# .env.development
DATABASE_URL=sqlite:///./habits.db
DEBUG=true
CORS_ORIGINS=["http://localhost:5173"]

# .env.production
DATABASE_URL=sqlite:///./data/habits.db
DEBUG=false
CORS_ORIGINS=["https://yourdomain.com"]
```

### 密钥管理（生产）

| 环境 | 解决方案 |
|-------------|----------|
| 本地 | `.env` 文件（gitignore） |
| Docker | 环境变量 / secrets |
| 云 | 平台 secrets（Railway、Fly.io） |
| 企业 | HashiCorp Vault、AWS Secrets Manager |

---

## 8. 生产中的数据库

### SQLite 注意事项

**SQLite 适用场景**：
- 单服务器部署
- 低写入并发
- 数据库 < 1TB
- 本地/个人应用

**何时迁移到 PostgreSQL**：
- 多服务器/负载均衡
- 高写入并发
- 需要复制/高可用

### 数据库文件位置

```python
# 不要存储在应用程序目录
# 不好
DATABASE_URL = "sqlite:///./habits.db"

# 好 - 绝对路径，在应用外
DATABASE_URL = "sqlite:////var/data/habit-tracker/habits.db"
```

### Docker 卷持久化

```yaml
# docker-compose.yml
services:
  backend:
    volumes:
      - db-data:/app/data

volumes:
  db-data:
```

### 使用 Litestream 备份

```yaml
# litestream.yml
dbs:
  - path: /data/habits.db
    replicas:
      - url: s3://bucket-name/habits
        sync-interval: 1s
        retention: 24h
```

```bash
# 使用 litestream 运行
litestream replicate -config litestream.yml
```

### 手动备份

```bash
# 使用 SQLite CLI 安全备份
sqlite3 /data/habits.db "VACUUM INTO '/backups/habits-$(date +%Y%m%d).db'"

# 或使用 Python
python -c "import sqlite3; src=sqlite3.connect('/data/habits.db'); dst=sqlite3.connect('/backups/backup.db'); src.backup(dst)"
```

### 使用 Alembic 迁移

```bash
# 生成迁移
alembic revision --autogenerate -m "Add description column"

# 应用迁移
alembic upgrade head

# 回滚
alembic downgrade -1
```

**生产部署：**
1. 创建备份
2. 运行迁移：`alembic upgrade head`
3. 启动应用
4. 验证健康检查

---

## 9. 监控与日志

### 结构化日志

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
        }
        return json.dumps(log_data)

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
```

### 请求日志中间件

```python
import time
import logging

logger = logging.getLogger(__name__)

@app.middleware("http")
async def log_requests(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    duration = time.time() - start_time

    logger.info(
        f"{request.method} {request.url.path} "
        f"status={response.status_code} "
        f"duration={duration:.3f}s"
    )
    return response
```

### 健康检查端点

```python
from fastapi import FastAPI, status
from fastapi.responses import JSONResponse

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

@app.get("/health/ready")
async def readiness_check(db: Session = Depends(get_db)):
    try:
        db.execute(text("SELECT 1"))
        return {"status": "ready"}
    except Exception as e:
        return JSONResponse(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            content={"status": "not ready", "error": str(e)},
        )
```

### 监控堆栈（可选）

| 工具 | 用途 |
|------|---------|
| Prometheus | 指标收集 |
| Grafana | 可视化 |
| Sentry | 错误跟踪 |
| Loki | 日志聚合 |

---

## 10. 云平台

### 平台对比

| 平台 | 定价 | 适合 | SQLite 支持 |
|----------|---------|----------|----------------|
| **Railway** | 按量计费 | 快速部署 | 有限 |
| **Render** | $7+/月 | 托管服务 | 有限 |
| **Fly.io** | $2+/月 | 全局、SQLite | 是（卷） |
| **DigitalOcean** | $4+/月 | VPS 控制 | 是 |
| **Hetzner** | $4+/月 | 欧洲、预算 | 是 |

### Fly.io 部署

```bash
# 安装 flyctl
curl -L https://fly.io/install.sh | sh

# 登录
fly auth login

# 启动应用
fly launch

# 部署
fly deploy

# 为 SQLite 创建卷
fly volumes create data --size 1

# 检查状态
fly status
```

**fly.toml:**
```toml
app = "habit-tracker"

[build]
  dockerfile = "Dockerfile"

[http_service]
  internal_port = 8000
  force_https = true

[mounts]
  source = "data"
  destination = "/data"
```

### Railway 部署

```bash
# 安装 Railway CLI
npm install -g @railway/cli

# 登录
railway login

# 初始化
railway init

# 部署
railway up
```

### VPS 部署清单

1. **服务器设置**
   ```bash
   sudo apt update && sudo apt upgrade
   sudo apt install nginx python3-pip python3-venv
   ```

2. **克隆仓库**
   ```bash
   git clone https://github.com/user/habit-tracker /var/www/habit-tracker
   ```

3. **设置后端**
   ```bash
   cd /var/www/habit-tracker/backend
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

4. **构建前端**
   ```bash
   cd /var/www/habit-tracker/frontend
   npm install && npm run build
   ```

5. **配置 systemd 服务**

6. **配置 Nginx**

7. **使用 Certbot 设置 SSL**

8. **配置防火墙**
   ```bash
   sudo ufw allow 80
   sudo ufw allow 443
   sudo ufw enable
   ```

---

## 11. 安全

### CORS 配置

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,  # 仅特定来源
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

### 安全头（Nginx）

```nginx
# 添加到 server 块
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;
```

### Docker 安全

```dockerfile
# 以非 root 用户运行
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# 使用特定版本
FROM python:3.11.7-slim

# 不要在镜像中存储密钥
# 使用运行时环境变量
```

### 到处使用 HTTPS

- 使用 Let's Encrypt 获取免费 SSL 证书
- 重定向 HTTP 到 HTTPS
- 启用 HSTS

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

### 环境安全

```bash
# 永远不要提交 .env 文件
echo ".env" >> .gitignore
echo ".env.*" >> .gitignore

# 设置限制性权限
chmod 600 .env
```

---

## 12. 单二进制部署

### PyInstaller

```bash
pip install pyinstaller

# 创建 spec 文件
pyi-makespec --onefile --name habittracker backend/app/main.py
```

**entrypoint.py:**
```python
import multiprocessing
import uvicorn

if __name__ == "__main__":
    multiprocessing.freeze_support()  # Windows 必需
    uvicorn.run("app.main:app", host="0.0.0.0", port=8000)
```

**构建：**
```bash
pyinstaller --onefile --add-data "frontend/dist:frontend/dist" entrypoint.py
```

### Tauri（桌面应用）

对于原生桌面包装器：

```bash
# 安装 Tauri CLI
cargo install tauri-cli

# 初始化
cargo tauri init

# 构建
cargo tauri build
```

**优势**：
- 原生 WebView（非捆绑浏览器）
- 小二进制（~10-50MB）
- 跨平台

---

## 部署场景

### 场景 1：本地/个人使用

```
┌─────────────────────────────────┐
│  uvicorn + 内嵌 React       │
│  SQLite 文件在 ./data          │
└─────────────────────────────────┘
```

**命令：**
```bash
cd backend && uvicorn app.main:app --port 8000
# 访问 http://localhost:8000
```

### 场景 2：自托管 VPS

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  Nginx   │──────│  FastAPI │──────│  SQLite  │
│  (SSL)   │      │ (systemd)│      │  (文件)  │
└──────────┘      └──────────┘      └──────────┘
```

**成本**：约 $4-5/月

### 场景 3：Docker Compose

```
┌──────────────────────────────────────────┐
│  docker-compose                          │
│  ┌────────────┐    ┌────────────┐        │
│  │  frontend  │    │  backend   │        │
│  │  (nginx)   │────│  (uvicorn) │        │
│  └────────────┘    └─────┬──────┘        │
│                          │               │
│                    ┌─────▼──────┐        │
│                    │   volume   │        │
│                    │  (sqlite)  │        │
│                    └────────────┘        │
└──────────────────────────────────────────┘
```

### 场景 4：云 PaaS（Fly.io）

```
┌──────────────────────────────────────────┐
│  Fly.io                                  │
│  ┌────────────────────────┐              │
│  │  Docker 容器            │              │
│  │  FastAPI + React       │              │
│  └───────────┬────────────┘              │
│              │                           │
│        ┌─────▼─────┐                     │
│        │  Volume   │                     │
│        │  (SQLite) │                     │
│        └───────────┘                     │
└──────────────────────────────────────────┘
```

**成本**：约 $2-5/月

---

## 快速参考

### 基本命令

```bash
# 开发
uvicorn app.main:app --reload
npm run dev

# 生产构建
npm run build
pip install -r requirements.txt

# Docker
docker-compose up --build
docker-compose logs -f

# 部署
fly deploy
railway up

# SSL
sudo certbot --nginx -d yourdomain.com

# 数据库备份
sqlite3 db.db "VACUUM INTO 'backup.db'"
```

### 端口参考

| 服务 | 默认端口 |
|---------|--------------|
| Vite 开发服务器 | 5173 |
| FastAPI/Uvicorn | 8000 |
| Nginx HTTP | 80 |
| Nginx HTTPS | 443 |
| PostgreSQL | 5432 |

---

## 资源

- [FastAPI 部署](https://fastapi.tiangolo.com/deployment/)
- [Vite 静态部署](https://vitejs.dev/guide/static-deploy.html)
- [Docker 文档](https://docs.docker.com/)
- [Nginx 文档](https://nginx.org/en/docs/)
- [Let's Encrypt](https://letsencrypt.org/)
- [Fly.io 文档](https://fly.io/docs/)
- [Litestream](https://litestream.io/)
