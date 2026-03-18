# OpenClaw 功能清单

**文档状态：** 参考文档  
**版本：** 1.0  
**创建日期：** 2025-03-18  
**说明：** 本文档整理自 OpenClaw 官方文档、社区资料及公开信息，供 SparkClaw 架构设计参考。

---

## 1. 核心定位

- **开源、本地优先的 AI Agent 平台**，定位为可自主执行任务的「数字员工」
- 支持 7×24 小时待命、主动执行任务
- 数据本地存储，隐私可控

---

## 2. 系统架构

| 层级 | 名称 | 功能 |
|------|------|------|
| **Channel** | 消息渠道接入层 | 连接 20+ 即时通讯平台，作为 Agent 的对外入口 |
| **Node** | 设备端执行节点 | 在本地执行操作（摄像头、录屏、终端命令等） |
| **Gateway** | 中央控制平面 | 维护 WebSocket、会话管理、任务调度，24 小时常驻 |

---

## 3. 通道适配器 (Channels)

### 3.1 国内主流

| 通道 | 说明 |
|------|------|
| 飞书 | 国内企业办公首选，原生内置支持 |
| QQ | 腾讯官方 Bot 能力，扫码即可绑定 |
| 微信 | 微信集成 |
| 钉钉 | 钉钉集成 |

### 3.2 国际主流

| 通道 | 说明 |
|------|------|
| WhatsApp | 海外用户使用率高，扫码登录 |
| Telegram | 官方推荐，5 分钟零门槛，长轮询无需公网 IP |
| Discord | 极客社区群组协作 |
| Slack | Slack 机器人 |
| Google Chat | Google Chat 集成 |
| Signal | Signal 集成 |
| MS Teams | Microsoft Teams 机器人 |

### 3.3 其他

| 通道 | 说明 |
|------|------|
| Web Chat | 网页聊天界面 |
| REST API | HTTP API 接入 |
| Email | 邮件入站/出站 |

---

## 4. 工具能力 (Tools)

### 4.1 按分组

| 分组 | 工具 | 说明 |
|------|------|------|
| **group:fs** | read, write, edit, apply_patch | 文件读写、编辑、结构化补丁 |
| **group:runtime** | exec, bash, process | Shell 执行、后台进程管理 |
| **group:web** | web_search, web_fetch | 网页搜索、URL 抓取 |
| **group:memory** | memory_search, memory_get | 记忆检索与获取 |
| **group:sessions** | sessions_list, sessions_history, sessions_send, sessions_spawn | 会话管理、子 Agent 孵化 |
| **group:messaging** | message | 跨通道消息发送（send, react, edit, delete, poll 等） |
| **group:automation** | cron, gateway | 定时任务、网关配置热更新 |
| **group:ui** | browser, canvas | 浏览器控制、Canvas 渲染 |
| **group:nodes** | nodes | 移动节点发现、通知、相机、录屏 |
| **独立工具** | image, image_generate, pdf | 图像分析/生成、PDF 分析 |
| **其他** | loop-detection | 工具调用循环检测与熔断 |

### 4.2 工具策略 (Tool Policies)

| 能力 | 说明 |
|------|------|
| tools.allow / tools.deny | 全局工具白名单/黑名单 |
| tools.profile | 预设配置：full, messaging, coding, minimal |
| tools.byProvider | 按模型提供商限制工具 |
| group:* 快捷引用 | group:fs, group:runtime 等分组 |
| 每 Agent 覆盖 | agents.list[].tools.* |

---

## 5. 记忆系统 (Memory)

### 5.1 四层记忆架构

| 层级 | 名称 | 说明 |
|------|------|------|
| **Session** | 实时情景 | 当前对话上下文，含 Pre-Compaction 自动记忆保存 |
| **USER** | 语义长期记忆 | `USER.md`、`MEMORY.md`，支持向量检索 |
| **TOOLS** | 动态工具 | 当前可用的工具与 Skills |
| **SOUL** | 不可变内核 | `SOUL.md`，定义人格、价值观与身份 |

### 5.2 记忆工具

| 工具 | 说明 |
|------|------|
| memory_get | 读取指定 Markdown 文件/行范围 |
| memory_search | 基于向量的语义检索 |

### 5.3 共享记忆 (openclaw-shared-memory 插件)

| 特性 | 说明 |
|------|------|
| 访问控制 | 三层：`private` / `shared-read` / `shared-write` |
| 存储 | SQLite + Markdown，零外部依赖 |
| 检索 | 向量语义搜索 |
| 配置 | 支持 OpenAI、Anthropic、Google、Voyage 或本地嵌入模型 |

---

## 6. 自动化 (Automation)

| 能力 | 说明 |
|------|------|
| **Heartbeat** | 周期性唤醒（默认 30 分钟），主动检查待办并汇报 |
| **Cron** | 定时任务：add, update, remove, run, runs, status, list |
| **activeHours** | 心跳活跃时段限制（时区感知） |
| **HEARTBEAT.md** | 工作区心跳检查清单 |
| **Manual wake** | 按需触发心跳 (system event) |

---

## 7. 会话与路由 (Sessions & Routing)

| 能力 | 说明 |
|------|------|
| 多 Agent 路由 | 按 workspace/sender 隔离会话 |
| 直接会话折叠 | 私聊合并为 main 会话 |
| 群组隔离 | 群聊各自独立会话 |
| @ 提及激活 | 群聊中基于 @ 唤醒 Agent |
| sessions_spawn | 子 Agent 孵化（subagent / ACP runtime） |

---

## 8. 应用场景

| 场景 | 能力 |
|------|------|
| **文件与系统** | 批量重命名、整理、备份、读写文档、执行终端命令 |
| **浏览器** | 自动搜索、填表、登录、抓取、监控、批量下载 |
| **办公** | 邮件筛选/回复/摘要、日程同步、会议纪要、周报、Excel 处理 |
| **开发** | 代码生成、调试、测试、PR 合并、CI/CD 辅助 |
| **跨工具协同** | 集成 20+ 通讯工具、批量消息、OA/CRM/API 对接 |
| **自定义扩展** | 自然语言创建流程、安装社区插件、编写 Skills |

---

## 9. 生态与扩展

| 组件 | 说明 |
|------|------|
| **ClawHub** | 官方 Skill 市场，13,000+ 技能 |
| **Moltbook** | AI Agent 社交网络，32,000+ Agent 注册 |
| **插件 SDK** | 扩展通道、工具、CLI |
| **MCP 协议** | 可桥接 Claude Code 等外部能力 |

---

## 10. 媒体与输入 (Media & Inputs)

| 能力 | 说明 |
|------|------|
| 图像入站/出站 | 图片消息、截图、生成图 |
| 音频入站/出站 | 语音消息、转录 |
| 文档 | PDF 分析、多模态文档 |
| 会话附件 | 内联文件支持 |
| 流式响应 | 长回复分块、流式输出 |

---

## 11. 移动节点 (Mobile Nodes)

| 平台 | 能力 |
|------|------|
| **Android** | 配对、Connect 标签、聊天、语音、Canvas/相机、设备命令、通知、通讯录/日历、运动、照片、SMS |
| **iOS** | 配对、Canvas、相机、录屏、定位、语音 |
| **macOS** | 菜单栏应用、system.run、system.notify |
| **WebChat** | 网页聊天客户端 |

---

## 12. 配置与安全

| 能力 | 说明 |
|------|------|
| 工具策略 | tools.allow/deny、tools.profile、tools.byProvider |
| 多 Agent 路由 | 按 workspace/sender 隔离会话 |
| Fallback 链 | 多模型降级，控制成本 |
| 预算控制 | maxCostPerDay、maxTokensPerDay |
| Gateway 认证 | Token 或密码认证 |

---

## 13. 与 SparkClaw 的关联

本项目中 [ARCH-multi-agent-consolidated-solution.md](./ARCH-multi-agent-consolidated-solution.md) 借鉴了 **openclaw-shared-memory** 的三层访问控制设计：

- `private`：仅创建者 Agent 可读写
- `shared-read`：所有 Agent 可读，仅创建者可写
- `shared-write`：所有 Agent 可读写（需 Reducer 合并）

---

## 14. 参考资源

- [OpenClaw 官网](https://openclaw.ai/)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw 官方文档](https://openclaws.io/docs/)
- [openclaw-shared-memory](https://github.com/dhruvja/openclaw-shared-memory)
- [ClawHub 技能市场](https://clawhub.ai/)
