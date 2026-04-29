# 模块二：Agent 流水线编排框架（`agent-pipeline`）

> 收敛共性 #2（多 Agent 流水线编排）、#10（人机协作审查节点）。

## 职责

提供基于 **DAG** 的多 Agent 任务编排引擎，统一各产品 Agent 的生命周期、状态机
和 ReviewAgent 协作模式，让业务团队**只写 Agent，不写编排**。

## 能力清单

1. **DAG 编排引擎**
   - 节点类型：`AgentNode`、`ReviewNode`、`HumanGateNode`、`BranchNode`、`MergeNode`。
   - 支持串行、并行、条件分支、循环（带最大次数）。
   - 失败重试：可配置 max_retries、退避策略。
2. **Agent 角色基类**
   - 标准接口：`prepare(input) → run(ctx) → review(output) → emit(result)`。
   - 强制输出 schema 校验（Pydantic / JSON Schema）。
   - 默认接入 [`llm-gateway`](./llm-gateway.md) 与
     [`knowledge-base`](./knowledge-base.md) 的 RAG 上下文。
3. **节点状态机**
   ```
   pending → running → review → approved → done
                    ↘ rejected → (重生成) → running
                    ↘ failed   → (重试)   → running
   ```
   状态变更同步推送到 WebSocket（参考 SafeWise 已有实时推送实现）。
4. **持久化与恢复**
   - 流水线快照（含上下文、节点状态）入库；进程重启可断点续跑。
   - 支持"恢复并修改参数配置"（对应 SmartInsight 项目审查恢复需求）。
5. **可观测性**
   - 每个节点的输入 / 输出 / 耗时 / token 消耗可查询。
   - 提供 trace_id 全链路打通 `llm-gateway` 调用日志。

## 流水线定义示例（伪 YAML）

```yaml
pipeline: stpa-stage-1
nodes:
  - id: loss
    type: AgentNode
    agent: LossAgent
  - id: hazard
    type: AgentNode
    agent: HazardAgent
    depends_on: [loss]
  - id: constraint
    type: AgentNode
    agent: SafetyConstraintAgent
    depends_on: [hazard]
  - id: review
    type: ReviewNode
    depends_on: [constraint]
    on_reject: { goto: constraint, max_retries: 3 }
```

## 业务方使用约束

- 业务 Agent 必须继承标准基类，**禁止自行实现 Agent 调度**。
- 节点状态变更**必须**走框架 API；不允许直接改库。
- 每条流水线必须含至少一个 ReviewNode 或 HumanGateNode（落实质量门禁）。

## 与其它模块的关系

- 调用 [`llm-gateway`](./llm-gateway.md) 完成推理。
- 复用 [`quality-gate`](./quality-gate.md) 的 ReviewAgent 与定向重生成接口。
- 通过 [`platform-core`](./platform-core.md) 的任务调度与 WebSocket 落地异步执行。
- 流水线产生的制品写入 [`traceability`](./traceability.md)，建立追踪关系。
