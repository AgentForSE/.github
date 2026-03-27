# .github
代码管理注意事项

## 提交规范

### ✅ 应当提交的内容

- **源代码**：所有业务逻辑、功能实现代码
- **环境定义文件**：如 `Dockerfile`、`docker-compose.yml`、`.env.example`（不含真实密钥）、`requirements.txt`、`package.json`、`pom.xml`、`build.gradle` 等
- **编译 / 启动 / 停止脚本**：如 `build.sh`、`start.sh`、`stop.sh`、`Makefile` 等
- **数据库定义及脚本**：如建表 SQL、数据迁移脚本、种子数据脚本等
- **配置模板**：不含敏感信息的配置示例文件（如 `config.example.yml`）
- **文档**：README、设计文档、接口文档等

### ❌ 不应提交的内容

- **依赖库目录**：如 `node_modules/`、`vendor/`、`.m2/`、`venv/`、`.venv/` 等（依赖应通过包管理工具安装，不直接入库）
- **编译产物 / 构建输出**：如 `dist/`、`build/`、`target/`、`*.class`、`*.jar`（可执行文件）等
- **测试环境相关文件**：如本地测试数据库文件、测试覆盖率报告（`htmlcov/`、`coverage.xml`）、`.tox/`、`.pytest_cache/` 等
- **IDE / 编辑器配置**：如 `.idea/`（部分）、`.vscode/`（部分，视团队约定而定）
- **敏感信息**：密码、Token、私钥等（使用 `.env.example` 代替 `.env`）
- **操作系统生成的临时文件**：如 `.DS_Store`、`Thumbs.db`
- **日志文件**：如 `*.log`

> **原则**：只要能通过脚本或包管理工具还原的内容，都不需要提交到代码库。
1、提交时只提交代码，包括环境定义、依赖定义、数据库定义及各种脚本，如编译、启动、停止等
2、提交时不要提交依赖库、工具、环境等
