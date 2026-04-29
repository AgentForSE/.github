# 六、统一技术栈与工程规范

为保障公共底座可复用、产品可整合、运维可控，组织内统一技术选型与工程规范。

## 1. 推荐技术栈

| 层 | 选型 | 备注 |
|----|------|------|
| **后端语言** | Python 3.11+ / FastAPI | 与 LLM / Agent 生态契合度最高；新建服务默认采用 |
| **前端** | Vue 3 + TypeScript（或 React + TS，二选一）| 组件库统一 Element Plus / Ant Design |
| **关系数据库** | PostgreSQL 15+ | 兼容 pgvector，便于知识库就地落地；MySQL 仅历史包袱兼容 |
| **向量数据库** | pgvector（中小规模）/ Milvus（大规模） | 二选一按部署量级 |
| **缓存 / 队列** | Redis 7+ | 兼任 Celery / Arq 后端 |
| **任务调度** | Celery / Arq / Temporal（择一统一） | 由 platform-core 统一封装 |
| **对象存储** | S3 协议（MinIO 私有化） | 文档、报告、PNG 等 |
| **容器** | Docker + Compose（开发） / K8s（生产） | 私有化部署优先 |
| **可观测性** | Prometheus + Grafana + Loki + OpenTelemetry | trace_id 统一打通 |

> 历史以 Java/Spring Boot 实现的服务可继续维护，但**新公共模块**默认采用 Python 栈。

## 2. 仓库与代码组织

### 2.1 仓库划分

| 仓库 | 角色 |
|------|------|
| `AgentForSE/.github` | 组织级规范、架构文档、工作流模板（即本仓库） |
| `AgentForSE/platform-*`（建议）| 八大公共模块，每模块独立 repo 或 monorepo 子目录 |
| `AgentForSE/safewise` | SafeWise 业务仓 |
| `AgentForSE/safematrix` | SafeMatrix 业务仓 |
| `AgentForSE/smartinsight` | SmartInsight 业务仓 |
| `AgentForSE/smartst` | SmartST 业务仓 |

### 2.2 公共模块发布

- 以**内部 PyPI**（建议 Nexus / Gitea Packages）发布 Python 包：
  `agentforse-llm-gateway`、`agentforse-agent-pipeline`、`agentforse-traceability` 等。
- 业务仓通过 `requirements.txt` / `pyproject.toml` 引用具体版本；
  禁止 `git+ssh://` 直接引用源码（除非特殊调试场景）。

### 2.3 版本策略

- 公共模块采用语义化版本（SemVer）。
- **破坏性变更**（major bump）必须先在本仓发起架构 PR，得到 SafeWise / SafeMatrix /
  SmartInsight / SmartST 各自维护者批准后再发布。

## 3. 配置与密钥

- 配置使用 `.env` + `pydantic-settings`；`.env.example` 提交，`.env` 不入库（已在 `.gitignore`）。
- 密钥统一走 K8s Secret / Vault；禁止入库。
- 模型 API Key 统一由 [`llm-gateway`](./platform-modules/llm-gateway.md) 持有，业务方拿不到原始 key。

## 4. CI/CD 规范

- 业务仓必须有的工作流（基于 [`/workflow-templates`](../../workflow-templates/)）：
  - **lint**：代码风格（Ruff / Black / ESLint / Prettier）。
  - **test**：单元测试 + 关键集成测试，覆盖率门槛 ≥ 60%（公共模块 ≥ 80%）。
  - **schema-contract**：校验对 `traceability` Artifact / Link schema 的兼容性。
  - **build**：Docker 镜像构建并推送内部仓。
- 主干保护：禁止直接 push 到 `main`；必须 PR + 评审 + CI 通过。
- 提交规范：建议 Conventional Commits；与 changelog 自动化集成。

## 5. Prompt 工程规范

- 所有 Prompt **必须**登记到 [`llm-gateway`](./platform-modules/llm-gateway.md)
  的 Prompt 模板库；源码中不允许出现超过 ~100 字的硬编码 Prompt。
- 模板 ID 命名：`<product>.<agent>.<purpose>.v<version>`，
  例：`safewise.hazard.identification.v3`。
- 系统 Prompt（角色定义、输出格式约束、装备 / 军工领域术语集）
  作为**公共片段**统一维护，业务模板通过 include 复用。
- 重要 Prompt 变更必须附 A/B 评估结果或回归 case。

## 6. 数据合规与安全

- 内网隔离：核心数据不出域（呼应"军工级合规"诉求）。
- 多租户隔离：所有库表带 `tenant` / `project` 列，查询强制带过滤。
- 审计：操作日志、模型调用日志、跨产品事件全部入审计。
- 模型与数据分离：不允许将客户数据用于训练私有模型，除非合同明确允许。

## 7. 文档规范

- 任何公共模块的**对外接口**变更，必须同步更新本目录中对应的 `*.md` 模块规范，
  以及 OpenAPI / GraphQL schema 文件。
- 业务仓的设计文档（README / 架构图）必须**链接回**本仓架构文档，
  避免重复编写共性内容。
