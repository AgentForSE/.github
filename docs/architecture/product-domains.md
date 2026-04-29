# 四、各产品专有领域（业务聚焦点）

在公共底座（[platform-modules/](./platform-modules/)）之上，各产品**只**需要开发
以下领域专有逻辑。任何不在此清单中的"基础能力"，都应从公共底座获取。

## SafeWise — 系统安全分析

| 类别 | 专有内容 |
|------|---------|
| 方法论实现 | STPA 三阶段：LossAgent / HazardAgent / SafetyConstraintAgent / RequirementAgent / ControlStructureAgent / UCAAgent / DesignConstraintAgent / CausalAgent / ScenarioAgent / TestCaseAgent |
| 可视化 | 控制结构图、STAMP 因果图（前端组件） |
| 行业模板 | GJB 900A 系统安全分析报告模板（注册到 `report-generator`） |
| 知识扩展 | STPA 模板库（损失 / 危险 / UCA 库），写入 `knowledge-base` |
| Artifact 类型 | `loss` / `hazard` / `uca` / `safety_constraint` / `causal_factor` |

## SafeMatrix — 软件功能安全分析

| 类别 | 专有内容 |
|------|---------|
| 方法论实现 | SFTA 故障树生成 / 模块级树扩展 Agent；FMEA / FMECA 评分 Agent；MCS 最小割集计算 |
| 可视化 | 故障树画布（AND / OR / 基本事件节点）、RPN 色标矩阵 |
| 行业模板 | DO-178C / GJB 报告模板 |
| 知识扩展 | 软件失效模式库（公共化后由 `knowledge-base` 承接） |
| Artifact 类型 | `fault_tree_node` / `failure_mode` / `safety_constraint`（与 SafeWise 共享） |

## SmartInsight — 智能代码走查

| 类别 | 专有内容 |
|------|---------|
| 方法论实现 | Agentic 代码仓搜索五步流程：list_dir / glob_search / grep_code / read_source_file / 结构化输出 |
| 工具适配 | 12+ 主流静态分析工具报告归一化适配器 |
| 仓结构 | RepoMap 代码仓结构地图智能生成 |
| 一致性比对 | 需求 - 设计 - 代码三方一致性核查逻辑 |
| Artifact 类型 | `code_unit` / `static_finding` / `review_issue` |

## SmartST — 系统测试自动化

| 类别 | 专有内容 |
|------|---------|
| 方法论实现 | 需求驱动测试用例生成 Agent、测试要素提取（前置上下文 / 边界值 / 异常路径 / 输入输出要点）、测试数据生成 |
| 流程 | 用例发布 / 执行回填 / 需求覆盖率统计 |
| 行业模板 | GJB 5000A 软件测试过程符合性报告 |
| Artifact 类型 | `test_case` / `test_run` / `defect` |

## SmartUT — 单元测试（预留）

> 当前处于早期研发阶段，按相同底座规范接入。专有领域：函数级用例生成、
> 语句 / 分支 / MC/DC 覆盖率分析、边界值推断、与 C++TEST / GoogleTest 集成。

## 公共边界检查清单（每个产品上线前自检）

业务团队在 PR / 设计评审时应核对：

- [ ] 是否所有 LLM 调用都经过 `llm-gateway`？
- [ ] 是否所有 Agent 都基于 `agent-pipeline` 基类？
- [ ] 文档解析是否使用 `doc-parser`，而非自实现？
- [ ] 知识 / 经验是否沉淀进 `knowledge-base`？
- [ ] 业务产出是否注册为 `traceability` Artifact，关系是否建立？
- [ ] 是否串入 `quality-gate`（输入确认 + 评审 + 定向重生成）？
- [ ] 报告导出是否使用 `report-generator` 模板？
- [ ] 是否复用 `platform-core` 的用户 / 项目 / 任务 / WebSocket / 日志？

任何"否"项，都应**优先**改为复用公共底座，而不是在产品里重复造轮子。
