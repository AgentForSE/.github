# 模块八：平台基础服务（`platform-core`）

> 收敛共性 #1、#4、#14、#15、#16、#17、#18 —— 即所有产品共用的平台基础能力。

## 职责

提供四款产品共享的**应用平台底座**：用户与权限、项目管理、异步任务调度、
WebSocket 实时推送、操作 / 任务日志、可视化看板数据接口。

## 能力清单

1. **用户与权限管理**
   - 鉴权：JWT / Token（兼容 SmartST 现有 X-Auth-Token 体系，逐步迁移）。
   - 角色 / 权限点：RBAC + 项目级成员权限。
   - 账号体系：登录 / 密码 / 重置 / 用户列表 / 禁用 / 审计字段。
2. **项目管理**
   - 多项目隔离（项目 ID 是所有业务表的强外键）。
   - 项目配置快照与恢复（参数配置、需求树、设计树、经验确认状态、
     工具扫描各层级智判结果——参考 SmartInsight 项目审查恢复需求）。
   - 项目级统计面板。
3. **异步任务调度**
   - 任务队列（Celery / Arq / Temporal 任选其一，组织内统一）。
   - 进度追踪、失败重试、可视化（参考 SmartST 解析任务、SafeWise 流水线任务）。
4. **WebSocket 实时推送**
   - 统一推送通道（按 `project` + `topic` 订阅）。
   - 与 [`agent-pipeline`](./agent-pipeline.md) 节点状态机联动；
     与 [`llm-gateway`](./llm-gateway.md) 流式输出桥接。
5. **操作日志与审计**
   - 关键接口与任务执行过程可追踪。
   - 统一日志页面：按项目 / 用户 / 时间筛选；错误告警与下载。
6. **可视化看板数据接口**
   - 覆盖率 / 进度 / 质量趋势统一聚合接口（前端按需调用）。
   - 与 [`traceability`](./traceability.md) 的查询打通，输出跟踪关系视图数据。
7. **对象存储**
   - 文档原件、报告产物、PNG 图等统一存储（S3 兼容协议）。
   - 提供签名 URL 下载。

## 对外接口（建议，节选）

| 接口 | 说明 |
|------|------|
| `POST /v1/auth/login`                | 登录 |
| `GET  /v1/me`                        | 当前用户 |
| `POST /v1/projects` / `GET /v1/projects/{id}` | 项目 |
| `POST /v1/tasks` / `GET /v1/tasks/{id}` | 异步任务 |
| `WS   /v1/ws?project=...`            | WebSocket 订阅 |
| `GET  /v1/dashboard?project=...`     | 看板聚合数据 |
| `GET  /v1/audit-log`                 | 审计日志 |

## 业务方使用约束

- **禁止**自建用户表 / 项目表；通过 SDK 引用平台核心。
- 长耗时操作**必须**走异步任务，禁止直接在 HTTP 请求上下文里跑。
- 实时进度**必须**通过 WebSocket，不允许前端轮询。
- 凡涉及核心数据访问，必须经过权限校验中间件，并记录审计日志。

## 与其它模块的关系

- 是其它所有模块的运行时宿主，提供租户 / 项目 / 用户的统一上下文。
- WebSocket 通道桥接 [`agent-pipeline`](./agent-pipeline.md) 状态与
  [`llm-gateway`](./llm-gateway.md) 流式输出。
- 看板聚合数据从 [`traceability`](./traceability.md) 与各业务库取数。
