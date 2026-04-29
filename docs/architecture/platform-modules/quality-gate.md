# 模块六：质量门禁与审查框架（`quality-gate`）

> 收敛共性 #9（质量门禁）、#10（人机协作审查节点）、#11（AI 误报消除 / 结果评审）。

## 职责

把"输入确认 → 生成 → 自动评审 → 人工确认 → 发布"这套质量保证流程做成
**可复用框架**，并提供 AI 误报二次校验与定向重生成能力，覆盖四款产品的共性需求。

## 标准门禁流程

```
                ┌──────────────┐
   原始输入 ──→ │ 输入确认门禁 │ → 仅 status=confirmed 的输入才允许下游生成
                └──────┬───────┘
                       ▼
                ┌──────────────┐
                │  Agent 生成  │   （由 agent-pipeline 调度）
                └──────┬───────┘
                       ▼
                ┌──────────────┐
                │ 自动评审节点 │ → ReviewAgent + 规则引擎二次校验，过滤低质量
                └──────┬───────┘
                       ▼
                ┌──────────────┐
                │ 人工确认门禁 │ → 接受 / 拒绝 / 标记问题
                └──────┬───────┘
                       │ rejected → 定向重生成（仅该条目）
                       ▼
                     发布
```

## 能力清单

1. **输入确认门禁**
   - 提供"确认 / 标记问题 / 重置"标准状态机（呼应 SmartST 已实现机制）。
   - 强制：下游生成接口必须传入 `parent_artifact_id`，框架自动校验
     该 Artifact `status=confirmed`，否则拒绝。
2. **自动评审（ReviewAgent + 规则引擎）**
   - 内置 ReviewAgent 基类，业务可继承定制评审 Prompt。
   - **规则引擎插件**：业务注册校验规则（如"故障树叶节点必须挂基本事件"、
     "测试用例必须含期望结果"）；规则用 JSON-Logic / CEL 等声明式表达。
   - 输出标注：`pass` / `warn` / `fail`，附理由。
3. **AI 误报消除**
   - 针对 SmartInsight 等以静态分析为输入的场景，提供"告警 → AI 二次确认 → 真伪标签"
     流水线模板，避免每个产品自己重写。
4. **定向重生成**
   - 接口：`POST /v1/regenerate?artifact_ids=...&reason=...`。
   - 仅对指定条目重新跑生成 Agent，避免全量重复消耗（呼应 SmartST"拒绝需求定向重生"）。
5. **变更过期标记联动**
   - 上游 Artifact 升版（[`traceability`](./traceability.md) 触发）时，
     自动调用本模块将下游标记为 `needs_review`。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/gate/confirm/{artifact_id}`         | 人工确认 |
| `POST /v1/gate/reject/{artifact_id}`          | 人工拒绝（携带原因） |
| `POST /v1/gate/regenerate`                    | 定向重生成 |
| `POST /v1/rules`                              | 注册业务规则插件 |
| `POST /v1/review`                             | 触发 ReviewAgent + 规则引擎评审 |

## 业务方使用约束

- 任何"AI 生成 → 入库 → 下游使用"场景**必须**串入本框架，不允许跳过。
- 规则引擎规则必须通过 PR 评审入仓，禁止运行时硬编码。
- 拒绝意见必须结构化（原因码 + 备注），用于后续闭环优化 Prompt。

## 与其它模块的关系

- 由 [`agent-pipeline`](./agent-pipeline.md) 在节点间调用本模块完成评审与门禁。
- 与 [`traceability`](./traceability.md) 强联动：门禁状态即 Artifact `status`。
- 评审结论与拒绝原因可沉淀到 [`knowledge-base`](./knowledge-base.md) 用于后续 RAG。
