# 五、跨产品整合方案与数据流

四款产品**通过公共底座（特别是 [`traceability`](./platform-modules/traceability.md)）
间接整合**，禁止两两点对点直连接口。本章定义关键数据流与集成场景。

## 总体集成原则

1. **以 Artifact 为通用语**：跨产品交换的**唯一**载体是
   [`traceability`](./platform-modules/traceability.md) 中的 `Artifact` 与 `TraceLink`。
2. **事件驱动 + 拉取并存**：上游产生新 Artifact 时通过 WebSocket / 内部事件总线广播，
   下游可订阅；同时下游可按需拉取。
3. **避免循环耦合**：禁止 A → B 后又 B → A 强同步；如有需要走 Trace Link 弱耦合。
4. **门禁前置**：跨产品消费的 Artifact，必须 `status=confirmed` 才能作为下游输入。

## 关键数据流

### 数据流 1：SafeWise → SafeMatrix（系统安全 → 软件功能安全）

```
 SafeWise STPA 输出
   ├─ Artifact: hazard / safety_constraint  (status=confirmed)
   └─→ TraceLink: derives
          ▼
 SafeMatrix 监听并触发 SFTA：
   将上游 safety_constraint 作为顶事件候选导入故障树
```

- 实现机制：SafeMatrix 订阅 `traceability` 的"新 Artifact / type=safety_constraint"事件。
- 用户体验：SafeMatrix 项目里出现"待导入安全约束"列表，一键纳入 SFTA。

### 数据流 2：SafeMatrix → SmartInsight（高风险模块 → 重点走查）

```
 SafeMatrix FMEA 输出
   ├─ Artifact: failure_mode（带 RPN 评分 / 关联 design_element）
   └─→ TraceLink: caused_by → design_element
          ▼
 SmartInsight 在代码走查时，对关联到高 RPN failure_mode 的
 design_element / code_unit 提升优先级与扫描深度
```

- 实现机制：SmartInsight 走查任务启动时，查询本项目下所有 `failure_mode` 中
  RPN ≥ 阈值的条目，回查到 `code_unit`，加入"重点关注"列表。

### 数据流 3：SmartInsight → SmartST（缺陷 → 回归测试）

```
 SmartInsight 走查输出
   ├─ Artifact: review_issue（关联 code_unit / requirement）
   └─→ TraceLink: references → requirement
          ▼
 SmartST 自动关联生成对应回归测试用例草稿，并挂在原需求下
```

- 实现机制：SmartST 监听新 `review_issue`；按规则触发"基于缺陷生成测试用例"流水线。

### 数据流 4：SmartST → 全局（测试结果反哺）

```
 SmartST 测试执行结果
   ├─ Artifact: test_run（pass / fail）
   └─→ TraceLink: verifies → test_case → covers → requirement
          ▼
 失败用例 → 自动建 defect Artifact → 再绑回需求 / 安全约束
   反向推动 SafeWise / SafeMatrix 复核
```

## 反向追溯链（合规交付）

任何一份测试用例都应能反向追溯到核心损失，构成完整的合规证据链：

```
TC-001 (test_case)
  └─→ verifies → SC-002 (safety_constraint)
        └─→ derives ← H-002 (hazard)
              └─→ derives ← L-001 (loss)

TC-001 (test_case)
  └─→ covers → REQ-005 (requirement)
        └─→ block_ref → SRS.md#3.2 (doc-parser block)
```

由 [`report-generator`](./platform-modules/report-generator.md) 的"追溯矩阵"模板
直接渲染。

## 跨产品 SSO 与权限

- 所有产品共享 [`platform-core`](./platform-modules/platform-core.md) 的用户与项目实体。
- 用户对项目 X 在 SafeWise 拥有 `analyst` 权限，自动在 SafeMatrix / SmartInsight / SmartST
  同项目下获得对应的查看权限（写权限按各产品角色矩阵控制）。

## 集成测试与契约约束

- 公共模块以**语义化版本**发布；破坏性变更必须先在本仓 `docs/architecture/` 提议
  并广播到所有业务仓 reviewer。
- 跨产品事件 schema（Artifact 类型字段、关系类型）以 JSON Schema 形式集中维护
  在 `docs/architecture/platform-modules/traceability.md`，业务方通过 PR 扩展。
- 各业务仓必须有"集成契约测试"流水线，验证自身对公共 schema 的兼容性
  （详见 [`/workflow-templates`](../../workflow-templates/)）。
