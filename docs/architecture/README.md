# AgentForSE 平台架构文档

本目录沉淀 AgentForSE 智能研发平台的**共性技术**与**公共开发底座**，作为组织内
SafeWise / SafeMatrix / SmartInsight / SmartST 等智能体软件研发的统一规范。

各业务团队在开发自身产品时，应优先依赖本目录中规定的公共模块与规范，
**只聚焦各自的领域业务逻辑**，不重复造轮子，并保证产品间可整合、数据可互通。

## 文档导航

| # | 文档 | 内容简介 |
|---|------|----------|
| 1 | [overview.md](./overview.md) | 四大产品的定位、分工与全生命周期映射 |
| 2 | [common-technologies.md](./common-technologies.md) | 19 项共性技术识别（架构 / 数据 / 质量 / 工程 / 平台） |
| 3 | [platform-modules/](./platform-modules/) | 八大公共底座模块规范（每个模块一份） |
| 4 | [product-domains.md](./product-domains.md) | 各产品在公共底座之上需自研的领域专有逻辑 |
| 5 | [integration.md](./integration.md) | 跨产品整合方案与数据流 |
| 6 | [tech-stack.md](./tech-stack.md) | 统一技术栈、仓库结构、CI/CD 与 Prompt 工程规范 |

## 公共底座模块一览

| 模块 | 目录 | 职责 |
|------|------|------|
| LLM 统一网关 | [`platform-modules/llm-gateway.md`](./platform-modules/llm-gateway.md) | 多模型接入、配额、Prompt、流式输出 |
| Agent 流水线编排 | [`platform-modules/agent-pipeline.md`](./platform-modules/agent-pipeline.md) | DAG 编排、Agent 基类、节点状态机 |
| 文档解析服务 | [`platform-modules/doc-parser.md`](./platform-modules/doc-parser.md) | Word/PDF/Markdown 解析与增量解析 |
| 知识库与 RAG | [`platform-modules/knowledge-base.md`](./platform-modules/knowledge-base.md) | 向量库、知识条目、检索增强生成 |
| 可追溯性数据模型 | [`platform-modules/traceability.md`](./platform-modules/traceability.md) | 制品注册、追踪关系、变更传播 |
| 质量门禁与审查 | [`platform-modules/quality-gate.md`](./platform-modules/quality-gate.md) | 输入确认、定向重生成、规则引擎 |
| 报告生成服务 | [`platform-modules/report-generator.md`](./platform-modules/report-generator.md) | Word/Excel 报告模板、图表嵌入 |
| 平台基础服务 | [`platform-modules/platform-core.md`](./platform-modules/platform-core.md) | 用户/权限、项目、任务、WebSocket、日志 |

## 使用约定

1. **本仓库是规范源**：架构变更必须通过 PR 修改本目录文档，并通知所有业务仓负责人。
2. **业务仓必须遵循**：SafeWise / SafeMatrix / SmartInsight / SmartST 在引入新组件前，
   优先检查公共底座是否已提供；如缺失，提出公共模块需求而非自行实现。
3. **跨产品集成走 `traceability`**：产品间数据流转**只能**通过统一可追溯性数据模型，
   不允许两两点对点直连接口（详见 [integration.md](./integration.md)）。
4. **CI/CD 复用模板**：业务仓的工作流应基于 [`/workflow-templates`](../../workflow-templates/) 派生。

## 修订记录

| 版本 | 日期 | 说明 |
|------|------|------|
| 0.1 | 2026-04 | 初版：基于 AgentForSE-0.1 PPT 与功能规格统计形成 |
