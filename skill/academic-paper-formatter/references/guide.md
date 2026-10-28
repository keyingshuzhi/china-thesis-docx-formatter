# 学位论文格式化使用指南

本文介绍面向柯影智航（KoyingPilot）及其他兼容智能体的 MCP Server 配置、工具路由和 CLI 备用流程。具体版式始终以学校模板和书面规范为准。

## 1. 选择接入方式

| 环境 | 建议方式 |
| --- | --- |
| 柯影智航或其他 Agent 支持 MCP，且可访问本地文件 | 配置本地 MCP Server，通过工具调用完成分析、格式化和审计。 |
| Agent 可运行终端，但不支持 MCP | 使用同一 uv 项目和锁文件调用 `scripts/*.py` CLI。 |
| Agent 不能访问本地文件或终端 | 请用户在可访问本地文件的环境运行工具，并提供生成的规则与审计结果供复核。 |

## 2. 安装与配置 MCP

MCP 项目在 `mcp/` 中，uv 管理依赖并将 Python 精确固定为 3.13.15：

```bash
uv python install 3.13.15
uv sync --locked --project mcp
```

柯影智航“添加 MCP 服务”表单按下列项目填写。这里的路径对应当前仓库部署位置；迁移仓库后要换成新位置。其他 MCP 客户端将相同信息映射到自身的 stdio 配置字段。

| 表单字段 | 填写值 |
| --- | --- |
| 连接方式 | **本地命令（stdio）** |
| 服务标识 | `academic-paper-formatter` |
| 显示名称 | `学位论文格式化` |
| 描述 | `依据学校 Word 模板分析、格式化和审计学位论文 DOCX；保留源文件，并将结果写入单独文件。` |
| 启动命令 | `/opt/homebrew/bin/uv` |
| 工作目录 | `/Volumes/柯影数智/研发部/mcp/mcp-academic-paper-formatter/mcp` |

参数框中每行填写一个参数，顺序如下：

```text
run
--locked
academic-paper-formatter-mcp
```

环境变量中添加：

```text
THESIS_FORMATTER_REPO_ROOT=/Volumes/柯影数智/研发部/mcp/mcp-academic-paper-formatter
```

`THESIS_FORMATTER_REPO_ROOT` 只用于定位源码脚本。MCP 优先读取客户端 Roots。若客户端不提供 Roots，Skill 会从当前 Agent 会话上下文取得选定工作区，并在 `get_mcp_version` 及后续文件工具调用中动态传入 `workspace_root`，切换工作区时无需修改 MCP 配置。无法由宿主提供 Roots、也无法在工具参数中传入动态工作区的旧客户端，才可选填 `THESIS_FORMATTER_WORKSPACE_ROOT` 作为固定兼容目录；该目录必须已存在且与源码仓库分离。不要将它设为 MCP 启动工作目录、源码仓库目录或其父目录。

凭据引用映射应为空：不要添加条目，并删除表单自动生成的空白行。本服务不需要 API 密钥；普通环境变量只放在“环境变量”栏。若看到 `MCP credentialEnv must map non-empty names to credential references`，检查并移除凭据引用栏里的空白行、`NAME=` 或未绑定的占位值。这是宿主校验提示，不表示 MCP Server 需要凭据。

路径与写入规则：

- 安装后先从当前 Agent 上下文取得选定工作区绝对路径（若可用），并调用 `get_mcp_version(workspace_root=...)`；无法取得时省略参数，让服务查询客户端 Roots。
- 若 Roots 为空且使用了 `workspace_hint`，每次文件工具调用都传入同一 `workspace_root`。该值按调用提供，不保存到服务配置或进程级状态。
- 若无法可靠取得当前工作区，停止文件操作；不要改用 MCP 启动目录、源码目录或凭文件路径猜测工作区。
- 输入和输出使用相对于唯一 Agent 工作区的路径，或已选工作区内的绝对路径；工作区外路径会被拒绝。多工作区时使用绝对路径。
- 输入文件必须存在；输出子目录会在工作区内自动创建。
- 输出必须与全部输入不同；默认拒绝覆盖已有输出，只有明确设置 `overwrite=true` 才覆盖。
- 工作区空时，先将待处理的模板、论文或 PDF 放入工作区。

## 3. MCP 工具路由

### 版本和工作区

开始文件操作前调用 `get_mcp_version`。结果包含服务版本、Python 版本、工作区列表及来源（`client_roots`、`workspace_hint`、`environment` 或 `unavailable`）。若客户端 Roots 为空，使用当前 Agent 会话上下文中的绝对路径传入 `workspace_root`，并将同一参数传给后续每个文件工具。只有 `workspace.configured=true` 且返回根目录对应当前 Agent 工作区时，才继续处理文件。服务不会跨调用记住该参数。

### MCP 调用与失败恢复

- 一次只执行一个文件工具调用，等待实际 MCP 结构化结果后再决定下一步；仅出现函数调用请求、不含工具结果时，不要重复发起同一写入请求。
- 依赖前一步输出的转换必须串行执行，并以 MCP 返回的 `output_path` 作为下一步输入。
- 写入返回 `Output already exists` 时，不自动覆盖或重试相同路径。先通过只读校验确认已有产物；若确需重新生成，使用新的唯一输出路径，只有用户明确要求覆盖时才传 `overwrite=true`。

### 模板和版式

1. 调用 `analyze_template`，输入学校 DOCX 模板并生成规则 JSON；需要结构提示时可传 `degree`。
2. 检查返回的 `extraction_warnings`、结构摘要和生成的 JSON，确认标题、摘要、关键词、题注、参考文献等角色。
3. 调用 `validate_rules_file` 校验规则。
4. 调用 `format_document_copy` 为论文生成排版副本。
5. 调用 `validate_document_copy` 比对格式并查看所有发现项。需严格检查结构时，同时提供 `degree` 和 `strict_degree=true`，且确认学校书面规范使相关项目都成为必需项。

### 内容占位符

先调用 `inspect_placeholders` 获取占位符和上下文。让用户审核每项替换文本后，再用 `apply_approved_replacements` 提供精确的 `replacements` 字典，并生成新 DOCX。跨 Word 文本片段的占位符可能无法安全替换，需按工具返回提示人工处理。

### 引文与参考文献

先调用 `audit_references` 查看缺失条目、未引用条目和重复编号。若用户提供经核验的参考文献文本，可用 `update_references_copy` 的 `approved_metadata` 插入；仅在缺失条目和重复编号均已解决后，才设置 `renumber_existing=true`。工具不会根据孤立引用自行寻找或生成来源。

### PDF 指南

调用 `parse_pdf_guide` 提取可复制文字并生成带页码的 JSON。逐项核实 `requirements` 中的原文证据，并查看 `extraction_warnings`。无文字页面、扫描页、图示和视觉排版需要单独查看；不能仅凭提取文本确定完整规范。

### 图片与 Word 字段

- `normalize_floating_figures_copy` 调整现有浮动图片的锚点位置，不更改图片字节或题注。
- `mark_word_fields_for_update_copy` 标记 Word 在打开文档时更新目录、交叉引用和页码字段。之后在 Word 中接受更新并重新渲染检查分页。

### 会话交付

最终交付物完成校验后，对每个需要交付的工作区文件调用 `load_generated_artifact`。该工具将 DOCX、PDF、JSON、Markdown 或 TXT 文件内容作为 MCP 嵌入资源返回，单个文件不得超过 32 MiB；Roots 不可用时传入当前 `workspace_root`。不要加载输入文件或流程中间文件。具体预览方式由 MCP 客户端决定。柯影智航当前客户端对该资源返回 `embedded resource unsupported`，不会生成可打开的附件卡片；此宿主能力留待后续柯影智航开发，不属于 v0.1.1。

## 4. CLI 备用流程

CLI 与 MCP 共用 `mcp/` 下的 uv 项目、Python 3.13.15 和 `uv.lock`。CLI 不做 MCP 的工作区边界检查，因此仍须把输入和输出路径明确写在 Agent 工作区内；脚本可从源码仓库运行，但不要把中间文件写进源码仓库：

```bash
# 从学校模板提取规则并校验
uv run --locked --project mcp python scripts/analyze_docx.py "/absolute/path/to/agent-workspace/school-template.docx" --degree master --output "/absolute/path/to/agent-workspace/school-rules.json"
uv run --locked --project mcp python scripts/validate_format.py "/absolute/path/to/agent-workspace/school-rules.json"

# 格式化论文副本并审计
uv run --locked --project mcp python scripts/format_document.py "/absolute/path/to/agent-workspace/thesis-source.docx" "/absolute/path/to/agent-workspace/thesis-formatted.docx" --rules "/absolute/path/to/agent-workspace/school-rules.json"
uv run --locked --project mcp python scripts/validate_format.py "/absolute/path/to/agent-workspace/school-rules.json" --document "/absolute/path/to/agent-workspace/thesis-formatted.docx" --degree master
```

`--degree` 可取 `undergraduate`、`master`、`doctoral` 或 `auto`。`--strict-degree` 会把常见结构提示提升为失败条件，仅当学校书面规范已确认所有被提示项目均为强制要求时才使用。

常用辅助命令：

```bash
# 查找占位符；审核映射后写入副本
uv run --locked --project mcp python scripts/complete_content.py "/absolute/path/to/agent-workspace/thesis.docx" --report "/absolute/path/to/agent-workspace/placeholders.json"
uv run --locked --project mcp python scripts/complete_content.py "/absolute/path/to/agent-workspace/thesis.docx" --replacements "/absolute/path/to/agent-workspace/approved-content.json" --output "/absolute/path/to/agent-workspace/thesis-completed.docx"

# 审计引用与参考文献
uv run --locked --project mcp python scripts/manage_references.py "/absolute/path/to/agent-workspace/thesis.docx" --rules "/absolute/path/to/agent-workspace/school-rules.json" --report "/absolute/path/to/agent-workspace/references-audit.json"

# 提取 PDF 文字证据
uv run --locked --project mcp python scripts/parse_pdf_rules.py "/absolute/path/to/agent-workspace/school-guide.pdf" --output "/absolute/path/to/agent-workspace/pdf-requirements.json"

# 规范浮动图片并请求 Word 更新字段
uv run --locked --project mcp python scripts/normalize_floating_figures.py "/absolute/path/to/agent-workspace/thesis.docx" "/absolute/path/to/agent-workspace/thesis-figures.docx" --horizontal center --vertical top
uv run --locked --project mcp python scripts/update_fields.py "/absolute/path/to/agent-workspace/thesis-figures.docx" "/absolute/path/to/agent-workspace/thesis-final.docx"
```

`requirements.txt` 保留用于兼容原有 pip 工作流；本项目的 MCP 和 CLI 均以 uv 项目配置与 `uv.lock` 为依赖和 Python 版本的唯一管理入口。

## 5. 交付前检查

- 原论文和模板未被改写；输出文件路径清楚且结果可打开。
- 规则 JSON 中的样式与角色已结合模板人工确认，提取警告已处理或记录。
- 所有格式差异均已检查；学位结构提示没有被误认为统一强制规则。
- 内容替换和新增参考文献有用户确认的文本来源。
- 在适用的 Word 环境中更新字段、重新渲染并逐页查看页面尺寸、中文字体、分页、表格、题注、页眉页脚和参考文献换行。
