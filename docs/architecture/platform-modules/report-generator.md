# 模块七：报告生成服务（`report-generator`）

> 收敛共性 #13（标准化报告导出）。

## 职责

提供**模板化**的 Word / Excel / PDF 报告生成能力，统一支持 GJB / DO-178C
等装备软件 / 航空航天领域的合规交付物，并能将 [`traceability`](./traceability.md)
的追溯矩阵、图表（故障树 PNG、控制结构图、覆盖率图）一并嵌入。

## 能力清单

1. **模板引擎**
   - Word：基于 `python-docx-template` / Apache POI 等模板渲染。
   - Excel：模板 + 数据填充（`openpyxl` 等）。
   - PDF：由 Word 模板转换或独立 PDF 模板。
2. **内置标准模板**（公共部分）
   - GJB 5000A 软件测试过程符合性报告。
   - GJB 900A 系统安全分析报告。
   - DO-178C 软件验证 / 分析报告。
   - 通用追溯矩阵 / 覆盖率统计。
3. **业务定制扩展**
   - 各产品可注册自有模板（SafeMatrix 故障树报告、SmartInsight 走查报告等）。
   - 模板版本化、租户级覆盖（公司专属页眉 / 页脚 / 编号规则）。
4. **图表与图像嵌入**
   - 矢量与位图（PNG）支持；故障树 / 控制结构图 / RPN 色标矩阵直接嵌入。
   - 图表数据可来自 [`platform-core`](./platform-core.md) 看板接口。
5. **完整交付包**
   - 一键打包：需求清单 + 用例 + 回填结果 + 走查问题 + 安全分析 + 报告 + 模板说明。
   - 输出 zip 包并归档（接 platform-core 的对象存储）。

## 对外接口（建议）

| 接口 | 说明 |
|------|------|
| `POST /v1/templates`                | 注册业务模板 |
| `GET  /v1/templates`                | 列表与版本 |
| `POST /v1/render`                   | 渲染单份报告（指定模板 + 数据 source） |
| `POST /v1/delivery-package`         | 打包完整交付物 |

## 业务方使用约束

- 业务方**禁止**直接拼接 Word / Excel；必须走模板。
- 模板新增 / 修改走 PR 评审，确保合规字段不漏。
- 报告中的追溯矩阵数据**必须**取自 [`traceability`](./traceability.md) 实时查询，
  不允许业务方提前拍平存储后渲染（避免数据漂移）。

## 与其它模块的关系

- 数据源主要来自 [`traceability`](./traceability.md) 与 [`platform-core`](./platform-core.md)。
- 渲染过程中的 AI 摘要（如执行摘要、风险综述）走 [`llm-gateway`](./llm-gateway.md)。
