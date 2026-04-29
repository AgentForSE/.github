# 组织级可复用 Workflow 模板

本目录是 AgentForSE 组织内 **GitHub Actions 工作流模板**集中地。

GitHub 自动将位于本目录的 `*.yml` 与同名 `*.properties.json` 暴露给本组织所有仓库，
开发者在仓库的 Actions → New workflow 页面会看到 "**By AgentForSE**" 一栏，
可以一键复用，落实 [架构文档](../docs/architecture/tech-stack.md#4-cicd-规范) 要求。

## 模板列表

| 文件 | 适用场景 |
|------|----------|
| [`python-ci.yml`](./python-ci.yml) | 公共模块或业务仓的 Python 后端：lint + 单测 + 覆盖率 |
| [`schema-contract.yml`](./schema-contract.yml) | 业务仓校验对 `traceability` Artifact / Link schema 的兼容性 |
| [`docker-build.yml`](./docker-build.yml) | Dockerfile 构建并推送内部镜像仓 |

## 使用方式

1. 在新建仓库的 Actions 页面选择 "**By AgentForSE**"，挑选模板。
2. GitHub 会把模板拷贝到当前仓库的 `.github/workflows/` 下。
3. 按提示替换占位变量（如包名、Python 版本、镜像名）。

## 维护约束

- 模板修改必须经过本仓 PR 评审；不会自动同步到下游仓库（拷贝模式），
  下游仓如需同步新版本，需手动 cherry-pick 或运行升级脚本。
- 重大变更应在 PR 中通过 `@AgentForSE/maintainers` 提醒各业务仓维护者。

参考：[GitHub 文档：创建组织级工作流模板](https://docs.github.com/en/actions/sharing-automations/creating-workflow-templates-for-your-organization)
