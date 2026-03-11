# Habit Tracker - 产品需求文档

## 1. 执行摘要

Habit Tracker是一款个人网页应用，旨在帮助用户通过简单的记录和连续打卡机制来建立和保持日常习惯。该应用提供了一种无缝的方式来记录每日习惯完成情况、可视化进度，并通过连续打卡和完成率指标保持动力。

核心价值主张是简洁：一款本地优先的Habit Tracker，没有账户管理、社交功能或过多定制选项的复杂性。用户可以纯粹地专注于培养习惯，不受干扰。

**MVP 目标：** 交付一款功能完善的习惯追踪应用，支持每日完成记录、连续打卡、计划缺勤支持和日历可视化——全部在本地运行，无需身份验证。

---

## 2. 使命

**使命声明：** 提供一款简洁、无干扰的工具，用于追踪日常习惯，并通过可见的进度指标保持动力。

### 核心原则

1. **简洁优先** — 最小化的功能，最大化的效用。每个功能都必须证明自己的存在价值。
2. **即时反馈** — 完成习惯应该立即且令人满足。
3. **动力而非内疚** — 积极展示进度；完成率与连续打卡并排显示，防止士气低落。
4. **本地且私密** — 无账户，无云端，数据不离开用户设备。
5. **每日聚焦** — 优化日常习惯；避免自定义日程的复杂性。

---

## 3. 目标用户

### 主要人物画像：自我驱动的个人

- **是谁：** 在本地运行应用供个人使用的单一用户
- **技术舒适度：** 熟练运行本地开发服务器或简单的可执行文件
- **目标：**
  - 建立一致的日常习惯（锻炼、阅读、冥想等）
  - 看到一致性的视觉证明以保持动力
  - 简单追踪，无需复杂应用的负担
- **痛点：**
  - 现有的Habit Tracker功能冗余
  - 不愿创建账户或分享数据
  - 纯连续打卡追踪在生活中出现问题时感觉像惩罚

---

## 4. MVP 范围

### 范围内

**核心功能**
- ✅ 创建、编辑和删除习惯
- ✅ 归档习惯（软删除，保留历史记录）
- ✅ 标记习惯在指定日期完成
- ✅ 标记日期为"跳过"（计划缺勤，不中断连续打卡）
- ✅ 撤销完成/跳过
- ✅ 查看今天所有习惯及其完成状态
- ✅ 计算每个习惯的当前连续打卡天数
- ✅ 最长连续打卡追踪（保留个人最佳）
- ✅ 每个习惯的完成率百分比
- ✅ 显示完成历史的月历视图

**技术方面**
- ✅ Python 后端，使用 FastAPI
- ✅ SQLite 数据库用于持久化
- ✅ React 前端，使用 Vite
- ✅ Tailwind CSS 用于样式设计
- ✅ RESTful API 设计
- ✅ 本地开发设置（双终端工作流）

### 范围外

**推迟的功能**
- ❌ 非日常习惯（每周、特定日期）
- ❌ 提醒/通知
- ❌ 习惯分类/标签
- ❌ 午夜后的宽限期
- ❌ 连续打卡冻结（计划缺勤已覆盖此用例）
- ❌ 数据导出（CSV、JSON）
- ❌ 深色模式
- ❌ 移动应用（仅响应式网页）
- ❌ 多用户/身份验证
- ❌ 云端同步
- ❌ 游戏化（徽章、积分）

---

## 5. 用户故事

### 主要用户故事

1. **作为一个用户，我想创建一个新习惯，以便能够开始每日追踪它。**
   - 示例：添加"晨间冥想"，描述为"10分钟正念"

2. **作为一个用户，我想通过单击标记习惯为完成，以便追踪无摩擦。**
   - 示例：点击"锻炼"旁边的复选框，立即看到视觉反馈

3. **作为一个用户，我想在一个屏幕上看到今天的所有习惯，以便我知道我需要做什么。**
   - 示例：仪表板显示5个习惯，已完成3个，剩余2个

4. **作为一个用户，我想看到每个习惯的当前连续打卡天数，以便保持动力维持它。**
   - 示例：突出显示"阅读：14天连续打卡"

5. **作为一个用户，我想将日期标记为"跳过"用于计划缺勤，以便假期不会中断我的连续打卡。**
   - 示例：将"锻炼"标记为旅行日跳过；当我回来时连续打卡继续

6. **作为一个用户，我想看到我的完成率和连续打卡并排显示，以便糟糕的一天不会感觉像失败。**
   - 示例："锻炼：5天连续打卡 | 87%完成率"

7. **作为一个用户，我想查看我的习惯历史日历，以便能够看到随时间的模式。**
   - 示例：月网格，绿色（已完成）、灰色（跳过）、红色（错过）指示

8. **作为一个用户，我想归档旧习惯而不丢失其历史记录，以便能够专注于当前目标。**
   - 示例：归档"学习西班牙语"，但如果需要仍可查看其历史数据

---

## 6. 核心架构与模式

### 高层架构

```
┌─────────────────┐     HTTP/JSON      ┌─────────────────┐
│                 │ ◄───────────────► │                 │
│  React + Vite   │                    │    FastAPI      │
│   (Frontend)    │                    │   (Backend)     │
│   Port 5173     │                    │   Port 8000     │
└─────────────────┘                    └────────┬────────┘
                                                │
                                                ▼
                                       ┌─────────────────┐
                                       │     SQLite      │
                                       │   (Database)    │
                                       └─────────────────┘
```

### 目录结构

```
habit-tracker/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI 应用入口
│   │   ├── database.py          # SQLite 连接与会话
│   │   ├── models.py            # SQLAlchemy ORM 模型
│   │   ├── schemas.py           # Pydantic 请求/响应模式
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── habits.py        # 习惯 CRUD 端点
│   │       └── completions.py   # 完成/跳过端点
│   ├── habits.db                # SQLite 数据库文件
│   ├── pyproject.toml           # Python 依赖
│   └── requirements.txt         # 固定依赖（可选）
│
├── frontend/
│   ├── src/
│   │   ├── components/          # 可复用 UI 组件
│   │   │   ├── HabitCard.jsx
│   │   │   ├── HabitForm.jsx
│   │   │   ├── Calendar.jsx
│   │   │   └── StreakBadge.jsx
│   │   ├── pages/               # 路由级组件
│   │   │   ├── Dashboard.jsx
│   │   │   └── HabitDetail.jsx
│   │   ├── api/                 # API 客户端函数
│   │   │   └── habits.js
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css            # Tailwind 导入
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── tailwind.config.js
│
├── .gitignore
├── .claude/
│   └── PRD.md                   # 本文档
└── README.md
```

### 关键设计模式

- **仓库模式** — 数据库操作在 models/routers 中抽象
- **Pydantic 模式** — 请求验证和响应序列化
- **组件组合** — React 组件小而可组合
- **API 客户端层** — 前端 API 调用集中在 `api/` 目录
- **乐观 UI** — 立即更新 UI，在后台与服务器同步

---

## 7. 功能

### 7.1 习惯管理

**目的：** 创建、更新和管理习惯

**操作：**
- 创建带名称和可选描述的习惯
- 编辑习惯名称/描述
- 归档习惯（从今日视图移除，保留历史）
- 永久删除习惯（需确认）

**关键功能：**
- 习惯名称必填，描述可选
- 可选颜色选择器用于视觉区分
- 归档习惯在主视图中隐藏但可访问

### 7.2 每日完成追踪

**目的：** 记录每日习惯完成

**操作：**
- 标记习惯今天为已完成
- 标记习惯为跳过（计划缺勤）
- 撤销完成或跳过
- 查看任何过去日期的完成状态

**关键功能：**
- 单击完成，即时视觉反馈
- 三种状态：已完成（绿色）、已跳过（灰色）、未完成（默认）
- 可修改今天和过去的日期（用于补记录）
- 完成基于日期（每个习惯每天一个）

### 7.3 连续打卡追踪

**目的：** 通过连续打卡可见性激励一致性

**计算：**
- **当前连续打卡：** 连续完成的天数（跳过日不中断连续打卡）
- **最长连续打卡：** 个人最佳，连续打卡中断后仍保留
- **完成率：** （已完成天数 / 习惯创建后的总天数）× 100

**关键功能：**
- 连续打卡在习惯卡片上突出显示
- 完成率与连续打卡并排显示以减轻压力
- 达成最长连续打卡时庆祝

### 7.4 日历视图

**目的：** 可视化随时间的习惯历史

**功能：**
- 月网格布局
- 颜色编码的日期：绿色（已完成）、灰色（跳过）、红色（错过）、默认（未来/未到期）
- 月份间导航
- 点击日期查看/编辑完成状态
- 突出显示今天的日期

---

## 8. 技术栈

### 后端

| 组件 | 技术 | 版本 |
|-----------|------------|---------|
| 框架 | FastAPI | ^0.100.0 |
| 服务器 | Uvicorn | ^0.23.0 |
| ORM | SQLAlchemy | ^2.0.0 |
| 验证 | Pydantic | ^2.0.0 |
| 数据库 | SQLite | 3.x（内置） |

### 前端

| 组件 | 技术 | 版本 |
|-----------|------------|---------|
| 框架 | React | ^18.x |
| 构建工具 | Vite | ^5.x |
| 路由 | react-router-dom | ^6.x |
| 服务器状态 | TanStack Query | ^5.x |
| 样式 | Tailwind CSS | ^3.x |
| 日期工具 | date-fns | ^3.x |
| 图标 | lucide-react | ^0.300.0 |

### 开发工具

| 工具 | 用途 |
|------|---------|
| Python venv | 虚拟环境 |
| npm/pnpm | 包管理 |
| Ruff | Python 代码检查/格式化 |
| ESLint | JavaScript 代码检查 |

---

## 9. 安全与配置

### 安全范围

**范围内：**
- ✅ 所有 API 端点的输入验证（Pydantic）
- ✅ 防止 SQL 注入（SQLAlchemy ORM）
- ✅ 本地开发的 CORS 配置

**范围外：**
- ❌ 身份验证/授权（单一本地用户）
- ❌ HTTPS（仅本地开发）
- ❌ 速率限制
- ❌ CSRF 保护

### 配置

**后端（环境变量）：**
```
DATABASE_URL=sqlite:///./habits.db
CORS_ORIGINS=http://localhost:5173
```

**前端（vite.config.js）：**
```javascript
server: {
  proxy: {
    '/api': 'http://localhost:8000'
  }
}
```

### 部署

MVP 通过开发服务器在本地运行：
- 后端：`uvicorn app.main:app --reload`
- 前端：`npm run dev`

---

## 10. API 规范

### 基础 URL
```
http://localhost:8000/api
```

### 端点

#### 习惯

**GET /api/habits**
列出所有习惯及计算统计。

响应：
```json
{
  "habits": [
    {
      "id": 1,
      "name": "Exercise",
      "description": "30 minutes of activity",
      "color": "#10B981",
      "currentStreak": 5,
      "longestStreak": 14,
      "completionRate": 0.87,
      "completedToday": true,
      "createdAt": "2025-01-01T00:00:00Z",
      "archivedAt": null
    }
  ]
}
```

**POST /api/habits**
创建新习惯。

请求：
```json
{
  "name": "Read",
  "description": "Read for 20 minutes",
  "color": "#3B82F6"
}
```

**PUT /api/habits/{id}**
更新习惯。

**DELETE /api/habits/{id}**
永久删除习惯。

**PATCH /api/habits/{id}/archive**
归档习惯。

#### 完成记录

**POST /api/habits/{id}/complete**
标记习惯在指定日期为已完成。

请求：
```json
{
  "date": "2025-01-04",
  "notes": "Ran 5k today"
}
```

**POST /api/habits/{id}/skip**
标记习惯在指定日期为跳过。

请求：
```json
{
  "date": "2025-01-04",
  "reason": "Traveling"
}
```

**DELETE /api/habits/{id}/completions/{date}**
移除完成或跳过记录。

**GET /api/habits/{id}/completions**
获取习惯的完成历史。

查询参数：`?start=2025-01-01&end=2025-01-31`

响应：
```json
{
  "completions": [
    {
      "date": "2025-01-01",
      "status": "completed",
      "notes": null
    },
    {
      "date": "2025-01-02",
      "status": "skipped",
      "notes": "Sick day"
    }
  ]
}
```

---

## 11. 成功标准

### MVP 成功定义

当用户能够做到以下时，MVP 成功：
1. 添加新习惯并在仪表板上看到
2. 每日完成习惯并看着他们的连续打卡增长
3. 为计划缺勤跳过一天而不丢失连续打卡
4. 在日历上查看完成历史
5. 被可见的进度指标激励

### 功能需求

- ✅ 创建、编辑、删除和归档习惯
- ✅ 单次交互标记习惯完成或跳过
- ✅ 准确计算和显示当前连续打卡
- ✅ 连续打卡中断后保留最长连续打卡
- ✅ 显示完成率为百分比
- ✅ 渲染显示完成历史的月历
- ✅ 跨浏览器会话持久化所有数据（SQLite）
- ✅ 正确处理日期边界（本地时间）

### 质量指标

- 页面加载在1秒内
- 完成操作反馈在100ms内
- 正常使用时无数据丢失
- 适用于 Chrome、Firefox、Safari
- 响应式布局（桌面 + 移动浏览器）

---

## 12. 实施阶段

### 阶段 1：后端基础

**目标：** 具有数据库持久化的功能性 API

**交付物：**
- ✅ 项目结构和 Python 虚拟环境
- ✅ SQLite 数据库，包含习惯和完成记录表
- ✅ 习惯的 CRUD 端点
- ✅ 完成/跳过端点
- ✅ 连续打卡计算逻辑
- ✅ 通过 Swagger UI 测试 API

**验证：** 可以通过 API 文档创建习惯和记录完成

---

### 阶段 2：前端基础

**目标：** 显示习惯的基本 React 应用

**交付物：**
- ✅ Vite + React 项目脚手架
- ✅ Tailwind CSS 配置
- ✅ 使用 TanStack Query 的 API 客户端
- ✅ 带习惯列表的仪表板页面
- ✅ 习惯创建表单
- ✅ 完成切换功能

**验证：** 可以在浏览器中添加习惯并标记为完成

---

### 阶段 3：核心功能

**目标：** 完整的 MVP 功能

**交付物：**
- ✅ 连续打卡和完成率显示
- ✅ 计划缺勤的跳过功能
- ✅ 编辑和归档习惯流程
- ✅ 日历视图组件
- ✅ 带历史记录的习惯详情页面

**验证：** 所有用户故事可实现

---

### 阶段 4：完善

**目标：** 可用于生产的 MVP

**交付物：**
- ✅ 加载和错误状态
- ✅ 空状态（尚无习惯）
- ✅ 破坏性操作的确认对话框
- ✅ 移动端响应式布局
- ✅ 带设置说明的 README

**验证：** 流畅的用户体验，无粗糙边缘

---

## 13. 未来考虑

### MVP 后增强

- **非日常习惯** — 每周或特定日期计划
- **数据导出** — 所有数据的 CSV/JSON 导出
- **深色模式** — 系统偏好检测
- **提醒** — 浏览器通知（需权限）
- **宽限期** — 午夜后 2-4 小时完成
- **习惯模板** — 预定义习惯供选择
- **周/月报告** — 进度摘要

### 技术改进

- **单一可执行文件** — 打包为独立应用（PyInstaller + 内嵌前端）
- **桌面应用** — Electron 或 Tauri 包装器
- **PWA 支持** — 可安装的离线能力网页应用
- **数据库迁移** — Alembic 用于模式演进

---

## 14. 风险与缓解

| 风险 | 影响 | 缓解 |
|------|--------|------------|
| **连续打卡计算错误** | 如果连续打卡不正确，用户会失去信任 | 全面单元测试连续打卡逻辑；手动测试边界情况 |
| **数据丢失** | 应用价值完全失效 | SQLite 健壮；记录备份程序；未来：导出功能 |
| **日期/时区问题** | 完成记录在错误的日期 | 使用一致的日期处理（date-fns）；跨时区测试 |
| **范围蔓延** | MVP 永远无法发布 | 严格遵守 MVP 范围；明确推迟功能 |
| **移动端 UX 差** | 在主要设备上无法使用 | 移动优先设计；尽早测试真实设备 |

---

## 15. 附录

### 数据库模式

```sql
CREATE TABLE habits (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    color TEXT DEFAULT '#10B981',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    archived_at TIMESTAMP
);

CREATE TABLE completions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    habit_id INTEGER NOT NULL,
    completed_date TEXT NOT NULL,  -- YYYY-MM-DD 格式
    status TEXT NOT NULL DEFAULT 'completed',  -- 'completed' 或 'skipped'
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (habit_id) REFERENCES habits(id) ON DELETE CASCADE,
    UNIQUE(habit_id, completed_date)
);

CREATE INDEX idx_completions_habit_date ON completions(habit_id, completed_date);
```

### 关键依赖

- [FastAPI 文档](https://fastapi.tiangolo.com/)
- [SQLAlchemy 2.0 文档](https://docs.sqlalchemy.org/)
- [React 文档](https://react.dev/)
- [Vite 文档](https://vitejs.dev/)
- [Tailwind CSS 文档](https://tailwindcss.com/)
- [TanStack Query 文档](https://tanstack.com/query/latest)
- [date-fns 文档](https://date-fns.org/)
