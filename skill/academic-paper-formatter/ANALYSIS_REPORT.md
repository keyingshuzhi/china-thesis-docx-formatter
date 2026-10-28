# 学位论文格式化 Skill — 分析报告（v0.1.1）

## 版本状态

**v0.1.1 已锁定。** 本版本包含 MCP Server、通用 Skill、`load_generated_artifact` 嵌入资源返回和 uv 管理的 Python 3.13.15 环境。柯影智航会话 UI 的资源卡片呈现不属于本版本范围，留待后续柯影智航开发。

## 目标

优先为柯影智航（KoyingPilot）提供中国学位论文 DOCX 分析与格式化能力，同时保持 MCP Server 和 Skill 对其他兼容智能体通用。项目拆分为两个配套部分：MCP Server 提供结构化本地工具，Skill 提供操作流程、安全边界和领域参考资料。MCP 项目环境统一由 uv 管理，Python 固定为 3.13.15。

## 本次升级内容

| 项目 | 类型 | 说明 |
| --- | --- | --- |
| MCP Server | 新增 | 封装模板提取、规则校验、排版、审计、占位符与参考文献处理，以及 PDF/图片/Word 字段辅助功能。 |
| Agent Skill 包 | 新增 | 按独立技能目录组织入口、参考指南、元数据、许可、版本记录与分析报告；优先适配柯影智航并保持可移植。 |
| Python 与依赖管理 | 新增 | 使用 `.python-version`、`pyproject.toml` 和 `uv.lock` 管理 MCP 环境，要求 Python 3.13.15。 |
| 文件访问边界 | 安全性 | MCP 按客户端 Roots 或本次调用传入的 `workspace_root` 限定当前 Agent 工作区；输出必须与输入分离，覆盖已有输出需显式选择。 |
| CLI 复用 | 架构 | MCP 直接复用 `scripts/` 中的逻辑，避免维护两套格式处理实现。 |
| PDF 解析接口 | 重构 | 将 PDF 规则提取逻辑拆成可调用函数，CLI 和 MCP 共用。 |
| 会话产物资源返回 | 新增 | 新增 `load_generated_artifact`，对工作区内 DOCX、PDF、JSON、Markdown 和 TXT 进行类型校验后，以 MCP 嵌入资源返回。 |
| 文档语言与组织 | 体验 | 将项目主说明和 Skill 用户文档统一为中文，并保留英文命令、参数和 API 标识符。 |

## 设计决策

1. **学校模板优先**：规则提取先于排版；无法从模板确认的要求保留为警告，交由用户复核。
2. **MCP 与 Skill 分工**：MCP 提供可调用工具，Skill 描述何时调用、处理顺序和安全约束；CLI 作为兼容入口保留。
3. **统一环境、锁文件可复现**：MCP Server 与 CLI 共用 `mcp/` 下的 uv 项目；`requires-python` 精确设为 `==3.13.15`，依赖由同一份 `uv.lock` 锁定。
4. **限制文件范围**：优先使用 MCP 客户端 Roots；Roots 不可用时按调用传入 `workspace_root`，仅允许访问当前 Agent 工作区。源码仓库及其父目录不作为论文工作区。
5. **原稿不变**：所有写操作输出到单独路径；已有目标文件默认拒绝覆盖。
6. **不生成学术事实**：内容替换和书目补充必须来自用户明确提供并审核的数据。
7. **提取结果需复核**：PDF 文本、学位结构提示和版式审计都不能代替学校书面要求与最终视觉检查。
8. `extension.json` 是 Skill 的项目本地元数据，不宣称为所有 Agent 平台都支持的标准格式。

## 风险与缓解

| 风险 | 影响 | 缓解措施 |
| --- | --- | --- |
| 模板提取遗漏或误认样式 | 论文版式与学校要求不符 | 返回提取警告；排版前审阅规则 JSON。 |
| 输出路径配置错误 | 访问或覆盖非预期文件 | 限制当前工作区；规范化路径；拒绝覆盖输入和已有输出。 |
| 学位层级清单被误当作强制要求 | 增加学校未要求的结构 | 默认只提供复核提示；仅按学校书面要求启用严格检查。 |
| 引文缺失时补造来源 | 学术信息失真 | 只接受用户提供的权威元数据；先审计再重编号。 |
| PDF 扫描页缺乏可提取文字 | 规范证据不完整 | 在结果中提示无文字页面；要求人工查看原始页面。 |
| Word 字段更新影响分页 | 目录或页码布局变化 | 仅标记打开时更新；之后在 Word 中更新并重新渲染检查。 |
| Python 版本不匹配 | MCP 无法启动 | `.python-version` 与运行时检查均固定到 3.13.15；使用 uv 启动。 |
| MCP 客户端不呈现嵌入资源 | 用户无法从会话直接打开生成文件 | `load_generated_artifact` 返回实际资源；Skill 要求报告客户端限制并提供工作区文件路径。柯影智航 UI 呈现能力留待宿主后续开发。 |

## 验收记录

- [x] MCP Server 与 CLI 共用 `.python-version`、`pyproject.toml`、Python 3.13.15 和 `uv.lock` 管理的环境。
- [x] Skill 目录采用独立命名目录，并包含 `SKILL.md`、`references/guide.md`、`extension.json`、`README.md`、`CHANGELOG.md`、`ANALYSIS_REPORT.md` 与 `LICENSE`。
- [x] MCP 输出写入单独路径，工具不修改输入论文或模板。
- [x] 柯影智航 E2E 确认 `load_generated_artifact` 可调用，并为 DOCX 返回正确 MIME 的嵌入资源（样例 39,842 bytes）。
- [x] 柯影智航 E2E 工作区边界负测拒绝外部路径；返回 `Path is outside the Agent workspace`。
- [x] 确认当前柯影智航 UI 将嵌入资源标记为 `embedded resource unsupported`，`file://` Markdown 链接也未呈现为可点击控件；问题归属宿主端并移交未来柯影智航开发。
- [x] `uv sync --locked --project mcp` 已在 Python 3.13.15 环境成功完成。
- [x] Skill Creator 的 `quick_validate.py` 已通过。
- [ ] 项目测试套件尚未运行。

## 待验证事项

- 在柯影智航后续开发中加入 MCP 嵌入资源的会话展示和直接打开能力，再重新验证文件下载/预览流程。
- 使用真实学校模板和论文验证各类规则抽取与版式检查的覆盖情况。
- 在 Microsoft Word 与学校指定的兼容软件中完成最终版面复核。

## 后续建议

- 根据真实论文处理流程收集 MCP 工具返回字段与错误信息的改进点。
- 明确各目标 Agent 平台安装 Skill 和配置 MCP 的具体格式。
- 如后续支持新增文件操作，继续遵循绝对路径、目录限制与输出副本原则。
- 将柯影智航对 MCP `EmbeddedResource` 的可视化支持作为宿主应用后续开发任务；不回写到已锁定的 v0.1.1。
