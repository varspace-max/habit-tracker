# SQLite 和 SQL 最佳实践参考指南

在 Python 应用程序中使用 SQLite 数据库的简明参考指南。

---

## 目录

1. [何时使用 SQLite](#1-何时使用-sqlite)
2. [数据库设计](#2-数据库设计)
3. [数据类型](#3-数据类型)
4. [索引](#4-索引)
5. [查询优化](#5-查询优化)
6. [SQLAlchemy 模式](#6-sqlalchemy-模式)
7. [数据完整性](#7-数据完整性)
8. [事务](#8-事务)
9. [Python 集成](#9-python-集成)
10. [性能调优](#10-性能调优)
11. [备份与恢复](#11-备份与恢复)
12. [反模式](#12-反模式)

---

## 1. 何时使用 SQLite

### 理想使用场景

- **嵌入式/物联网设备**：移动应用、桌面应用、本地工具
- **应用程序文件格式**：用于应用数据的单文件数据库
- **低至中等流量的网站**：每日请求量低于 10 万次
- **开发和测试**：快速设置，无需服务器
- **数据分析**：导入 CSV，运行 SQL 查询
- **缓存层**：远程数据的本地缓存
- **单用户应用程序**：个人工具、本地应用

### 不适合使用 SQLite 的场景

- **高并发写入**：SQLite 一次只允许一个写入操作
- **网络文件系统**：NFS、SMB 可能导致数据损坏
- **多台服务器**：无法在多台机器间共享 SQLite
- **超大数据集**：超过 1TB 可能需要分布式方案
- **高流量生产环境**：考虑使用 PostgreSQL

### 关键特性

| 特性 | 数值 |
|---------|-------|
| 库大小 | <600KB |
| 最大数据库大小 | 281 TB |
| 最大行大小 | 1 GB |
| 并发读取器 | 无限制 |
| 并发写入器 | 1 |
| ACID 兼容 | 是 |

---

## 2. 数据库设计

### 主键

```sql
-- 推荐：INTEGER PRIMARY KEY（别名 rowid，自动递增）
CREATE TABLE habits (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    created_at TEXT NOT NULL
);

-- 使用显式 AUTOINCREMENT（删除后防止重用 rowid）
CREATE TABLE habits (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);

-- 复合主键
CREATE TABLE completions (
    habit_id INTEGER NOT NULL,
    completed_date TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'completed',
    PRIMARY KEY (habit_id, completed_date)
);
```

### 外键

```sql
-- 外键默认禁用 - 必须在每个连接中启用
PRAGMA foreign_keys = ON;

CREATE TABLE completions (
    id INTEGER PRIMARY KEY,
    habit_id INTEGER NOT NULL,
    completed_date TEXT NOT NULL,
    FOREIGN KEY (habit_id) REFERENCES habits(id) ON DELETE CASCADE
);
```

**级联操作**：

| 操作 | 行为 |
|--------|----------|
| `NO ACTION` | 如果存在子行则拒绝（默认） |
| `CASCADE` | 删除/更新子行 |
| `SET NULL` | 将外键设置为 NULL |
| `SET DEFAULT` | 将外键设置为默认值 |
| `RESTRICT` | 类似 NO ACTION 但立即生效 |

### 表约束

```sql
CREATE TABLE habits (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    color TEXT DEFAULT '#10B981',
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    archived_at TEXT,

    -- 检查约束
    CHECK (length(name) > 0),
    CHECK (color GLOB '#[0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f][0-9A-Fa-f]')
);

CREATE TABLE completions (
    id INTEGER PRIMARY KEY,
    habit_id INTEGER NOT NULL,
    completed_date TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'completed',

    FOREIGN KEY (habit_id) REFERENCES habits(id) ON DELETE CASCADE,
    UNIQUE (habit_id, completed_date),
    CHECK (status IN ('completed', 'skipped'))
);
```

### WITHOUT ROWID 表

```sql
-- 用于非整数或复合主键
CREATE TABLE settings (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL
) WITHOUT ROWID;
```

**使用场景**：
- 非整数主键
- 复合主键
- 行较小
- 频繁的主键查询

**避免使用**：
- 大主键（所有索引中都会复制）
- 多个辅助索引
- 大行

---

## 3. 数据类型

### SQLite 类型亲和性

SQLite 使用动态类型 - 类型与值关联，而非与列关联。

**五种存储类**：

| 类 | 描述 |
|-------|-------------|
| `NULL` | 空值 |
| `INTEGER` | 有符号整数（1-8 字节） |
| `REAL` | 8 字节 IEEE 浮点数 |
| `TEXT` | UTF-8/UTF-16 字符串 |
| `BLOB` | 二进制数据 |

**类型亲和性规则**（基于声明的类型名称）：
1. 包含 "INT" → INTEGER
2. 包含 "CHAR"、"CLOB"、"TEXT" → TEXT
3. 包含 "BLOB" 或无类型 → BLOB
4. 包含 "REAL"、"FLOA"、"DOUB" → REAL
5. 其他 → NUMERIC

### 日期/时间存储

**选项 1：TEXT（ISO 8601）- 推荐**

```sql
CREATE TABLE completions (
    id INTEGER PRIMARY KEY,
    completed_date TEXT NOT NULL,  -- 'YYYY-MM-DD'
    created_at TEXT NOT NULL DEFAULT (datetime('now'))  -- 'YYYY-MM-DD HH:MM:SS'
);

-- 查询示例
SELECT * FROM completions WHERE completed_date = '2025-01-15';
SELECT * FROM completions WHERE completed_date >= '2025-01-01' AND completed_date < '2025-02-01';
SELECT * FROM completions WHERE completed_date BETWEEN '2025-01-01' AND '2025-01-31';
```

**优点**：人类可读、字典序可排序、与 SQLite 日期函数配合使用。

**选项 2：INTEGER（Unix 时间戳）**

```sql
CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    timestamp INTEGER NOT NULL DEFAULT (strftime('%s', 'now'))
);

-- 查询示例
SELECT * FROM events WHERE timestamp >= strftime('%s', '2025-01-01');
SELECT datetime(timestamp, 'unixepoch') as readable_time FROM events;
```

**优点**：存储更小（8 字节），比较更快。

### 布尔值处理

```sql
-- SQLite 没有原生的 BOOLEAN - 使用 INTEGER 0/1
CREATE TABLE habits (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1 CHECK (is_active IN (0, 1))
);

-- TRUE 和 FALSE 是 1 和 0 的别名（SQLite 3.23.0+）
INSERT INTO habits (name, is_active) VALUES ('Exercise', TRUE);
SELECT * FROM habits WHERE is_active = TRUE;
```

### JSON 存储

```sql
-- 存储为 TEXT，使用 JSON 函数查询（SQLite 3.38.0+）
CREATE TABLE habits (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    settings TEXT  -- JSON 字符串
);

INSERT INTO habits (name, settings)
VALUES ('Exercise', '{"reminder_time": "09:00", "notifications": true}');

-- 查询 JSON
SELECT
    name,
    json_extract(settings, '$.reminder_time') as reminder
FROM habits
WHERE json_extract(settings, '$.notifications') = 1;
```

### STRICT 表（SQLite 3.37.0+）

```sql
-- 强制类型检查
CREATE TABLE habits (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    count INTEGER NOT NULL
) STRICT;

-- 这将失败：INSERT INTO habits (name, count) VALUES ('Test', 'not a number');
```

---

## 4. 索引

### 何时创建索引

- WHERE 子句中的列
- JOIN 条件中的列
- ORDER BY 子句中的列
- 外键列（对 CASCADE 操作至关重要）

### 索引类型

```sql
-- 单列索引
CREATE INDEX idx_habits_name ON habits(name);

-- 复合索引（列顺序很重要！）
CREATE INDEX idx_completions_habit_date ON completions(habit_id, completed_date);

-- 唯一索引
CREATE UNIQUE INDEX idx_habits_name_unique ON habits(name);

-- 部分索引（索引行的子集）
CREATE INDEX idx_active_habits ON habits(name) WHERE archived_at IS NULL;

-- 表达式索引
CREATE INDEX idx_habits_lower_name ON habits(lower(name));
```

### 复合索引列顺序

复合索引中列的顺序很重要：

```sql
CREATE INDEX idx_completions ON completions(habit_id, completed_date);

-- 使用索引（habit_id 是最左边的列）
SELECT * FROM completions WHERE habit_id = 1;

-- 使用索引（两列，按顺序）
SELECT * FROM completions WHERE habit_id = 1 AND completed_date = '2025-01-15';

-- 不能高效使用索引（completed_date 不是最左边的列）
SELECT * FROM completions WHERE completed_date = '2025-01-15';
```

### 覆盖索引

```sql
-- 包含查询需要的所有列，避免表查找
CREATE INDEX idx_completions_covering ON completions(habit_id, completed_date, status);

-- 这个查询完全由索引满足
SELECT completed_date, status FROM completions WHERE habit_id = 1;
```

### 索引权衡

| 优点 | 缺点 |
|---------|------|
| 读取更快 | 写入更慢 |
| ORDER BY 更快 | 占用更多磁盘空间 |
| JOIN 更快 | 内存开销 |

**经验法则**：每个辅助索引大约会使 INSERT 慢 5 倍。

---

## 5. 查询优化

### EXPLAIN QUERY PLAN

```sql
EXPLAIN QUERY PLAN
SELECT h.name, COUNT(*) as completions
FROM habits h
JOIN completions c ON h.id = c.habit_id
WHERE c.completed_date >= '2025-01-01'
GROUP BY h.id;

-- 输出解释：
-- SCAN = 全表扫描（通常不好）
-- SEARCH = 使用索引（好）
-- USING INDEX = 纯索引访问（最好）
-- USING COVERING INDEX = 无需表访问（最好）
```

### 查询技巧

```sql
-- 不好：SELECT *
SELECT * FROM habits;

-- 好：只选择需要的列
SELECT id, name, created_at FROM habits;

-- 不好：在没有索引的情况下使用 LIKE 进行前缀搜索
SELECT * FROM habits WHERE name LIKE '%exercise%';

-- 好：前缀 LIKE 可以使用索引
SELECT * FROM habits WHERE name LIKE 'exercise%';

-- 不好：对索引列使用函数
SELECT * FROM habits WHERE lower(name) = 'exercise';

-- 好：创建表达式索引，或规范化数据
CREATE INDEX idx_lower_name ON habits(lower(name));

-- 不好：不同列上的 OR（难以优化）
SELECT * FROM habits WHERE name = 'Exercise' OR description = 'workout';

-- 好：对复杂的 OR 条件使用 UNION
SELECT * FROM habits WHERE name = 'Exercise'
UNION
SELECT * FROM habits WHERE description = 'workout';
```

### 高效的日期查询

```sql
-- 对于 TEXT 日期（ISO 8601）
SELECT * FROM completions
WHERE completed_date >= '2025-01-01'
  AND completed_date < '2025-02-01';

-- 对于当前月份
SELECT * FROM completions
WHERE completed_date >= date('now', 'start of month')
  AND completed_date < date('now', 'start of month', '+1 month');

-- 不要使用 LIKE 查询日期
-- 不好：WHERE completed_date LIKE '2025-01%'
```

### 运行 ANALYZE

```sql
-- 更新查询计划器的统计信息
ANALYZE;

-- 关闭连接前运行（SQLite 3.18.0+）
PRAGMA optimize;
```

---

## 6. SQLAlchemy 模式

### 引擎设置

```python
from sqlalchemy import create_engine, event

engine = create_engine(
    "sqlite:///habits.db",
    connect_args={"check_same_thread": False},  # 用于多线程应用
    echo=False,  # 设置 True 以记录 SQL
)

# 每次连接时应用 PRAGMA 设置
@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA journal_mode=WAL")
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.execute("PRAGMA synchronous=NORMAL")
    cursor.execute("PRAGMA cache_size=-64000")  # 64MB
    cursor.execute("PRAGMA temp_store=MEMORY")
    cursor.close()
```

### 模型定义

```python
from sqlalchemy import Column, Integer, String, Text, ForeignKey, UniqueConstraint, CheckConstraint
from sqlalchemy.orm import relationship, declarative_base

Base = declarative_base()

class Habit(Base):
    __tablename__ = "habits"

    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    description = Column(Text)
    color = Column(String(7), default="#10B981")
    created_at = Column(String(19), nullable=False)  # YYYY-MM-DD HH:MM:SS
    archived_at = Column(String(19))

    completions = relationship(
        "Completion",
        back_populates="habit",
        cascade="all, delete-orphan",
        lazy="selectin",
    )

    __table_args__ = (
        CheckConstraint("length(name) > 0", name="name_not_empty"),
    )


class Completion(Base):
    __tablename__ = "completions"

    id = Column(Integer, primary_key=True)
    habit_id = Column(Integer, ForeignKey("habits.id", ondelete="CASCADE"), nullable=False, index=True)
    completed_date = Column(String(10), nullable=False)  # YYYY-MM-DD
    status = Column(String(10), default="completed")
    notes = Column(Text)

    habit = relationship("Habit", back_populates="completions")

    __table_args__ = (
        UniqueConstraint("habit_id", "completed_date", name="uq_habit_date"),
        CheckConstraint("status IN ('completed', 'skipped')", name="valid_status"),
    )
```

### 关系加载策略

| 策略 | 使用场景 |
|----------|----------|
| `lazy="select"` | 默认，访问时会出现 N+1 问题 |
| `lazy="joined"` | 多对一关系 |
| `lazy="selectin"` | 一对多集合 |
| `lazy="raise"` | 防止意外的延迟加载 |

```python
from sqlalchemy.orm import joinedload, selectinload

# 查询中预加载
habits = session.query(Habit).options(selectinload(Habit.completions)).all()

# 对于多对一
completions = session.query(Completion).options(joinedload(Completion.habit)).all()
```

### 会话管理

```python
from sqlalchemy.orm import sessionmaker, Session

SessionLocal = sessionmaker(bind=engine)

# 上下文管理器模式（推荐）
def get_habits():
    with Session(engine) as session:
        return session.query(Habit).all()

# 带事务处理
def create_habit(name: str):
    with Session(engine) as session, session.begin():
        habit = Habit(name=name, created_at=datetime.now().isoformat())
        session.add(habit)
        # 成功时自动提交，异常时自动回滚
        return habit

# 用于 FastAPI 依赖
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 7. 数据完整性

### 约束汇总

```sql
CREATE TABLE example (
    id INTEGER PRIMARY KEY,                    -- 主键
    name TEXT NOT NULL,                        -- 必填字段
    email TEXT UNIQUE,                         -- 不允许重复
    age INTEGER CHECK (age >= 0),              -- 值验证
    category_id INTEGER REFERENCES categories(id),  -- 外键
    status TEXT DEFAULT 'active'               -- 默认值
);
```

### 强制执行外键

```python
# 必须在每个连接中启用外键
@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.close()

# 验证已启用
result = connection.execute("PRAGMA foreign_keys").fetchone()
assert result[0] == 1
```

### 软删除

```sql
-- 不使用 DELETE，而是设置 archived_at
UPDATE habits SET archived_at = datetime('now') WHERE id = 1;

-- 查询活动记录
SELECT * FROM habits WHERE archived_at IS NULL;

-- 为活动记录创建部分索引
CREATE INDEX idx_active_habits ON habits(name) WHERE archived_at IS NULL;
```

---

## 8. 事务

### 基本事务

```sql
BEGIN TRANSACTION;
INSERT INTO habits (name, created_at) VALUES ('Exercise', datetime('now'));
INSERT INTO completions (habit_id, completed_date) VALUES (last_insert_rowid(), '2025-01-15');
COMMIT;

-- 发生错误时
ROLLBACK;
```

### Python 事务

```python
# 显式事务
with engine.begin() as connection:
    connection.execute(text("INSERT INTO habits ..."))
    connection.execute(text("INSERT INTO completions ..."))
    # 成功时自动提交，异常时自动回滚

# SQLAlchemy ORM
with Session(engine) as session, session.begin():
    habit = Habit(name="Exercise")
    session.add(habit)
    completion = Completion(habit=habit, completed_date="2025-01-15")
    session.add(completion)
    # 自动提交/回滚
```

### 隔离级别

SQLite 默认支持可序列化隔离。写入操作会阻塞其他写入操作，但读取操作永远不会阻塞。

---

## 9. Python 集成

### 连接基础

```python
import sqlite3

# 上下文管理器（成功时提交，但不会关闭！）
with sqlite3.connect("habits.db") as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM habits")
    rows = cursor.fetchall()

# 显式关闭
conn = sqlite3.connect("habits.db")
try:
    # ... 操作
    conn.commit()
finally:
    conn.close()
```

### 参数化查询（防止 SQL 注入）

```python
# 问号占位符
cursor.execute(
    "INSERT INTO habits (name, description) VALUES (?, ?)",
    (name, description)
)

# 命名占位符
cursor.execute(
    "INSERT INTO habits (name, description) VALUES (:name, :desc)",
    {"name": name, "desc": description}
)

# 用于 IN 子句
ids = [1, 2, 3]
placeholders = ",".join("?" * len(ids))
cursor.execute(f"SELECT * FROM habits WHERE id IN ({placeholders})", ids)

# 永远不要使用字符串格式化！
# 不好：cursor.execute(f"SELECT * FROM habits WHERE name = '{user_input}'")
```

### 行工厂

```python
# 将行作为字典返回
conn.row_factory = sqlite3.Row
cursor = conn.cursor()
cursor.execute("SELECT * FROM habits")
row = cursor.fetchone()
print(row["name"])  # 按列名访问
print(row[0])       # 按索引访问
print(dict(row))    # 转换为字典
```

### 批量操作

```python
# executemany 用于批量插入
data = [("Exercise",), ("Reading",), ("Meditation",)]
cursor.executemany("INSERT INTO habits (name) VALUES (?)", data)

# 为提高性能而包装在事务中
conn.execute("BEGIN")
try:
    for chunk in chunks(large_data, 1000):
        cursor.executemany("INSERT INTO habits (name) VALUES (?)", chunk)
    conn.commit()
except:
    conn.rollback()
    raise
```

---

## 10. 性能调优

### 必需的 PRAGMA 设置

```sql
-- 每次连接时运行
PRAGMA journal_mode = WAL;        -- 预写日志（更好的并发性）
PRAGMA synchronous = NORMAL;      -- 与 WAL 一起使用安全，比 FULL 更快
PRAGMA foreign_keys = ON;         -- 启用外键强制
PRAGMA cache_size = -64000;       -- 64MB 页面缓存（负数 = KB）
PRAGMA temp_store = MEMORY;       -- 将临时表存储在内存中
PRAGMA mmap_size = 268435456;     -- 256MB 内存映射 I/O

-- 定期或关闭前运行
PRAGMA optimize;                   -- 优化查询计划器统计信息
```

### PRAGMA 参考

| PRAGMA | 目的 | 推荐值 |
|--------|---------|-------------|
| `journal_mode` | 事务日志 | `WAL` |
| `synchronous` | 磁盘同步频率 | `NORMAL`（配合 WAL） |
| `foreign_keys` | 外键强制 | `ON` |
| `cache_size` | 页面缓存大小 | `-64000`（64MB） |
| `temp_store` | 临时表位置 | `MEMORY` |
| `busy_timeout` | 锁等待时间（毫秒） | `5000` |

### WAL 模式

```sql
PRAGMA journal_mode = WAL;
```

**优点**：
- 读取器不会阻塞写入器
- 写入器不会阻塞读取器
- 更好的崩溃恢复
- 对于大多数工作负载都更快

**限制**：
- 不适用于网络文件系统
- 会在数据库旁边创建 `-wal` 和 `-shm` 文件

### 数据库维护

```sql
-- 碎片整理和优化（在维护窗口期间运行）
VACUUM;

-- 更新查询计划器的统计信息
ANALYZE;

-- 重建索引
REINDEX;

-- 检查数据库完整性
PRAGMA integrity_check;
```

---

## 11. 备份与恢复

### 安全的备份方法

```python
import sqlite3

def backup_database(source_path: str, dest_path: str):
    """使用 SQLite 的备份 API 进行安全备份。"""
    source = sqlite3.connect(source_path)
    dest = sqlite3.connect(dest_path)

    with dest:
        source.backup(dest)

    dest.close()
    source.close()
```

```sql
-- VACUUM INTO 创建真空副本（SQLite 3.27.0+）
VACUUM INTO '/path/to/backup.db';
```

### 不应该做的事情

```bash
# 永远不要在活动数据库上使用 cp/copy - 不是事务安全的！
cp database.db backup.db  # 不好！
```

### Litestream（持续备份）

```yaml
# litestream.yml
dbs:
  - path: /data/habits.db
    replicas:
      - url: s3://bucket-name/habits
        sync-interval: 1s
```

### 完整性检查

```sql
-- 完整完整性检查
PRAGMA integrity_check;

-- 快速检查（更快）
PRAGMA quick_check;

-- 如果健康则返回 'ok'
```

---

## 12. 反模式

### 配置错误

| 错误 | 解决方案 |
|---------|----------|
| 未启用外键 | 每个连接使用 `PRAGMA foreign_keys=ON` |
| 使用默认日志模式 | 启用 WAL：`PRAGMA journal_mode=WAL` |
| SQLite 在网络文件系统上 | 只使用本地文件系统 |

### 设计错误

| 错误 | 解决方案 |
|---------|----------|
| 存储逗号分隔的列表 | 使用适当的关联表 |
| 未索引外键 | 始终为外键列创建索引 |
| 过度索引 | 只为频繁查询的列创建索引 |
| 使用错误的日期格式 | 使用 ISO 8601：`YYYY-MM-DD` |

### 查询错误

| 错误 | 解决方案 |
|---------|----------|
| `SELECT *` | 只选择需要的列 |
| LIKE 用于日期查询 | 使用日期比较 |
| 对索引列使用函数 | 创建表达式索引 |
| 不使用 EXPLAIN | 分析慢查询 |

### Python 错误

| 错误 | 解决方案 |
|---------|----------|
| 字符串格式化 SQL | 使用参数化查询 |
| 未关闭连接 | 使用上下文管理器 |
| 每次请求创建引擎 | 创建一次，复用 |
| 忽略 N+1 查询 | 使用预加载 |

---

## 快速参考

### 连接设置模板

```python
from sqlalchemy import create_engine, event

engine = create_engine("sqlite:///habits.db", connect_args={"check_same_thread": False})

@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA journal_mode=WAL")
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.execute("PRAGMA synchronous=NORMAL")
    cursor.execute("PRAGMA cache_size=-64000")
    cursor.execute("PRAGMA temp_store=MEMORY")
    cursor.close()
```

### SQLite 日期函数

```sql
-- 当前日期/时间
SELECT date('now');                    -- 2025-01-15
SELECT datetime('now');                -- 2025-01-15 12:30:00
SELECT strftime('%s', 'now');          -- Unix 时间戳

-- 日期运算
SELECT date('now', '-7 days');         -- 7 天前
SELECT date('now', '+1 month');        -- 1 个月后
SELECT date('now', 'start of month');  -- 当月第一天

-- 提取部分
SELECT strftime('%Y', '2025-01-15');   -- 2025
SELECT strftime('%m', '2025-01-15');   -- 01
SELECT strftime('%d', '2025-01-15');   -- 15
```

---

## 资源

- [SQLite 文档](https://sqlite.org/docs.html)
- [SQLite 何时使用](https://sqlite.org/whentouse.html)
- [SQLAlchemy 2.0 文档](https://docs.sqlalchemy.org/en/20/)
- [Litestream](https://litestream.io/)
