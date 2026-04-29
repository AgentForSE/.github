# 模块五：可追溯性数据模型（`traceability`）

> 收敛共性 #8（可追溯性数据模型）。**这是跨产品整合的核心枢纽。**

## 职责

提供**统一制品（Artifact）模型**与**追踪关系（Trace Link）模型**，承载
"需求 → 危险 → 安全约束 → 设计 → 代码 → 用例 → 缺陷 → 测试结果"全链路追溯，
并驱动变更影响分析与跨产品集成。

## 数据模型

### 1. 制品（Artifact）

统一的多态实体，所有上层产品的产出物都注册为 Artifact，便于跨产品引用。

| 字段 | 说明 |
|------|------|
| `id`           | 全局唯一 ID（建议 ULID） |
| `type`         | 类型（见下表） |
| `tenant`       | 租户 |
| `project`      | 所属项目 |
| `source_app`   | 产生应用：`safewise` / `safematrix` / `smartinsight` / `smartst` / `manual` |
| `external_ref` | 业务侧主键（如 SafeWise 的 H-002、SmartST 的 TC-001） |
| `block_ref`    | 关联的 [`doc-parser`](./doc-parser.md) `block_id`（如适用） |
| `version`      | 版本号 |
| `status`       | `draft` / `confirmed` / `obsolete` |
| `payload`      | JSON 业务字段 |
| `created_at` / `updated_at` | 时间戳 |

### 2. 制品类型字典（可扩展）

| 类型 | 主要来源 |
|------|----------|
| `requirement`        | 需求文档解析、SafeWise SafetyConstraint |
| `loss` / `hazard` / `uca` / `causal_factor` | SafeWise |
| `safety_constraint`  | SafeWise / SafeMatrix |
| `fault_tree_node`    | SafeMatrix |
| `failure_mode`       | SafeMatrix |
| `design_element`     | SDD 解析 |
| `code_unit`          | 仓库代码（函数 / 类 / 模块） |
| `static_finding`     | SmartInsight 静态分析归一化 |
| `review_issue`       | SmartInsight |
| `test_case`          | SmartST / SmartUT |
| `test_run`           | SmartST 执行回填 |
| `defect`             | 通用缺陷 |

### 3. 追踪关系（Trace Link）

| 字段 | 说明 |
|------|------|
| `from_artifact_id` / `to_artifact_id` | 两端制品 |
| `relation` | `derives` / `verifies` / `mitigates` / `implements` / `covers` / `caused_by` / `references` |
| `confidence` | 0–1，AI 生成时需附置信度 |
| `created_by` | `human` / `agent:<name>` |
| `evidence` | JSON：来源、Prompt id、出处 block_id 等 |

## 能力清单

1. **制品 CRUD + 版本**：保留历史，支持回查任意时间点状态。
2. **关系图谱查询**：
   - 正向追溯：从需求查所有派生制品。
   - 反向追溯：从测试用例查溯回到核心损失。
   - 影响分析：某需求变更，列出所有下游受影响 Artifact。
3. **变更传播**：当上游 Artifact `version` 升级时，自动将其下游
   `status` 标记为 `obsolete`（呼应 SafeMatrix"过期"标记机制）。
4. **图查询接口**：支持深度 N 的 BFS / 路径查询；提供 GraphQL 风格端点。
5. **可视化数据接口**：返回前端绘制追溯链 / 矩阵 / 桑基图所需的节点 / 边数据。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/artifacts`                       | 注册制品 |
| `POST /v1/links`                           | 创建追踪关系 |
| `GET  /v1/artifacts/{id}/trace?direction=` | 追溯查询（forward / backward） |
| `GET  /v1/artifacts/{id}/impact`           | 变更影响分析 |
| `POST /v1/artifacts/{id}/versions`         | 升版（自动触发 obsolete 传播） |

## 业务方使用约束

- 任何产品的核心产出（损失 / 故障树节点 / 走查问题 / 测试用例……）
  **必须**注册到 Artifact，禁止只存在于业务库。
- 跨产品引用**必须**通过 Trace Link，**禁止**业务库间直连外键。
- AI 自动建立的关系**必须**带置信度与 evidence。

## 与其它模块的关系

- 是 [integration.md](../integration.md) 跨产品数据流的载体。
- 与 [`doc-parser`](./doc-parser.md) 通过 `block_ref` 关联到原始文档位置。
- 与 [`quality-gate`](./quality-gate.md) 联动：`status=confirmed` 是下游生成的前置门禁。
- 与 [`report-generator`](./report-generator.md) 联动：报告中的追溯矩阵直接来自本模块查询结果。
