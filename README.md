# 中国学位论文格式化工具

面向柯影智航（KoyingPilot）优先适配、同时支持其他通用智能体的中国学位论文 Word `.docx` 格式分析与排版工具。v0.1.1 将项目拆分为两个可组合部分：MCP Server 提供通用工具接口，Skill 提供可移植的使用流程和规则。学校提供的模板与书面规范始终是排版依据。

## 项目组成

| 部分 | 路径 | 用途 |
| --- | --- | --- |
| MCP Server | [`mcp/`](mcp/README.md) | 通过 MCP 暴露模板分析、格式化、审计、辅助文档和会话产物交付工具；运行环境由 uv 管理，Python 固定为 3.13.15。 |
| Skill | [`skill/academic-paper-formatter/`](skill/academic-paper-formatter/README.md) | 优先适配柯影智航，也可移植到其他支持 Agent Skill 的智能体。 |
| CLI | [`scripts/`](scripts/) | 保留现有命令行工具；与 MCP 共用 uv 环境和 Python 3.13.15，供没有 MCP 接口的环境使用。 |

## MCP 快速开始

```bash
uv python install 3.13.15
uv sync --locked --project mcp
uv run --locked --project mcp academic-paper-formatter-mcp
```

MCP Server 使用 stdio 传输。可在柯影智航或其他支持本地 MCP Server 的客户端配置 `uv` 启动项，参数为 `--directory <仓库>/mcp run academic-paper-formatter-mcp`；具体字段位置与格式按宿主版本适配。`THESIS_FORMATTER_REPO_ROOT` 用于定位源码脚本。MCP 优先读取客户端提供的 Roots；客户端不提供 Roots 时，Skill 会将当前会话工作区通过每次工具调用的 `workspace_root` 参数动态传入，不需要为每个工作区修改 MCP 配置。`THESIS_FORMATTER_WORKSPACE_ROOT` 仅保留给无法传递动态工作区的旧客户端作为固定兼容目录。详见 [MCP 使用指南](mcp/README.md)。

## CLI 快速开始

CLI 与 MCP 共用 `mcp/` 下的 uv 项目、Python 3.13.15 和 `uv.lock`：

```bash
uv sync --locked --project mcp
uv run --locked --project mcp python scripts/analyze_docx.py "/absolute/path/to/agent-workspace/school-template.docx" --degree master --output "/absolute/path/to/agent-workspace/school-rules.json"
uv run --locked --project mcp python scripts/validate_format.py "/absolute/path/to/agent-workspace/school-rules.json"
uv run --locked --project mcp python scripts/format_document.py "/absolute/path/to/agent-workspace/thesis-source.docx" "/absolute/path/to/agent-workspace/thesis-formatted.docx" --rules "/absolute/path/to/agent-workspace/school-rules.json"
uv run --locked --project mcp python scripts/validate_format.py "/absolute/path/to/agent-workspace/school-rules.json" --document "/absolute/path/to/agent-workspace/thesis-formatted.docx"
```

更多流程与安全约束见 [Skill 入口](skill/academic-paper-formatter/SKILL.md)。

## v0.1.1 冻结状态

v0.1.1 的功能范围和版本号保持锁定。端到端验证确认 MCP 正确识别工作区、拒绝工作区外路径，并从 `load_generated_artifact` 返回 DOCX 嵌入资源；柯影智航客户端呈现环节未通过：它将资源标记为 `embedded resource unsupported`，没有展示可打开的附件卡片，`file://` Markdown 链接也未渲染为可点击链接。宿主端附件呈现能力留待后续柯影智航开发处理，不纳入本版本。

## 原则与边界

- 原始论文和学校模板保持不变；处理结果写入新文件。
- 不推断或编造学校格式要求、论文内容、实验结果和参考文献。
- PDF 提取结果是供人工核验的文本证据；扫描页与视觉版式需要另行检查。
- 学位层级仅用于提示人工复核结构，不替代学校的书面要求。
- `templates/` 中的规则 JSON 仅是通用起点，不能当作任何学校的提交标准。

## 目录结构

```text
mcp/                            # uv 管理的 Python 3.13.15 MCP Server
skill/academic-paper-formatter/ # 可分发的 Agent Skill
scripts/                        # 既有 CLI 实现
templates/                      # 通用规则起点
tests/                          # 项目原有测试
test_data/                      # 演示输入和样例输出
```
