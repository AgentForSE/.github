# 模块一：LLM 统一网关（`llm-gateway`）

> 收敛共性 #3（LLM 统一接入层）、#19（模型与 Prompt 配置管理）。

## 职责

为所有上层智能体提供**唯一**的 LLM 调用入口，屏蔽各家模型厂商的协议差异，统一处理
鉴权、计费、限流、监控、Prompt 与流式输出。

## 能力清单

1. **多模型接入**
   - 兼容 OpenAI 协议族：DeepSeek、GPT-4o、Qwen、Claude（OpenAI 兼容端点）。
   - 兼容私有化推理引擎：vLLM / Ollama / 自研推理服务。
   - 统一 SDK：`chat()` / `embedding()` / `rerank()` 三类接口。
2. **配额与计费**
   - 按租户 / 项目 / Agent 三维度统计 token 使用量。
   - 支持配额上限与告警阈值；超限时熔断或降级到备用模型。
3. **Prompt 模板管理**
   - 模板版本化（含元数据：作者、变更说明、绑定 Agent、目标模型）。
   - 模板按"系统 Prompt + 业务 Prompt + 输出格式约束"三段式组织。
   - 支持 A/B 灰度与回滚。
4. **流式输出**
   - 上行：调用方传入 `stream=true`。
   - 下行：网关将 SSE / chunk 流转封装为统一事件，桥接到
     [`platform-core`](./platform-core.md) 的 WebSocket 推送。
5. **可观测性**
   - 全量调用日志（脱敏后入数仓）、调用耗时、首 token 时延、失败率。
   - 提供 Prompt 评估接口，便于知识库 / 经验闭环（参考 SmartInsight 经验库做法）。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/chat/completions`         | OpenAI 兼容 chat 接口 |
| `POST /v1/embeddings`               | 向量化 |
| `POST /v1/rerank`                   | 重排（可选，对接 RAG） |
| `GET  /v1/models`                   | 当前可用模型与配额 |
| `POST /v1/prompts/{id}/render`      | 渲染指定版本 Prompt（变量占位符替换） |
| `GET  /v1/usage?project=...`        | 用量查询 |

## 业务方使用约束

- **禁止**绕过网关直接调用厂商 API。
- 所有 Agent 的 Prompt 必须登记到 Prompt 模板库，禁止在源码中硬编码长 Prompt。
- 调用时必须传 `tenant`、`project`、`agent`、`prompt_template_id` 四个上下文标签，
  以便统计与排障。

## 与其它模块的关系

- 上游：[`agent-pipeline`](./agent-pipeline.md) 中的 Agent 通过本模块调用 LLM。
- 下游：日志与计费数据接入 [`platform-core`](./platform-core.md) 看板。
- 联动：[`knowledge-base`](./knowledge-base.md) 通过本模块的 `embeddings` 接口生成向量。
