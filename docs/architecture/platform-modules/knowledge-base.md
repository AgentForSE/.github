# 模块四：知识库与检索增强（`knowledge-base`）

> 收敛共性 #12（知识 / 经验库）。

## 职责

统一管理装备软件研发领域的**结构化知识**与**历史经验**，并提供
RAG（检索增强生成）能力，让各产品的 Agent 在生成时具备领域上下文支撑。

## 能力清单

1. **知识条目类型**（可由各业务域注册扩展）
   - 故障模式库（SafeMatrix 内置软件失效模式 → 公共化）。
   - STPA 损失 / 危险 / UCA 模板（SafeWise）。
   - 代码缺陷模式与误报特征（SmartInsight）。
   - 测试要素与边界值规则（SmartST）。
   - 规范条款（GJB 5000A / GJB 900A / DO-178C 条款级化）。
2. **存储**
   - 元数据：PostgreSQL（条目 ID、类型、租户 / 项目作用域、版本、引用来源）。
   - 向量：pgvector（小规模） / Milvus（大规模），二选一按部署规模。
   - Embedding 通过 [`llm-gateway`](./llm-gateway.md) 的 `/embeddings` 接口生成。
3. **RAG 检索**
   - 标准接口：`retrieve(query, types=[...], top_k=N, filters={...})`。
   - 支持混合检索（关键词 + 向量）、重排（rerank）、租户隔离。
   - 结果带置信度与来源（保证 Agent 输出可附带 Evidence 链，呼应 SmartInsight 证据链原则）。
4. **经验闭环**
   - Agent 输出经过人工确认后，可一键沉淀回经验库（标记为"已验证案例"）。
   - 提供经验导入 / 导出（CSV / JSON），支持跨项目复用。
5. **业务域扩展机制**
   - 各产品通过注册 `KnowledgeType` 与 schema 来扩展自有条目类型。
   - 提供管理后台 UI 模板（各业务可嵌入）。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/kb/types`                | 注册业务域知识类型 |
| `POST /v1/kb/items`                | 写入知识条目 |
| `POST /v1/kb/retrieve`             | RAG 检索 |
| `POST /v1/kb/items/{id}/promote`   | 将临时经验提升为已验证条目 |

## 业务方使用约束

- 业务方**禁止**自建独立向量库或经验表；新增条目类型走"注册"机制。
- 检索时**必须**携带 `tenant` 与 `project` 过滤，避免跨项目数据泄漏。
- 沉淀经验前必须通过 [`quality-gate`](./quality-gate.md) 审查通过的条目。

## 与其它模块的关系

- 依赖 [`llm-gateway`](./llm-gateway.md) 提供 embedding 与 rerank。
- 输出供 [`agent-pipeline`](./agent-pipeline.md) 注入到 Agent 上下文。
- 与 [`traceability`](./traceability.md) 互通：经验条目可关联到具体制品作为"参考案例"。
