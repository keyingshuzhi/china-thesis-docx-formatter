# 更新日志

重要变更记录于此。格式参考 [Keep a Changelog](https://keepachangelog.com/)。

## [0.1.1] — 2026-10-28（已冻结）

### 新增

- 新增 `load_generated_artifact`，将工作区内经校验的 DOCX、PDF、JSON、Markdown 或 TXT 文件以 MCP 嵌入资源返回给客户端，单文件上限为 32 MiB。
- 当 MCP 客户端未提供 Roots 时，文件工具接受按调用传入的当前 Agent 工作区 `workspace_root`，不必为不同工作区修改 MCP 配置或在服务进程间共享路径。
- 新增本地 MCP Server，暴露模板分析、规则校验、论文格式化、文档审计、PDF 指南提取、占位符处理、引文审计、浮动图片规范和 Word 字段更新工具。
- 新增 `get_mcp_version`，返回服务版本、Python 版本和 Agent 工作区配置状态。
- MCP 文件工具优先从客户端自动读取当前 Agent 工作区，支持工作区相对路径，并在需要时创建输出子目录；源码仓库不再作为文件工作区。
- 新增独立 Skill 包，包含 `SKILL.md`、`references/guide.md`、参考资料、元数据、分析报告和更新日志。
- 补充柯影智航（KoyingPilot）接入说明，同时保留 MCP Server 和 Skill 对其他兼容智能体的通用性。
- MCP 依赖由 uv 管理，锁定 Python 3.13.15 并提交 `uv.lock`。
- CLI 与 MCP 共用 uv 项目、Python 3.13.15 和锁文件。
- MCP 强制使用独立的工作区路径，并在生成结果时保护源文件。

### 变更

- 将原有 `references/` 资料迁入 `skill/academic-paper-formatter/references/`。
- 抽取 PDF 规则解析函数，供 CLI 和 MCP 共用。
- 清理 `requirements.txt` 中重复的 `lxml` 依赖声明。
- 将用户指南和技能文档统一为中文；命令、API 名称与字段保留英文。

### 修复

- MCP 输入路径现在先执行工作区边界校验，再检查文件存在性和扩展名，避免工作区外路径因格式错误掩盖边界拒绝结果。
- 补充串行 MCP 调用和输出已存在时的恢复规则，避免未收到真实工具结果就重复写入或意外覆盖已有产物。

### 端到端验收与已知限制

- [x] 柯影智航重新加载 MCP 后发现 `load_generated_artifact`；版本、Python 版本和 Agent 工作区均符合预期。
- [x] `load_generated_artifact` 为工作区中的 DOCX 返回 MCP `EmbeddedResource`，样例文件大小 39,842 bytes，MIME 为 `application/vnd.openxmlformats-officedocument.wordprocessingml.document`。
- [x] 工作区外路径负测返回 `Path is outside the Agent workspace`，未读取工作区外文件。
柯影智航当前客户端未呈现嵌入资源，界面返回 `embedded resource unsupported`，没有可打开附件卡片；`file://` Markdown 链接也未形成可点击控件。此宿主功能延期至后续柯影智航开发。
- [ ] 自动化项目测试套件未运行。

v0.1.1 的功能范围和版本号保持锁定。柯影智航宿主端资源呈现能力不纳入此版本。
