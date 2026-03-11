# FastAPI 最佳实践参考

构建生产级 FastAPI 应用的简明参考指南。

---

## 目录

1. [项目结构](#1-项目结构)
2. [路由与端点](#2-路由与端点)
3. [Pydantic 模型与验证](#3-pydantic-模型与验证)
4. [依赖注入](#4-依赖注入)
5. [错误处理](#5-错误处理)
6. [数据库集成](#6-数据库集成)
7. [性能与异步](#7-性能与异步)
8. [测试](#8-测试)
9. [安全](#9-安全)
10. [配置](#10-配置)
11. [反模式](#11-反模式)

---

## 1. 项目结构

### 小型项目（文件类型结构）

```
app/
├── main.py           # FastAPI 应用实例
├── routers/          # API 路由处理器
├── models.py         # SQLAlchemy 模型
├── schemas.py        # Pydantic 模式
├── database.py       # 数据库配置
├── dependencies.py   # 共享依赖
└── config.py         # 设置
```

### 大型项目（领域驱动结构）

```
src/
├── habits/
│   ├── router.py
│   ├── schemas.py
│   ├── models.py
│   ├── service.py
│   ├── dependencies.py
│   └── exceptions.py
├── completions/
│   └── (相同结构)
├── shared/
│   ├── config.py
│   ├── database.py
│   └── exceptions.py
└── main.py
```

### 核心原则

- **关注点分离**：保持路由、模型、模式、服务分离
- **一个路由器一个领域**：将相关端点分组
- **明确导入**：跨包导入时使用完整的模块路径

```python
# 推荐：明确导入
from src.habits import service as habits_service
from src.habits import constants as habits_constants

# 避免：模糊导入
from src.habits.service import *
```

---

## 2. 路由与端点

### 使用 APIRouter

```python
from fastapi import APIRouter, Depends, status

router = APIRouter(
    prefix="/habits",
    tags=["habits"],
    responses={404: {"description": "Not found"}},
)

@router.get("/", response_model=list[HabitResponse])
async def list_habits():
    pass

@router.post("/", response_model=HabitResponse, status_code=status.HTTP_201_CREATED)
async def create_habit(habit: HabitCreate):
    pass
```

### 包含在主应用中

```python
from fastapi import FastAPI
from .routers import habits, completions

app = FastAPI()
app.include_router(habits.router, prefix="/api")
app.include_router(completions.router, prefix="/api")
```

### 路径参数 vs 查询参数

```python
from typing import Annotated
from fastapi import Path, Query

@router.get("/habits/{habit_id}")
async def get_habit(
    # 路径参数：资源标识
    habit_id: Annotated[int, Path(ge=1, description="习惯 ID")],
    # 查询参数：过滤/选项
    include_stats: Annotated[bool, Query()] = False,
):
    pass
```

**经验法则**：
- 路径参数用于资源标识：`/habits/{id}`
- 查询参数用于过滤、排序、分页：`/habits?status=active&page=1`

### 响应模型和状态码

```python
from fastapi import status
from fastapi.responses import JSONResponse

@router.post("/", response_model=HabitResponse, status_code=status.HTTP_201_CREATED)
async def create_habit(habit: HabitCreate):
    return created_habit

@router.delete("/{habit_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_habit(habit_id: int):
    return None  # 204 没有响应体

@router.get("/{habit_id}")
async def get_habit(habit_id: int):
    if not habit:
        raise HTTPException(status_code=404, detail="习惯未找到")
    return habit
```

### API 版本控制

```python
# URL 路径版本控制（推荐）
app.include_router(v1_router, prefix="/api/v1")
app.include_router(v2_router, prefix="/api/v2")

# 或使用子应用
v1_app = FastAPI()
v2_app = FastAPI()
app.mount("/api/v1", v1_app)
app.mount("/api/v2", v2_app)
```

---

## 3. Pydantic 模型与验证

### 基础模式模式

```python
from datetime import datetime
from pydantic import BaseModel, ConfigDict

class BaseSchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)  # 用于 ORM 兼容性
```

### 请求/响应模式模式

```python
from pydantic import BaseModel, Field

# 共享属性
class HabitBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: str | None = None

# 创建模式 - 客户端发送的内容
class HabitCreate(HabitBase):
    color: str = Field(default="#10B981", pattern=r"^#[0-9A-Fa-f]{6}$")

# 更新模式 - 所有字段可选
class HabitUpdate(BaseModel):
    name: str | None = Field(None, min_length=1, max_length=100)
    description: str | None = None
    color: str | None = Field(None, pattern=r"^#[0-9A-Fa-f]{6}$")

# 响应模式 - 包含数据库字段，排除敏感数据
class HabitResponse(HabitBase):
    id: int
    color: str
    created_at: datetime
    current_streak: int
    completion_rate: float

    model_config = ConfigDict(from_attributes=True)
```

### 字段验证

```python
from pydantic import BaseModel, Field, field_validator, model_validator

class HabitCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    target_days: list[str] = Field(default_factory=list)

    @field_validator("name")
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("名称不能为空")
        return v.strip()

    @field_validator("target_days")
    @classmethod
    def validate_days(cls, v: list[str]) -> list[str]:
        valid_days = {"mon", "tue", "wed", "thu", "fri", "sat", "sun"}
        for day in v:
            if day.lower() not in valid_days:
                raise ValueError(f"无效的日期: {day}")
        return [d.lower() for d in v]

    @model_validator(mode="after")
    def check_consistency(self):
        # 跨字段验证
        return self
```

### 嵌套模型

```python
class CompletionResponse(BaseModel):
    date: str
    status: str
    notes: str | None

class HabitDetailResponse(HabitResponse):
    completions: list[CompletionResponse]
    longest_streak: int
```

### OpenAPI 示例

```python
class HabitCreate(BaseModel):
    name: str
    description: str | None = None

    model_config = ConfigDict(
        json_schema_extra={
            "examples": [
                {
                    "name": "晨练",
                    "description": "30 分钟有氧运动",
                }
            ]
        }
    )
```

---

## 4. 依赖注入

### 基本依赖

```python
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@router.get("/habits")
def list_habits(db: Session = Depends(get_db)):
    return db.query(Habit).all()
```

### 验证依赖

```python
from fastapi import Depends, HTTPException

async def valid_habit_id(habit_id: int, db: Session = Depends(get_db)) -> Habit:
    habit = db.query(Habit).filter(Habit.id == habit_id).first()
    if not habit:
        raise HTTPException(status_code=404, detail="习惯未找到")
    return habit

@router.get("/habits/{habit_id}")
async def get_habit(habit: Habit = Depends(valid_habit_id)):
    return habit  # 已验证并获取

@router.delete("/habits/{habit_id}")
async def delete_habit(habit: Habit = Depends(valid_habit_id), db: Session = Depends(get_db)):
    db.delete(habit)
    db.commit()
```

### 链式依赖

```python
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    # 解码并验证 token
    return user

async def get_current_active_user(user: User = Depends(get_current_user)) -> User:
    if not user.is_active:
        raise HTTPException(status_code=400, detail="用户未激活")
    return user

@router.get("/me")
async def read_current_user(user: User = Depends(get_current_active_user)):
    return user
```

### 类依赖

```python
class Pagination:
    def __init__(self, skip: int = 0, limit: int = Query(default=100, le=100)):
        self.skip = skip
        self.limit = limit

@router.get("/habits")
async def list_habits(pagination: Pagination = Depends()):
    return habits[pagination.skip : pagination.skip + pagination.limit]
```

### 关键点

- 默认情况下，依赖在**请求内被缓存**
- 使用 `Depends(dep, use_cache=False)` 禁用缓存
- 尽可能使用 `async def` 定义依赖
- 依赖可以依赖其他依赖（链接它们）

---

## 5. 错误处理

### HTTPException

```python
from fastapi import HTTPException, status

@router.get("/habits/{habit_id}")
async def get_habit(habit_id: int):
    habit = get_habit_by_id(habit_id)
    if not habit:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="习惯未找到",
            headers={"X-Error-Code": "HABIT_NOT_FOUND"},
        )
    return habit
```

### 自定义异常类

```python
# exceptions.py
class AppException(Exception):
    def __init__(self, status_code: int, detail: str, error_code: str):
        self.status_code = status_code
        self.detail = detail
        self.error_code = error_code

class HabitNotFoundError(AppException):
    def __init__(self, habit_id: int):
        super().__init__(
            status_code=404,
            detail=f"未找到 ID 为 {habit_id} 的习惯",
            error_code="HABIT_NOT_FOUND",
        )

class DuplicateCompletionError(AppException):
    def __init__(self, habit_id: int, date: str):
        super().__init__(
            status_code=409,
            detail=f"习惯 {habit_id} 在 {date} 已存在完成记录",
            error_code="DUPLICATE_COMPLETION",
        )
```

### 全局异常处理器

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": exc.error_code,
            "detail": exc.detail,
            "path": str(request.url),
        },
    )

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "error": "VALIDATION_ERROR",
            "detail": exc.errors(),
        },
    )
```

### 一致的错误响应格式

```json
{
    "error": "HABIT_NOT_FOUND",
    "detail": "未找到 ID 为 123 的习惯",
    "path": "/api/habits/123"
}
```

---

## 6. 数据库集成

### SQLAlchemy 设置

```python
# database.py
from sqlalchemy import create_engine, event
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = "sqlite:///./habits.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False},  # 仅 SQLite
)

# 应用 SQLite 的 PRAGMA 设置
@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA journal_mode=WAL")
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.execute("PRAGMA synchronous=NORMAL")
    cursor.close()

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
```

### 模型定义

```python
# models.py
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from datetime import datetime

class Habit(Base):
    __tablename__ = "habits"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    description = Column(Text)
    color = Column(String(7), default="#10B981")
    created_at = Column(DateTime, default=datetime.utcnow)
    archived_at = Column(DateTime, nullable=True)

    completions = relationship(
        "Completion",
        back_populates="habit",
        cascade="all, delete-orphan",
        lazy="selectin",  # 适合集合
    )

class Completion(Base):
    __tablename__ = "completions"

    id = Column(Integer, primary_key=True, index=True)
    habit_id = Column(Integer, ForeignKey("habits.id", ondelete="CASCADE"), nullable=False, index=True)
    completed_date = Column(String(10), nullable=False)  # YYYY-MM-DD
    status = Column(String(10), default="completed")  # completed, skipped
    notes = Column(Text)

    habit = relationship("Habit", back_populates="completions")

    __table_args__ = (
        UniqueConstraint("habit_id", "completed_date", name="uq_habit_date"),
    )
```

### 上下文管理器的依赖

```python
from typing import Generator
from sqlalchemy.orm import Session

def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 仓储模式（可选）

```python
# repositories/habits.py
class HabitRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_by_id(self, habit_id: int) -> Habit | None:
        return self.db.query(Habit).filter(Habit.id == habit_id).first()

    def get_all(self, include_archived: bool = False) -> list[Habit]:
        query = self.db.query(Habit)
        if not include_archived:
            query = query.filter(Habit.archived_at.is_(None))
        return query.all()

    def create(self, data: HabitCreate) -> Habit:
        habit = Habit(**data.model_dump())
        self.db.add(habit)
        self.db.commit()
        self.db.refresh(habit)
        return habit
```

### 预加载

```python
from sqlalchemy.orm import joinedload, selectinload

# 对于多对一关系
habit = db.query(Habit).options(joinedload(Habit.category)).first()

# 对于一对多集合
habits = db.query(Habit).options(selectinload(Habit.completions)).all()
```

---

## 7. 性能与异步

### 异步 vs 同步函数

| 工作类型 | 函数定义 | 原因 |
|-----------|---------------------|--------|
| 异步 I/O（数据库、HTTP） | `async def` | 非阻塞 |
| 同步/阻塞 I/O | `def` | 在线程池中运行 |
| CPU 密集型 | 外部 worker | 避免阻塞 |

```python
# 推荐：I/O 操作使用异步
@router.get("/habits")
async def list_habits(db: AsyncSession = Depends(get_async_db)):
    result = await db.execute(select(Habit))
    return result.scalars().all()

# 推荐：阻塞操作使用同步（FastAPI 处理线程池）
@router.get("/file")
def read_file():
    with open("file.txt") as f:
        return f.read()

# 错误：异步函数中调用阻塞操作
@router.get("/bad")
async def bad_endpoint():
    time.sleep(5)  # 阻塞整个事件循环！
    return {"status": "done"}
```

### 后台任务

```python
from fastapi import BackgroundTasks

def send_notification(email: str, message: str):
    # 长时间运行的任务
    pass

@router.post("/habits")
async def create_habit(habit: HabitCreate, background_tasks: BackgroundTasks):
    created_habit = create_habit_in_db(habit)
    background_tasks.add_task(send_notification, "user@example.com", "习惯已创建！")
    return created_habit
```

### 缓存

```python
from functools import lru_cache

# 缓存设置（调用一次）
@lru_cache
def get_settings():
    return Settings()

# 对于 Redis 缓存，使用 fastapi-cache
from fastapi_cache.decorator import cache

@router.get("/stats")
@cache(expire=60)
async def get_stats():
    return calculate_expensive_stats()
```

---

## 8. 测试

### 基本测试设置

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine, StaticPool
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.database import Base, get_db

@pytest.fixture(name="session")
def session_fixture():
    engine = create_engine(
        "sqlite:///:memory:",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    Base.metadata.create_all(engine)
    SessionLocal = sessionmaker(bind=engine)
    with SessionLocal() as session:
        yield session

@pytest.fixture(name="client")
def client_fixture(session):
    def get_session_override():
        return session

    app.dependency_overrides[get_db] = get_session_override
    client = TestClient(app)
    yield client
    app.dependency_overrides.clear()
```

### 编写测试

```python
# tests/test_habits.py
def test_create_habit(client):
    response = client.post(
        "/api/habits",
        json={"name": "Exercise", "description": "Daily workout"},
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Exercise"
    assert "id" in data

def test_create_habit_validation_error(client):
    response = client.post("/api/habits", json={"name": ""})
    assert response.status_code == 422

def test_get_habit_not_found(client):
    response = client.get("/api/habits/999")
    assert response.status_code == 404

def test_list_habits(client, session):
    # 设置
    habit = Habit(name="Test Habit")
    session.add(habit)
    session.commit()

    # 测试
    response = client.get("/api/habits")
    assert response.status_code == 200
    assert len(response.json()) == 1
```

### 异步测试

```python
import pytest
from httpx import AsyncClient, ASGITransport

@pytest.mark.anyio
async def test_async_endpoint():
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test",
    ) as client:
        response = await client.get("/api/habits")
        assert response.status_code == 200
```

---

## 9. 安全

### 输入验证

FastAPI + Pydantic 处理大部分验证。额外措施：

```python
from pydantic import BaseModel, Field, field_validator
import bleach

class CommentCreate(BaseModel):
    content: str = Field(..., max_length=1000)

    @field_validator("content")
    @classmethod
    def sanitize_content(cls, v: str) -> str:
        return bleach.clean(v)  # 移除 XSS 向量
```

### CORS 配置

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],  # 特定来源，不要使用 "*"
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

### SQL 注入防护

```python
# 推荐：使用 ORM
habit = db.query(Habit).filter(Habit.id == habit_id).first()

# 推荐：参数化查询
result = db.execute(text("SELECT * FROM habits WHERE id = :id"), {"id": habit_id})

# 错误：字符串格式化（存在漏洞！）
db.execute(f"SELECT * FROM habits WHERE id = {habit_id}")  # 绝对不要这样做
```

### 限流

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.get("/api/habits")
@limiter.limit("100/minute")
async def list_habits(request: Request):
    pass
```

---

## 10. 配置

### 使用 Pydantic Settings

```python
# config.py
from pydantic_settings import BaseSettings, SettingsConfigDict
from functools import lru_cache

class Settings(BaseSettings):
    app_name: str = "习惯追踪器"
    database_url: str = "sqlite:///./habits.db"
    debug: bool = False
    cors_origins: list[str] = ["http://localhost:5173"]

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### 使用

```python
from .config import get_settings

settings = get_settings()
print(settings.database_url)
```

### .env 文件

```env
DATABASE_URL=sqlite:///./habits.db
DEBUG=true
CORS_ORIGINS=["http://localhost:5173"]
```

---

## 11. 反模式

### 需要避免的关键反模式

| 反模式 | 问题 | 解决方案 |
|--------------|---------|----------|
| 在 `async def` 中阻塞 I/O | 停止事件循环 | 使用异步库或普通 `def` |
| 端点到端点调用 | 紧耦合 | 使用服务层 |
| 全局可变状态 | 竞态条件 | 使用 Redis/数据库 |
| 返回 ORM 对象 | 暴露内部细节 | 使用响应模式 |
| 不使用 Pydantic | 缺少验证 | 始终定义模式 |
| SQL 字符串格式化 | SQL 注入 | 使用 ORM 或参数化查询 |
| 硬编码配置 | 缺乏灵活性 | 使用环境变量 |

### 常见错误

```python
# 错误：异步中阻塞
async def bad():
    time.sleep(5)  # 阻塞事件循环

# 错误：未关闭连接
def bad_db():
    return SessionLocal()  # 永远不会关闭！

# 错误：请求间共享会话
db = SessionLocal()  # 全局会话 - 竞态条件！

# 错误：暴露内部模型
@router.get("/habits")
def list_habits(db: Session = Depends(get_db)):
    return db.query(Habit).all()  # 返回 ORM 对象

# 推荐：使用响应模型
@router.get("/habits", response_model=list[HabitResponse])
def list_habits(db: Session = Depends(get_db)):
    return db.query(Habit).all()  # 通过 Pydantic 序列化
```

---

## 生命周期事件

### 现代模式（推荐）

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动
    print("正在启动...")
    # 初始化资源（数据库连接池、缓存等）
    yield
    # 关闭
    print("正在关闭...")
    # 清理资源

app = FastAPI(lifespan=lifespan)
```

---

## 快速参考

### HTTP 状态码

| 码 | 常量 | 使用场景 |
|------|----------|----------|
| 200 | `HTTP_200_OK` | 成功的 GET、PUT |
| 201 | `HTTP_201_CREATED` | 成功的 POST |
| 204 | `HTTP_204_NO_CONTENT` | 成功的 DELETE |
| 400 | `HTTP_400_BAD_REQUEST` | 无效请求 |
| 404 | `HTTP_404_NOT_FOUND` | 资源未找到 |
| 409 | `HTTP_409_CONFLICT` | 重复资源 |
| 422 | `HTTP_422_UNPROCESSABLE_ENTITY` | 验证错误 |
| 500 | `HTTP_500_INTERNAL_SERVER_ERROR` | 服务器错误 |

### 常用导入

```python
from fastapi import FastAPI, APIRouter, Depends, HTTPException, status, Query, Path, Body
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field, field_validator, ConfigDict
from sqlalchemy.orm import Session
from typing import Annotated
```

---

## 资源

- [FastAPI 文档](https://fastapi.tiangolo.com/)
- [Pydantic 文档](https://docs.pydantic.dev/)
- [SQLAlchemy 2.0 文档](https://docs.sqlalchemy.org/en/20/)
- [FastAPI 最佳实践 (GitHub)](https://github.com/zhanymkanov/fastapi-best-practices)
