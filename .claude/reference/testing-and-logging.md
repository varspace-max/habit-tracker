# 测试与日志最佳实践参考指南

结构化日志（structlog）和综合测试策略的简明参考指南。

---

## 目录

**第一部分：使用 structlog 进行日志记录**
1. [为什么选择 structlog](#1-为什么选择-structlog)
2. [配置](#2-配置)
3. [FastAPI 集成](#3-fastapi-集成)
4. [上下文绑定](#4-上下文绑定)
5. [异常日志记录](#5-异常日志记录)
6. [使用 structlog 进行测试](#6-使用-structlog-进行测试)

**第二部分：测试策略**
7. [测试金字塔](#7-测试金字塔)
8. [单元测试 (Python)](#8-单元测试-python)
9. [集成测试 (FastAPI)](#9-集成测试-fastapi)
10. [React 组件测试](#10-react-组件测试)
11. [使用 Playwright 进行 E2E 测试](#11-使用-playwright-进行-e2e-测试)
12. [测试组织](#12-测试组织)

---

# 第一部分：使用 structlog 进行日志记录

## 1. 为什么选择 structlog

### 相比标准日志的优势

| 特性 | 标准日志 | structlog |
|---------|------------------|-----------|
| 输出格式 | 纯文本 | 结构化键值对 |
| 上下文 | 每次调用手动添加 | 绑定日志器携带上下文 |
| 配置 | 复杂的层级结构 | 声明式处理器链 |
| JSON 输出 | 需要自定义格式化器 | 内置支持 |
| 性能 | 良好 | 启用缓存后极佳 |

### 主要优势

- **结构化数据**：日志以键值对形式输出，便于解析
- **绑定日志器**：一次性添加上下文，会自动出现在所有后续日志中
- **处理器管道**：通过可组合函数转换日志
- **环境感知**：开发环境使用美化控制台输出，生产环境使用 JSON

---

## 2. 配置

### 基础设置

```python
# app/logging_config.py
import logging
import structlog

def configure_logging(json_format: bool = False):
    """Configure structlog for the application."""

    shared_processors = [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
    ]

    if json_format:
        # Production: JSON output
        processors = shared_processors + [
            structlog.processors.dict_tracebacks,
            structlog.processors.JSONRenderer(),
        ]
    else:
        # Development: Pretty console output
        processors = shared_processors + [
            structlog.processors.format_exc_info,
            structlog.dev.ConsoleRenderer(colors=True),
        ]

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(),
        cache_logger_on_first_use=True,
    )
```

### 基于环境的配置

```python
import os
import sys

def configure_logging():
    # Auto-detect: JSON for production/CI, console for development
    use_json = (
        os.environ.get("LOG_JSON", "false").lower() == "true"
        or os.environ.get("CI", "false").lower() == "true"
        or not sys.stderr.isatty()
    )
    configure_logging(json_format=use_json)
```

### 在 FastAPI 中初始化

```python
# app/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from app.logging_config import configure_logging

@asynccontextmanager
async def lifespan(app: FastAPI):
    configure_logging()
    yield

app = FastAPI(lifespan=lifespan)
```

---

## 3. FastAPI 集成

### 请求日志中间件

```python
# app/middleware.py
import time
import uuid
import structlog
from starlette.middleware.base import BaseHTTPMiddleware
from fastapi import Request

logger = structlog.get_logger()

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Clear context and bind request info
        structlog.contextvars.clear_contextvars()

        request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))

        structlog.contextvars.bind_contextvars(
            request_id=request_id,
            method=request.method,
            path=request.url.path,
        )

        start_time = time.perf_counter()

        try:
            response = await call_next(request)
            duration_ms = (time.perf_counter() - start_time) * 1000

            logger.info(
                "Request completed",
                status_code=response.status_code,
                duration_ms=round(duration_ms, 2),
            )

            response.headers["X-Request-ID"] = request_id
            return response

        except Exception as exc:
            duration_ms = (time.perf_counter() - start_time) * 1000
            logger.exception(
                "Request failed",
                duration_ms=round(duration_ms, 2),
            )
            raise
```

### 向应用添加中间件

```python
# app/main.py
from app.middleware import LoggingMiddleware

app.add_middleware(LoggingMiddleware)
```

---

## 4. 上下文绑定

### 请求级别的上下文

```python
import structlog

# In middleware or early in request handling
structlog.contextvars.bind_contextvars(
    request_id="abc-123",
    user_id=42,
    path="/api/habits",
)

# All subsequent logs include this context automatically
logger = structlog.get_logger()
logger.info("Processing request")  # Includes request_id, user_id, path
logger.info("Fetching data")       # Same context
```

### 临时上下文

```python
# Add temporary context for a code block
with structlog.contextvars.bound_contextvars(operation="streak_calculation"):
    logger.info("Starting calculation")
    # ... do work
    logger.info("Calculation complete")
# Context is restored after the block
```

### 逐日志器绑定

```python
# Create a logger with bound context
logger = structlog.get_logger().bind(
    component="habit_service",
    version="1.0",
)

logger.info("Service started")  # Includes component, version
```

---

## 5. 异常日志记录

### 记录异常

```python
logger = structlog.get_logger()

try:
    risky_operation()
except Exception:
    # Option 1: exc_info=True
    logger.error("Operation failed", exc_info=True)

    # Option 2: .exception() method (same as error with exc_info=True)
    logger.exception("Operation failed")
```

### 结构化异常输出

对于 JSON 日志记录，配置 `dict_tracebacks` 处理器：

```python
structlog.processors.dict_tracebacks
```

这会生成 JSON 可序列化的异常数据，而不是多行字符串。

---

## 6. 使用 structlog 进行测试

### 使用 capture_logs

```python
import structlog
from structlog.testing import capture_logs

def test_logs_habit_creation():
    with capture_logs() as captured:
        # Call function that logs
        create_habit("Exercise")

    assert captured == [
        {
            "event": "Habit created",
            "habit_name": "Exercise",
            "log_level": "info",
        }
    ]
```

### Pytest Fixture

```python
# tests/conftest.py
import pytest
import structlog
from structlog.testing import LogCapture

@pytest.fixture
def log_output():
    return LogCapture()

@pytest.fixture(autouse=True)
def configure_structlog(log_output):
    structlog.configure(processors=[log_output])
    yield
    structlog.reset_defaults()
```

```python
# tests/test_service.py
def test_service_logs_correctly(log_output):
    do_something()

    assert log_output.entries == [
        {"event": "something happened", "log_level": "info"}
    ]
```

---

# 第二部分：测试策略

## 7. 测试金字塔

### 分布

| 层级 | 占比 | 速度 | 范围 |
|-------|------------|-------|-------|
| 单元测试 | 70% | 毫秒级 | 单个函数/类 |
| 集成测试 | 20% | 秒级 | 多个组件 |
| E2E 测试 | 10% | 分钟级 | 完整系统 |

### 什么测试应该放在哪里

**单元测试：**
- 纯函数（连续天数计算、日期工具函数）
- Pydantic 验证器
- 使用模拟依赖的业务逻辑

**集成测试：**
- 使用真实数据库的 API 端点
- 仓储层操作
- 使用真实依赖的服务层

**E2E 测试：**
- 仅关键的用户流程
- 完整的前端 + 后端交互
- 视觉回归测试

---

## 8. 单元测试 (Python)

### 结构

```python
# tests/unit/test_streak_calculator.py
import pytest
from datetime import date
from app.services.streak import calculate_streak

class TestStreakCalculation:
    def test_returns_zero_for_empty_completions(self):
        result = calculate_streak([])
        assert result == 0

    def test_returns_one_for_single_completion_today(self):
        result = calculate_streak([date.today()])
        assert result == 1

    def test_counts_consecutive_days(self):
        completions = [date(2025, 1, 1), date(2025, 1, 2), date(2025, 1, 3)]
        result = calculate_streak(completions)
        assert result == 3

    def test_breaks_on_gap(self):
        completions = [date(2025, 1, 1), date(2025, 1, 3)]  # Gap on Jan 2
        result = calculate_streak(completions)
        assert result == 1  # Only Jan 3 counts
```

### 参数化测试

```python
@pytest.mark.parametrize("completions,expected", [
    ([], 0),
    ([date(2025, 1, 1)], 1),
    ([date(2025, 1, 1), date(2025, 1, 2)], 2),
    ([date(2025, 1, 1), date(2025, 1, 3)], 1),  # Gap breaks streak
])
def test_streak_calculation(completions, expected):
    assert calculate_streak(completions) == expected
```

### 模拟

```python
from unittest.mock import Mock, patch

def test_service_calls_repository():
    mock_repo = Mock()
    mock_repo.get_by_id.return_value = Habit(id=1, name="Exercise")

    service = HabitService(repository=mock_repo)
    result = service.get_habit(1)

    mock_repo.get_by_id.assert_called_once_with(1)
    assert result.name == "Exercise"
```

---

## 9. 集成测试 (FastAPI)

### 测试设置

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine, StaticPool
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.database import Base, get_db

@pytest.fixture(scope="function")
def db_session():
    """Create a fresh database for each test."""
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    Base.metadata.create_all(engine)
    SessionLocal = sessionmaker(bind=engine)

    session = SessionLocal()
    try:
        yield session
    finally:
        session.close()
        Base.metadata.drop_all(engine)

@pytest.fixture(scope="function")
def client(db_session):
    """Create test client with overridden database."""
    def override_get_db():
        try:
            yield db_session
        finally:
            pass

    app.dependency_overrides[get_db] = override_get_db

    with TestClient(app) as test_client:
        yield test_client

    app.dependency_overrides.clear()
```

### API 测试

```python
# tests/integration/test_api_habits.py

class TestHabitAPI:
    def test_create_habit_returns_201(self, client):
        response = client.post(
            "/api/habits",
            json={"name": "Exercise", "description": "Daily workout"}
        )

        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Exercise"
        assert "id" in data

    def test_create_habit_without_name_returns_422(self, client):
        response = client.post("/api/habits", json={})

        assert response.status_code == 422

    def test_get_habit_returns_habit(self, client, db_session):
        # Setup: Create habit in database
        habit = Habit(name="Test", created_at=datetime.utcnow())
        db_session.add(habit)
        db_session.commit()

        # Test
        response = client.get(f"/api/habits/{habit.id}")

        assert response.status_code == 200
        assert response.json()["name"] == "Test"

    def test_get_nonexistent_habit_returns_404(self, client):
        response = client.get("/api/habits/99999")

        assert response.status_code == 404
```

### 使用事务实现数据库隔离

```python
@pytest.fixture
def db_session():
    """Rollback after each test for isolation."""
    connection = engine.connect()
    transaction = connection.begin()
    session = Session(bind=connection)

    yield session

    session.close()
    transaction.rollback()
    connection.close()
```

---

## 10. React 组件测试

### 使用 Vitest 进行设置

```javascript
// vite.config.js
export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.js',
  },
});

// src/test/setup.js
import '@testing-library/jest-dom';
```

### 组件测试

```javascript
// src/features/habits/__tests__/HabitCard.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { HabitCard } from '../components/HabitCard';

describe('HabitCard', () => {
  const mockHabit = {
    id: 1,
    name: 'Exercise',
    currentStreak: 5,
    completedToday: false,
  };

  it('renders habit name', () => {
    render(<HabitCard habit={mockHabit} />);

    expect(screen.getByText('Exercise')).toBeInTheDocument();
  });

  it('displays current streak', () => {
    render(<HabitCard habit={mockHabit} />);

    expect(screen.getByText(/5.*streak/i)).toBeInTheDocument();
  });

  it('calls onComplete when button clicked', async () => {
    const onComplete = vi.fn();
    render(<HabitCard habit={mockHabit} onComplete={onComplete} />);

    await userEvent.click(screen.getByRole('button', { name: /complete/i }));

    expect(onComplete).toHaveBeenCalledWith(1);
  });

  it('shows completed state', () => {
    const completedHabit = { ...mockHabit, completedToday: true };
    render(<HabitCard habit={completedHabit} />);

    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### 使用 Providers 进行测试

```javascript
// src/test/utils.jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';

export function renderWithProviders(ui) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return render(
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        {ui}
      </BrowserRouter>
    </QueryClientProvider>
  );
}
```

### 查询优先级（按顺序使用）

1. `getByRole` - 可访问名称（最佳）
2. `getByLabelText` - 表单标签
3. `getByText` - 文本内容
4. `getByTestId` - 最后手段

```javascript
// 推荐
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText('Email');

// 避免
screen.getByTestId('submit-button');  // Only when necessary
```

---

## 11. 使用 Playwright 进行 E2E 测试

### Playwright MCP 服务器设置

```bash
# Add Playwright MCP to Claude Code
claude mcp add playwright npx @playwright/mcp@latest
```

### 配置

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

### 页面对象模型

```javascript
// tests/e2e/pages/DashboardPage.js
export class DashboardPage {
  constructor(page) {
    this.page = page;
    this.addHabitButton = page.getByRole('button', { name: /add habit/i });
    this.habitList = page.getByTestId('habit-list');
  }

  async goto() {
    await this.page.goto('/');
  }

  async addHabit(name) {
    await this.addHabitButton.click();
    await this.page.getByLabel('Habit name').fill(name);
    await this.page.getByRole('button', { name: /save/i }).click();
  }

  async completeHabit(name) {
    const habitCard = this.page.getByTestId(`habit-${name}`);
    await habitCard.getByRole('button', { name: /complete/i }).click();
  }

  async getHabitStreak(name) {
    const habitCard = this.page.getByTestId(`habit-${name}`);
    return habitCard.getByTestId('streak-count').textContent();
  }
}
```

### E2E 测试

```javascript
// tests/e2e/habits.spec.js
import { test, expect } from '@playwright/test';
import { DashboardPage } from './pages/DashboardPage';

test.describe('Habit Tracking', () => {
  test('user can create and complete a habit', async ({ page }) => {
    const dashboard = new DashboardPage(page);

    await dashboard.goto();
    await dashboard.addHabit('Exercise');

    // Verify habit appears
    await expect(page.getByText('Exercise')).toBeVisible();

    // Complete the habit
    await dashboard.completeHabit('Exercise');

    // Verify streak updated
    await expect(page.getByTestId('streak-count')).toHaveText('1');
  });

  test('streak increments on consecutive days', async ({ page }) => {
    // Test with seeded data for multi-day scenarios
  });
});
```

### 视觉测试

```javascript
test('dashboard matches snapshot', async ({ page }) => {
  await page.goto('/');

  // Wait for data to load
  await expect(page.getByTestId('habit-list')).toBeVisible();

  // Compare screenshot
  await expect(page).toHaveScreenshot('dashboard.png', {
    mask: [page.locator('.timestamp')],  // Mask dynamic content
  });
});
```

### 运行 E2E 测试

```bash
# Run all E2E tests
npx playwright test

# Run with UI mode (debugging)
npx playwright test --ui

# Run specific test file
npx playwright test habits.spec.js

# Update snapshots
npx playwright test --update-snapshots
```

---

## 12. 测试组织

### 目录结构

```
tests/
├── conftest.py                 # Shared fixtures
├── pytest.ini                  # Pytest configuration
├── unit/
│   ├── conftest.py             # Unit test fixtures
│   ├── test_streak.py
│   └── test_validators.py
├── integration/
│   ├── conftest.py             # Integration fixtures (db, client)
│   ├── test_api_habits.py
│   └── test_api_completions.py
└── e2e/
    ├── playwright.config.js
    ├── pages/
    │   └── DashboardPage.js
    └── habits.spec.js

frontend/
└── src/
    ├── features/
    │   └── habits/
    │       └── __tests__/
    │           ├── HabitCard.test.jsx
    │           └── useHabits.test.js
    └── test/
        ├── setup.js
        └── utils.jsx
```

### Pytest 标记

```ini
# pytest.ini
[pytest]
markers =
    unit: Unit tests (fast, no I/O)
    integration: Integration tests (database, API)
    slow: Slow running tests
```

```python
@pytest.mark.unit
def test_calculate_streak():
    pass

@pytest.mark.integration
def test_api_creates_habit():
    pass
```

```bash
# Run by marker
pytest -m unit
pytest -m integration
pytest -m "not slow"
```

### 覆盖率配置

```toml
# pyproject.toml
[tool.coverage.run]
source = ["app"]
omit = ["*/tests/*", "*/__pycache__/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
]
fail_under = 80
```

```bash
# Run with coverage
pytest --cov=app --cov-report=html --cov-report=term-missing
```

---

## 快速参考

### 测试命令

```bash
# Backend
pytest                              # All tests
pytest tests/unit                   # Unit tests only
pytest tests/integration            # Integration tests only
pytest -m unit                      # By marker
pytest --cov=app                    # With coverage
pytest -x                           # Stop on first failure
pytest -v                           # Verbose output

# Frontend
npm test                            # All tests
npm test -- --watch                 # Watch mode
npm test -- --coverage              # With coverage

# E2E
npx playwright test                 # All E2E tests
npx playwright test --ui            # UI mode
npx playwright test --debug         # Debug mode
```

### 断言速查表

```python
# Pytest
assert result == expected
assert result is not None
assert "text" in result
assert len(items) == 3
pytest.raises(ValueError)
```

```javascript
// React Testing Library
expect(element).toBeInTheDocument();
expect(element).toBeVisible();
expect(element).toHaveText('text');
expect(element).toBeDisabled();
expect(mockFn).toHaveBeenCalledWith(arg);
```

```javascript
// Playwright
await expect(locator).toBeVisible();
await expect(locator).toHaveText('text');
await expect(page).toHaveURL('/path');
await expect(page).toHaveScreenshot();
```

---

## 资源

- [structlog 文档](https://www.structlog.org/)
- [pytest 文档](https://docs.pytest.org/)
- [FastAPI 测试](https://fastapi.tiangolo.com/tutorial/testing/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright 文档](https://playwright.dev/)
- [Playwright MCP](https://github.com/microsoft/playwright-mcp)
