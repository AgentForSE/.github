# 二、共性技术识别

下列 19 项共性技术从四款产品的现有实现与规划中归纳得到，是抽取
[公共底座模块](./platform-modules/)的依据。

## 2.1 架构共性

| # | 共性 | 说明 |
|---|------|------|
| 1 | B/S 前后端分离 | 四款均为 Web 平台；前端可视化展示，后端提供 REST + WebSocket API |
| 2 | 多 Agent 流水线编排 | 均采用 DAG 串联多个专职 Agent（Loss/Hazard/UCA、SFTA/FMEA、Search/Read 等），各 Agent 单一职责 |
| 3 | LLM 统一接入层 | 均需对接 DeepSeek / GPT-4o / Qwen / Claude 与私有化推理引擎，需统一 SDK |
| 4 | 异步任务与实时推送 | 分析任务耗时长，均依赖 WebSocket 推送进度与状态可视化 |

## 2.2 数据处理共性

| # | 共性 | 说明 |
|---|------|------|
| 5 | 文档解析流水线 | SRS / SDD / 测试输入 / 代码文件均需解析 → 结构化 → 入库 |
| 6 | 增量分析机制 | 需求变更 / 代码 diff 仅触发增量分析，避免重复消耗 token |
| 7 | 版本管理与 Diff 对比 | 多版本分析结果可追踪、可比对（SafeMatrix 已有版本 Diff 面板可作参照） |
| 8 | 可追溯性数据模型 | 维护制品间追踪关系链（需求→用例→缺陷→代码→安全约束等），支持正/反向追溯 |

## 2.3 质量管控共性

| # | 共性 | 说明 |
|---|------|------|
| 9  | 质量门禁（Quality Gate） | "仅对已确认输入生成输出"，防止脏数据进入下游（SmartST 已落地 confirmed 门禁） |
| 10 | 人机协作审查节点 | 流水线关键节点插入 ReviewAgent + 人工确认；被拒绝条目支持定向重生成 |
| 11 | AI 误报消除 / 结果评审 | LLM 输出做二次校验（规则引擎或评审 Agent），过滤低质量结果（SmartInsight 误报确认能力可作参照） |

## 2.4 工程化共性

| # | 共性 | 说明 |
|---|------|------|
| 12 | 知识 / 经验库 | 故障模式库、历史经验库、规则库，供 LLM 检索增强（RAG） |
| 13 | 标准化报告导出 | GJB 5000A / GJB 900A / DO-178C 等规范的 Word/Excel 报告模板 |
| 14 | CI/CD 集成 | 代码提交时自动触发对应分析任务（走查 / 测试 / 安全分析） |
| 15 | 可视化看板 | 项目级覆盖率、进度、质量趋势统计面板 |

## 2.5 平台管理共性

| # | 共性 | 说明 |
|---|------|------|
| 16 | 项目管理 | 项目 CRUD、进入项目、配置快照与恢复 |
| 17 | 用户与权限管理 | Token 鉴权、角色 / 权限点细粒度控制、项目级成员权限 |
| 18 | 日志与审计 | 操作日志 + 任务执行日志，按项目 / 用户 / 时间筛选 |
| 19 | 模型与 Prompt 配置管理 | 各 Agent 独立配置 LLM 模型与 Prompt，便于调优 |

## 共性 → 模块映射

| 共性条目 | 收敛入模块 |
|----------|-----------|
| 3, 19 | [llm-gateway](./platform-modules/llm-gateway.md) |
| 2, 10 | [agent-pipeline](./platform-modules/agent-pipeline.md) |
| 5, 6, 7 | [doc-parser](./platform-modules/doc-parser.md) |
| 12 | [knowledge-base](./platform-modules/knowledge-base.md) |
| 8 | [traceability](./platform-modules/traceability.md) |
| 9, 10, 11 | [quality-gate](./platform-modules/quality-gate.md) |
| 13 | [report-generator](./platform-modules/report-generator.md) |
| 1, 4, 14, 15, 16, 17, 18 | [platform-core](./platform-modules/platform-core.md) |
