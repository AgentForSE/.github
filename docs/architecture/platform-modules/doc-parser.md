# 模块三：文档解析服务（`doc-parser`）

> 收敛共性 #5（文档解析流水线）、#6（增量分析机制）、#7（版本管理与 Diff 对比）。

## 职责

将装备软件研发场景中的**输入文档**（SRS、SDD、测试输入、规范文件、代码文件）
统一解析为**结构化数据**，支持版本管理、增量解析与 Diff，为上层 Agent
提供干净、可定位、可追溯的输入。

## 能力清单

1. **多格式解析**
   - Word（.docx）、PDF（含扫描件 OCR）、Markdown、HTML、纯文本。
   - 代码文件：以 [`tree-sitter`](https://tree-sitter.github.io/) 类工具构建 AST。
2. **结构化提取**
   - 章节树（标题层级 / 编号识别）、段落、表格、图片、公式。
   - 跨章节引用、编号识别（如"见 3.2 节"、"REQ-001"）。
   - 输出统一 `Document` 模型 + `Block` 列表（块级唯一 ID 用于追溯）。
3. **增量解析**
   - 文档新版本上传时，按章节级 / 块级做 diff。
   - 对未变更块**保留**其下游已确认的分析结果（呼应 SmartST"保留已确认需求"机制）。
   - 输出变更标签：`added` / `modified` / `removed` / `unchanged`。
4. **版本管理与 Diff 视图**
   - 同一文档的多版本入库，提供版本树查询。
   - 提供 Diff API（块级三向对比）供前端可视化（参考 SafeMatrix Diff 面板）。
5. **任务化与重试**
   - 解析以异步任务形式提交，失败可重试，进度可查（参考 SmartST 解析流水线）。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/docs`                          | 上传文档，返回 doc_id + version |
| `POST /v1/docs/{doc_id}/parse`           | 触发（增量 / 全量）解析任务 |
| `GET  /v1/docs/{doc_id}/blocks`          | 拉取结构化块列表 |
| `GET  /v1/docs/{doc_id}/diff?from&to`    | 版本 Diff |
| `GET  /v1/parse-tasks/{task_id}`         | 任务进度 |

## 业务方使用约束

- 业务方**禁止**自行解析文档；如格式不支持需求驱动到本模块新增 parser。
- 解析结果必须以 `block_id` 形式被引用，禁止直接拷贝原文到下游表。
- 增量解析时，业务方应据 `block_id` 与变更标签决定下游是否重新分析。

## 与其它模块的关系

- 输出的 `block_id` 是 [`traceability`](./traceability.md) 中"需求 / 设计 / 代码"
  类制品的天然主键之一。
- 增量解析结果驱动 [`quality-gate`](./quality-gate.md) 的"变更影响 → 标记下游过期"逻辑。
- [`agent-pipeline`](./agent-pipeline.md) 的 Agent 以 `Block` 为标准输入。
